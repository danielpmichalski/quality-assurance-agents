# AGENTS.md

Guidance for AI coding agents (and humans) working in this repository.

## What this repo is

`quality-assurance-agents` is a **Claude Code plugin** (no application code, no build step) that
ships nine **test role agents** based on the ISTQB® syllabi, plus one shared skill. Every agent
is a Markdown file with YAML frontmatter; the "source code" is prompt text. Quality of the
project = quality, precision and reliability of those prompts.

Status: `0.1.0`, **not yet verified** (see the warning in `README.md`). The roadmap is to
improve the agents against public ISTQB® material and prove their reliability with
`claude plugin eval`.

## Repository layout

```
.claude-plugin/plugin.json        Plugin manifest (name, version, keywords)
.claude-plugin/marketplace.json   Marketplace entry (source "./")
agents/<role>.md                  One agent per file; filename == `name:` in frontmatter
skills/qa-id-scheme/SKILL.md      Shared ID scheme, preloaded into every agent (user-invocable: false)
README.md                         User-facing docs (agent table, pipeline, ID summary, install)
LICENSE                           MIT
```

`.idea/` and `*.iml` are git-ignored. There are no tests, scripts or CI yet.

## The agents

| Agent | Qualifier | Basis | Tools | Default output |
|---|---|---|---|---|
| `acceptance-tester` | `ACC` | Acceptance Testing, CTFL 4.0 | R/W | `<input>.acceptance.md` |
| `test-analyst` (base role) | none | CTFL 4.0, CTAL-TA 4.0 | R/W | `<input>.test-design.md` |
| `security-tester` | `SEC` | Security Tester, OWASP, STRIDE, CWE | R/W | `<input>.security-test-design.md` |
| `usability-tester` | `UX` | Usability Testing, ISO 9241, WCAG 2.2 | R/W | `<input>.usability-test-design.md` |
| `performance-tester` | `PERF` | Performance Testing, ISO/IEC 25010 | R/W | `<input>.performance-test-design.md` |
| `model-based-tester` | `MBT` | Model-Based Tester | R/W | `<input>.mbt.md` |
| `technical-test-analyst` | `TECH` | CTAL-TTA | R/W + **Bash** | `technical-test-design.md` |
| `test-automation-engineer` | `AUTO` | CTAL-TAE | R/W + **Bash** | `<input>.automation.md` + test code |
| `reviewer` | `REV` | CTFL 4.0 static testing, ISO/IEC 20246 | R/W | `<reviewed-file>.review.md` |

"R/W" = `Read, Write, Edit, Glob, Grep`. Only the two agents that must run existing tools get
`Bash`; do not add tools to an agent without a concrete need.

Typical flow: `acceptance-tester` → `test-analyst` → (`security-tester`, `performance-tester`,
`usability-tester`, `model-based-tester`) → `test-automation-engineer`; `technical-test-analyst`
works from code; `reviewer` can review any output. Each agent must also work standalone.

## Anatomy of an agent file

Frontmatter: `name`, `description` (multi-line `>` block; it is the **routing text**, so it
must contain the "Use when asked to …" trigger phrases), `tools`, `model: inherit`, and
`skills: [qa-id-scheme]`.

Body, in this order (keep it when editing — the agents are deliberately parallel):

1. Role statement (syllabi, Glossary, standards; what the role does **not** do)
2. `## Shared ID scheme` (role qualifier, defers to the skill)
3. `## Input` (what the caller provides, default output path, "never overwrite the input",
   update existing output and preserve IDs)
4. `## Working method` (numbered steps ending in a **self-check before writing**)
5. `## Output document structure` (fixed numbered sections in a fenced Markdown template)
6. `## Rules` (terminology, no invented requirements, GFM tables, input language, and the
   exact short reply the agent gives the caller — it must not paste the document back)

## Invariants — do not break these

- **ID scheme**: `skills/qa-id-scheme/SKILL.md` is the single source of truth. `AC-` IDs are
  never qualified and only `acceptance-tester` creates new ones; IDs are stable, never
  renumbered, never reused (mark *withdrawn*); two-digit zero padding. If you add a role-specific
  ID or output file, update the skill, the agent, **and** the README table.
- **No invented requirements.** Gaps become findings (`F-`), assumptions (`ASM-`) or open
  questions, each labelled.
- **Design agents only design.** `security-tester` never attacks, `performance-tester` never
  generates load, `usability-tester` never claims to have observed users.
