# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

```bash
# Setup
python3 -m venv hexstrike_env
source hexstrike_env/bin/activate
pip3 install -r requirements.txt

# Run the server (default port 8888, host 127.0.0.1)
python3 hexstrike_server.py
python3 hexstrike_server.py --debug
python3 hexstrike_server.py --port 8888

# Run the MCP client (connects to a running server)
python3 hexstrike_mcp.py --server http://127.0.0.1:8888
python3 hexstrike_mcp.py --debug

# Verify server is running
curl http://localhost:8888/health
```

Override host/port via environment: `HEXSTRIKE_PORT` and `HEXSTRIKE_HOST`.

## Architecture

Two-script system with a clear separation of concerns:

- **`hexstrike_server.py`** — Flask REST API server. Hosts all 150+ security tool integrations, the `IntelligentDecisionEngine`, `ModernVisualEngine`, process management, LRU caching, and AI agent classes. Exposes endpoints under `/api/` (e.g. `/api/command`, `/api/intelligence/*`, `/api/processes/*`).

- **`hexstrike_mcp.py`** — FastMCP client that wraps the server's REST API as MCP tools, making them consumable by AI agents (Claude Desktop, VS Code Copilot, Cursor, Roo Code, etc.). Connects via `http://127.0.0.1:8888` by default. The `HexStrikeClient` class handles retries and health-checks on startup.

### Key classes in `hexstrike_server.py`

| Class | Role |
|---|---|
| `ModernVisualEngine` | Terminal color/progress-bar rendering |
| `IntelligentDecisionEngine` | AI-powered tool selection and parameter optimization |
| `BugBountyWorkflowManager` | Bug bounty hunting automation |
| `CTFWorkflowManager` | CTF challenge solving |
| `CVEIntelligenceManager` | CVE monitoring and exploit analysis |
| `AIExploitGenerator` | Automated exploit development |
| `VulnerabilityCorrelator` | Attack chain discovery |
| `FailureRecoverySystem` | Error handling and graceful degradation |

### MCP integration config

`hexstrike-ai-mcp.json` is a template MCP server config. Replace `/path/hexstrike_mcp.py` with the actual path and `IPADDRESS` with the server IP before using it with an AI client.

## Python dependencies

Core: `flask`, `requests`, `psutil`, `fastmcp`, `aiohttp`  
Web automation: `beautifulsoup4`, `selenium`, `webdriver-manager`  
Proxy: `mitmproxy`  
Binary analysis: `pwntools`, `angr` (conditionally imported — heavy deps)  
Pin: `bcrypt==4.0.1` (required for pwntools/passlib compatibility)

External security tools (nmap, gobuster, nuclei, sqlmap, etc.) must be installed separately from their official sources — see `requirements.txt` comments for the full categorized list.
