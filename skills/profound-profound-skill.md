---
name: Profound
description: Use when building AI analytics integrations, querying brand visibility and citation data, tracking AI crawler activity, managing knowledge bases, building and running AI agents, or connecting to AI tools via Model Context Protocol (MCP).
metadata:
    mintlify-proj: profound
    version: "1.0"
---

# Profound Skill

## Product summary

Profound is an AI analytics platform that tracks how AI systems (ChatGPT, Claude, Gemini, Perplexity) interact with your website and brand. It provides visibility into AI-driven traffic, citations, sentiment analysis, and crawler activity through a REST API, Python/JavaScript SDKs, and Model Context Protocol (MCP) integration. The primary documentation is at https://docs.tryprofound.com. Key entry points: REST API at `https://api.tryprofound.com`, MCP server at `https://mcp.tryprofound.com/mcp`, and SDKs available via `pip install profound` (Python) or npm (JavaScript). Authentication uses API keys generated in Settings → API Keys.

## When to use

Reach for Profound when:
- Querying AI visibility metrics, citation data, or sentiment analysis for a brand or category
- Tracking which AI crawlers (GPTBot, PerplexityBot, etc.) visit a domain and how often
- Building custom reports or dashboards using raw API data
- Integrating Profound analytics into an AI agent or MCP-connected tool (Claude, ChatGPT, Copilot Studio, etc.)
- Managing knowledge bases and searching documents via API
- Building, publishing, or running Profound Agents programmatically
- Analyzing raw per-execution prompt-answer data
- Setting up Agent Analytics to track AI interactions on a website

## Quick reference

### Authentication
- **API Key**: Generate in Settings → API Keys (Enterprise plan required)
- **Header method** (recommended): `X-API-Key: your_api_key_here`
- **Rate limit**: 600 requests/hour per key; check `X-RateLimit-Remaining` header
- **Errors**: 401 = invalid key, 403 = insufficient permissions, 429 = rate limit exceeded

### Core API endpoints
| Endpoint | Purpose |
|----------|---------|
| `POST /v2/reports/visibility` | Brand visibility in AI answers (share of voice, position) |
| `POST /v2/reports/citations` | Which domains/pages AI cites (count, share, rank) |
| `POST /v2/reports/sentiment` | Sentiment analysis (positive/negative percentages) |
| `POST /v2/reports/query-fanouts` | Search queries AI generates behind the scenes |
| `POST /v2/reports/factcheck` | FactCheck scores and accuracy metrics |
| `POST /v1/agents/{agent_id}/run` | Start an agent run |
| `GET /v1/agents/{agent_id}/runs/{run_id}` | Poll agent run status and results |
| `POST /v1/knowledge-bases/{kb_id}/search` | Search knowledge base documents |
| `POST /v1/knowledge-bases/{kb_id}/documents` | Add/update documents |
| `GET /v1/org/categories` | List categories (required for visibility reports) |
| `GET /v1/org/domains` | List tracked domains (required for traffic reports) |

### MCP server
- **URL**: `https://mcp.tryprofound.com/mcp`
- **Authentication**: OAuth 2.0 (sign in with Profound account)
- **Supported clients**: Claude, ChatGPT, Cursor, Gemini CLI, Copilot Studio, VS Code, WRITER, others
- **Key tools**: `list_agents`, `run_agent`, `get_agent_run`, `get_visibility_report`, `get_citations_report`, `get_sentiment_report`, `get_bots_report`, `get_referrals_report`

### Date handling
- Format: `YYYY-MM-DD` (ISO 8601)
- **Critical**: `end_date` is exclusive — to include all of 2026-05-10, send `end_date="2026-05-11"`
- Timezone: UTC by default; specify in request if needed

### SDKs
| Language | Install | Import |
|----------|---------|--------|
| Python | `pip install profound` | `from profound import Profound` |
| JavaScript | `npm install @profound/sdk` | `import { Profound } from '@profound/sdk'` |

## Decision guidance

| Scenario | Use REST API | Use MCP | Use SDK |
|----------|-------------|--------|--------|
| Building a custom dashboard or report | ✓ | | ✓ |
| Integrating into Claude, ChatGPT, or Copilot | | ✓ | |
| Automating data retrieval in Python/JS | | | ✓ |
| One-off queries or testing | ✓ | | |
| Building an AI agent that needs Profound data | | ✓ | |
| Knowledge base operations | ✓ | | ✓ |
| Agent creation and publishing | | ✓ | ✓ |

| Report type | Endpoint | Scope | Key metric |
|------------|----------|-------|-----------|
| Brand visibility | `/v2/reports/visibility` | Category + date range | `visibility_score` |
| Citations | `/v2/reports/citations` | Category + date range | `citation_share` |
| Sentiment | `/v2/reports/sentiment` | Category + date range | `positive_sentiment`, `negative_sentiment` |
| Bot traffic | `/v2/reports/bots` | Domain + date range | `count`, `citations` |
| Referral traffic | `/v2/reports/referrals` | Domain + date range | `visits` |

## Workflow

