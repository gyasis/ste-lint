# ste-lint — Agent Instructions

`ste-lint` is a deterministic, dependency-free prose linter. It checks Markdown,
plain text, and source-code comments against an ASD-STE100-inspired rule set. No
LLM, no network, no install beyond copying the file.

---

## When to use it

Run it on any human-written prose that other people read: documentation,
docstrings, code comments, commit messages, README files, design documents, wiki
pages, changelogs. Do not run it on generated text, template stubs, or code itself
(identifiers, expressions, queries).

---

## Core flags — memorise these four

```bash
ste-lint FILE                          # human-readable report
ste-lint FILE --json                   # machine-readable violations (use this)
ste-lint FILE --max-score 2.0          # CI gate — exit 1 if score > 2.0
ste-lint FILE --lang py                # force language (for stdin or odd extensions)
ste-lint FILE --glossary terms.txt     # whitelist project-specific jargon
ste-lint FILE --skill                  # skill-arg check (automatic for SKILL.md)
ste-lint NEW --source OLD              # fact-coverage: fail if the rewrite lost a fact
ste-lint NEW --source LOG --facts code,ident  # a summary kept every identifier
ste-lint FILE --diff                   # only lines changed since HEAD
ste-lint FILE --lines 3-10,15          # only these lines
ste-lint -                             # read from stdin
```

**Always use `--json` in agent workflows.** The human-readable report is for
presenting results to a person. JSON gives you structured violations with exact
line, column, category, found text, and replacement — enough to fix without
re-reading the file.

---

## Score interpretation

| Score (violations / 100 verbs) | Meaning |
|---|---|
| < 1.5 | Clean — matches STE research target |
| 1.5 – 2.5 | Acceptable for most technical prose |
| 2.5 – 4.36 | Needs editing |
| > 4.36 | Baseline (no STE applied) — fix before shipping |
| `null` / undefined | Zero verbs detected; score is meaningless — check the input |

The second metric (`score_per_100_words`) is heuristic-free and good for comparing
files of different lengths. Use `per_verbs` as the primary gate.

---

## The lint loop (fix a file)

```python
# 1. Run
result = json.loads(subprocess.check_output(["ste-lint", file, "--json"]))

# 2. Check
if result["ok"] or result["score_per_100_verbs"] < 2.0:
    break  # done

# 3. Fix — work from result["violations"], highest severity first
for v in result["violations"]:
    if v["category"] == "punctuation":
        continue  # advisory only — fix last
    # apply v["replace"] at v["line"]:v["col"]

# 4. Repeat until score < 2.0 or no scored violations remain
```

Fix `skill-arg` and `fact-coverage` first: they are correctness errors, not
style problems. They do not count in the score, but any hit fails the run.
When you rewrite existing text, always pass `--source` with the original.
Keep the best-scoring pass that has no `fact-coverage` hit, and stop after 3
passes: a later pass can score worse.
Then fix categories in this order: `history-comment` → `warning-label` → `procedure-form` → `deny-list` → `multi-action` →
`telegraphic` → `sentence-length` → `paragraph-length` → `passive-voice` →
`synonym-rotation` → `punctuation` (advisory). Punctuation is never counted in the
score unless you pass `--only punctuation` explicitly.

---

## Cross-project audit

Sweep all prose files in a project and report the worst offenders:

```bash
find . -type f \( \
  -name "*.md" -o -name "*.py" -o -name "*.ts" -o -name "*.js" \
  -o -name "*.go" -o -name "*.rs" -o -name "*.sql" -o -name "*.sh" \
\) | while read f; do
  score=$(ste-lint "$f" --quiet 2>/dev/null | grep -oP '[\d.]+(?= violations per 100)' | head -1)
  echo "$score $f"
done | sort -n | tail -20
```

Or with JSON for more control:

```bash
find . -name "*.md" -o -name "*.py" | xargs -I{} sh -c \
  'ste-lint "{}" --json 2>/dev/null | python3 -c \
  "import json,sys; d=json.load(sys.stdin); \
   print(d[\"score_per_100_verbs\"], d[\"scored_violations\"], sys.argv[1])" "{}"' \
| sort -n
```

---

## Reporting to a human — be terse

Do not paste the full violation list. Report:

```
FILE (score X.XX/100v, N violations):
  line 12: "leverage" → use
  line 34: sentence 31 words — split
  line 67: passive "is checked" — rewrite active
```

Three violations per file maximum in a summary. If there are more, say "N more".
Link to the file and line. Do not explain what ASD-STE100 is unless asked.

---

## Supported file types (auto-detected from extension)

`.py .pyi .js .jsx .ts .tsx .svelte .go .rs .java .c .cpp .h .sh .bash .rb .sql .lua .css .yaml .yml .toml .r`

For markdown, plain text, and stdin: the tool checks prose as-is (no comment
extraction needed). Pass `--lang md` or `--lang txt` to disable source extraction
on stdin.

---

## Glossary pattern (project-specific jargon)

When a project uses domain terms the deny list would flag (e.g., `broker`,
`wuphf`, `sidecar`, `litellm`), create a glossary file:

```
# ste-lint glossary for twicedata_intra
broker
wuphf
sidecar
litellm
PromptChain
```

Pass it on every run: `ste-lint FILE --glossary .ste-glossary`. The terms are
never flagged regardless of which rule would normally match them.

---

## What the tool does NOT do

- It does not rewrite prose. It flags and suggests.
- It does not check grammar, spelling, or factual accuracy.
- It does not understand context — a sentence that is technically fine but
  says nothing useful will score 0.0.
- It is not a style guide. It enforces the *form* (short sentences, plain
  verbs, active voice), not the substance.

---

## Exit codes

| Code | Meaning |
|---|---|
| 0 | No `--max-score` set, or score ≤ threshold |
| 1 | `--max-score` set and score exceeded, or any `skill-arg` or `fact-coverage` hit |
| 2 | Usage error or file not readable |
