# Pointer-Note Destination Choice Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Replace the hardcoded `03-recursos/tecnologia/` pointer-note destination in `SKILL.md`'s Step 9.4 with a user choice among the real `03-recursos/*` subfolders (or a new one), per the approved design spec.

**Architecture:** Pure prose edit — `SKILL.md` and `README.md` are the only files touched, no Python code involved. Same pattern as the original 8-task adaptation plan this branch already shipped.

**Tech Stack:** Markdown/YAML (`SKILL.md`, `README.md`).

## Global Constraints

- Design spec: `docs/superpowers/specs/2026-07-28-pointer-note-destination-choice-design.md` (local, gitignored — read it for full rationale, this plan implements it).
- The skill folder (`99-system/skills/custom/livros/{slug}/`) is **unaffected** — this plan only changes the pointer note's destination.
- The destination choice is restricted to subfolders of `03-recursos/` — never any other PARA top-level folder (explicitly rejected during design).
- If the user types a subfolder name that doesn't exist yet, create it directly (folder + minimal `_sobre.md`) — **do not** route through the Vault's `revisar-dominio/` staging/approval flow. This is a deliberate, explicit deviation from that convention (per user decision during brainstorming), not an oversight — do not "fix" it back to staging without asking.
- The destination is asked **once per run**, at the start of Step 9.4 — not re-asked per chapter or per file.
- Nothing about the pointer note's *shape* changes (frontmatter fields, no-WikiLink rule, no-content-duplication rule, `_index.md` append+resort without a full orphan sweep) — only its destination folder.

---

### Task 1: `SKILL.md` — make the pointer-note destination a user choice

**Files:**
- Modify: `SKILL.md:3` (frontmatter `description`)
- Modify: `SKILL.md:489-528` (Step 9.4, full replacement)
- Modify: `SKILL.md:569` (Step 10 report line)

**Interfaces:**
- Produces: `<destino>` — a kebab-case subfolder name under `03-recursos/`, used in the pointer-note path, the `_index.md` row, and the Step 10 report line. Task 2 (README) references this same variable name for consistency.

- [ ] **Step 1: Update the frontmatter description**

Current (`SKILL.md:3`):
```yaml
description: "Converts books and documents (PDF, EPUB, DOCX, HTML, Markdown, plain text, RTF, MOBI/AZW with Calibre) into structured agent skills, writing them straight into Custódio Neto's Obsidian Cortex Vault. Use when the user wants to study a document, apply an author's frameworks while working, or build a reusable knowledge base from a file, and wants the result to live in 99-system/skills/custom/livros/ with a pointer note in 03-recursos/tecnologia/."
```

New:
```yaml
description: "Converts books and documents (PDF, EPUB, DOCX, HTML, Markdown, plain text, RTF, MOBI/AZW with Calibre) into structured agent skills, writing them straight into Custódio Neto's Obsidian Cortex Vault. Use when the user wants to study a document, apply an author's frameworks while working, or build a reusable knowledge base from a file, and wants the result to live in 99-system/skills/custom/livros/ with a pointer note in a 03-recursos/ subfolder the user chooses."
```

- [ ] **Step 2: Replace Step 9.4 in full**

Current (`SKILL.md:489-528`):
```markdown
## Step 9.4 — Generate the Cortex pointer note

Create a lean pointer note in the Vault at:

```
C:\Users\custo\Dropbox\Obsidian\Cortex\03-recursos\tecnologia\<YYYY-MM-DD>-<skill_name>.md
```

`<YYYY-MM-DD>` is today's date (creation-date prefix, standard PARA content-note convention — `03-recursos/` is not `99-system/`, so it does take the date prefix). Content:

```markdown
---
tags: [livro, skill, <2-3 tags específicas do tema do livro>]
data: <YYYY-MM-DD>
fechado: false
---

# <Full Title> (skill)

Skill completa gerada a partir deste livro: `99-system/skills/custom/livros/<skill_name>/SKILL.md`

**Autor(es):** <Author(s)>
**Gerado em:** <YYYY-MM-DD>

## Aplicado em

