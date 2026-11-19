**Company:** Meridian Data Systems, which sells an observability stack with a
natural-language query layer.

**Project:** *QueryBridge*, a Python MCP (Model Context Protocol) server that lets
AI assistants explore Meridian's columnar analytics database ("PrismDB") over HTTP.
It exposes tools such as `read_query` (run a SELECT), `list_tables`, and
`test_connection`, built on a popular open-source Python MCP server framework.
Deployment is flexible: run it
locally over stdio, or as an HTTP service in Kubernetes via the bundled Helm chart,
with OAuth bearer-token auth available behind an OAuth-enablement flag.

**Your task:** You are the security engineer reviewing QueryBridge's default
posture before a customer rollout. The product brief claims "authentication is
supported." Verify what actually happens when an operator runs the server with no
configuration at all — in particular, who can reach the tool endpoints over the
network and whether any authentication stands between them and the database. Then
assess how the Helm chart amplifies or mitigates this.

**Scope note:** Focus on the settings module (`config.py`: the `ServerConfig`
dataclass defaults and `load_server_config`) and the server module (`server.py`:
how `_auth` is constructed from `oauth_enabled`, and the startup path in `main()`).
Also inspect the bundled Kubernetes chart templates (`deployment.yaml`,
`service.yaml`, `_helpers.tpl`) and the example env file for what shipped
deployments inherit.
