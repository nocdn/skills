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

The routes table itself MUST still list every route the service exposes, including `/`, `/api`, and `/api/health`, so the table remains an honest map of the API surface.

### Routes that MUST NOT have their own per-route section

Even though these routes appear in the routes table, do NOT create a per-route section for any of them:

- `GET /` (the usage reference itself)
- `GET /api` (service metadata / defaults)
- `GET /api/health` (service status)
- any other `*/health` or status-only endpoint

These routes are self-evident from the routes table and would only pad the page with noise. The per-route sections exist for routes a developer actually needs to learn how to call.

### 3. Per-route sections

For every route in the routes table EXCEPT the routes listed in "Routes that MUST NOT have their own per-route section" above, in the SAME importance order as the table (not alphabetical, not implementation order), produce a section consisting of:

1. A `METHOD /path` line (e.g. `POST /api/download`), followed on the next line by a row of `=` characters whose length matches the `METHOD /path` line exactly (Setext-style underline).
2. The curl example (see "Curl examples" below).
3. `REQUEST FIELDS:` colon-heading + table - **only if the route accepts request fields** (body, query params, path params, custom headers). If the route has no request fields at all, OMIT this entire block. Do not insert a placeholder table with rows like `none | never | -`.
4. `EXAMPLE RESPONSE:` colon-heading + pretty-printed JSON response (see "Example Response" below).
5. (Optional) `ERROR RESPONSE:` colon-heading + pretty-printed example error JSON, when useful.

#### Per-route section separator

Per-route sections do not use a closing divider. The exact spacing between two adjacent per-route sections is:

```
[last content of section A]
[blank]
[blank]
[blank]
[METHOD /path of section B]
[=== underline of section B]
```

That is: three blank lines between the last content of one section and the `METHOD /path` line of the next. This gives a clear visual break between routes without becoming excessive.

You MAY include short informational paragraphs of plain prose within a per-route section where genuinely useful: explaining non-obvious behavior, retry/fallback logic, platform-specific edge cases, what a particular field actually does, when to prefer one mode over another, etc.

- Concise, technical, filler-free. No marketing language, no restatement of the fields table, no obvious statements.
- Place them where they fit best: directly after the curl, after a specific table, or before the example response.
- Use sparingly: only when the information is actually useful to a developer using the API and is not already obvious from the curl, fields table, or response.

#### Line style for informational paragraphs

Informational paragraphs are NOT flowing prose. Every distinct statement (what would be a separate sentence in normal prose) goes on its own line, and the page must not contain long wrapped sentences in these blocks.

- One statement per line. Break at every place you would otherwise put a `.` followed by a space.
- No full stops at the end of any of these lines.
- Every line starts with a lowercase letter, including the first line of the block. Do NOT capitalize the first word just because it begins a sentence. The only exceptions are real proper nouns, acronyms, code identifiers, and quoted literals (e.g. `YouTube`, `URL`, `PROXY_URL`, `FORCE_PROXY_SERVICES`, `"both"`, `audioQuality`) - those keep their natural casing.
- Do not artificially split mid-clause - the unit is the statement (roughly one sentence), not the wrapped terminal line.

Example - wrong (one long line, full stops, capitalized starts):

```
quality only applies when mode is "both" and accepts best, 2160p, 1440p, 1080p, 720p, 480p, or 360p. audioQuality only applies when mode is "audio" and accepts best, high, medium, low, or lowest, or a raw bitrate like "128K". When playlist is true, the URL must be a playlist and the response is a .zip file. proxy uses PROXY_URL when configured; FORCE_PROXY_SERVICES can force proxy use by service.
```

Example - right (one statement per line, no trailing `.`, lowercase starts where possible):

```
quality only applies when mode is "both" and accepts best, 2160p, 1440p, 1080p, 720p, 480p, or 360p
audioQuality only applies when mode is "audio" and accepts best, high, medium, low, or lowest, or a raw bitrate like "128K"
when playlist is true, the URL must be a playlist and the response is a .zip file
proxy uses PROXY_URL when configured; FORCE_PROXY_SERVICES can force proxy use by service
```

