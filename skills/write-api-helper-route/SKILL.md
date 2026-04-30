---
name: write-api-helper-route
description: "Write the GET / helper route of an HTTP API as a plain-text quick-reference page with a routes table, curl examples, request-field tables, and example responses. Use when adding or updating a GET / usage page on an API (Hono, Express, Fastify, etc.), when asked for an API usage page, GET / quick reference, or to keep the helper route in sync with the actual routes."
---

# Write GET / API Helper Route

Build the root route (`GET /`) of an HTTP API as a plain-text quick-reference that is fast to read in a terminal (`curl localhost:PORT/`) and dense with usage information. This is not the README - the README handles long-form prose.

## Response

- `Content-Type: text/plain`
- The body ends with exactly one trailing newline character. No extra blank lines at the end of the response.

## Top-level rules

- No markdown formatting (no `#`, no `**`, no `_`, no code fences, no backticks, no `-` lists) EXCEPT for: markdown tables, the Setext-style title underline, and the ALL-CAPS colon-headings described below.
- Exactly three blank lines between each top-level section (title block → routes table → first per-route section).
- Per-route sections are separated differently - see "Per-route sections" below.
- Indentation is allowed where it improves readability (JSON pretty-printing, curl line continuations), but the `curl` line itself must start at column 0 with no leading indentation.

## Page structure

The page has three top-level sections, separated by three blank lines:

1. **Title block** - project name, Setext underline, one-line description.
2. **Routes table** - one markdown table listing every route.
3. **Per-route sections** - one section per route, in importance order.

### 1. Title block

- Line 1: project name as-is.
- Line 2: a row of `=` characters whose length matches the project name exactly (Setext-style underline). This is the one exception to the no-markdown rule because it doubles as nice plain-text styling.
- Line 3: blank.
- Line 4: a single concise, technical, all-lowercase sentence describing what the project does. No filler, no marketing language - just the technical fact of what it is. Example: `wraps yt-dlp and streams the resulting media file over http`.

### 2. Routes table

Preceded by the colon-heading `ROUTES:` on its own line, then a blank line, then the table.

Columns: `Method`, `Path`, `Description`. Description is a few words at most.

Order rows by importance: the primary endpoint (the one users actually came here to use) first, then the next most useful, and so on. Utility endpoints like `/api/health` go last.

### 3. Per-route sections

For every route in the routes table, in the SAME importance order as the table (not alphabetical, not implementation order), produce a section consisting of:

1. A bare `METHOD /path` line (e.g. `POST /api/download`).
2. The curl example (see "Curl examples" below).
3. `REQUEST FIELDS:` colon-heading + table - **only if the route accepts request fields** (body, query params, path params, custom headers). If the route has no request fields at all, OMIT this entire block. Do not insert a placeholder table with rows like `none | never | -`.
4. `EXAMPLE RESPONSE:` colon-heading + pretty-printed JSON response (see "Example Response" below).
5. (Optional) `ERROR RESPONSE:` colon-heading + pretty-printed example error JSON, when useful.

#### Per-route section separator

Each per-route section ends with a horizontal divider: a line of exactly five `/` characters (`/////`) on its own line, as the section's closing marker. The divider appears in EVERY per-route section, including the last one.

The exact spacing between two adjacent per-route sections is:

```
[last content of section A]
[blank]
/////
[blank]
[blank]
[blank]
[METHOD /path of section B]
```

That is: one blank line, then the divider, then three blank lines, then the next section starts. This gives a clear visual break between routes without becoming excessive.

You MAY include short informational paragraphs of plain prose within a per-route section where genuinely useful: explaining non-obvious behavior, retry/fallback logic, platform-specific edge cases, what a particular field actually does, when to prefer one mode over another, etc.

- Concise, technical, filler-free. No marketing language, no restatement of the fields table, no obvious statements.
- Place them where they fit best: directly after the curl, after a specific table, or before the example response.
- Use sparingly: only when the information is actually useful to a developer using the API and is not already obvious from the curl, fields table, or response.

## Curl examples

The `curl` line MUST start at column 0 - no leading indentation on the line that begins with `curl`, and no leading indentation on any of its continuation lines either. Do not wrap the entire curl block in outer indentation.

Continuation lines start with 2 spaces of indent ONLY for relative readability of `-H`, `-d`, `--output`, etc. - that is the only kind of indentation allowed inside the curl block.

- For requests that take a body, headers, or query params, write the curl pretty: backslash line continuations + small inner indents (e.g. JSON body indented 2 spaces inside the `-d` argument). Include as many request fields as possible, including optional ones.
- For simple requests with no body and no headers (e.g. a plain `GET /api/health`), write the curl on a single line with no backslashes. Do not pretty-format curls that have nothing to format.

The hostname/port must be `http://localhost:$PORT` matching the service's actual configured port, used consistently across every curl example.

## Markdown table style

Standard markdown table structure only: header row, header separator row, data rows. **Do NOT add a top border row or a bottom border row** - just the standard three parts.