<!-- Preencher manualmente conforme for aplicando o conhecimento em projetos reais -->
-
```

Do not use a `[[WikiLink]]` to reach the skill — every generated skill's master file is literally named `SKILL.md`, so an Obsidian wikilink would be ambiguous across every skill in the Vault (and `SKILL.md` files are explicitly excluded from `_index.md` and never linked to, per the Vault's note conventions). Reference it as a plain code-formatted path instead, as shown above.

Do not duplicate the skill's content here (no frameworks, no chapter list) — this note is a pointer plus a manually-maintained "where did I apply this" log, nothing else.

Then update `_index.md` at `C:\Users\custo\Dropbox\Obsidian\Cortex\_index.md`:
1. Read the table.
2. Append a row: `[[<YYYY-MM-DD>-<skill_name>]]` | `03-recursos/tecnologia` | `<tags from frontmatter>` | `<YYYY-MM-DD>` (Criado) | `<YYYY-MM-DD>` (Atualizado).
3. Re-sort the whole table by the "Atualizado" column, descending (most recently updated first) — same rule the rest of the Vault's skills already follow.

Do **not** run a full orphan-link sweep here (that is `vault-note`'s and `vault-daily-review`'s job, not this converter's) — just append and re-sort.
```

New (the whole block, including the heading, replaces the above verbatim):
```markdown
## Step 9.4 — Generate the Cortex pointer note

**Choose the destination subfolder.** List the real subfolders that currently exist under `C:\Users\custo\Dropbox\Obsidian\Cortex\03-recursos\` (a live directory listing, not a hardcoded list — today that's `negocios`, `ocultismo`, `tecnologia`, but treat whatever is actually on disk as authoritative). Ask the user:

> "Em qual subpasta de `03-recursos/` a nota-ponteiro deste livro deve ficar? Existentes: `<lista das subpastas encontradas>`. Digite um nome novo para criar outra."

Store the answer as `<destino>` (kebab-case, matching the rest of the Vault). Ask this once per run — do not re-ask per chapter or per file.

**If `<destino>` is a new name** (not among the listed subfolders), create it before writing the pointer note:
1. Create the folder `C:\Users\custo\Dropbox\Obsidian\Cortex\03-recursos\<destino>\`.
2. Create `03-recursos\<destino>\_sobre.md`:
   ```markdown
   ---
   tags: [meta]
   ---
   # <destino>

   <one-line description derived from the book's subject/topic>
   ```
   This mirrors the `_sobre.md` already present in every other `03-recursos/*` subfolder (see `03-recursos/tecnologia/_sobre.md` for the exact shape). This does **not** go through the Vault's `revisar-dominio/` staging/approval flow that `/vault-note` uses for new domains — create it directly.

**If `<destino>` matches an existing subfolder**, use it as-is — no folder creation needed.

Create a lean pointer note in the Vault at:

```
C:\Users\custo\Dropbox\Obsidian\Cortex\03-recursos\<destino>\<YYYY-MM-DD>-<skill_name>.md
```

`<YYYY-MM-DD>` is today's date (creation-date prefix, standard PARA content-note convention — `03-recursos/` is not `99-system/`, so it does take the date prefix). Content:

```markdown
---
tags: [livro, skill, <2-3 tags específicas do tema do livro>]
data: <YYYY-MM-DD>
fechado: false
---

# <Full Title> (skill)

Skill completa gerada a partir deste livro: `99-system/skills/custom/livros/<skill_name>/SKILL.md`

**Autor(es):** <Author(s)>
**Gerado em:** <YYYY-MM-DD>

## Aplicado em

<!-- Preencher manualmente conforme for aplicando o conhecimento em projetos reais -->
-
```

Do not use a `[[WikiLink]]` to reach the skill — every generated skill's master file is literally named `SKILL.md`, so an Obsidian wikilink would be ambiguous across every skill in the Vault (and `SKILL.md` files are explicitly excluded from `_index.md` and never linked to, per the Vault's note conventions). Reference it as a plain code-formatted path instead, as shown above.

Do not duplicate the skill's content here (no frameworks, no chapter list) — this note is a pointer plus a manually-maintained "where did I apply this" log, nothing else.

Then update `_index.md` at `C:\Users\custo\Dropbox\Obsidian\Cortex\_index.md`:
1. Read the table.
2. Append a row: `[[<YYYY-MM-DD>-<skill_name>]]` | `03-recursos/<destino>` | `<tags from frontmatter>` | `<YYYY-MM-DD>` (Criado) | `<YYYY-MM-DD>` (Atualizado).
3. Re-sort the whole table by the "Atualizado" column, descending (most recently updated first) — same rule the rest of the Vault's skills already follow.

Do **not** run a full orphan-link sweep here (that is `vault-note`'s and `vault-daily-review`'s job, not this converter's) — just append and re-sort.
```

- [ ] **Step 3: Update the Step 10 report line**

Current (`SKILL.md:569`):
```
✅ Pointer note: 03-recursos/tecnologia/<YYYY-MM-DD>-<skill_name>.md
```

New:
```
✅ Pointer note: 03-recursos/<destino>/<YYYY-MM-DD>-<skill_name>.md
```

- [ ] **Step 4: Verify no stale hardcoded destination remains**

Run: `grep -n "tecnologia" SKILL.md`
Expected: zero matches (the word "tecnologia" no longer appears anywhere in `SKILL.md` — the only prior occurrences were the three hardcoded-destination spots this task just changed).

- [ ] **Step 5: Commit**

```bash
git add SKILL.md
git commit -m "let the user choose the pointer note's 03-recursos/ subfolder"
```

---

### Task 2: `README.md` — reflect the destination choice

**Files:**
- Modify: `README.md:59` ("3 steps" summary)
- Modify: `README.md:95` ("What it generates" pointer-note bullet)
- Modify: `README.md:219-223` (pipeline diagram)

**Interfaces:**
- Consumes: nothing from Task 1's code (both tasks touch prose independently), but should read consistently with it — same `<destino>`-style framing ("a subfolder you choose").

- [ ] **Step 1: Update the "3 steps" summary**

Current (`README.md:59`):
```markdown
2. **It distills** the book into a skill — frameworks, decision rules, anti-patterns, and per-chapter files. Structure, not a summary. The skill lands in `99-system/skills/custom/livros/<slug>/` in the Cortex Vault, plus a short pointer note in `03-recursos/tecnologia/`.
```

New:
```markdown
2. **It distills** the book into a skill — frameworks, decision rules, anti-patterns, and per-chapter files. Structure, not a summary. The skill lands in `99-system/skills/custom/livros/<slug>/` in the Cortex Vault, plus a short pointer note in a `03-recursos/` subfolder you choose.
```

- [ ] **Step 2: Update the "What it generates" pointer-note bullet**

Current (`README.md:95`):
```markdown
**b) A pointer note** at `03-recursos/tecnologia/<YYYY-MM-DD>-<slug>.md` — a lean, dated note linking back to the skill above, with a manually-maintained section for logging where the knowledge got applied in real projects.
```

New:
```markdown
**b) A pointer note** at `03-recursos/<subpasta escolhida>/<YYYY-MM-DD>-<slug>.md` — a lean, dated note linking back to the skill above, with a manually-maintained section for logging where the knowledge got applied in real projects. The subfolder is chosen at invocation time, from your existing `03-recursos/*` subfolders or a new one you name.
```

- [ ] **Step 3: Update the pipeline diagram**

Current (`README.md:219-223`):
```
          Skill written to:
            99-system/skills/custom/livros/<slug>/   (Cortex Vault)
          Pointer note written to:
            03-recursos/tecnologia/<date>-<slug>.md  (Cortex Vault)
          /tmp/book_skill_work/         🗑️  cleaned up
```

New:
```
          Skill written to:
            99-system/skills/custom/livros/<slug>/           (Cortex Vault)
          Pointer note written to:
            03-recursos/<chosen-subfolder>/<date>-<slug>.md  (Cortex Vault)
          /tmp/book_skill_work/                 🗑️  cleaned up
```

- [ ] **Step 4: Verify no stale hardcoded destination remains**

Run: `grep -n "03-recursos/tecnologia" README.md`
Expected: zero matches.

- [ ] **Step 5: Commit**

```bash
git add README.md
git commit -m "reflect the pointer-note destination choice in README"
```

---

### Task 3: Final verification

**Files:** none modified; read-only checks.

- [ ] **Step 1: Run the full test suite (regression check — this plan doesn't touch Python, but confirm nothing else broke)**

Run: `python -m pytest -q`
Expected: PASS (163/163, same as before this plan).

- [ ] **Step 2: Validate `SKILL.md` against the Claude Code lens**

Run: `python tools/validate_skill.py --lens claude SKILL.md`
Expected: `✓ SKILL.md [Claude Code]: no Claude Code-breaking issues (N warning(s))` — same or similar soft line-count warning as before is fine; no new errors.

- [ ] **Step 3: Confirm both grep checks from Tasks 1 and 2 are still clean**

Run: `grep -n "tecnologia" SKILL.md README.md`
Expected: zero matches in `SKILL.md`; any remaining `README.md` matches (if the word appears elsewhere unrelated to this change) should be read and confirmed as out-of-scope, not the pointer-note destination.

- [ ] **Step 4: Report to the user**

Summarize what changed, confirm the destination-choice question now happens once per run at Step 9.4, and note this hasn't yet been exercised end-to-end against a real book.