Another right example:

```
/api/message only accepts application/json
send audio and video uploads to /api/transcribe
provider values are openai, gemini, fireworks, openrouter, or cerebras
effort values are none, minimal, low, medium, high, or xhigh
```

## Curl examples

The `curl` line MUST start at column 0 - no leading indentation on the line that begins with `curl`, and no leading indentation on any of its continuation lines either. Do not wrap the entire curl block in outer indentation.

Continuation lines start with 2 spaces of indent ONLY for relative readability of `-H`, `-d`, `--output`, etc. - that is the only kind of indentation allowed inside the curl block.

- For requests that take a body, headers, or query params, write the curl pretty: backslash line continuations + small inner indents (e.g. JSON body indented 2 spaces inside the `-d` argument). Include as many request fields as possible, including optional ones.
- For simple requests with no body and no headers (e.g. a plain `GET /api/health`), write the curl on a single line with no backslashes. Do not pretty-format curls that have nothing to format.

The hostname/port must be `http://localhost:$PORT` matching the service's actual configured port, used consistently across every curl example.

## Markdown table style

Markdown-like tables only - NOT standard markdown tables. The outer `|` characters (the leftmost wall and the rightmost wall) MUST be omitted on every row, including the header separator row. Only the interior `|` characters between columns remain. The first character of every row is the first character of the first cell (no leading `|`, no leading space), and the last character of every row is the last character of the last cell (no trailing space, no trailing `|`).

- Header row, header separator row, data rows. **Do NOT add a top border row or a bottom border row.**
- All cells in a column must be padded with trailing spaces so every cell in that column has equal width.
- The header separator row's dash count for each column must match the padded width of that column exactly.
- All interior `|` characters must align vertically across every row.
- Cramped tables where every cell is sized to its own content are not acceptable. The page is read as plain text in a terminal, not just rendered, so alignment matters.

Example of the expected style:

