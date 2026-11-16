# awardsignal

Which federal contractors are quietly becoming acquisition targets, and what has a given company actually won lately? awardsignal is a local forwarding client for the hosted federal contractor-signals MCP service: it answers `tools/list` locally from the captured tool definitions and forwards every call to the service, returning the answers exactly as the service sends them back.

The headline read, `contractor_signals`, returns a 0–100 acquisition-likelihood score for a named contractor, with the reasoning shown, built from public USAspending and EDGAR data. That score is market intelligence about a company — a probabilistic, reasoning-transparent estimate — not a compliance verdict, an exclusion check, or investment advice, and this client treats it that way: it forwards the question and passes the answer through. The other two tools are a daily ranked feed of rising contractors over the same public data and a direct live search of the public USAspending awards dataset.

All 3 captured tools are built; there are no omissions.

## The tools

- **`contractor_signals`** — The acquisition-likelihood / rising-contractor signal profile for one federal contractor: a 0–100 pattern-based score with full reasoning, plus award velocity, multi-year trajectory, agency and NAICS mix, biggest win, set-aside footprint, and, when the company is public, its EDGAR ticker and recent corporate-event filings. `name` is required.
- **`rising_contractors`** — The current ranked "Rising Contractors / Likely M&A Targets" feed: small and mid-size federal contractors showing the strongest leading acquisition signals, refreshed daily. Cap the rows with `limit` (default 20). No required parameters.
- **`search_awards`** — A direct live search of US federal awards in USAspending.gov, filterable by `naics` codes, `agency`, `recipient`, `keyword`, a `start`/`end` date range, and a `minAmount`/`maxAmount` dollar range, with a `limit` on rows returned. No required parameters.

## Install and run

As an MCP server via npm:

```bash
npx -y @reneturcios/awardsignal mcp
```

MCP client configuration:

```json
{
  "mcpServers": {
    "awardsignal": {
      "command": "npx",
      "args": ["-y", "@reneturcios/awardsignal", "mcp"]
    }
  }
}
```

As a container:

```bash
docker run -i ghcr.io/cloudcorprecords/awardsignal
docker run --rm ghcr.io/cloudcorprecords/awardsignal rising_contractors '{}'
```

## CLI

One binary serves both the command line and the MCP server. Each tool is a subcommand taking a single JSON string with that tool's arguments:

```bash
awardsignal contractor_signals '{"name":"ECS Federal"}'
awardsignal rising_contractors '{"limit":10}'
awardsignal search_awards '{"naics":["541512"],"minAmount":1000000}'
awardsignal --version
```

Exit codes: `0` when the forwarded call succeeds, `1` when the upstream result is an error, and `2` for usage problems such as an unknown subcommand or a malformed JSON argument.

## Configuration

- `GOVCONSIGNALS_MCP_URL` — Overrides the upstream endpoint. The default is the pinned hosted endpoint, `https://ma.dropwatchhq.com/mcp`. (The override variable is a reconstruction added by this rebuild.)

The upstream registry record declares no headers, so no authentication header is ever sent; the tool surface was captured keyless.

Upstream failures — a JSON-RPC error from the service, a non-2xx response, an unreadable body, or a request that fails outright — are reported to the caller as an error result whose text starts with `awardsignal error:`.

## How forwarding works

The first forwarded call opens a session: an `initialize` request (protocol version `2025-03-26`), followed by a `notifications/initialized` notification, then the `tools/call` itself. If the service answers the initialize request with an `Mcp-Session-Id` header, that value is carried on every later request, and the open session is reused for the rest of the process. Replies are accepted either as plain JSON or as server-sent events. Successful results pass through verbatim, including any `isError` flag and any extra fields the service includes.

## Development

```bash
npm install
npm run build   # tsc -> dist/ (dist/cli.js is the bin)
npm test        # tsx --test test/*.test.ts — stubbed upstream, no network
npm run dev     # run the CLI from source
```

The test suite checks the served tool list against the captured surface (all 3 tools, deep-equal, in order, with no titles invented and no `required` key added to the two tools that carry none), exercises the session handshake and its reuse, verifies that no `Authorization` header is ever sent under any environment, and verifies that arguments go upstream untouched and results come back untouched.

## License

MIT — see `LICENSE`. Copyright (c) 2026 Rene Turcios.
