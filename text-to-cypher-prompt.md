## 🧠 CTI Knowledge Graph Extraction Prompt

This prompt defines the entity and relationship schema, allowed graph structure,
and strict extraction rules for converting unstructured Cyber Threat Intelligence
(CTI) reports into a Neo4j knowledge graph.

---

## 📦 Entity Definitions

```python
ENTITIES = [
  # Core actors & assets
  {"label":"ThreatActor","properties":[
    {"name":"name","type":"STRING"},
    {"name":"sponsor","type":"STRING"},
    {"name":"motivation","type":"STRING"},
    {"name":"summary","type":"STRING"}
  ]},
  {"label":"Malware","properties":[
    {"name":"name","type":"STRING"},
    {"name":"family","type":"STRING"},
    {"name":"summary","type":"STRING"}
  ]},
  {"label":"Tool","properties":[
    {"name":"name","type":"STRING"},
    {"name":"summary","type":"STRING"}
  ]},
  {"label":"Victim","properties":[
    {"name":"name","type":"STRING"},
    {"name":"sector","type":"STRING"},
    {"name":"country_iso2","type":"STRING"},
    {"name":"summary","type":"STRING"}
  ]},
  {"label":"C2_Infrastructure","properties":[
    {"name":"service","type":"STRING"},
    {"name":"summary","type":"STRING"}
  ]},
  {"label":"Campaign","properties":[
    {"name":"name","type":"STRING"},
    {"name":"summary","type":"STRING"}
  ]},

  # Time & events
  {"label":"Incident","properties":[
    {"name":"id","type":"STRING"},
    {"name":"date","type":"DATE"},
    {"name":"category","type":"STRING"},
    {"name":"type","type":"STRING"},
    {"name":"exfiltration","type":"BOOLEAN"},
    {"name":"double_extortion","type":"BOOLEAN"},
    {"name":"cost_estimate","type":"FLOAT"},
    {"name":"summary","type":"STRING"}
  ]},
  {"label":"Date","properties":[
    {"name":"value","type":"DATE"}
  ]},

  # Context
  {"label":"Sector","properties":[{"name":"name","type":"STRING"}]},
  {"label":"Region","properties":[{"name":"name","type":"STRING"}]},
  {"label":"Country","properties":[
    {"name":"iso2","type":"STRING"},
    {"name":"name","type":"STRING"}
  ]},

  # TTPs, vulnerabilities, mitigations
  {"label":"Technique","properties":[
    {"name":"technique_id","type":"STRING"},
    {"name":"name","type":"STRING"},
    {"name":"tactic","type":"STRING"},
    {"name":"summary","type":"STRING"}
  ]},
  {"label":"CVE","properties":[
    {"name":"id","type":"STRING"},
    {"name":"published_date","type":"DATE"},
    {"name":"summary","type":"STRING"}
  ]},
  {"label":"Motivation","properties":[
    {"name":"type","type":"STRING"}
  ]},
  {"label":"Mitigation","properties":[
    {"name":"name","type":"STRING"},
    {"name":"framework","type":"STRING"},
    {"name":"control_id","type":"STRING"},
    {"name":"summary","type":"STRING"}
  ]},
  {"label":"Capability","properties":[
    {"name":"name","type":"STRING"},
    {"name":"summary","type":"STRING"}
  ]},

  # Sources
  {"label":"Source","properties":[
    {"name":"id","type":"STRING"},
    {"name":"title","type":"STRING"},
    {"name":"publisher","type":"STRING"},
    {"name":"published_on","type":"DATE"},
    {"name":"url","type":"STRING"}
  ]}
]
```

---

## 🔗 Relationship Definitions

```python
RELATIONS = [
  {"label":"attacked","description":"ThreatActor attacked Victim"},
  {"label":"uses","description":"Actor or Malware uses Tool or Malware"},
  {"label":"exploits","description":"Malware or Tool exploits CVE"},
  {"label":"abuses","description":"Actor or Malware abuses C2"},
  {"label":"targets","description":"Campaign targets Victim"},
  {"label":"includes","description":"Campaign includes Actor or Malware"},
  {"label":"occurred_on","description":"Event occurred on Date"},
  {"label":"has_alias","description":"ThreatActor alias"},
  {"label":"attributed_to","description":"Incident attributed to Actor"},
  {"label":"involved_malware","description":"Malware involved in Incident"},
  {"label":"involved_tool","description":"Tool involved in Incident"},
  {"label":"used_technique","description":"Technique used in Incident or by Actor"},
  {"label":"occurred_in","description":"Incident occurred in Region/Country"},
  {"label":"targets_sector","description":"Incident targets Sector"},
  {"label":"located_in","description":"Victim located in Region/Country"},
  {"label":"motivated_by","description":"Actor motivation"},
  {"label":"exploited_in","description":"Incident exploited CVE"},
  {"label":"mitigates","description":"Mitigation mitigates Incident or Technique"},
  {"label":"leverages","description":"Entity leverages Capability"},
  {"label":"supported_by","description":"Fact supported by Source"}
]
```

---

## 🧩 Allowed Graph Schema

```python
POTENTIAL_SCHEMA = [
  ("ThreatActor","uses","Malware"),
  ("ThreatActor","uses","Tool"),
  ("ThreatActor","abuses","C2_Infrastructure"),
  ("Malware","exploits","CVE"),
  ("Malware","abuses","C2_Infrastructure"),
  ("ThreatActor","attacked","Victim"),
  ("Campaign","includes","ThreatActor"),
  ("Campaign","includes","Malware"),
  ("Campaign","targets","Victim"),
  ("Campaign","occurred_on","Date"),
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
  ("ThreatActor","supported_by","Source"),
  ("Malware","supported_by","Source"),
  ("Incident","supported_by","Source"),
  ("CVE","supported_by","Source"),
  ("Technique","supported_by","Source"),
  ("Mitigation","supported_by","Source")
]
```

---

## 🔑 Primary Keys

```python
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
  "Country":"iso2",
  "Technique":"technique_id",
  "Motivation":"type",
  "Mitigation":"name",
  "Capability":"name",
  "Source":"id"
}
```

---

## 📝 Extraction Rules

```text
TASK
- From the input text, extract concrete, evidence-backed facts
- Output a SINGLE Neo4j Cypher import query

STRICT RULES
- Output ONLY raw Cypher; no explanations
- Use MERGE keyed by PRIMARY_PROP
- Never overwrite non-null values with null or empty
- Use date('YYYY-MM-DD'); do not invent dates
- Countries must use ISO-3166-1 alpha-2
- Normalize sectors (Energy, Healthcare, Finance, Government, etc.)
- Map techniques to MITRE ATT&CK IDs
- Explicitly model AI/Automation via Capability nodes
- Attach evidence (source_id, page, quote) whenever available
```
