# Vulnerability and Fix

## GitHub Security Advisory

- [GHSA-73cv-556c-w3g6: Unauthenticated tool invocation via default oauth_enabled=False + host 0.0.0.0 bind](https://github.com/startreedata/mcp-pinot/security/advisories/GHSA-73cv-556c-w3g6)
- Affected versions: <= 3.0.1
- Patched version: 3.1.0

## Fixed Code Commit

- [Fix merge commit 1c7d3f9cd384854bf72c127d230bdb32299475ad](https://github.com/startreedata/mcp-pinot/commit/1c7d3f9cd384854bf72c127d230bdb32299475ad)
- [Fix pull request #95](https://github.com/startreedata/mcp-pinot/pull/95)

## Vulnerability

This is an insecure-by-default network exposure that also creates a confused-deputy condition.

`ServerConfig` defaults to HTTP on `0.0.0.0:8080`, and `OAUTH_ENABLED` defaults to `false`. When OAuth is disabled, `server.py` constructs the FastMCP server with `_auth = None`. The HTTP startup path does not check that a network-facing listener has authentication.

As a result, an unauthenticated network-adjacent caller can invoke MCP tools, including `read_query`, `list_tables`, schema tools, and table-config tools. The server performs these operations using its own Pinot credentials, so the caller inherits the privileges of that server-side account. This can expose table data and metadata and may allow schema or table-config changes, depending on the Pinot account's permissions.

The Helm chart amplifies the issue: it defaults `mcp.transport` to `http`, `mcp.host` to `0.0.0.0`, `mcp.port` to `8000`, and `mcp.oauth.enabled` to `false`. The deployment template passes those values directly to the container, while the chart's default service configuration exposes port 80 to the cluster. Merely deploying the shipped chart can therefore make the unauthenticated HTTP service available to workloads able to route to the Kubernetes service.

## Fix

The fix should make network exposure opt-in and fail closed:

1. Change the default HTTP bind address from `0.0.0.0` to loopback (`127.0.0.1`) in `ServerConfig`, environment loading, `.env.example`, and Helm values.
2. In `main()`, refuse to start HTTP or HTTPS on a non-loopback host unless `OAUTH_ENABLED=true`.
3. Make Helm exposure local-only by default, with a Service disabled unless explicitly enabled. Require OAuth before rendering or starting a non-loopback network exposure.
4. Continue to allow loopback HTTP for local use, and allow stdio as the safe local transport.

The upstream fix also adds parser-backed, single-statement read-only validation for `read_query`, reducing but not replacing the need for authentication and network-exposure controls.

## Example fixed startup guard

```python
def _is_loopback(host: str) -> bool:
    return host in {"127.0.0.1", "::1", "localhost"}


if (
    server_config.transport != "stdio"
    and not server_config.oauth_enabled
    and not _is_loopback(server_config.host)
):
    raise SystemExit(
        "Refusing to start: network HTTP transport requires OAuth "
        "or a loopback MCP_HOST."
    )
```