### Query visibility data via REST API
1. **Get your category ID**: Call `GET /v1/org/categories` to list categories, pick one by name
2. **Construct the request**: Build a POST body with `category_id`, `start_date`, `end_date`, `metrics` (e.g., `["visibility_score"]`), and optional `dimensions` (e.g., `["model", "date"]`)
3. **Remember end_date is exclusive**: To include all of 2026-05-10, send `end_date="2026-05-11"`
4. **Send the request**: `POST /v2/reports/visibility` with `X-API-Key` header
5. **Parse the response**: Check `info.query.metrics` to find the position of each metric in the `metrics` array of each row (don't hardcode positions)
6. **Handle pagination**: Use `cursor` from the response to fetch the next page if needed

### Run an existing agent
1. **List agents**: Call `GET /v1/agents` or use MCP `list_agents` to find the agent ID
2. **Get input schema**: Call `GET /v1/agents/{agent_id}` to see what inputs the agent expects
3. **Start the run**: Call `POST /v1/agents/{agent_id}/run` with inputs matching the schema
4. **Poll for completion**: Call `GET /v1/agents/{agent_id}/runs/{run_id}` repeatedly until `status` is `succeeded`, `failed`, or `cancelled`
5. **Extract outputs**: Read the `outputs` object from the final run response

### Build and publish a new agent (via MCP)
1. **Start a build session**: Call `start_agent_build_session` with a plain-language intent
2. **Explore node types**: Call `list_agent_node_types` to see available building blocks
3. **Create a draft**: Call `create_agent_definition` with `preview: true` to review, then `preview: false` to save
4. **Validate**: Call `validate_agent_definition` to catch structural issues
5. **Publish**: Call `publish_agent_definition` with `preview: true` to review, then `preview: false` to go live

### Search and manage knowledge base
1. **List knowledge bases**: Call `GET /v1/knowledge-bases` to find the KB ID
2. **Search**: Call `POST /v1/knowledge-bases/{kb_id}/search` with `query`, `top_k`, and optional `filters` (tags, folders)
3. **Add documents**: Call `POST /v1/knowledge-bases/{kb_id}/documents` with `name`, `text`, and optional `folder`
4. **Update documents**: Call `PUT /v1/knowledge-bases/{kb_id}/documents` with the same fields to overwrite
5. **Delete documents**: Call `DELETE /v1/knowledge-bases/{kb_id}/documents` with `name`

### Connect MCP to an AI tool
1. **Choose your tool**: Identify the MCP client (Claude Desktop, ChatGPT, Copilot Studio, etc.)
2. **Add the server**: In the tool's settings, add MCP server at `https://mcp.tryprofound.com/mcp`
3. **Authenticate**: Select OAuth 2.0, sign in with your Profound account
4. **Verify**: Confirm the Profound tools appear in the tool's available resources
5. **Use in prompts**: Reference Profound tools directly in your prompts (e.g., "Get visibility report for category X")

## Common gotchas

- **end_date is exclusive**: Sending `end_date="2026-05-10"` excludes that date. Always add 1 day to the inclusive end date you want.
- **Don't hardcode metric positions**: The API returns `info.query.metrics` to tell you the order. Use `order.index("metric_name")` to find positions, not hardcoded integers.
- **API key is sensitive**: Never commit it to version control. Use environment variables (`PROFOUND_API_KEY`) or secure vaults.
- **Category ID is required for visibility reports**: You can't query visibility without a category. Use `GET /v1/org/categories` first.
- **Domain must be exact hostname**: `www.example.com` and `example.com` are different. Use the exact value from `GET /v1/org/domains`.
- **Agent must be published to run**: Drafts cannot be executed. Call `publish_agent_definition` first.
- **Inputs must match schema**: Running an agent with wrong input keys will fail. Always call `GET /v1/agents/{agent_id}` to inspect `input_schema` first.
- **Rate limit is per key, not per user**: Each API key has its own 600 req/hour limit. Check `X-RateLimit-Remaining` header to avoid surprises.
- **MCP OAuth requires browser sign-in**: You can't use API keys with MCP. You must authenticate via OAuth in the MCP client.
- **Knowledge base search returns snippets by default**: Set `return_full_page: true` to get full document content instead of matched snippets.
- **V1 report endpoints are deprecated**: Use `/v2/reports/*` endpoints instead. V1 endpoints remain active but will be sunset.

## Verification checklist

Before submitting work with Profound:

- [ ] API key is stored securely (environment variable, not hardcoded)
- [ ] Date ranges use exclusive `end_date` (add 1 day to the inclusive end date)
- [ ] Metric positions are read from `info.query.metrics`, not hardcoded
- [ ] Category ID or domain is correct (verified with `GET /v1/org/categories` or `GET /v1/org/domains`)
- [ ] Agent is published before attempting to run it
- [ ] Agent run inputs match the `input_schema` from `GET /v1/agents/{agent_id}`
- [ ] Rate limit headers are checked; no 429 errors in logs
- [ ] Knowledge base documents use correct folder paths and tags
- [ ] MCP client is authenticated via OAuth (not API key)
- [ ] Pagination cursors are used for large result sets (don't assume all data fits in one response)
- [ ] Error responses are handled (401, 403, 429, 422 validation errors)

## Resources

- **Comprehensive page index**: https://docs.tryprofound.com/llms.txt — paste this into your AI assistant for full documentation navigation
- **REST API reference**: https://docs.tryprofound.com/rest-api/introduction — all endpoints, parameters, and response formats
- **Cookbook (recipes)**: https://docs.tryprofound.com/cookbook/introduction — end-to-end examples for common tasks (visibility, citations, leaderboards)
- **MCP capabilities**: https://docs.tryprofound.com/mcp/capabilities/analytics-capabilities — full list of MCP tools and how to use them
- **Agent Analytics setup**: https://docs.tryprofound.com/agent-analytics/self-serve-onboarding — configure tracking for your domain

---

> For additional documentation and navigation, see: https://docs.tryprofound.com/llms.txt