- **Numbers come from tools.** `technical-test-analyst` reports coverage/mutation/static-analysis
  figures only from deterministic tools, with command + report provenance. "Not measured" is valid.
- **Never weaken an assertion** to make a test pass (`test-automation-engineer`); a mismatch is
  a suspected defect. Project conventions (this file's equivalent in the *target* repo) win.
- **Inputs are never modified**; each agent writes only its own output file (`reviewer` is
  strictly read-only toward the work product).
- **Bash agents are read-only toward the codebase** except the automation engineer's test files.

## ISTQB® content policy (important for the improvement work)

- ISTQB® is a registered trademark. This project is **not affiliated with or endorsed by ISTQB®**;
  keep that disclaimer in `README.md` and the manifests.
- Agent text is **original work that refers to** syllabi and the Glossary. **Do not copy
  syllabus or Glossary text** (definitions, learning-objective wording, tables, exam material)
  into agents, skills or docs — paraphrase, cite the source document, version and section, and
  keep it short. Check the licence terms of each public ISTQB® document before using it, and
  record what was used (see "Planned work").
- Use Glossary terms precisely (test basis, test condition, coverage item, test oracle, defect vs
  failure, confirmation vs regression testing…). Do not invent terms.
- Prefer verifying against the current syllabus version named in each agent (CTFL v4.0,
  CTAL-TA v4.0, CTAL-TTA, CTAL-TAE, specialist syllabi). If a newer version exists, update the
  agent's wording *and* its description/README mention together.

## Working on the agents

- Edit prompts, not just wording: every change should make behaviour more precise, more
  verifiable or more reliable. Keep proportionality guidance ("depth follows risk") and
  explicit stop conditions ("ask for a path, never invent a test basis").
- Keep instructions imperative and testable — each rule should be something an eval can check
  (an ID format, a required section, a forbidden action, a reply shape).
- Keep the parallel structure across agents; if you improve a cross-cutting rule (e.g.
  self-check, reply format, update-in-place behaviour), apply it to **all nine** agents.
- Keep descriptions accurate; routing quality depends on them. Don't make them so broad that
  they overlap another agent (e.g. `reviewer` vs. the review step inside each designer).
- Language: instructions are English; agents answer in the input document's language.
- Bump `version` in `.claude-plugin/plugin.json` for behaviour-changing releases and keep the
  agent count/wording in `plugin.json`, `marketplace.json` and `README.md` consistent.

## Validating changes

Currently manual:

```bash
claude --plugin-dir /path/to/quality-assurance-agents      # load the local plugin
claude plugin validate .                                    # manifest/frontmatter check, if available in your version
```

Then run the changed agent on a small fixture (a short story with ACs) and check: output
file location, section structure, ID format, traceability completeness (no orphan AC/TC),
no modified input, and the short reply.

### Planned work: evals with `claude plugin eval`

Not set up yet. When adding it, follow `claude plugin eval --help` / the Claude Code docs for
the current suite format rather than guessing, and:

- Put suites and fixtures in a dedicated top-level directory (e.g. `evals/`), not in `agents/`.
- Use small, realistic fixtures per agent (stories with deliberate ambiguities, boundaries,
  missing error behaviour; code + coverage reports for the technical agents; a work product
  with seeded defects for `reviewer`).
- Prefer **deterministic assertions** (file exists, required headings, ID regexes, every AC
  appears in the traceability matrix, input file unchanged, no `Bash` write outside tests)
  over model-graded checks; use graded checks only for judgment (finding quality, risk levels).
- Include **negative/guardrail cases**: no path given → agent asks; invented requirement bait;
  request to attack a live system or generate load; request to report coverage without a tool;
  failing test the agent is tempted to "fix" by weakening.
- Include **cross-agent traceability** cases (acceptance → analyst → automation) and
  **idempotence** (re-run preserves IDs).
- Track ISTQB® sources used for improvements in a short `docs/sources.md` (document, version,
  URL, licence note, which agent/section it informed).

## Conventions

- Commit messages: Conventional Commits (`feat:`, `fix:`, `docs:`, `chore:`), as in the
  existing history. Default branch: `main`.
- Markdown: GFM tables with escaped `|`, ~95-column wrapping in agent files, fenced blocks
  tagged with a language.
- Don't commit generated outputs from running the agents (`*.test-design.md`, `*.review.md`, …)
  unless they are deliberate eval fixtures/goldens in `evals/`.
- Don't edit `.idea/`.
