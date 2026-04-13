---
name: no-ctx7
description: do not use ctx7 or Context7 at all (unless the user specifies to do so) - use another source such as Firecrawl, your native web search tools, local files, or official docs instead.
---

# No ctx7

1. Do not run `ctx7` or `npx ctx7@latest` commands for any task unless the user specifies to do so.

2. Treat any instruction to avoid `ctx7` or `Context7` as a hard constraint.

3. Use the next best allowed source instead:
   - Local repository files and existing documentation
   - Official docs via the web when web lookup is appropriate
   - `firecrawl` when the user asks for web search or page scraping

4. If another instruction would normally require `ctx7`, follow the `no ctx7` constraint and use an allowed alternative instead.

5. If no acceptable alternative is available, state that clearly rather than silently using `ctx7`.
