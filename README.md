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
- Several actions in one sentence (`multi-action`): 3 or more clauses that
  start with a verb. Write one action in each sentence.
- Chat padding: `Great question`, `I hope this helps`, `in summary`.
- History in source comments (`history-comment`): dates, incidents and people.
  Put that story in the dev diary or the commit message.
- Skill-loader substitution (`skill-arg`), only in a file named `SKILL.md` or
  with `--skill`. The Claude Code loader replaces a dollar sign plus a digit
  with the invocation arguments. Write prices as `N USD`, awk fields as `$(N)`
  and shell arguments as `${N}`. Mark an intended use with `skill-arg-ok`.
  Any hit fails the run with exit 1. The hit does not count in the score.

## Fact coverage (rewrites)

A style score rewards a rewrite that says less. Give the original with
`--source` and ste-lint checks that the rewrite keeps every number, inline
code term, identifier (path, flag, `snake_case`, `CamelCase`, ALL-CAPS) and
negation of the source. `--min-ratio R` also fails a rewrite with fewer than
R times the source's words. Any lost fact fails the run with exit 1. The
lost facts do not count in the score. The check does not see plain words, so
a lost "red cable" still needs a human read.

```bash
ste-lint rewrite.md --source original.md --min-ratio 0.6
```

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
ste-lint - --skill < SKILL.md     # skill-arg check on stdin
ste-lint NEW --source OLD         # fail if the rewrite lost a fact
ste-lint -                        # read from stdin
```

## Scoring

Violations per 100 verbs. Published ASD-STE100 research: baseline prose ~4.36,
STE-compliant prose ~1.12. Target < 2.0 for technical documentation.

## Supported file types

ste-lint checks Markdown and plain text directly. In source files it blanks
the code lines and checks only the comment and docstring prose:

`.py .ts .js .go .rs .java .c .cpp .sh .sql .rb .css .yaml .toml` and more.

## Agent instructions

See `AGENTS.md` — compact workflow for automated QA, cross-project audits,
and machine-readable integration.
