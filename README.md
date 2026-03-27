# 🗺️ Location Intelligence ADK Agent

An AI agent built with **Google ADK** and **Gemini 3.1 Pro**, connected to two remote MCP servers — **BigQuery** for enterprise data and **Google Maps** for real-world location analysis. Built as part of Track 2 Lab 1.

---

## 🤖 What It Does

The agent answers complex business questions by combining data from two sources:

- **BigQuery MCP** — queries demographic, foot traffic, pricing, and sales data from the `mcp_bakery` dataset
- **Google Maps MCP** — finds real-world locations, searches for competitors, and calculates travel routes

### Example queries it can answer:
- "Find the zip code in LA with the highest morning foot traffic score"
- "Search for bakeries in that zip code to see if it's saturated"
- "What is the maximum price charged for a Sourdough Loaf in the LA Metro area?"
- "Find the closest Restaurant Depot and check if drive time is under 30 minutes"

---

## 🏗️ Architecture
```
User Query
    │
    ▼
root_agent (Gemini 3.1 Pro via Vertex AI)
    │
    ├── BigQuery MCP Toolset
    │   └── https://bigquery.googleapis.com/mcp
    │       OAuth2 token via google.auth
    │       Tables: demographics, foot_traffic,
    │               bakery_prices, sales_history_weekly
    │
    └── Maps MCP Toolset
        └── https://mapstools.googleapis.com/mcp
            API Key authentication
            Tools: search_places, directions, distance
```

---

## 📁 Project Structure
```
launchmybakery/
├── data/                        # Pre-generated CSV files for BigQuery
├── adk_agent/                   # AI Agent Application (ADK)
│   └── mcp_bakery_app/          # App directory
│       ├── agent.py             # Agent definition
│       ├── tools.py             # MCP toolset configuration
│       └── .env                 # Project config and API keys
├── setup/                       # Infrastructure setup scripts
├── cleanup/                     # Infrastructure cleanup scripts
├── README.md                    # This file
└── TROUBLESHOOTING.md           # Common issues and fixes
```

---

## 🚀 Setup & Run

### Prerequisites
- Google Cloud project with billing enabled
- Cloud Shell (recommended)

### Step 1 — Clone the repo
```bash
git clone https://github.com/google/mcp.git
cd mcp/examples/launchmybakery
```

### Step 2 — Authenticate
```bash
gcloud auth application-default login
```

### Step 3 — Run setup scripts
```bash
# Enable APIs and create .env with API keys
chmod +x setup/setup_env.sh
./setup/setup_env.sh

# Create BigQuery dataset and tables
chmod +x setup/setup_bigquery.sh
./setup/setup_bigquery.sh
```

### Step 4 — Install ADK
```bash
python3 -m venv .venv
source .venv/bin/activate
pip install google-adk
```

### Step 5 — Start the agent
```bash
cd adk_agent

# Use this command in Cloud Shell (fixes 403 issue)
adk web --host 0.0.0.0 --port 8080 --allow_origins "*"
```

Open **Web Preview → port 8080** in Cloud Shell.

---

## 🧪 Sample Test Queries
```
1. "I want to open a bakery in Los Angeles. Find the zip code with the highest morning foot traffic score."

2. "Can you search for Bakeries in that zip code to see if it's saturated?"

3. "What is the maximum price being charged for a Sourdough Loaf in the LA Metro area?"

4. "Run a revenue forecast for December 2025 using my best performing store data for Sourdough Loaf at $18 per unit."

5. "Find the closest Restaurant Depot to the proposed area and confirm drive time is under 30 minutes."
```

---

## ⚙️ BigQuery Dataset

The `mcp_bakery` dataset contains 4 tables:

| Table | Description |
|---|---|
| `demographics` | Census data and population characteristics by zip code |
| `bakery_prices` | Competitor pricing and product details for baked goods |
| `sales_history_weekly` | Weekly sales performance by store and product |
| `foot_traffic` | Estimated foot traffic scores by zip code and time of day |

---

## 🛠️ Tech Stack

| Component | Technology |
|---|---|
| Agent Framework | Google ADK (google-adk) |
| LLM | Gemini 3.1 Pro Preview via Vertex AI |
| Data Source | BigQuery MCP Server |
| Location Source | Google Maps MCP Server |
| Auth — BigQuery | OAuth2 via google.auth |
| Auth — Maps | API Key |
| Language | Python 3.12 |
| Environment | Google Cloud Shell |

---

## ⚠️ Known Issues & Fixes

See [TROUBLESHOOTING.md](./TROUBLESHOOTING.md) for detailed fixes for:
- 403 Forbidden on POST requests in Cloud Shell
- Places API not enabled error
- API Key service blocked
- MCP Tools timeout
- Port 8000 Web Preview failure

---

## 🧹 Cleanup
```bash
chmod +x ../cleanup/cleanup_env.sh
./../cleanup/cleanup_env.sh
```

---

## 📚 References

- [Google ADK Documentation](https://google.github.io/adk-docs/)
- [BigQuery MCP Server](https://docs.cloud.google.com/bigquery/docs/use-bigquery-mcp)
- [Google Maps MCP Server](https://developers.google.com/maps/ai/grounding-lite)
- [Codelab: Build a Location Intelligence ADK Agent](https://codelabs.developers.google.com/adk-mcp-bigquery-maps)
