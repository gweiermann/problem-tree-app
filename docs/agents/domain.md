# Domain documentation

This repository uses a single-context domain-documentation layout.

## Canonical glossary

The root `CONTEXT.md` is the canonical glossary for the product domain. Read it before naming domain concepts in issues, specifications, tests, or code. Use its terms consistently and update it when a domain term is resolved or materially sharpened.

Keep `CONTEXT.md` free of implementation details. It describes the domain language and invariants, not technical architecture.

## Architecture decisions

Hard-to-reverse architectural decisions belong in `docs/adr/`. Create that directory lazily when the first ADR is warranted.

Before proposing or recording a decision, check existing ADRs. Surface conflicts explicitly instead of silently replacing an earlier decision.
