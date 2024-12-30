# Neo4j Cricket Analytics with Python and Cypher Queries

This documentation provides a comprehensive guide to creating a cricket analytics project using **Neo4j Aura** as the graph database, **Cypher queries** for data analysis, and Python for programmatic interaction. The project focuses on analyzing cricket match data, player statistics, and team performances using a graph-based approach.

---

## **Project Overview**

Cricket analytics involves exploring and analyzing cricket match data to extract insights. Graph databases like Neo4j are well-suited for such projects due to their ability to model complex relationships between players, teams, matches, and events.

Key Features:
1. Create a graph database representing cricket data (teams, players, matches, dismissals).
2. Perform complex analyses using Cypher queries.
3. Interact with Neo4j using Python for automation and dynamic insights.

---

## **Tools and Technologies**

- **Neo4j Aura**: A cloud-based Neo4j database.
- **Cypher**: Neo4j's query language for graph data.
- **Python**: To automate queries and extract insights.
- **neo4j Python Driver**: For connecting and querying Neo4j from Python.

---

## **Setup**

### **1. Neo4j Aura Account**

1. Sign up at [Neo4j Aura](https://neo4j.com/cloud/aura/).
2. Create a free database (Aura Free Tier).
3. Note down the connection credentials:
   - Bolt URI
   - Username
   - Password

### **2. Install Python Dependencies**

Ensure Python is installed on your system. Install the Neo4j Python driver:

```bash
pip install neo4j
```

---

## **Dataset**

We generate a fake cricket dataset using Python to populate the database. The dataset includes:

- Players: Name, role (batsman, bowler, all-rounder), batting and bowling statistics.
- Teams: Name, matches won and lost.
- Matches: Match ID, date, location, teams, and events.

### **Data Model**

The data is modeled as a graph with the following nodes and relationships:

- **Nodes**:
  - `Player`
  - `Team`
  - `Match`

- **Relationships**:
  - `PLAYS_FOR`: Links a `Player` to a `Team`.
  - `PLAYED_IN`: Links a `Player` to a `Match`.
  - `WON` / `LOST`: Links a `Team` to a `Match`.
  - `DISMISSED_BY`: Links a `Player` (batsman) to another `Player` (bowler).

---

## **Step 1: Generate Data**

We use Python to create a dataset and upload it to Neo4j.

```python
from random import randint, choice
from datetime import datetime, timedelta
import json

# Generate fake players
roles = ["Batsman", "Bowler", "All-rounder"]
teams = ["Team A", "Team B", "Team C", "Team D"]

players = []
for i in range(1, 51):
    player = {
        "name": f"Player {i}",
        "role": choice(roles),
        "team": choice(teams),
    }
    players.append(player)

# Generate fake matches
matches = []
locations = ["Mumbai", "Sydney", "London", "Dubai"]

start_date = datetime(2020, 1, 1)
for i in range(1, 21):
    match_date = start_date + timedelta(days=randint(0, 1000))
    match = {
        "match_id": i,
        "date": match_date.strftime('%Y-%m-%d'),
        "location": choice(locations),
        "teams": [choice(teams), choice(teams)],
    }
    matches.append(match)

# Save data to JSON
with open("players.json", "w") as f:
    json.dump(players, f, indent=4)

with open("matches.json", "w") as f:
    json.dump(matches, f, indent=4)

print("Data generated and saved to players.json and matches.json.")
```

---

## **Step 2: Import Data into Neo4j**

Use Cypher queries in Neo4j to load the generated data.

### **1. Upload Player Data**

```cypher
LOAD CSV WITH HEADERS FROM 'file:///players.json' AS row
CREATE (:Player {name: row.name, role: row.role});
```

### **2. Upload Team Data**

```cypher
UNWIND $teams AS team
CREATE (:Team {name: team});
```

### **3. Upload Match Data**

```cypher
LOAD CSV WITH HEADERS FROM 'file:///matches.json' AS row
CREATE (:Match {match_id: row.match_id, date: row.date, location: row.location});
```

---

## **Step 3: Analysis Using Cypher Queries**

Here are some advanced queries to analyze the data:

### **1. List All Players in a Specific Team**
```cypher
MATCH (p:Player)-[:PLAYS_FOR]->(t:Team {name: 'Team A'})
RETURN p.name AS Player, p.role AS Role;
```

### **2. Find Matches Played by a Team**
```cypher
MATCH (t:Team {name: 'Team B'})-[:WON|LOST]->(m:Match)
RETURN m.match_id AS MatchID, m.date AS Date, m.location AS Location;
```

### **3. Find Players Dismissed by a Bowler**
```cypher
MATCH (b:Player {name: 'Player 1'})-[:DISMISSED_BY]->(p:Player)
RETURN p.name AS Batsman;
```

### **4. Player Performance in All Matches**
```cypher
MATCH (p:Player {name: 'Player 10'})-[:PLAYED_IN]->(m:Match)
RETURN m.match_id AS MatchID, m.date AS Date, m.location AS Location;
```

---

## **Step 4: Automate Queries in Python**

Use the following Python code to execute queries programmatically.

```python
from neo4j import GraphDatabase

# Connect to Neo4j
driver = GraphDatabase.driver("neo4j+s://<bolt-uri>", auth=("<username>", "<password>"))

def run_query(tx, query, parameters=None):
    result = tx.run(query, parameters)
    return [record for record in result]

def list_players_in_team(team_name):
    query = """
    MATCH (p:Player)-[:PLAYS_FOR]->(t:Team {name: $team_name})
    RETURN p.name AS Player, p.role AS Role
    """
    with driver.session() as session:
        return session.read_transaction(run_query, query, {"team_name": team_name})

players = list_players_in_team("Team A")
print(players)

# Close connection
driver.close()
```

---

## **Conclusion**

This project demonstrates how to use Neo4j Aura and Python to analyze cricket data effectively. By modeling relationships in a graph, you can perform complex queries that are difficult in relational databases. 
