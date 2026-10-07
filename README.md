# ste-lint

ASD-STE100-inspired prose linter. Checks Markdown, plain text, and source-code
comments against Simplified Technical English rules. Stdlib only — one file,
no install.

## What it checks

- Sentences over 20 words (instructions) or 25 words (descriptions)
- ~200 deny-list entries: inflated verbs (`leverage`, `utilize`, `facilitate`),
  hedges (`essentially`, `potentially`), marketing adjectives (`robust`,
  `seamless`), AI tells (`delve`, `tapestry`, `synergy`), nominalizations,
  phrasal verbs
- Passive voice
- Punctuation symptoms (em dashes, semicolons, comma-heavy sentences)
- Synonym rotation (calling the same thing by different names)

## Install

```bash
cp ste-lint ~/.local/bin/ste-lint
chmod +x ~/.local/bin/ste-lint
```

## Usage

```bash
ste-lint FILE                     # human-readable report
ste-lint FILE --json              # machine-readable (for scripts and agents)
ste-lint FILE --max-score 2.0     # CI gate — exit 1 if score exceeds 2.0
ste-lint FILE --lang py           # force language (stdin or non-standard ext)
ste-lint FILE --glossary proj.txt # whitelist project-specific jargon
ste-lint -                        # read from stdin
```

## Scoring

Violations per 100 verbs. Published ASD-STE100 research: baseline prose ~4.36,
STE-compliant prose ~1.12. Target < 2.0 for technical documentation.

## Supported file types

Markdown and plain text are checked directly. Source files have code lines
blanked; comment and docstring prose is extracted and checked:

`.py .ts .js .go .rs .java .c .cpp .sh .sql .rb .css .yaml .toml` and more.

## Agent instructions

See `AGENTS.md` — compact workflow for automated QA, cross-project audits,
and machine-readable integration.
