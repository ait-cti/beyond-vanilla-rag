## Prompt (V2): CTI Report → Neo4j Cypher Import Query

Use the following prompt to instruct an LLM to extract concrete, evidence-backed facts from unstructured CTI text and output a **single** Neo4j Cypher import query.

```text
Write me a neo4j Cypher import query using the Text.

Use the following schema to guide your reasoning:

ENTITIES = [
  # Core actors & assets
  {"label":"ThreatActor","properties":[
    {"name":"name","type":"STRING"},               # primary
    {"name":"sponsor","type":"STRING"},
    {"name":"motivation","type":"STRING"},
    {"name":"summary","type":"STRING"}             # text for RAG
  ]},
  {"label":"Malware","properties":[
    {"name":"name","type":"STRING"},               # primary
    {"name":"family","type":"STRING"},
    {"name":"summary","type":"STRING"}
  ]},
  {"label":"Tool","properties":[
    {"name":"name","type":"STRING"},               # primary
    {"name":"summary","type":"STRING"}
  ]},
  {"label":"Victim","properties":[
    {"name":"name","type":"STRING"},               # primary (org/org-unit)
    {"name":"sector","type":"STRING"},
    {"name":"country_iso2","type":"STRING"},
    {"name":"summary","type":"STRING"}
  ]},
  {"label":"C2_Infrastructure","properties":[
    {"name":"service","type":"STRING"},            # primary (e.g., Telegram, Discord, VPS brand)
    {"name":"summary","type":"STRING"}
  ]},
  {"label":"Campaign","properties":[
    {"name":"name","type":"STRING"},               # primary
    {"name":"summary","type":"STRING"}
  ]},

  # Time & events
  {"label":"Incident","properties":[
    {"name":"id","type":"STRING"},                 # primary (stable per import batch)
    {"name":"date","type":"DATE"},
    {"name":"category","type":"STRING"},           # e.g., C1..C6 (optional)
    {"name":"type","type":"STRING"},               # ransomware, intrusion, etc.
    {"name":"exfiltration","type":"BOOLEAN"},
    {"name":"double_extortion","type":"BOOLEAN"},
    {"name":"cost_estimate","type":"FLOAT"},
    {"name":"summary","type":"STRING"}             # narrative for RAG
  ]},
  {"label":"Date","properties":[
    {"name":"value","type":"DATE"}                 # primary
  ]},

  # Context
  {"label":"Sector","properties":[{"name":"name","type":"STRING"}]},
  {"label":"Region","properties":[{"name":"name","type":"STRING"}]},
  {"label":"Country","properties":[
    {"name":"iso2","type":"STRING"},               # primary
    {"name":"name","type":"STRING"}
  ]},

  # TTPs, CVEs, mitigations, capabilities
  {"label":"Technique","properties":[
    {"name":"technique_id","type":"STRING"},       # primary, e.g., T1566
    {"name":"name","type":"STRING"},
    {"name":"tactic","type":"STRING"},
    {"name":"summary","type":"STRING"}
  ]},
  {"label":"CVE","properties":[
    {"name":"id","type":"STRING"},                 # primary
    {"name":"published_date","type":"DATE"},
    {"name":"summary","type":"STRING"}
  ]},
  {"label":"Motivation","properties":[
    {"name":"type","type":"STRING"}                # primary (e.g., financial, espionage, hacktivist)
  ]},
  {"label":"Mitigation","properties":[
    {"name":"name","type":"STRING"},               # primary (e.g., MFA, EDR, Segmentation)
    {"name":"framework","type":"STRING"},
    {"name":"control_id","type":"STRING"},
    {"name":"summary","type":"STRING"}
  ]},
  {"label":"Capability","properties":[
    {"name":"name","type":"STRING"},               # primary (e.g., AI/GenAI, Automation)
    {"name":"summary","type":"STRING"}
  ]},

  # Optional: Source for citations
  {"label":"Source","properties":[
    {"name":"id","type":"STRING"},                 # primary (doc-id or stable slug)
    {"name":"title","type":"STRING"},
    {"name":"publisher","type":"STRING"},
    {"name":"published_on","type":"DATE"},
    {"name":"url","type":"STRING"}
  ]}
]

RELATIONS = [
  # Activity & usage
  {"label":"attacked","description":"ThreatActor attacked Victim",
   "properties":[{"name":"date","type":"DATE"},{"name":"evidence","type":"STRING"},{"name":"source_id","type":"STRING"},{"name":"page","type":"STRING"}]},
  {"label":"uses","description":"ThreatActor or Malware using Malware or Tool",
   "properties":[{"name":"date","type":"DATE"},{"name":"evidence","type":"STRING"},{"name":"source_id","type":"STRING"},{"name":"page","type":"STRING"}]},
  {"label":"exploits","description":"Malware or Tool exploiting a CVE",
   "properties":[{"name":"date","type":"DATE"},{"name":"evidence","type":"STRING"},{"name":"source_id","type":"STRING"},{"name":"page","type":"STRING"}]},
  {"label":"abuses","description":"Malware or ThreatActor abusing C2",
   "properties":[{"name":"date","type":"DATE"},{"name":"evidence","type":"STRING"},{"name":"source_id","type":"STRING"},{"name":"page","type":"STRING"}]},
  {"label":"targets","description":"Campaign targeting Victim",
   "properties":[{"name":"date","type":"DATE"},{"name":"evidence","type":"STRING"},{"name":"source_id","type":"STRING"},{"name":"page","type":"STRING"}]},
  {"label":"includes","description":"Campaign includes ThreatActor or Malware",
   "properties":[{"name":"date","type":"DATE"},{"name":"evidence","type":"STRING"},{"name":"source_id","type":"STRING"},{"name":"page","type":"STRING"}]},
  {"label":"occurred_on","description":"Date on which an attack or event occurred",
   "properties":[{"name":"date","type":"DATE"}]},
  {"label":"has_alias","description":"ThreatActor alias",
   "properties":[{"name":"date","type":"DATE"}]},

  # Incidents & analytics (hub-and-spoke)
  {"label":"attributed_to","description":"Incident attributed to ThreatActor",
   "properties":[{"name":"confidence","type":"STRING"},{"name":"evidence","type":"STRING"},{"name":"source_id","type":"STRING"},{"name":"page","type":"STRING"}]},
  {"label":"involved_malware","description":"Malware observed in Incident",
   "properties":[{"name":"evidence","type":"STRING"},{"name":"source_id","type":"STRING"},{"name":"page","type":"STRING"}]},
  {"label":"involved_tool","description":"Tool observed in Incident",
   "properties":[{"name":"evidence","type":"STRING"},{"name":"source_id","type":"STRING"},{"name":"page","type":"STRING"}]},
  {"label":"used_technique","description":"Technique used in Incident/Actor",
   "properties":[{"name":"phase","type":"STRING"},{"name":"details","type":"STRING"},{"name":"evidence","type":"STRING"},{"name":"source_id","type":"STRING"},{"name":"page","type":"STRING"}]},
  {"label":"occurred_in","description":"Incident occurred in Region/Country","properties":[]},
  {"label":"targets_sector","description":"Incident targets Sector","properties":[]},
  {"label":"located_in","description":"Victim located in Country/Region","properties":[]},
  {"label":"motivated_by","description":"ThreatActor motivation","properties":[]},
  {"label":"exploited_in","description":"Incident exploited CVE",
   "properties":[{"name":"first_seen","type":"DATE"},{"name":"evidence","type":"STRING"},{"name":"source_id","type":"STRING"},{"name":"page","type":"STRING"}]},
  {"label":"mitigates","description":"Mitigation mitigates Technique/Incident",
   "properties":[{"name":"status","type":"STRING"},{"name":"observed_effectiveness","type":"STRING"},{"name":"evidence","type":"STRING"},{"name":"source_id","type":"STRING"},{"name":"page","type":"STRING"}]},
  {"label":"leverages","description":"Actor/Technique/Incident/Mitigation leverages Capability",
   "properties":[{"name":"details","type":"STRING"},{"name":"evidence","type":"STRING"},{"name":"source_id","type":"STRING"},{"name":"page","type":"STRING"}]},

  # Optional: connect facts to sources
  {"label":"supported_by","description":"Node/edge supported by Source",
   "properties":[{"name":"page","type":"STRING"},{"name":"quote","type":"STRING"}]}
]

POTENTIAL_SCHEMA = [
  # Capabilities & behaviors
  ("ThreatActor","uses","Malware"),
  ("ThreatActor","uses","Tool"),
  ("ThreatActor","abuses","C2_Infrastructure"),
  ("Malware","exploits","CVE"),
  ("Malware","abuses","C2_Infrastructure"),
  ("Malware","attacked","Victim"),
  ("ThreatActor","attacked","Victim"),
  ("ThreatActor","has_alias","ThreatActor"),

  # Campaigns
  ("Campaign","includes","ThreatActor"),
  ("Campaign","includes","Malware"),
  ("Campaign","targets","Victim"),
  ("Campaign","occurred_on","Date"),

  # Incidents as analytics hub
  ("Incident","attributed_to","ThreatActor"),
  ("Incident","involved_malware","Malware"),
  ("Incident","involved_tool","Tool"),
  ("Incident","used_technique","Technique"),
  ("Incident","exploited_in","CVE"),
  ("Incident","targets","Victim"),
  ("Incident","targets_sector","Sector"),
  ("Incident","occurred_in","Region"),
  ("Incident","occurred_in","Country"),
  ("ThreatActor","motivated_by","Motivation"),
  ("Mitigation","mitigates","Technique"),
  ("Mitigation","mitigates","Incident"),
  ("ThreatActor","leverages","Capability"),
  ("Technique","leverages","Capability"),
  ("Incident","leverages","Capability"),
  ("Mitigation","leverages","Capability"),
  ("Victim","located_in","Country"),
  ("Victim","located_in","Region"),

  # Evidence linkage
  ("ThreatActor","supported_by","Source"),
  ("Malware","supported_by","Source"),
  ("Incident","supported_by","Source"),
  ("CVE","supported_by","Source"),
  ("Technique","supported_by","Source"),
  ("Mitigation","supported_by","Source")
]

PRIMARY_PROP = {
  "ThreatActor":"name",
  "Malware":"name",
  "Victim":"name",
  "CVE":"id",
  "Tool":"name",
  "C2_Infrastructure":"service",
  "Campaign":"name",
  "Date":"value",
  "Incident":"id",
  "Sector":"name",
  "Region":"name",
  "Country":"iso2",               # << iso2, not name
  "Technique":"technique_id",
  "Motivation":"type",
  "Mitigation":"name",
  "Capability":"name",
  "Source":"id"
}


TASK
- From the input Text, extract concrete, evidence-backed facts and output a SINGLE Neo4j Cypher import query.

WHY THESE FIELDS
- Ensure the graph supports:
  Q1 actor activity by year & motivation; Q2 malware family YoY trends; Q3 CVE time-to-exploit after publication; 
  Q4 sector volumes; Q5 regional increases; Q6 ransomware exfil/double-extortion share; 
  Q7 initial-access mix; Q8 top MITRE techniques; Q9 mitigation effectiveness; Q10 AI/automation influence.

STRICT RULES
- Output ONLY the raw Cypher; no commentary.
- Use MERGE for nodes keyed by PRIMARY_PROP; set attributes via ON CREATE SET / ON MATCH SET; NEVER overwrite non-null with null/empty.
- Use unique variable names; do NOT reuse a variable for different nodes.
- Dates: use date('YYYY-MM-DD') when full date is known; if only year-month known, use the first of the month; if only year known, use date('YYYY-01-01'). Do not invent dates.
- Booleans: set Incident.exfiltration and Incident.double_extortion to true/false when stated; otherwise omit.
- Countries: create Country nodes with ISO-3166-1 alpha-2 in Country.iso2. If only a country name is present and you cannot resolve iso2 confidently, skip Country node.
- Sectors: normalize common sectors {Energy, Healthcare, Finance, Government, Education, Manufacturing, Technology, Retail, Transportation, Media, Legal, Other}.
- Techniques: map to MITRE ATT&CK technique_id (e.g., T1566) and set tactic (e.g., "Initial Access"). For entry methods (phishing, RDP abuse, supply chain), create (Incident)-[:used_technique {phase:'Initial Access', details:'phishing/RDP/supply chain'...}]->(Technique).
- CVEs: connect (Incident)-[:exploited_in {first_seen:<date-if-stated>, evidence, source_id, page}]->(CVE). This enables time-to-exploit analytics against CVE.published_date.
- Mitigations: when usage or recommendation is stated, create (:Mitigation)-[:mitigates {status:'applied'|'recommended'|'not_applicable', observed_effectiveness:'<free text>', evidence, source_id, page}]->(:Incident OR :Technique).
- AI/Automation: when mentioned offensively or defensively, create (:Capability {name:'AI/GenAI'|'Automation'|...}) and link from ThreatActor/Technique/Incident/Mitigation via [:leverages {details, evidence, source_id, page}].
- Evidence: when source id, page, or quotations are available, populate edge properties and optionally (:Source) nodes and [:supported_by] links.

OUTPUT PATTERN (example snippet, fix values to match the Text):
MERGE (ta1:ThreatActor {name:'The Dukes'})
MERGE (v1:Victim {name:'US Government', sector:'Government', country_iso2:'US'})
MERGE (i1:Incident {id:'INC-2013-0001'})
  ON CREATE SET i1.date=date('2013-01-01'), i1.type='intrusion', i1.exfiltration=false, i1.double_extortion=false, i1.summary:'...'
MERGE (d1:Date {value: date('2013-01-01')})
MERGE (i1)-[:attributed_to {confidence:'medium', evidence:'...', source_id:'SRC-001', page:'5'}]->(ta1)
MERGE (i1)-[:targets]->(v1)
MERGE (i1)-[:occurred_on]->(d1)

Do not add any explanation; return only the Cypher!

Text:
<PASTE_INPUT_TEXT_HERE>
```
