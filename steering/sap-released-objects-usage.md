---
inclusion: auto
name: sap-released-objects-usage
description: "How to use the sap-released-objects MCP server — URL-based, no install, typical prompts for Clean Core compliance checks and released successor lookups."
---

# Using the SAP Released Objects MCP Server

This server is your **Clean Core compliance checker** — it searches the SAP Cloudification Repository for released APIs, checks Clean Core levels (A / B / C / D), and finds released successors for deprecated or non-released objects. Use it when you're writing new code on ABAP Cloud or migrating existing code that references SAP standard objects.

## No installation required

The server is hosted publicly at:

```
https://sap-released-objects-server-production.up.railway.app/mcp
```

No credentials, no local setup. Kiro connects directly via the URL.

## Configure Kiro

Add this entry to `.kiro/settings/mcp.json` (workspace) or `~/.kiro/settings/mcp.json` (user):

```json
{
  "mcpServers": {
    "sap-released-objects": {
      "type": "url",
      "url": "https://sap-released-objects-server-production.up.railway.app/mcp"
    }
  }
}
```

## Tools

| Tool | Purpose |
|------|---------|
| `sap_search_objects` | Search for released APIs by name or business concept |
| `sap_get_object_details` | Get Clean Core Level, state, and successor info for a specific object |
| `sap_find_successor` | Find released replacements for deprecated or non-released objects |
| `sap_check_clean_core_compliance` | Batch-check a list of objects for compliance |
| `sap_list_object_types` | List available TADIR object types with counts |
| `sap_get_statistics` | Repository-wide statistics by level, type, and state |
| `sap_list_versions` | List available S/4HANA versions for private cloud / on-premise |

## Typical prompts

- "Is table `MARA` released for ABAP Cloud?"
- "Find the released successor for `BAPI_MATERIAL_GET_ALL`."
- "Batch-check these objects for Level A compliance: `BAPI_SALESORDER_CREATEFROMDAT2`, `CL_CRM_BOL_CORE`, `RSAQ_REMOTE_QUERY_CALL`."
- "Show me all released APIs matching `*_EASY`."

## Auto-approve (optional)

For faster iteration, auto-approve the read-only tools:

```json
{
  "mcpServers": {
    "sap-released-objects": {
      "type": "url",
      "url": "https://sap-released-objects-server-production.up.railway.app/mcp",
      "autoApprove": [
        "sap_search_objects",
        "sap_get_object_details",
        "sap_find_successor",
        "sap_check_clean_core_compliance",
        "sap_list_object_types",
        "sap_get_statistics",
        "sap_list_versions"
      ]
    }
  }
}
```

## Troubleshooting

### Slow or unresponsive
This is a free hosted service; it can occasionally be slow or temporarily unavailable. Verify internet connectivity and try again after a few minutes.

### No results for a known SAP object
The Cloudification Repository covers released APIs and their successors. Custom objects (`Z*`, `Y*`) are not in the repository — use the ABAP ADT server for those.