```
Method | Path           | Description
------ | -------------- | -----------------------------------
GET    | /api/health    | service status and configuration
POST   | /api/check     | run a fact-check (direct or queued)
GET    | /api/check/:id | poll a queued job
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
- **Type** - the JSON/TypeScript primitive type ONLY: `string`, `string[]`, `boolean`, `number`, `File`, etc. **Never enumerate allowed values in the Type column**, even when the field is a strict enum / zod literal union / oneOf. The Type column is for the type, not for the value set.
  - Wrong: `"openai" or "gemini" or "fireworks" or "openrouter" or "cerebras"`
  - Wrong: `"none" or "minimal" or "low" or "medium" or "high" or "xhigh"`
  - Wrong: `"ar" or "de" or "el" or "en" or "es" or "fr" or "it" or "ja" or "ko" or "nl" or "pl" or "pt" or "vi" or "zh"`
  - Right: `string` (in all three cases above)
  - The allowed values belong in the informational paragraph for the route (e.g. `provider values are openai, gemini, fireworks, openrouter, or cerebras`), not in the table.
  - The `or` form is still allowed for genuine **type** unions where the alternatives are different types, not different string literals - e.g. `string or null`, `number or string`, `string or string[]`.
- **Default** - the literal value in JSON form:
  - Quoted strings: `"both"`, `"1080p"`
  - Unquoted `null`, unquoted booleans (`true`, `false`)
  - Unquoted numbers: `8080`, `3.14`
  - A single dash `-` if the field is required with no default.

Order fields with required fields first, then optional fields in importance order. Use the same field order in the JSON body inside the curl example so the curl and the table line up.

## Example Response

Under the `EXAMPLE RESPONSE:` colon-heading, write a pretty-printed JSON example (2-space indentation) of what the route actually returns on success, following the "JSON response shape" rules below.

When the success response is NOT JSON (binary file, audio/video stream, plain text, octet-stream, etc.), do NOT invent fake JSON metadata describing the response. Instead, write a single concise plain-text sentence under `EXAMPLE RESPONSE:` describing what the success response actually is. Example: `Streams the downloaded media file with Content-Type matching the source format and Content-Disposition: attachment.`

In that case, optionally follow `EXAMPLE RESPONSE:` (after a blank line) with an `ERROR RESPONSE:` colon-heading and an example error to document the error shape, since errors will typically still be JSON even when success is binary.

## JSON response shape (applies to EXAMPLE RESPONSE and ERROR RESPONSE)

This rule applies to **every** JSON example on the page, both under `EXAMPLE RESPONSE:` and under `ERROR RESPONSE:`. There is no exception for "complex" or "nested" or "array-shaped" responses - always strip the outermost `{` and `}`.

Do NOT wrap the example in the outermost JSON object braces `{` and `}`. Show only the inner key/value lines exactly as they would appear inside the outermost JSON object, with no surrounding braces. Outdent the inner content by one level so the top-level keys sit at column 0; nested objects, arrays, and their inner braces keep their normal 2-space indentation relative to that.

Single-key example:

- Wrong:
  ```
  ERROR RESPONSE:

  {
    "error": "Missing media file. Upload an audio or video file in the file field."
  }
  ```
- Right:
  ```
  ERROR RESPONSE:

  "error": "Missing media file. Upload an audio or video file in the file field."
  ```

Multi-key example:

- Wrong:
  ```
  ERROR RESPONSE:

  {
    "error": "yt-dlp failed",
    "details": "ERROR: unable to download media"
  }
  ```
- Right:
  ```
  ERROR RESPONSE:

  "error": "yt-dlp failed",
  "details": "ERROR: unable to download media"
  ```

Nested / array example (EXAMPLE RESPONSE with arrays and nested objects):

- Wrong:
  ```
  EXAMPLE RESPONSE:

  {
    "history": [
      {
        "id": 42,
        "url": "https://example.com",
        "status": "success"
      }
    ],
    "limit": 25,
    "offset": 0
  }
  ```
- Right:
  ```
  EXAMPLE RESPONSE:

  "history": [
    {
      "id": 42,
      "url": "https://example.com",
      "status": "success"
    }
  ],
  "limit": 25,
  "offset": 0
  ```

Note how the top-level keys (`"history"`, `"limit"`, `"offset"`) sit at column 0 with no leading indentation, while the array elements and nested object continue to use 2-space indentation relative to their parent. Commas between top-level keys are preserved exactly as they would be inside the original JSON object.

If the top-level response is itself an array (not an object), keep the outer `[` and `]` - this rule only strips the outermost object braces. Same for top-level strings/numbers/booleans/null.

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

For a yt-dlp wrapper exposing `POST /api/download`, `GET /api`, and `GET /api/health`. Note: `GET /api`, `GET /api/health`, and `GET /` appear in the routes table but DO NOT get per-route sections.

```
yt-dlp-api
==========

wraps yt-dlp and streams the resulting media file over http



ROUTES:

Method | Path          | Description
------ | ------------- | -------------------------
POST   | /api/download | download and stream media
GET    | /api          | service info and defaults
GET    | /api/health   | service status
GET    | /             | usage reference



POST /api/download
==================

curl -X POST http://localhost:3447/api/download \
  -H "Content-Type: application/json" \
  -d '{
    "url": "https://www.youtube.com/watch?v=dQw4w9WgXcQ",
    "mode": "both",
    "quality": "1080p",
    "proxyUrl": "http://127.0.0.1:8080"
  }' \
  --output download.bin

YouTube URLs always require a working proxy
if the proxy is unreachable the request fails with a 502
mode values are audio, video, or both
quality values are best, 2160p, 1440p, 1080p, 720p, 480p, or 360p

REQUEST FIELDS:

Field    | Type           | Default
-------- | -------------- | -------
url      | string         | -
mode     | string         | "both"
quality  | string         | "1080p"
proxyUrl | string or null | null

EXAMPLE RESPONSE:

Streams the downloaded media file with Content-Type matching the source format and Content-Disposition: attachment.

ERROR RESPONSE:

"error": {
  "message": "Playlist downloads are not supported."
}
```
