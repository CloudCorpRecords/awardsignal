# Changelog

## 0.1.0 (2026-10-28)

- First release: a forwarding client for the hosted federal contractor-signals MCP server. All 3 captured tools are answered locally for tools/list, and every tools/call is forwarded upstream over a streamable-HTTP JSON-RPC session (initialize, notifications/initialized, tools/call, honouring the Mcp-Session-Id header when the upstream issues one).
- Tools: `contractor_signals`, `rising_contractors`, `search_awards`. Full surface, no omissions. The capture carries no `title` keys, so none are served; `rising_contractors` and `search_awards` carry no `required` key in the capture and none is served.
- Upstream results pass through verbatim, including the isError flag; upstream failures surface as `awardsignal error:` text results.
- One bin serves the CLI (one subcommand per tool, plus `--version`) and the stdio MCP server (`mcp` subcommand). The upstream endpoint can be overridden via `GOVCONSIGNALS_MCP_URL`. The upstream record declares no headers, so no auth header is ever sent.
