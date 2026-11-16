# @pipeworx/cn-mee-enforcement

Has China's Ministry of Ecology and Environment (生态环境部) publicly penalised or named this company: which decision, on what date, what was found and what was ordered (fine, correction)? Rows come from the Ministry's exposure board (曝光台): nuclear-safety and radiation administrative penalty decisions, enforcement information and circulars. Firms only.

Part of [Pipeworx](https://pipeworx.io) — an MCP gateway connecting AI agents to 2006+ live data sources. This is an independent, unofficial integration — not affiliated with, endorsed by, or published by the upstream provider.

## Tools

- `cn_mee_notices_list(stream?, query?, page?)` -- one page (15 rows, newest first) of the exposure board. `stream` is `penalties` (administrative penalty and correction decisions, default), `enforcement` (typical cases and environmental-incident investigation results that name the responsible company) or `notices` (circulars). Each row gives the title, date, the company named, and the link to the official notice. `query` is Chinese text matched against the titles on the page read only (the publisher has no keyword search); `page` goes back in time and the response gives `pages_total`. Also returns `searched`, `individual_rows_omitted` and `data_as_of`.
- `cn_mee_penalty_get(notice_url)` -- one penalty or correction decision against a company, from a `notice_url` of a `penalties` row: the party and its unified social credit code, registered address, document number, date, the facts found, the decision, the fine amounts, whether correction was ordered, and the legal provisions cited.

## Auth

Keyless.

## Data sources

- Ministry of Ecology and Environment of China (生态环境部), exposure board: administrative penalty decisions, enforcement information and circulars (曝光台). Read from the publisher on each call; nothing is stored.

## Individuals are not returned

Some decisions are against natural persons. A row whose party is a person is left out and counted (`individual_rows_omitted`); `cn_mee_penalty_get` on such a decision returns `found: false, reason: individual_party` and no name. In a decision against a company, the legal representative and any sentence naming or describing a person (court sentences, named staff) are left out and counted (`person_lines_omitted`). Every response carries a `note` saying so; the tests prove a person's name never appears in a response.

## Behaviour worth knowing

- The exposure board lists decisions the Ministry chose to publish, not a register of every penalty (provincial bureaus publish their own). No match is never evidence that a company was not penalised.
- The penalties stream is mostly nuclear-safety and radiation decisions (quality-assurance failures at nuclear equipment makers, radioactive-waste handling), 2019 onward, about 5 pages.
- Typical-case releases in the `enforcement` stream anonymise the companies ("某公司"); incident investigation results name them.
- A page that cannot be read is an `upstream_down` error, never an empty result. A page read successfully with no title match says so and names the page (`empty_reason: no_title_match_on_this_page`).
- Text is the Ministry's own wording, in simplified Chinese.

## Quick Start

Add to your MCP client (Claude Desktop, Cursor, Windsurf, etc.):

```json
{
  "mcpServers": {
    "cn-mee-enforcement": {
      "url": "https://gateway.pipeworx.io/cn-mee-enforcement/mcp"
    }
  }
}
```

### What this endpoint actually serves

`tools/list` at `https://gateway.pipeworx.io/cn-mee-enforcement/mcp` returns the tools in the table
above **plus the shared Pipeworx meta-tools** — `ask_pipeworx`,
`discover_tools`, `search_within`, `remember`/`recall` and the rest of the
gateway-wide set. So the tool count you see is larger than this table: a
single-pack endpoint currently lists roughly 30 shared tools alongside the
pack's own. The connection's `initialize` response states its exact scope, and
is the authoritative answer for a given day.

This is deliberate, not multiplexing by accident. The meta-tools are what let a
scoped connection answer a question this pack does not cover — via
`ask_pipeworx`, which routes across the whole catalog — without you adding a
second MCP server. There is currently no way to mount a pack endpoint without
them; if the extra schemas cost you more context than the routing is worth,
connect to the full gateway once rather than to several pack endpoints.

Or connect to the full Pipeworx gateway to get every pack's tools listed
directly, instead of just this one's:

```json
{
  "mcpServers": {
    "pipeworx": {
      "url": "https://gateway.pipeworx.io/mcp"
    }
  }
}
```

Both URLs reach the same gateway and the same 2006+ data sources. The
only difference is which pack's tools are listed **directly**; `ask_pipeworx`
reaches all of them from either one.

## No MCP client? Call it over HTTP

```bash
curl -X POST https://gateway.pipeworx.io/v1/tools/cn_mee_notices_list \
  -H 'Content-Type: application/json' \
  -d '{"stream":"penalties","page":1}'
```

No account needed for the first calls. Inspect any tool: `GET https://gateway.pipeworx.io/v1/tools/cn_mee_notices_list`. Find one: `POST https://gateway.pipeworx.io/v1/tools/search_packs` with `{"query":"..."}`.

## Run it from npm

`@pipeworx/mcp-cn-mee-enforcement` is a local stdio bridge to the Pipeworx gateway: your MCP client
starts it with `npx`, and it forwards every call to the same
`https://gateway.pipeworx.io/cn-mee-enforcement/mcp` endpoint shown above. It needs network access;
there is no offline mode.

```json
{
  "mcpServers": {
    "cn-mee-enforcement": {
      "command": "npx",
      "args": ["-y", "@pipeworx/mcp-cn-mee-enforcement"]
    }
  }
}
```

Without a key it calls anonymously, capped at 50 calls a day. A free account
raises that to 200 a day: get a key at https://pipeworx.io/signup and add
`"env": { "PIPEWORX_API_KEY": "<your key>" }` to the server entry.
The key is sent as a Bearer token, the same way a hosted connection sends it,
so calls are metered and billed exactly as they are there.

It lists **only** this pack's tools by default. Set `PIPEWORX_META_TOOLS=1` to
also list `ask_pipeworx` and the other gateway-wide tools.

Behind an HTTPS proxy: Node's built-in `fetch` does not read `HTTPS_PROXY` by
default. Newer Node releases honour it when `NODE_USE_ENV_PROXY=1` is set; check
your Node version's documentation.

## Using with ask_pipeworx

Instead of calling tools directly, you can ask questions in plain English —
this works on the pack endpoint above as well as on the full gateway:

```
ask_pipeworx({ question: "your question about Cn Mee Enforcement data" })
```

The gateway picks the right tool and fills the arguments automatically.

## More

- [Docs and guides](https://pipeworx.io/docs)
- [pipeworx.io](https://pipeworx.io)

## License

MIT
