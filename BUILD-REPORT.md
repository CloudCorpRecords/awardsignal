# BUILD REPORT — awardsignal

Clean-room rebuild of `com.dropwatchhq/govcon-contractor-signals`
v1.0.0 (remote-only MCP server at `https://ma.dropwatchhq.com/mcp`),
built 2026-10-28 as a forwarding client. Package:
`@reneturcios/awardsignal` 0.1.0, registry name
`io.github.CloudCorpRecords/awardsignal`.

## Surface built

Full surface: **all 3 captured tools built**, no omissions.

| Tool | Required fields as served |
|------|---------------------------|
| `contractor_signals` | `name` |
| `rising_contractors` | NO `required` key in the capture — served verbatim, none invented |
| `search_awards` | NO `required` key in the capture — served verbatim, none invented |

Each served tool carries exactly the keys `name`, `description`,
`inputSchema`, exactly as captured: the capture carries NO `title`
keys, so none are served. The tool literals were generated
programmatically from
`harness/fixtures/govconsignals-tools-original.json` by the script at
`/tmp/b46-awardsignal-scratch/gen_forward.py` (parse + re-serialise
into `src/forward.ts`) — never retyped — and a post-generation check
(the generator's own round-trip assertion, the registration-integrity
test suite, plus the parity harness) confirmed the served literals
deep-equal the fixture's 3 tool objects, in order. Tools are
registered via the MCP SDK low-level Server API
(`ListToolsRequestSchema` / `CallToolRequestSchema` handlers), not
McpServer+zod.

## Fixture conflicts

**None.** The spec and the fixture agree on all 3 tools' names,
descriptions, required fields (including the two tools with no
`required` key: `rising_contractors`, `search_awards`) and property
shapes. The spec's note that the capture carries no `title` keys
matches the fixture exactly.

## Labelled reconstructions (from the spec's Wire section)

- Upstream endpoint pinned: `https://ma.dropwatchhq.com/mcp`.
- Env override `GOVCONSIGNALS_MCP_URL` — labelled reconstruction.
- The registry record declares NO headers on the remote: no auth
  header is ever sent. Tests assert that no `Authorization` header
  is sent under any environment — including with key-like env vars
  (`GOVCONSIGNALS_API_KEY`, `API_KEY`, `AUTHORIZATION`) set — and
  that no `X-API-Key` header is ever sent; upstream requests carry
  only `Content-Type`, `Accept` and, when issued, `Mcp-Session-Id`.
- Error rendering `awardsignal error:` text for upstream failures —
  labelled reconstruction.
- Upstream results pass through verbatim, including `isError` and
  extra fields. tools/call is forwarded as streamable-HTTP
  JSON-RPC: initialize (protocolVersion `2025-03-26`) →
  notifications/initialized → tools/call, honouring
  `Mcp-Session-Id` when issued; plain-JSON and SSE replies are
  both parsed. The forwarding core takes an injectable fetch;
  tests stub the upstream — no live network calls in tests.
- Contractor-score judgment (from the spec, recorded by the
  coordinator on the firsthand capture read): `contractor_signals`
  returns a 0–100 acquisition-likelihood / rising-contractor
  pattern score for a named federal CONTRACTOR (a company),
  computed from PUBLIC USAspending + EDGAR data, with its
  reasoning shown. This is built, and distinguished on the record
  from the screening SAFETY family (entity-screen's onboarding
  risk score, exclude-feed's OFAC/LEIE screens, this batch's
  omitted `screen_entity`): those return compliance/fraud/
  exclusion verdicts about a subject for gatekeeping decisions.
  An acquisition-likelihood estimate is market intelligence about
  companies — probabilistic, reasoning-transparent, about no
  person, checking no exclusion list, gating nothing. The same
  distinction covers `rising_contractors` (a published ranked feed
  over the same public data). `search_awards` is a direct live
  search of the public USAspending awards dataset. No personal
  data, no payment, metering, contact, signup or send tool exists
  in the surface.

## Deviations

None.

## Lockfile

- Route: generated clean from birth in the build dir with no
  `node_modules` and no pre-existing lockfile:
  `npm install --package-lock-only --ignore-scripts --offline`
  (Route 3).
- Parses as JSON; `name` == `@reneturcios/awardsignal`.
- Package entries: **128**.
- sha256: `db3238ee7010261d25600b74a54fbf528d73b5cac61b1176817a8ff61bb810da`
  (re-verified unchanged after the subsequent offline `npm ci`).

## server.json

- Description exactly: "Federal contractor signals:
  acquisition-likelihood profiles, rising contractors, award
  search." — **94 characters**.
- npm package entry carries `"version": "0.1.0"`; the OCI entry
  (`ghcr.io/cloudcorprecords/awardsignal:0.1.0`) carries **no**
  `version` field. Both entries carry packageArguments
  `[{type: positional, value: mcp}]` and transport stdio.
  Dockerfile (node:20-alpine, both stages) carries the label
  `io.modelcontextprotocol.server.name="io.github.CloudCorpRecords/awardsignal"`.
  Container workflow `IMAGE` is `ghcr.io/cloudcorprecords/awardsignal`
  and its smoke test asserts the server name `awardsignal`.

## Verification (all run 2026-10-28)

1. `npx tsc --noEmit` → exit 0.
2. `npm run build` → exit 0; real `dist/` produced (cli, forward,
   index, mcp, types — .js, .d.ts, .js.map each).
3. `node dist/cli.js --version` → `0.1.0`.
4. `npm test` → **54 tests, 54 pass, 0 fail** (registration-integrity
   suite deep-equals the served tools/list against the fixture's 3
   tools, including the no-title and no-required assertions, plus
   session handling/reuse, env override, the
   no-Authorization-header assertions under any environment,
   verbatim argument forwarding and result pass-through including
   upstream isError, SSE parsing, error rendering, MCP-layer
   validation, and CLI behaviour).
5. Parity: `python3 /home/hatch/workspace/mcp-rebuild-100/harness/parity.py --original /home/hatch/workspace/mcp-rebuild-100/harness/fixtures/govconsignals-tools-original.json --rebuild "stdio:node /home/hatch/workspace/mcp-rebuild-100/builds/awardsignal/dist/cli.js mcp" --out /home/hatch/workspace/mcp-rebuild-100/builds/awardsignal/parity-out` → **verdict PASS, parity 100.0**
   (3/3 identical, 0 missing, 0 extra, no schema diffs). Output in
   `parity-out/`.
