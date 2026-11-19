# Hints

## Rung 1
**Hint 1 (cheap):** Trace two values end-to-end: the address the HTTP server
   binds, and whether the MCP server instance receives an auth provider at all. Both
   come straight from config defaults when the operator sets nothing.

## Rung 2
**Hint 2 (narrower):** In `config.py`, `ServerConfig.host` defaults to a literal
   string and `load_server_config` reads the host override from the environment with
   the same insecure fallback; three lines down, `oauth_enabled` parses its
   enablement flag with a `"false"` default. In
   `server.py`, `_auth` stays `None` unless `server_config.oauth_enabled` is true.

## Rung 3
**Hint 3 (narrow):** The dangerous pattern is insecure-by-default composition:
   binding every interface (`0.0.0.0`) *and* constructing the server app with
   `auth=None`
   means a default deployment is a network-reachable, unauthenticated API. Neither
   misconfiguration is exotic — they are the defaults, and the Helm chart's
   `_helpers.tpl`/deployment template propagate the same host value into clusters.

## Rung 4
**Hint 4 (spicy):** With defaults, `main()` starts uvicorn on
   `host="0.0.0.0", port=8080` with `auth=None`; anyone who can route to the pod can
   invoke `read_query` with arbitrary SQL text — no token, no handshake. There is
   also no startup guard: nothing refuses to boot when HTTP is exposed without
   OAuth.

## Rung 5
**Hint 5 (giveaway):** Exploit: deploy the chart unmodified (or run the server
   bare with HTTP transport); from any network-adjacent host send a JSON-RPC
   `tools/call` request for `read_query` with `{"query": "SELECT ..."}` (or a
   write-capable statement against the backing DB) directly to :8080 — full
   unauthenticated data access. Fix shape: default the bind to loopback
   (`127.0.0.1`) and fail closed at startup if HTTP would be served on a
   non-loopback address without OAuth enabled.