- All cells in a column must be padded with trailing spaces so every cell in that column has equal width.
- The header separator row's dash count for each column must match the padded width of that column exactly.
- All `|` characters must align vertically across every row.
- Cramped tables where every cell is sized to its own content are not acceptable. The page is read as plain text in a terminal, not just rendered, so alignment matters.

Example of the expected style:

```
| Method | Path            | Description                         |
| ------ | --------------- | ----------------------------------- |
| GET    | /api/health     | service status and configuration    |
| POST   | /api/check      | run a fact-check (direct or queued) |
| GET    | /api/check/:id  | poll a queued job                   |
```

### Pipes inside table cells

Do NOT use the `|` character inside table cells. Escaping it as `\|` leaves a visible backslash in the plain-text output, which is wrong. For union-type values (where TypeScript would use `|`), use the word `or` instead.

- Wrong: `"audio" \| "video" \| "both"`
- Right: `"audio" or "video" or "both"`
- Wrong: `string \| null`
- Right: `string or null`

The `|` character is reserved exclusively for the table's column separators.

## Request Fields table

Columns: `Field`, `Type`, `Default`.

- **Field** - the field name as it appears in the request (JSON key, query param, header).
- **Type** - TypeScript-style notation: `string`, `string[]`, `boolean`, `number`, etc. Use `or` for unions (see above).
- **Default** - the literal value in JSON form:
  - Quoted strings: `"both"`, `"1080p"`
  - Unquoted `null`, unquoted booleans (`true`, `false`)
  - Unquoted numbers: `8080`, `3.14`
  - A single dash `-` if the field is required with no default.

Order fields with required fields first, then optional fields in importance order. Use the same field order in the JSON body inside the curl example so the curl and the table line up.

## Example Response

Under the `EXAMPLE RESPONSE:` colon-heading, write a pretty-printed JSON example (2-space indentation) of what the route actually returns on success.

When the success response is NOT JSON (binary file, audio/video stream, plain text, octet-stream, etc.), do NOT invent fake JSON metadata describing the response. Instead, write a single concise plain-text sentence under `EXAMPLE RESPONSE:` describing what the success response actually is. Example: `Streams the downloaded media file with Content-Type matching the source format and Content-Disposition: attachment.`

In that case, optionally follow `EXAMPLE RESPONSE:` (after a blank line) with an `ERROR RESPONSE:` colon-heading and a pretty-printed example error JSON to document the error shape, since errors will typically still be JSON even when success is binary.

## Colon-headings

Every markdown table and example response must be preceded by a one-line plain-text heading in ALL CAPS ending with a colon. ALL CAPS is used because the page is plain text - markdown bold/italic do not render - so capitalization is the only available form of emphasis. The required headings are:

- `ROUTES:` before the routes table
- `REQUEST FIELDS:` before a per-route fields table
- `EXAMPLE RESPONSE:` before the example response
- `ERROR RESPONSE:` before an optional error response example

A single blank line separates the colon-heading from the content below it. These colon-headings are plain text, not markdown headings - just ALL-CAPS words ending with a colon on their own line.

## Maintenance

When you add, change, or remove API routes, update `GET /` in the same change:

- Every route must appear in the routes table and have its own per-route section.
- If `GET /` and the actual routes disagree, treat it as a bug and fix `GET /` in the same change.

## Worked example

For a yt-dlp wrapper exposing `POST /api/download`, `GET /api`, and `GET /api/health`:

```
yt-dlp-api
==========

wraps yt-dlp and streams the resulting media file over http



ROUTES:

| Method | Path          | Description               |
| ------ | ------------- | ------------------------- |
| POST   | /api/download | download and stream media |
| GET    | /api          | service info and defaults |
| GET    | /api/health   | service status            |



POST /api/download

curl -X POST http://localhost:3447/api/download \
  -H "Content-Type: application/json" \
  -d '{
    "url": "https://www.youtube.com/watch?v=dQw4w9WgXcQ",
    "mode": "both",
    "quality": "1080p",
    "proxyUrl": "http://127.0.0.1:8080"
  }' \
  --output download.bin

YouTube URLs always require a working proxy; if the proxy is unreachable the
request fails with a 502.

REQUEST FIELDS:

| Field    | Type                         | Default |
| -------- | ---------------------------- | ------- |
| url      | string                       | -       |
| mode     | "audio" or "video" or "both" | "both"  |
| quality  | string                       | "1080p" |
| proxyUrl | string or null               | null    |

EXAMPLE RESPONSE:

Streams the downloaded media file with Content-Type matching the source
format and Content-Disposition: attachment.

ERROR RESPONSE:

{
  "error": {
    "message": "Playlist downloads are not supported."
  }
}

/////



GET /api

curl http://localhost:3447/api

EXAMPLE RESPONSE:

{
  "defaultMode": "both",
  "defaultQuality": "1080p",
  "name": "yt-dlp-api",
  "port": 3447,
  "proxyConfigured": false,
  "status": "ok"
}

/////



GET /api/health

curl http://localhost:3447/api/health

EXAMPLE RESPONSE:

{
  "status": "ok"
}

/////
```
