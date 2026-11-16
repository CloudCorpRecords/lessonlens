# BUILD REPORT — lessonlens

Clean-room rebuild of `com.lessonvo/site` v1.0.1
(remote-only MCP server at `https://lessonvo.com/mcp`), built
2026-10-28 as a forwarding client. Package: `@reneturcios/lessonlens`
0.1.0, registry name `io.github.CloudCorpRecords/lessonlens`.

## Surface built

Complete surface: **7 of 7 captured tools built**, no omissions.

| Tool | Required fields as served |
|------|---------------------------|
| `dataset_columns` | *(none — no `required` key)* |
| `dataset_search` | `query` |
| `dataset_row` | `column`, `value` |
| `dataset_compare` | `column`, `values` |
| `dataset_stats` | `column` |
| `dataset_top` | `column` |
| `dataset_provenance` | *(none — no `required` key)* |

Each served tool carries exactly the keys `name`, `title`,
`description`, `inputSchema`, with the `inputSchema.$schema` marker
(`http://json-schema.org/draft-07/schema#`) exactly as captured.
The tool literals were generated programmatically from
`harness/fixtures/lessonvo-tools-original.json` by the script at
`/tmp/laneb3-lessonlens-scratch/gen.py` (parse + re-serialise,
splice into `src/forward.ts`) — never retyped — and the generator's
own post-generation check, the registration-integrity test suite,
and the parity harness each confirmed the served literals
deep-equal the fixture's 7 tool objects, in order. Tools are
registered via the MCP SDK low-level Server API
(`ListToolsRequestSchema` / `CallToolRequestSchema` handlers),
not McpServer+zod.

## Fixture conflicts

**None.** The spec and the fixture agree on the tools' names,
titles, descriptions, required fields (`dataset_columns` and
`dataset_provenance` carry no `required` key; `dataset_search`
requires `query`; `dataset_row` requires `column` + `value`;
`dataset_compare` requires `column` + `values`; `dataset_stats`
and `dataset_top` require `column`) and property shapes.

## Labelled reconstructions (from the spec's Wire section)

- Upstream endpoint pinned: `https://lessonvo.com/mcp`.
- Env override `LESSONVO_MCP_URL` — labelled reconstruction.
- The registry record declares NO headers on the remote: no auth
  header is ever sent, under any environment. Tests assert that no
  `Authorization` header (and no `X-API-Key` header) is sent with an
  empty environment, with a `LESSONVO_API_KEY`-style variable
  present, and with `AUTHORIZATION` / `API_KEY` / `LESSONVO_TOKEN`
  variables present.
- Error rendering `lessonlens error:` text for upstream failures —
  labelled reconstruction.
- Upstream results pass through verbatim, including `isError` and
  extra fields. tools/call is forwarded as streamable-HTTP JSON-RPC:
  initialize (protocolVersion `2025-03-26`) →
  notifications/initialized → tools/call, honouring `Mcp-Session-Id`
  when issued; plain-JSON and SSE replies are both parsed. The
  forwarding core takes an injectable fetch; tests stub the
  upstream — no live network calls in tests.

## Deviations

None.

## Lockfile

- Route: generated clean from birth in the build dir with no
  `node_modules` and no pre-existing lockfile:
  `npm install --package-lock-only --ignore-scripts --offline`
  (Route 3), then populated with `npm install --ignore-scripts --offline`.
- Parses as JSON; `name` == `@reneturcios/lessonlens`.
- Package entries: **128**.
- sha256: `881c3ab1bbbfb96ab447d16a56b05d4e9cbd9a780311d90fe9bf4405016099ab`
  (re-verified unchanged after the subsequent offline `npm install`).

## server.json

- Description exactly: "Lessonvo site dataset: search, compare
  and cite the site's published dataset." — **77 characters**.
- npm package entry carries `"version": "0.1.0"`; the OCI entry
  (`ghcr.io/cloudcorprecords/lessonlens:0.1.0`) carries **no**
  `version` field. Both entries carry packageArguments
  `[{type: positional, value: mcp}]` and transport stdio. Dockerfile
  (node:20-alpine, both stages) carries the label
  `io.modelcontextprotocol.server.name="io.github.CloudCorpRecords/lessonlens"`.
  Container workflow `IMAGE` is `ghcr.io/cloudcorprecords/lessonlens`
  and its smoke test asserts the server name `lessonlens`.

## Verification (all run 2026-10-28)

1. `npx tsc --noEmit` → exit 0.
2. `npm run build` → exit 0; real `dist/` produced (cli, forward,
   index, mcp, types — .js, .d.ts, .js.map each).
3. `node dist/cli.js --version` → `0.1.0`.
4. `npm test` → **62 tests, 62 pass, 0 fail** (registration-integrity
   suite deep-equals the served tools/list against the fixture's 7
   tools, plus session handling/reuse, env override, the
   no-Authorization-header assertions under every environment,
   verbatim argument forwarding and result pass-through including
   upstream isError, SSE parsing, error rendering, MCP-layer
   validation, and CLI behaviour).
5. Parity: `python3 /home/hatch/workspace/mcp-rebuild-100/harness/parity.py --original /home/hatch/workspace/mcp-rebuild-100/harness/fixtures/lessonvo-tools-original.json --rebuild "stdio:node /home/hatch/workspace/mcp-rebuild-100/builds/lessonlens/dist/cli.js mcp" --out /home/hatch/workspace/mcp-rebuild-100/builds/lessonlens/parity-out` → **verdict PASS, parity 100.0**
   (7/7 identical, 0 missing, 0 extra, no schema diffs). Output in
   `parity-out/`.
