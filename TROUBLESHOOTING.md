# Troubleshooting Notes — Location Intelligence ADK Agent

Issues encountered and fixes applied during Track 2 Lab 1.

## Issue 1 — 403 Forbidden on POST requests in ADK Web UI

### What happened
POST requests to /apps/.../sessions and /run_sse returned 403 Forbidden.
GET requests worked fine.

### Why it occurred
Cloud Shell Web Preview proxy adds an authentication layer. POST requests from the ADK web UI do not carry the right headers expected by the proxy. This is a CORS/origin mismatch between the ADK server and Cloud Shell proxy layer.

### Fix
Instead of plain adk web, run:
adk web --host 0.0.0.0 --port 8080 --allow_origins "*"

### How to avoid next time
Always use --allow_origins "*" and port 8080 in Cloud Shell.
Add this alias to save time:
echo 'alias adkweb="adk web --host 0.0.0.0 --port 8080 --allow_origins \"*\""' >> ~/.bashrc
source ~/.bashrc

---

## Issue 2 — Places API not enabled

### What happened
403: Places API (New) has not been used in project 496499909513

### Why it occurred
The Maps API key belonged to a different project than the active working project.
API keys are tied to the project they were created in.
The Places API was never enabled on the key's own project.

### Fix
gcloud services enable places.googleapis.com maps-backend.googleapis.com geocoding-backend.googleapis.com --project=496499909513

### How to avoid next time
Before starting any lab, verify which project your API key belongs to.
Enable all required APIs on that same project, not just your active project.

---

## Issue 3 — API Key service blocked

### What happened
API_KEY_SERVICE_BLOCKED: Requests to places.googleapis.com are blocked.

### Why it occurred
The API key had API restrictions set — only specific APIs were allowed.
Places API (New) was not in the allowed list.

### Fix
Go to console.developers.google.com/apis/credentials
Click the key, go to API restrictions, set to Don't restrict key, then Save.

### How to avoid next time
For lab environments, always set API key restrictions to None.
Test your key before writing any agent code:
curl "https://places.googleapis.com/v1/places:searchText" -H "X-Goog-Api-Key: YOUR_KEY" -H "X-Goog-FieldMask: places.displayName" -H "Content-Type: application/json" -d '{"textQuery": "bakery in London"}'

---

## Issue 4 — MCP Tools timeout

### What happened
TimeoutError and ConnectionError: Failed to get tools from MCP server.
Maps MCP connected but timed out fetching the tools list.

### Why it occurred
mapstools.googleapis.com is a cloud-hosted MCP endpoint with higher latency.
The default timeout in StreamableHTTPConnectionParams was too short.

### Fix
Added timeout=30.0 in tools.py:
tools = MCPToolset(
    connection_params=StreamableHTTPConnectionParams(
        url=MAPS_MCP_URL,
        headers={"X-Goog-Api-Key": maps_api_key},
        timeout=30.0
    )
)

### Timeout reference
Local process MCP      — Default timeout (5s)
Cloud-hosted MCP       — 30 seconds
Slow external APIs     — 60 seconds

### How to avoid next time
Always set timeout explicitly for remote or cloud-hosted MCP servers.

---

## Issue 5 — Port 8000 Web Preview failure

### What happened
Unable to forward your request to a backend.
Couldn't connect to a server on port 8000.

### Why it occurred
Cloud Shell Web Preview is most stable on port 8080.
Port 8000 sometimes has proxy routing issues in Cloud Shell.

### Fix
adk web --host 0.0.0.0 --port 8080 --allow_origins "*"

### How to avoid next time
Always use port 8080 in Cloud Shell for Web Preview.

---

## Pre-Lab Checklist

Run these before starting every lab:

1. Confirm active project
gcloud config get-value project

2. Enable required APIs
gcloud services enable places.googleapis.com maps-backend.googleapis.com bigquery.googleapis.com aiplatform.googleapis.com run.googleapis.com

3. Test Maps API key
curl "https://places.googleapis.com/v1/places:searchText" -H "X-Goog-Api-Key: YOUR_KEY" -H "X-Goog-FieldMask: places.displayName" -H "Content-Type: application/json" -d '{"textQuery": "coffee shop"}'

4. Start ADK correctly
adk web --host 0.0.0.0 --port 8080 --allow_origins "*"

5. Always set timeout=30.0 in StreamableHTTPConnectionParams for remote MCP servers
