# FAERS Graph Analytics with Python and Cypher Queries

This documentation provides a comprehensive guide to creating a FAERS (FDA Adverse Event Reporting System) analytics project using **Neo4j** as the graph database, **Cypher queries** for data analysis, and Python for programmatic interaction. The project focuses on analyzing drug adverse event reports, their outcomes, and related details using a graph-based approach.

---

## **Project Overview**

FAERS analytics involves exploring and analyzing adverse event reports to extract meaningful insights. Neo4j's ability to model complex relationships makes it ideal for understanding the intricate connections between cases, drugs, reactions, manufacturers, and outcomes.

Key Features:
1. Create a graph database representing FAERS data (cases, drugs, reactions, outcomes, manufacturers).
2. Perform advanced analyses using Cypher queries.
3. Automate queries and extract insights programmatically using Python.

---

## **Tools and Technologies**

- **Neo4j**: Graph database to store and query FAERS data.
- **Cypher**: Query language for Neo4j.
- **Python**: Automates interactions with Neo4j and extracts data insights.
- **Neo4j Python Driver**: Enables Python programs to interact with Neo4j.

---

## **Setup**

### **1. Neo4j Setup**

1. Install Neo4j Community Edition or sign up for Neo4j Aura.
2. Create a new database and note the connection details:
   - **Bolt URI**
   - **Username**
   - **Password**

### **2. Install Python Dependencies**

Ensure Python is installed. Then, install the Neo4j Python driver:

```bash
pip install neo4j
```

---

## **Dataset**

### **Overview**

The dataset includes the following entities and their relationships:

- **Nodes**:
  - `Case`: Individual reports of adverse events.
  - `Drug`: Medications involved in the cases.
  - `Reaction`: Adverse reactions reported.
  - `Outcome`: Long-term results of adverse events.
  - `Manufacturer`: Companies producing the drugs.
  - `AgeGroup`: Categorized age groups of the individuals in cases.

- **Relationships**:
  - `IS_PRIMARY_SUSPECT`, `IS_SECONDARY_SUSPECT`, `IS_CONCOMITANT`: Drug roles in a case.
  - `HAS_REACTION`: Links cases to reported reactions.
  - `RESULTED_IN`: Links cases to outcomes.
  - `REGISTERED`: Links drugs to manufacturers.
  - `FALLS_UNDER`: Links cases to age groups.

### **Data Model**

The FAERS data is structured as a graph with nodes and relationships:

```plaintext
(:Case {primaryid, age, gender, reportDate, ...})
  -[:IS_PRIMARY_SUSPECT]-> (:Drug {name, primarySubstance, ...})
  -[:REGISTERED]-> (:Manufacturer {manufacturerName, ...})
  -[:HAS_REACTION]-> (:Reaction {description, ...})
  -[:RESULTED_IN]-> (:Outcome {outcome, ...})
```

---

## **Cypher Queries**

Below are key Cypher queries to analyze the FAERS graph data:

### **1. Retrieve All Drugs Prescribed for a Specific Case**

```cypher
MATCH (c:Case {primaryid: 100654764})
OPTIONAL MATCH (c)-[:IS_PRIMARY_SUSPECT]->(d:Drug)
OPTIONAL MATCH (c)-[:IS_SECONDARY_SUSPECT]->(ds:Drug)
OPTIONAL MATCH (c)-[:IS_CONCOMITANT]->(dc:Drug)
RETURN c.primaryid AS CaseID,
       COLLECT(d.name) AS PrimarySuspectDrugs,
       COLLECT(ds.name) AS SecondarySuspectDrugs,
       COLLECT(dc.name) AS ConcomitantDrugs;
```

### **2. Get Reactions Reported for a Specific Drug**

```cypher
MATCH (d:Drug {name: "DEXAMETHASONE"})<-[:IS_PRIMARY_SUSPECT]-(c:Case)-[:HAS_REACTION]->(r:Reaction)
RETURN d.name AS DrugName, COLLECT(r.description) AS Reactions;
```

### **3. Count Cases for Each Outcome**

```cypher
MATCH (c:Case)-[:RESULTED_IN]->(o:Outcome)
RETURN o.outcome AS Outcome, COUNT(c) AS CaseCount;
```

### **4. Retrieve Cases Involving a Specific Age Group**

```cypher
MATCH (c:Case)-[:FALLS_UNDER]->(a:AgeGroup {ageGroup: "Adult"})
RETURN c.primaryid AS CaseID, c.age AS Age, c.gender AS Gender, c.reporterOccupation AS Reporter;
```

### **5. Identify Manufacturers Linked to Adverse Reactions**

```cypher
MATCH (m:Manufacturer)<-[:REGISTERED]-(d:Drug)<-[:IS_PRIMARY_SUSPECT]-(c:Case)-[:HAS_REACTION]->(r:Reaction {description: "Insomnia"})
RETURN m.manufacturerName AS Manufacturer, COUNT(c) AS CaseCount;
```

---

## **Python Integration**

### **Connecting to Neo4j**

The following Python script demonstrates how to run Cypher queries:

```python
from neo4j import GraphDatabase

# Neo4j credentials
uri = "bolt://<HOST>:<PORT>"  # Replace with your Neo4j host and port
username = "neo4j"            # Your Neo4j username
password = "<PASSWORD>"       # Your Neo4j password

# Connect to Neo4j
driver = GraphDatabase.driver(uri, auth=(username, password))

# Function to execute a query
def run_query(query):
    with driver.session() as session:
        result = session.run(query)
        return [record.data() for record in result]

# Example: Query for reactions to a drug
query = """
MATCH (d:Drug {name: "DEXAMETHASONE"})<-[:IS_PRIMARY_SUSPECT]-(c:Case)-[:HAS_REACTION]->(r:Reaction)
RETURN d.name AS DrugName, COLLECT(r.description) AS Reactions;
"""
results = run_query(query)

# Print results
for record in results:
    print(record)

# Close connection
driver.close()
```

---

## **Conclusion**

This project showcases how to leverage Neo4j and Python to analyze complex datasets like FAERS. By modeling relationships as a graph, the project enables detailed insights that are challenging to achieve with traditional databases. Future extensions could include integrating machine learning for predictive analytics or building a user interface for easier data exploration.
