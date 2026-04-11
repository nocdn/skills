---
name: generating-llms-txt
description: "Generate llms.txt and llms-full.txt files for a repository or documentation site following the llmstxt.org specification. Use when asked to create llms.txt, generate llms-full.txt, document for LLMs, make repo readable for AI, or create machine-readable documentation."
---

# Generate llms.txt and llms-full.txt

Create both `llms.txt` (concise index with links) and `llms-full.txt` (expanded full-content version) at the repository root, following the spec at https://llmstxt.org/.

## What Each File Does

**`llms.txt`** — A curated markdown index. Lists the project name, summary, and links to key documentation files with one-line descriptions. Small enough to fit in a single LLM context window. This is the entry point.

**`llms-full.txt`** — The full-content expansion. Contains the actual content of every page/file referenced in `llms.txt`, inlined under markdown headings. No links to follow — everything is self-contained. Use this when the consumer needs complete documentation in one file without fetching URLs.

## Step 1: Read the Full Spec

Fetch and read **both** of these pages before writing anything:
- **https://llmstxt.org/** — The official specification (format, section order, rules, examples)
- **https://llmstxt.org/intro.html** — The `llms_txt2ctx` CLI docs (shows how llms.txt is parsed and expanded into context files, which informs how to write `llms-full.txt`)

Confirm you understand:
- The required section order (H1 → blockquote → detail paragraphs → H2 file lists)
- The `## Optional` section's special meaning (skippable for shorter context)
- The link format: `- [Title](url): description`
- Detail paragraphs between the blockquote and H2 sections **must not contain any headings**
- The file can live at root (`/llms.txt`) or in a subpath (`/docs/llms.txt`)

## Step 2: Analyze the Repository

Examine the full repository structure. Identify:

1. **Project purpose** — Read `README.md`, `package.json`, `pyproject.toml`, `Cargo.toml`, `go.mod`, or equivalent
2. **Documentation files** — `docs/`, `spec/`, `*.md` files, API docs, changelogs
3. **Code entry points** — Main source directories, key modules, public API surfaces
4. **Examples** — `examples/`, sample code, tutorials, quickstarts
5. **Configuration** — Setup guides, env templates, CI configs relevant to users
6. **Specs/schemas** — OpenAPI specs, GraphQL schemas, protocol definitions, type definitions

Catalog every file that would help an LLM understand the project. Note file sizes — if a file is very large (>50KB), consider summarizing or splitting its content in `llms-full.txt`.

## Step 3: Write llms.txt

Place at the repository root. Follow this exact structure:

```markdown
# Project Name

> One-to-two sentence summary. State what it is, what it does, and its primary use case.

Key facts an LLM must know to interpret the rest of the file correctly:
- Framework/language and major dependencies
- What this project is NOT (common misconceptions)
- Architectural patterns or conventions used
- Any non-obvious terminology

## Docs

- [Getting Started](docs/getting-started.md): Setup instructions and first-run walkthrough
- [API Reference](docs/api.md): Complete public API with parameters, return types, and examples

## Examples

- [Basic Usage](examples/basic.py): Minimal working example showing core functionality
- [Advanced Patterns](examples/advanced.py): Real-world usage with error handling and configuration

## Optional

- [Architecture](docs/architecture.md): System design and component relationships
- [Contributing](CONTRIBUTING.md): Development setup and contribution guidelines
- [Changelog](CHANGELOG.md): Version history and breaking changes
```

### llms.txt Rules

