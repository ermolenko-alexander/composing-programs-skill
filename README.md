# composing-programs — Agent Skill

Agent skill generated from *Composing Programs* by John DeNero ([composingprograms.com](https://composingprograms.com), the UC Berkeley CS61A textbook) with [book-to-skill](https://github.com/virgiliojr94/book-to-skill).

The content is **synthesized summaries, not the book text**: 29 chapter files with frameworks, mental models, code examples and worked examples, plus a glossary, a patterns file and a decision-rule cheatsheet. Chapter files load on demand, so an agent only pays for the chapter it reads.

## Why this matters — for vibe-coders and everyone else

If you build software by describing what you want and letting an AI agent write the code, the quality ceiling of what you get is set by the *judgment* available in context — not by the model's raw ability to emit syntax. This skill puts the distilled judgment of the canonical programming-fundamentals book (Berkeley's CS61A, the Python successor to SICP) into that context.

What changes in practice:

- **Your agent stops copy-pasting loop skeletons.** The book's higher-order-function discipline ("two functions share a skeleton → extract the pattern") is in the cheatsheet as a decision rule, so refactors happen at generation time, not review time.
- **Fewer state bugs.** Identity vs equality, mutable default arguments, when `nonlocal` state is justified and when it's a smell — the exact class of bugs that silently corrupts vibe-coded projects after week two.
- **Performance fixes become one-liners.** "Pure function recomputed on repeated args → memoize (Θ(φⁿ)→Θ(n))" is a lookup, not an investigation.
- **Abstraction barriers survive growth.** The number-one reason AI-generated codebases rot is code reaching past interfaces into representations. The book names the discipline; the skill makes it checkable.
- **You learn while shipping.** Every answer cites the section it came from, so the skill doubles as a study guide: ask for `ch16` and you get closures explained the way DeNero teaches them.

Fundamentals don't age. Frameworks churn every season; the environment model, data abstraction, recursion discipline and orders of growth are the same in 2026 as in 1985 — and they're exactly what distinguishes code that survives from code that demos well.

## Install

```bash
npx skills add https://github.com/ermolenko-alexander/composing-programs-skill --skill composing-programs
```

Works with Claude Code, GitHub Copilot CLI, Amp, Codex and other Agent Skills hosts.

## Contents

- `SKILL.md` — core frameworks, chapter index, topic index
- `chapters/` — 29 section summaries (ch11–ch48)
- `glossary.md` — key terms with chapter references
- `patterns.md` — 13 techniques with when/how/trade-offs
- `cheatsheet.md` — decision rules, thresholds, tells & smells

## License

The source book is licensed [CC BY-SA 3.0](https://creativecommons.org/licenses/by-sa/3.0/) by John DeNero. These derived summaries are distributed under the same license, with attribution to the original work.
