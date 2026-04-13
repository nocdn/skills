---
name: generate-commit-message
description: generate a one-line commit message that matches the repository's existing style by inspecting recent commit subjects and the current staged or unstaged changes. use when the user asks for a commit message, git commit text, or help summarizing current changes.
---

# Generate Commit Message

1. Inspect recent commit subjects to learn the repo's style and conventions.
   - Prefer: `git log --no-merges --pretty=format:"%s" -n 20`

2. Inspect the current changes.
   - Use the history of the current chat with the user and also:
   - Check staged changes first.
   - If nothing is staged, inspect unstaged changes.
   - Use lightweight summaries before full diffs when helpful.

3. Infer the dominant commit style in the repo.
   - For example: imperative verbs, sentence case, lowercase, conventional commits, ticket prefixes, scope prefixes.

4. Draft a descriptive enough but concise one-line commit message that matches that style and accurately reflects the current changes.

5. Return only the proposed commit message unless the user asks for alternatives.