1. **H1** — Project name. Required. Only one.
2. **Blockquote** — Summary with key context. Strongly recommended.
3. **Detail paragraphs** — Zero or more paragraphs, lists, or other markdown (but **no headings** — H1–H6 are not allowed here). Put critical disambiguation here (e.g., "This is NOT a REST API framework").
4. **H2 sections** — File lists. Each entry is `- [Title](url): description`.
5. **`## Optional`** — Special section. Content here can be dropped when context is tight. Put secondary/supplementary docs here.
6. **URLs** — Use relative paths for repo files (`docs/api.md`). Use absolute URLs for external resources (`https://...`). For documentation sites, use the `.md`-appended URL pattern when available (e.g., `https://example.com/docs/page.html.md`).
7. **Descriptions** — The spec makes descriptions optional, but always include them. Every link should have a `: description` after the URL. Be specific. Bad: "API docs". Good: "Complete REST API reference with request/response schemas and authentication details".
8. **Section names** — Use clear, descriptive H2 names: `Docs`, `API`, `Examples`, `Guides`, `Specifications`, `Configuration`, `Optional`.
9. **No empty sections** — Only include sections that have entries.
10. **Conciseness** — `llms.txt` should be under 5KB. It's an index, not a document. Curate aggressively — include only files that meaningfully help an LLM understand and work with the project.
11. **`.md` URL convention** — For documentation sites, link to the `.md` version of pages when available (append `.md` to the HTML URL, e.g., `https://example.com/docs/guide.html.md`). For pages without filenames in the URL, append `index.html.md`. This gives LLMs clean markdown instead of HTML.
12. **External links are allowed** — `llms.txt` can reference URLs outside the repo/site if they help LLMs understand the project (e.g., linking to an upstream framework's docs).

### What to Include vs Exclude

**Include:**
- README, getting-started guides, quickstarts
- API references, interface definitions, type definitions
- Usage examples and tutorials
- Configuration/setup docs
- Specifications, schemas, protocol definitions
- Architecture and design docs (in Optional)

**Exclude:**
- Build artifacts, generated files, lock files
- Internal implementation details not useful for consumers
- Redundant files (don't list both `README.md` and `docs/overview.md` if they say the same thing)
- CI/CD configs, test fixtures, editor configs
- Files already fully covered by another listed document

## Step 4: Write llms-full.txt

Place at the repository root alongside `llms.txt`. This file inlines the full content of every resource referenced in `llms.txt`.

### Structure

```markdown
# Project Name

> Same summary as llms.txt.

Same detail paragraphs as llms.txt.

## Docs

### Getting Started

[Full content of docs/getting-started.md inlined here, verbatim]

### API Reference

[Full content of docs/api.md inlined here, verbatim]

## Examples

### Basic Usage

[Full content of examples/basic.py inlined here, in a fenced code block]

## Optional

### Architecture

[Full content of docs/architecture.md inlined here]
```

### llms-full.txt Rules

1. **Mirror the llms.txt structure** — Same H1, blockquote, detail paragraphs, and H2 sections in the same order.
2. **Inline content under H3 headings** — Each file from `llms.txt` becomes an H3 under its parent H2 section. The H3 title matches the link title from `llms.txt`.
3. **Verbatim content** — Copy file contents as-is. Do not summarize or truncate unless a file exceeds ~50KB, in which case include the most relevant portions and note what was omitted.
4. **Code files** — Wrap source code in fenced code blocks with the correct language identifier (````python`, ````typescript`, etc).
5. **External URLs** — Fetch the content if accessible. If not fetchable, include the URL and a note: `> Content available at: https://...`
6. **No dangling links** — Every link in `llms.txt` must have its content represented in `llms-full.txt`.
7. **Keep the `## Optional` section** — Include it with full content. Consumers can still choose to drop it.
8. **Size awareness** — `llms-full.txt` will be large. Target under 500KB. If the total content exceeds this, prioritize non-Optional sections and trim Optional content first.

## Step 5: Validate

Check both files:

- [ ] `llms.txt` has exactly one H1
- [ ] `llms.txt` blockquote is present and informative
- [ ] Every link in `llms.txt` uses the format `- [Title](url): description`
- [ ] All relative paths in `llms.txt` resolve to real files
- [ ] `llms-full.txt` mirrors the same structure as `llms.txt`
- [ ] Every file referenced in `llms.txt` has its content inlined in `llms-full.txt`
- [ ] No placeholder text remains (`[TODO]`, `[your description]`, etc.)
- [ ] Language in both files is concise, specific, and jargon-free (or jargon is defined)
- [ ] `## Optional` section is used for secondary content, not core docs

## Writing Quality Guidelines

- **Be specific in descriptions.** Not "Docs for the API" but "REST API endpoints for user management, including authentication, CRUD operations, and webhook configuration".
- **Front-load important information.** Put the most critical docs in the first H2 section. Put nice-to-haves in `## Optional`.
- **Disambiguate the project.** If the project name is generic or commonly confused with something else, use the detail paragraphs to clarify what it is and is NOT.
- **Use expert-level language.** The audience is LLMs and developers. Skip introductory fluff. State facts directly.
- **Include version/compatibility notes** when relevant (e.g., "Requires Node.js 18+", "Compatible with React 18 and 19").
