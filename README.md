# lessonlens

Look into the Lessonvo dataset without keeping a local copy of it. lessonlens is a compact forwarding client for the hosted Lessonvo MCP service: the tool list is served from your own machine using the captured definitions, and each query is sent upstream, with the service's answer returned exactly as it arrived.

The whole surface is read-only. You can inspect the dataset's shape and origin, search its rows for a fragment of text, fetch the rows matching an exact key, set several values side by side, and summarise or rank its numeric columns. lessonlens stores no data and performs no computation of its own — it is a lens onto the published dataset, not a mirror of it.

## The tools

- **`dataset_columns`** — Every column in the dataset, which ones are numeric, the total row count and the provenance banner. The sensible first call when the schema is still unknown. No arguments.
- **`dataset_search`** — Rows where any cell contains the query text, matched case-insensitively, up to 50 rows. `query` is required; `limit` optionally narrows the result (1–50).
- **`dataset_row`** — Rows where one column equals a value exactly, compared case-insensitively. `column` and `value` are both required.
- **`dataset_compare`** — Rows whose column matches any of the supplied values, returned in the order the values were given — the shape of an "X vs Y" question. Requires `column` and a `values` list of 2–10 entries.
- **`dataset_stats`** — count, min, max, mean, median and sum for a numeric column. Grouping commas and currency symbols are understood; rows that do not parse as numbers are left out and counted separately. `column` is required.
- **`dataset_top`** — Rows ranked by a numeric column, highest first — "which is the most X". `column` is required; `limit` caps the list (1–50) and `ascending: true` flips it to lowest first.
- **`dataset_provenance`** — The dataset's source, the date it was computed, its licence and the citation to use when quoting a figure. No arguments.

## Install and run

As an MCP server via npm:

```bash
npx -y @reneturcios/lessonlens mcp
```

MCP client configuration:

```json
{
  "mcpServers": {
    "lessonlens": {
      "command": "npx",
      "args": ["-y", "@reneturcios/lessonlens", "mcp"]
    }
  }
}
```

As a container:

```bash
docker run -i ghcr.io/cloudcorprecords/lessonlens
docker run --rm ghcr.io/cloudcorprecords/lessonlens dataset_provenance '{}'
```

## CLI

A single binary covers both the command line and the MCP server. Every tool is a subcommand whose one argument is a JSON string of that tool's inputs:

```bash
lessonlens dataset_columns '{}'
lessonlens dataset_search '{"query":"lesson"}'
lessonlens dataset_row '{"column":"city","value":"Fresno"}'
lessonlens dataset_compare '{"column":"city","values":["Fresno","Modesto"]}'
lessonlens dataset_stats '{"column":"score"}'
lessonlens dataset_top '{"column":"score","limit":10,"ascending":true}'
lessonlens dataset_provenance '{}'
lessonlens --version
```

Exit codes: `0` for a successful forwarded call, `1` when the result that comes back is an error, and `2` for usage mistakes like an unknown subcommand or a JSON argument that does not parse.

## Configuration

- `LESSONVO_MCP_URL` — Replaces the upstream endpoint. Defaults to the pinned hosted endpoint, `https://lessonvo.com/mcp`. (This override variable is a reconstruction introduced by this rebuild.)

The Lessonvo registry record lists no headers for its remote, so lessonlens never attaches an authentication header — every query goes upstream keyless.

When the upstream cannot answer — it returns a JSON-RPC error, answers with a non-2xx status, sends a body that cannot be read, or the request itself fails — the caller receives an error result whose text begins with `lessonlens error:`.

## How forwarding works

The first query of a process opens a session against the upstream: an `initialize` request (protocol version `2025-03-26`), a `notifications/initialized` notification, and then the `tools/call`. If the service issues an `Mcp-Session-Id` header during initialize, lessonlens repeats it on subsequent requests and keeps reusing that one session until the process ends. Replies are understood in both plain JSON and server-sent-events framing. Successful results are passed through untouched, `isError` flag and any extra fields included.

## Development

```bash
npm install
npm run build   # tsc -> dist/ (dist/cli.js is the bin)
npm test        # tsx --test test/*.test.ts — stubbed upstream, no network
npm run dev     # run the CLI from source
```

The suite compares the served tool list against the captured surface tool for tool, walks the session handshake and its reuse, proves no `Authorization` header is sent under any environment, and confirms that arguments travel upstream unchanged and results return unchanged.

## License

MIT — see `LICENSE`. Copyright (c) 2026 Rene Turcios.
