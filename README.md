# Fiskmas MCP — server metadata (enabler repo)

Fiskmås is hosting for AI agents: the AI client you already use (Claude,
ChatGPT, Cursor) talks to [Fiskmås](https://fiskmas.dev) over MCP, and that
client **is** the deployment console. No web dashboard, no CI YAML — your agent
creates the project, pushes a Docker image, deploys it to a live HTTPS URL, and
watches the rollout health-gate.

This repository is **server metadata only** (description + client configuration
for directories and MCP clients that read repos). The product itself is
hosted, not self-hosted: the server source is private and the deployment
target is the Fiskmås platform (2-node k3s, per-tenant namespaces, digest-pinned
health-gated rollouts). Pricing and limits are on the landing page.

## What the server does

| Capability | Notes |
|---|---|
| Accounts | `create_account` starts onboarding (email → verify link → one API token); `whoami` returns the identity + plan limits; `reissue_token` recovers a lost/rotated token (one-time email verify link). |
| Projects | `create_project` / `list_projects` / `get_project` / `set_env` / `list_env` / `delete_project`. |
| Deploys | `deploy_project` pulls an image by digest and runs one container with the plan's CPU/memory caps; `start_project` / `stop_project` / `restart_project` manage the lifecycle. |
| Observation | `get_status` (status, URL, digest, last deploy error), `get_logs` (recent log lines, plan-windowed), `get_stats` (live CPU/mem vs caps). |
| Knowledge | `list_skills` / `get_skill` serve the platform skill files (the agent onboarding doc). |
| Feedback | `send_feedback` routes problems back to the platform team. |

One token covers everything: the MCP `Authorization` header **and**
`docker login registry.fiskmas.dev` (the registry's token service issues
short-lived RS256 JWTs from it).

## Connecting

Free tier: 1 always-on project. Create an account at
<https://fiskmas.dev> — the verify page shows your API token once.

```bash
# Claude Code (Streamable HTTP, the MCP server is Bearer-authed)
claude mcp add --transport http fiskmas https://mcp.fiskmas.dev/mcp \
  --header "Authorization: Bearer <your-token>"

# then push + deploy with the same token:
docker login registry.fiskmas.dev -u <your-token>
```

The full tool contract (parameter lists, limits table, app contract) is served
by the server itself:

- skill: `https://fiskmas.dev/skill.md`
- machine-readable overview: `https://fiskmas.dev/llms.txt`
- tool schemas: `https://fiskmas.dev/docs.html`

`mcp.json` (also in this repo) is the same endpoint in generic MCP-client form.
