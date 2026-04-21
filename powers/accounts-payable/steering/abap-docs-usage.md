# Using the ABAP Docs MCP Server

This server is your **ABAP knowledge base** — it searches offline ABAP/RAP documentation, SAP Help content, SAP Community posts, and Software Heroes articles, plus exposes an ABAP feature-matrix lookup and local ABAP linting. Use it whenever you need authoritative ABAP / CDS / RAP reference material or community examples.

## No installation required

The server is hosted publicly at:

```
https://mcp-abap.marianzeis.de/mcp
```

No credentials, no local setup. Kiro connects directly via the URL.

## Configure Kiro

Add this entry to `.kiro/settings/mcp.json` (workspace) or `~/.kiro/settings/mcp.json` (user):

```json
{
  "mcpServers": {
    "abap-docs": {
      "type": "url",
      "url": "https://mcp-abap.marianzeis.de/mcp"
    }
  }
}
```

## Tools

| Tool | Purpose |
|------|---------|
| `search` | Unified search across offline ABAP/RAP docs + optional online sources (SAP Help, SAP Community, Software Heroes) |
| `fetch` | Retrieve the full document or community post by ID |
| `abap_feature_matrix` | Look up which ABAP features are available at a given release |
| `sap_community_search` | Dedicated SAP Community search for troubleshooting, error messages, workarounds |
| `abap_lint` | Local ABAP linting via the server |

## Typical prompts

- "What does the `FILTER` operator do in ABAP 7.40 SP08+?"
- "Find SAP Community posts about error `CX_SY_ITAB_DUPLICATE_KEY` in RAP."
- "Is `WITH PRIVILEGED ACCESS` available in SAP_BASIS 7.52?"
- "Fetch the full SAP Help page for the `VIRTUAL` keyword in CDS."

## Auto-approve (recommended)

Since all tools are read-only, you can safely auto-approve them:

```json
{
  "mcpServers": {
    "abap-docs": {
      "type": "url",
      "url": "https://mcp-abap.marianzeis.de/mcp",
      "autoApprove": [
        "search",
        "fetch",
        "abap_feature_matrix",
        "sap_community_search",
        "abap_lint"
      ]
    }
  }
}
```

## Troubleshooting

### Slow or unresponsive
This is a free hosted service. If it's slow, try again after a few minutes or narrow your search query.

### No results
The offline index covers the official SAP ABAP / RAP documentation and selected community content. Very new SAP announcements may not be indexed yet — fall back to a direct web search.
