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
- Telegraphic instructions (`telegraphic`): 6 or more words with no article,
  preposition or short verb. An example is `Remove cap, check seal, replace
  gasket if worn`. Write `Remove the cap`.
- Safety labels (`warning-label`): `WARNING` is for a risk of injury,
  `CAUTION` for a risk of damage, `NOTE` for information. A `WARNING` or
  `CAUTION` starts with a command.
- Long paragraphs (`paragraph-length`): more than 6 sentences.
- Procedures in running text (`procedure-form`): 3 or more instructions in one
  paragraph or in one list item. Write a numbered list with one instruction in
  each step.
- History in source comments (`history-comment`): dates, incidents and people.
  Put that story in the dev diary or the commit message.
- Skill-loader substitution (`skill-arg`), only in a file named `SKILL.md` or
  with `--skill`. The Claude Code loader replaces a dollar sign plus a digit
  with the invocation arguments. Write prices as `N USD`, awk fields as `$(N)`
  and shell arguments as `${N}`. Mark an intended use with `skill-arg-ok`.
  Any hit fails the run with exit 1. The hit does not count in the score.

## Fact coverage (rewrites)

A style score rewards a rewrite that says less. Give the original with
`--source` and ste-lint checks that the rewrite keeps these facts of the
source:

- every number, inline code term and identifier (path, flag, `snake_case`,
  `CamelCase`, ALL-CAPS);
- every negation (`not`, `never`, `no`);
- every value word. A value word is a plain word that stands for a value:

| Class | Words | Value |
|---|---|---|
| color | red, green, `#f00` | hex code |
| polarity | positive, plus, `+`, negative, ground | sign |
| side, vertical, depth | left, top, rear | axis value |
| rotation, radial, compass | clockwise, inward, north | axis value |
| state | open, closed, enabled, true, `required` | state |
| limit | maximum, at least, below, `<=` | operator |
| order | first, last, before, after | position |
| quantity | `12 V`, `30 min`, `5 volts` | number and unit |

ste-lint compares the words of one class by value, so `red` and `#ff0000`
match, and `12 V` and `12 volts` match. A changed value is a swap: "the source says
'clockwise' and the rewrite has 'counterclockwise' instead".

`--min-ratio R` also fails a rewrite with fewer than R times the source's
words. Any lost fact fails the run with exit 1. The lost facts do not count
in the score. Other plain words, such as "terminal" or "cable", are not
checked, so a human still reads the rewrite.

```bash
ste-lint rewrite.md --source original.md --min-ratio 0.6
```

### Summaries of logs and other files

A summary of a long log must keep the error names, paths and flags, but not
every number in the log. `--facts` limits the check to some fact kinds:
`code`, `ident`, `number`, `negation` and `value`.

```bash
ste-lint handoff.md --source raw.log --facts code,ident
```

Log levels and safety labels (`INFO`, `ERROR`, `WARNING`, `NOTE`) are not
facts.

## Changed lines only

`--diff [REF]` reports only the paragraphs with a line added or changed since
the git ref (default `HEAD`). A file that git does not track counts whole.
`--lines 3-10,15` does the same for a given line range, also on stdin. Use
either option to gate new text without first fixing every old line.

```bash
ste-lint README.md --diff --max-score 2.0
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
ste-lint NEW --source LOG --facts code,ident  # summary kept the identifiers
ste-lint FILE --diff              # only lines changed since HEAD
ste-lint FILE --lines 3-10,15     # only these lines
ste-lint -                        # read from stdin
```

## Scoring

Violations per 100 verbs. Published ASD-STE100 research: baseline prose ~4.36,
STE-compliant prose ~1.12. Target < 2.0 for technical documentation.

The report also shows a density figure:
(verbs + fact terms + value words) / words, and the mean words per sentence.
Filler lowers the density. Numbers, identifiers and actions raise it. The
density is for comparison and for debugging, never a gate.

## Supported file types

ste-lint checks Markdown and plain text directly. In source files it blanks
the code lines and checks only the comment and docstring prose:

`.py .ts .js .go .rs .java .c .cpp .sh .sql .rb .css .yaml .toml` and more.

## Agent instructions

See `AGENTS.md` — compact workflow for automated QA, cross-project audits,
and machine-readable integration.
