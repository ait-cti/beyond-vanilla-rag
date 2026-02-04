# GraphRAG Prompts

```python
# graphrag_pipeline.py
from __future__ import annotations

from typing import Literal, Annotated, List, Optional, Callable, Dict
from operator import add

from typing_extensions import TypedDict
from pydantic import BaseModel, Field

from langchain_core.prompts import ChatPromptTemplate
from langchain_core.output_parsers import StrOutputParser
from langchain_openai import ChatOpenAI, OpenAIEmbeddings
from langchain_neo4j import GraphCypherQAChain, Neo4jVector  # (GraphCypherQAChain not used but kept if you extend)
from langchain_core.example_selectors import SemanticSimilarityExampleSelector

from langchain_neo4j.chains.graph_qa.cypher_utils import CypherQueryCorrector, Schema
from neo4j.exceptions import CypherSyntaxError

# -----------------------------
# Shared types for the graph
# -----------------------------
class InputState(TypedDict):
    question: str

class OverallState(TypedDict):
    question: str
    next_action: str
    cypher_statement: str
    cypher_errors: List[str]
    database_records: List[dict] | str | None
    steps: Annotated[List[str], add]

class OutputState(TypedDict):
    answer: str
    steps: List[str]
    cypher_statement: str

# -----------------------------
# Public factory (what you import)
# -----------------------------
def create_graphrag(
    *,
    enhanced_graph,
    examples: List[Dict[str, str]],
    model: Any,#str = "gpt-4.1-mini",
    temperature: float = 0.5,
) -> Dict[str, Callable]:
    """
    Build the full LangGraph-based pipeline and return:
    - graph: the compiled graph
    - ask:   convenience function -> str question in, dict result out
    Requirements:
      - `enhanced_graph` must expose `.schema`, `.structured_schema`, and `.query(sql, params?)`.
      - `examples` is a list of { "question": str, "query": str }.
    """

    # -----------------------------
    # LLMs & prompts
    # -----------------------------
    llmlg = model

    guardrails_system = """
    As an intelligent assistant, your primary objective is to decide whether a given question is related to Cyber Threat Intelligence (CTI). 
    If the question is related to CTI, output "cti". Otherwise, output "end".
    Provide only the specified output: "cti" or "end".
    """

    guardrails_prompt = ChatPromptTemplate.from_messages(
        [
            ("system", guardrails_system),
            ("human", "{question}"),
        ]
    )

    class GuardrailsOutput(BaseModel):
        decision: Literal["cti", "end"] = Field(
            description="Decision on whether the question is related to CTI"
        )

    guardrails_chain = guardrails_prompt | llmlg.with_structured_output(GuardrailsOutput)

    # Example selector
    example_selector = SemanticSimilarityExampleSelector.from_examples(
        examples, OpenAIEmbeddings(), Neo4jVector, k=3, input_keys=["question"]
    )

    # Text → Cypher
    text2cypher_prompt = ChatPromptTemplate.from_messages(
        [
            (
                "system",
                (
                    "Given an input question, convert it to a Cypher query. No pre-amble."
                    "Do not wrap the response in any backticks or anything else. Respond with a Cypher statement only!"
                    "Only use labels, relationship types, and properties that appear in the provided schema. Do not invent fields; if unsure, return only known ones."
                ),
            ),
            (
                "human",
                (
                    """You are a Neo4j expert. Given an input question, create a syntactically correct Cypher query to run.
Do not wrap the response in any backticks or anything else. Respond with a Cypher statement only!
Only use labels, relationship types, and properties that appear in the provided schema.

Schema:
{schema}

Examples:
{fewshot_examples}

User input: {question}
Cypher query:"""
                ),
            ),
        ]
    )
    text2cypher_chain = text2cypher_prompt | llmlg | StrOutputParser()

    # Validate Cypher
    validate_cypher_system = "You are a Cypher expert reviewing a statement written by a junior developer."
    validate_cypher_user = """You must check the following:
* Are there any syntax errors in the Cypher statement?
* Are there any missing or undefined variables in the Cypher statement?
* Are any node labels missing from the schema?
* Are any relationship types missing from the schema?
* Are any of the properties not included in the schema?
* Does the Cypher statement include enough information to answer the question?

Schema:
{schema}

Question:
{question}

Cypher:
{cypher}
"""

    validate_cypher_prompt = ChatPromptTemplate.from_messages(
        [("system", validate_cypher_system), ("human", validate_cypher_user)]
    )

    class Property(BaseModel):
        node_label: str
        property_key: str
        property_value: str

    class ValidateCypherOutput(BaseModel):
        errors: Optional[List[str]] = None
        filters: Optional[List[Property]] = None

    validate_cypher_chain = validate_cypher_prompt | llmlg.with_structured_output(
        ValidateCypherOutput
    )

    # Corrector
    corrector_schema = [
        Schema(el["start"], el["type"], el["end"])
        for el in enhanced_graph.structured_schema.get("relationships")
    ]
    cypher_query_corrector = CypherQueryCorrector(corrector_schema)

    # Final answer
    generate_final_prompt = ChatPromptTemplate.from_messages(
        [
            ("system", "You are a helpful assistant"),
            (
                "human",
                """Use the following results retrieved from a database to provide
a succinct, definitive answer to the user's question.

Results: {results}
Question: {question}""",
            ),
        ]
    )
    generate_final_chain = generate_final_prompt | llmlg | StrOutputParser()
