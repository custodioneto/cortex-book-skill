# Cortex Livro-Skill Adaptation Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Adapt this fork of `book-to-skill` (upstream: github.com/virgiliojr94/book-to-skill, MIT) so that running its converter always writes into Custódio Neto's Obsidian Cortex Vault, following the Vault's PARA/staging/naming conventions, and is invoked as `/vault-livro-to-skill` instead of `/book-to-skill`.

**Architecture:** `SKILL.md` is the actual generator spec — Claude reads it and does the extraction/writing itself; `scripts/extract.py` (the `book_to_skill` package) only does raw text extraction into a temp working dir. Almost all of the required behavior change lives in `SKILL.md` prose, not in Python. The one piece of real code affected is `tools/scan_generated_skill.py`, which hardcodes the `chapters/` folder name when it collects files to security-scan.

**Tech Stack:** Python 3.9+ (extraction engine, untouched logic), Markdown/YAML (`SKILL.md`, generated skill files), pytest.

## Global Constraints

- Every path written to disk by the generated skill must be **kebab-case**, matching the rest of the Vault (user instruction #4).
- The generated **skill folder** goes to `C:\Users\custo\Dropbox\Obsidian\Cortex\99-system\skills\custom\livros\{titulo-slug}\` — **no date prefix** (99-system/ is dateless kebab-case, per global `C:\Users\custo\CLAUDE.md` and `AGENTS.md`).
- The generated **pointer note** goes to `C:\Users\custo\Dropbox\Obsidian\Cortex\03-recursos\tecnologia\{YYYY-MM-DD}-{titulo-slug}.md` — **with** a creation-date prefix (standard PARA content-note convention) and must not duplicate skill content.
- This fork targets **Claude Code / the Cortex Vault only** — per explicit user decision, drop GitHub Copilot CLI / Amp multi-host destination logic from `SKILL.md` rather than keep it as an alternate branch.
- Chapter files inside the renamed `capitulos/` folder keep the **English `chNN-<slug>.md` prefix** (per explicit user decision) — only the containing folder name changes to Portuguese.
- `pyproject.toml`'s package name and `book-to-skill` console-script entry point are **out of scope** — only the slash-command identity (`SKILL.md` `name:` frontmatter) changes, per user ask #3, which is scoped to "o comando de invocação", not the pip package.
- `docs/ARCHITECTURE.md`, `docs/PERFORMANCE.md`, `docs/index.md`, `CHANGELOG.md`, `CONTRIBUTING.md`, `BACKERS.md`, `SECURITY.md`, `mkdocs.yml` are **not touched** by this plan — none of the 6 user asks mention the public mkdocs docs site; only `README.md` was explicitly named (ask #6).
- `LICENSE.md` is **not touched** — MIT requires only that the original copyright notice survive, and it already does.
- The structural addition of a new `livros/` sub-namespace under `99-system/skills/custom/` must go through `06-metadata/staging/` for the user's manual approval before being treated as an accepted convention (user ask #5, and the Vault's own critical-structural-change rule in `CLAUDE.md`/`AGENTS.md`/`cortex-status.md`).

---

### Task 1: Vault staging proposal for the new `livros/` skill sub-namespace

**Files:**
- Create (in the Vault repo, not this repo): `C:\Users\custo\Dropbox\Obsidian\Cortex\06-metadata\staging\2026-07-28-skills-custom-livros.md`

**Interfaces:**
- Produces: the approved path convention `99-system/skills/custom/livros/{titulo-slug}/` that Task 3 hardcodes into `SKILL.md`. Do not start Task 3 until this staging doc is approved (or the user explicitly says to proceed in parallel).

- [ ] **Step 1: Write the staging proposal**

Create the file with this exact content:

```markdown
---
tags: [meta]
---

# Proposta — nova sub-pasta `livros/` em `99-system/skills/custom/`

Rascunho de planejamento. **Nada foi executado.** Nenhuma pasta real foi
criada fora desta nota de staging. Aguarda aprovação.

## Contexto

O fork `cortex-book-skill` (github.com/custodioneto/cortex-book-skill,
baseado em github.com/virgiliojr94/book-to-skill) está sendo adaptado para
gravar as skills que gera diretamente no Vault, em vez de num destino solto
fora do Cortex. Isso introduz uma categoria nova dentro de
`99-system/skills/custom/`: skills derivadas de livros processados, uma por
livro, cada uma dona de um `SKILL.md` completo (núcleo + capítulos +
glossário + patterns + cheatsheet) — mesmo formato de skill que as `vault-*`
já usam, mas com conteúdo gerado a partir de um livro em vez de escrito à mão.

## Estrutura proposta

```diff
  99-system/skills/custom/
  ├── vault-close/
  ├── vault-daily-review/
  ├── ...
+ └── livros/
+     ├── _sobre.md
+     └── {titulo-slug}/
+         ├── SKILL.md
+         ├── cheatsheet.md
+         ├── patterns.md
+         ├── glossary.md
+         └── capitulos/
+             └── ch01-*.md ...
```

- `livros/` **não** leva prefixo de data — é `99-system/`, mesma regra
  dateless kebab-case do resto dessa pasta.
- `{titulo-slug}` em kebab-case (ex.: `designing-data-intensive-apps`,
  `cialdini-influence`), igual ao resto do Vault.
- `livros/_sobre.md` (novo, `tags: [meta]`) explica o propósito da pasta,
  seguindo o padrão de `_sobre.md` já usado em domínios do PARA.
- Cada `livros/{slug}/` é auto-descoberta como skill via o symlink já
  existente `.claude\plugins\custom\` → `99-system/skills/custom/`, sem
  configuração adicional — mesmo mecanismo que já expõe `vault-note` etc.
  **Não confirmado ainda:** se o `sync-commands` do PowerShell profile (que
  gera `.claude\commands\*.md` a partir de `SKILL.md`) enxerga subpastas
  aninhadas (`livros/{slug}/SKILL.md`) ou só o primeiro nível
  (`vault-note/SKILL.md`). Validar na primeira execução real do
  `/vault-livro-to-skill` — se não gerar slash command automaticamente,
  isso é cosmético (a skill ainda é carregável por invocação direta) e
  vira um item de backlog separado, não bloqueia esta proposta.

## Por que staging

Mudança estrutural crítica (nova sub-pasta dentro de `99-system/skills/custom/`)
— regra do Cortex é propor em `06-metadata/staging/` antes de criar no disco
de verdade, mesmo quando quem pediu foi o próprio Neto.

## Depois de aprovado

Este documento sai de `staging/`, vira
`06-metadata/logs/2026-07-28-skills-custom-livros.md` (registro histórico),
e ganha uma linha resumida em `99-system/contexto/decisoes.md` linkando pro
registro completo. A pasta `livros/` em si só é criada de fato na primeira
vez que `/vault-livro-to-skill` processar um livro — esta aprovação libera a
convenção, não cria a pasta antecipadamente.
```

- [ ] **Step 2: Ask the user to approve**

Report the file path and ask the user to review it and confirm approval (mirrors how `vault-note`'s own domain-approval flow works: nothing is moved into `06-metadata/logs/` or referenced as decided until the user says so).

---

### Task 2: Rebrand the skill identity in `SKILL.md`

**Files:**
- Modify: `SKILL.md:1-16` (frontmatter + cross-agent notes comment)
- Modify: `SKILL.md:82-93` (Step 0 out-of-scope message)

**Interfaces:**
- Produces: `name: vault-livro-to-skill` frontmatter that Task 7 (README) and Task 1's staging doc both refer to by that name.

- [ ] **Step 1: Replace the frontmatter and cross-agent comment block**

Current (`SKILL.md:1-17`):
```markdown
---
name: book-to-skill
description: "Converts books and documents (PDF, EPUB, DOCX, HTML, Markdown, plain text, RTF, MOBI/AZW with Calibre) into structured agent skills, extracting frameworks, mental models, principles, techniques, and anti-patterns. Use when the user wants to study a document through GitHub Copilot CLI, Amp, or Claude Code, apply an author's frameworks while working, or build a reusable knowledge base from a file."
---

<!--
Cross-agent notes (informational; ignored by host agents):
  - Compatible skill roots: GitHub Copilot CLI (~/.copilot/skills, ~/.agents/skills,
    .github/skills, .claude/skills, .agents/skills), Amp (.agents/skills,
    ~/.config/agents/skills, ~/.config/amp/skills), Claude Code (~/.claude/skills).
  - `allowed-tools` is intentionally omitted to stay agent-neutral: Copilot CLI uses
    `shell`/MCP-server names, Claude uses `Bash`/`Read`/`Write`/`Glob`/`Grep`, Amp
    adds `shell_command`. The skill needs shell (to run extract.py) and file
    read/write — each host will prompt for those on first use.
  - Argument hint: <path-to-document-folder-or-glob>... [skill-name-slug]
-->

# Book-to-Skill Converter
```

New:
```markdown
---
name: vault-livro-to-skill
description: "Converts books and documents (PDF, EPUB, DOCX, HTML, Markdown, plain text, RTF, MOBI/AZW with Calibre) into structured agent skills, writing them straight into Custódio Neto's Obsidian Cortex Vault. Use when the user wants to study a document, apply an author's frameworks while working, or build a reusable knowledge base from a file, and wants the result to live in 99-system/skills/custom/livros/ with a pointer note in 03-recursos/tecnologia/."
---

<!--
Fork notes (informational; ignored by host agents):
  - This fork targets Claude Code + the Cortex Vault only. It always writes the
    generated skill to the Vault path in Step 5/6, not a probed host-specific root.
  - `allowed-tools` is intentionally omitted; the skill needs Bash (to run
    extract.py) and file read/write — Claude Code will prompt for those on
    first use.
  - Argument hint: <path-to-document-folder-or-glob>... [skill-name-slug]
-->

# Vault Livro-to-Skill Converter
```

- [ ] **Step 2: Update the Step 0 error message**

Current (`SKILL.md:82-86`):
```markdown
## Step 0 — Out-of-scope check

If no arguments are provided, stop and respond:
> "book-to-skill requires a supported document path, folder, or glob pattern. Usage: `book-to-skill <path-to-document-folder-or-glob>... [skill-name-slug]`"
```

New:
```markdown
## Step 0 — Out-of-scope check

If no arguments are provided, stop and respond:
> "vault-livro-to-skill requires a supported document path, folder, or glob pattern. Usage: `/vault-livro-to-skill <path-to-document-folder-or-glob>... [skill-name-slug]`"
```

- [ ] **Step 3: Verify no other literal `book-to-skill` command references remain outside the "Modes of Operation" prose**

Run: `grep -n "book-to-skill" SKILL.md`
Expected: only prose mentions of the upstream project name in the philosophy section (if any), no more usage examples or error strings. Fix any you find using the same substitution as Step 2.

- [ ] **Step 4: Commit**

```bash
git add SKILL.md
git commit -m "rebrand skill as vault-livro-to-skill"
```

---

### Task 3: Replace multi-host destination logic with the fixed Cortex Vault path

**Files:**
- Modify: `SKILL.md:63-80` (delete "Skill Locations" section)
- Modify: `SKILL.md:127-160` (Step 2 `SCRIPT_PATH` probing loop)
- Modify: `SKILL.md:294-324` (Step 5 "Determine skill name")
- Modify: `SKILL.md:327-331` (Step 6 "Create skill directory structure")

**Interfaces:**
- Produces: `SKILLS_HOME` is now always `C:\Users\custo\Dropbox\Obsidian\Cortex\99-system\skills\custom\livros\`, referenced literally by Task 4 and Task 5's new step.

- [ ] **Step 1: Delete the "Skill Locations" section**

Delete `SKILL.md:63-80` in full (the `## Skill Locations` heading through the paragraph ending "...ask the user once and remember the answer for the session — do not silently default.", plus the trailing `---`). Nothing replaces it — Step 5 now states the fixed destination directly.

- [ ] **Step 2: Simplify the `SCRIPT_PATH` probing loop in Step 2**

Current (`SKILL.md:131-147`):
```bash
SCRIPT_PATH=""
for candidate in \
  "$HOME/.copilot/skills/book-to-skill/scripts/extract.py" \
  "$HOME/.agents/skills/book-to-skill/scripts/extract.py" \
  "$HOME/.claude/skills/book-to-skill/scripts/extract.py" \
  ".github/skills/book-to-skill/scripts/extract.py" \
  ".claude/skills/book-to-skill/scripts/extract.py" \
  ".agents/skills/book-to-skill/scripts/extract.py" \
  "$HOME/.config/agents/skills/book-to-skill/scripts/extract.py" \
  "$HOME/.config/amp/skills/book-to-skill/scripts/extract.py"
do
  if [ -f "$candidate" ]; then
    SCRIPT_PATH="$candidate"
    break
  fi
done
```

New:
```bash
SCRIPT_PATH=""
for candidate in \
  "$HOME/.claude/skills/vault-livro-to-skill/scripts/extract.py" \
  ".claude/skills/vault-livro-to-skill/scripts/extract.py"
do
  if [ -f "$candidate" ]; then
    SCRIPT_PATH="$candidate"
    break
  fi
done
```

- [ ] **Step 3: Replace Step 5's destination-selection logic**

Current (`SKILL.md:294-324`):
```markdown
## Step 5 — Determine skill name

If `SKILL_NAME` was provided, use it as the skill slug.
Otherwise, propose two options and let the user choose:
- **By author-concept**: `{author-lastname}-{core-concept}` (e.g. `cialdini-influence`, `meadows-systems`)
- **By title**: lowercase hyphens from book title (e.g. `designing-data-intensive-apps`)

Default to author-concept format if the book has a strong methodological identity.

Choose the destination skill root (`SKILLS_HOME`). Probe the user's filesystem for existing skill homes and pick by **the host the user is running in**:

| Host agent | Personal skill root (probe in order) | Project-local root |
|---|---|---|
| **GitHub Copilot CLI** | `~/.copilot/skills` → `~/.agents/skills` | `.github/skills` → `.claude/skills` → `.agents/skills` |
| **Amp** | `~/.agents/skills` → `~/.config/agents/skills` → `~/.config/amp/skills` | `.agents/skills` |
| **Claude Code** | `~/.claude/skills` | `.claude/skills` |

Selection rules:
1. If **exactly one** of the host's candidate roots exists on disk, use it without asking.
2. If **none** exist (fresh machine), ask the user which root to create — present the host-appropriate options and remember the choice for the session. Do not silently pick.
3. If the user explicitly asked for project-local output, prefer the project-local row.
4. If you cannot identify the host, ask: "Which agent are you running this in — GitHub Copilot CLI, Amp, or Claude Code?"

Set `SKILLS_HOME` to the selected root and check if `$SKILLS_HOME/<skill_name>/` already exists.
If it does, prompt the user to choose:
1. **Update / Fold-in** (Mode 4) — integrate new files/content into the existing skill components.
2. **Overwrite** — delete and regenerate the skill from scratch.
3. **Rename** — append `-2` or use a different custom slug.

If the user selects **Update / Fold-in**, proceed immediately to the **Update / Fold-in Workflow** section after Step 2.5 (skipping Steps 3, 4, 6, 7, 8, 9).
```

New:
```markdown
## Step 5 — Determine skill name

If `SKILL_NAME` was provided, use it as the skill slug.
Otherwise, propose two options and let the user choose:
- **By author-concept**: `{author-lastname}-{core-concept}` (e.g. `cialdini-influence`, `meadows-systems`)
- **By title**: lowercase hyphens from book title (e.g. `designing-data-intensive-apps`)

Default to author-concept format if the book has a strong methodological identity.
The slug MUST be kebab-case — lowercase letters, digits, and hyphens only, matching the rest of the Cortex Vault.

The destination is always fixed — this fork writes only into the Cortex Vault, never a probed host root:

```
SKILLS_HOME = C:\Users\custo\Dropbox\Obsidian\Cortex\99-system\skills\custom\livros\
```

Set `SKILLS_HOME` to that path and check if `$SKILLS_HOME/<skill_name>/` already exists.
If it does, prompt the user to choose:
1. **Update / Fold-in** (Mode 4) — integrate new files/content into the existing skill components.
2. **Overwrite** — delete and regenerate the skill from scratch.
3. **Rename** — append `-2` or use a different custom slug.

If the user selects **Update / Fold-in**, proceed immediately to the **Update / Fold-in Workflow** section after Step 2.5 (skipping Steps 3, 4, 6, 7, 8, 9).
```

- [ ] **Step 4: Update Step 6's `mkdir`**

Current (`SKILL.md:327-331`):
```markdown
## Step 6 — Create skill directory structure

```bash
mkdir -p "$SKILLS_HOME/<skill_name>/chapters"
```
```

New (folder rename to `capitulos` happens together with Task 4 — write it here already so Task 4 doesn't have to touch this line again):
```markdown
## Step 6 — Create skill directory structure

```bash
mkdir -p "$SKILLS_HOME/<skill_name>/capitulos"
```
```

- [ ] **Step 5: Commit**

```bash
git add SKILL.md
git commit -m "fix destination to the Cortex Vault instead of probing multiple hosts"
```

---

### Task 4: Rename `chapters/` to `capitulos/` throughout `SKILL.md`

**Files:**
- Modify: `SKILL.md` (Step 7 heading text and folder path, Step 9's Chapter Index template, Step 9.5, Update/Fold-in Workflow section, Step 10 report)

**Interfaces:**
- Consumes: `SKILLS_HOME` fixed path from Task 3.
- Produces: every generated book skill uses `capitulos/ch<NN>-<slug>.md` (folder in Portuguese, per-file prefix stays `chNN` per user decision).

- [ ] **Step 1: Step 7 — chapter file path**

Current (`SKILL.md:361`):
```markdown
Create `$SKILLS_HOME/<skill_name>/chapters/ch<NN>-<slug>.md` using the structure below.
```

New:
```markdown
Create `$SKILLS_HOME/<skill_name>/capitulos/ch<NN>-<slug>.md` using the structure below.
```

- [ ] **Step 2: Step 9 — Chapter Index template**

Current (`SKILL.md:490-496`):
```markdown
## Chapter Index

| # | Title | Key Frameworks |
|---|-------|----------------|
| [ch01](chapters/ch01-<slug>.md) | <Title> | <framework1>, <framework2> |
| [ch02](chapters/ch02-<slug>.md) | <Title> | <framework1>, <framework2> |
...
```

New:
```markdown
## Chapter Index

| # | Title | Key Frameworks |
|---|-------|----------------|
| [ch01](capitulos/ch01-<slug>.md) | <Title> | <framework1>, <framework2> |
| [ch02](capitulos/ch02-<slug>.md) | <Title> | <framework1>, <framework2> |
...
```

- [ ] **Step 3: Step 9.5 — scanner invocation stays path-generic, no edit needed**

`SKILL.md:521-530` (Step 9.5) calls `scan_generated_skill.py "$SKILLS_HOME/<skill_name>"` — it doesn't hardcode `chapters` itself (the Python script does; that's Task 6). Confirm with:

Run: `grep -n "chapters" SKILL.md`
Expected after Steps 1-2 and Step 4 below: zero remaining matches.

- [ ] **Step 4: Update/Fold-in Workflow section**

Current (`SKILL.md:589-608`, relevant excerpts):
```markdown
### 1. Read Existing Skill Structure
Read and parse the existing skill's files:
- Read `$SKILLS_HOME/<skill_name>/SKILL.md` to parse the existing **Chapter Index**, **Topic Index**, metadata (author, total chapters), and **Core Frameworks**.
- List all files in `$SKILLS_HOME/<skill_name>/chapters/` to find the highest chapter number (e.g. `ch12`).
- Read `$SKILLS_HOME/<skill_name>/glossary.md`, `$SKILLS_HOME/<skill_name>/patterns.md`, and `$SKILLS_HOME/<skill_name>/cheatsheet.md` to see what terms and frameworks are already indexed.

### 2. Match Content & Identify Revisions vs. Additions
Analyze the new extracted text in `<tempdir>/book_skill_work/full_text.txt` to identify if the new content represents:
- **Updates/Revisions to existing chapters**: If a section of the new content directly updates or expands an existing chapter's topic, read the existing chapter file, merge the new details into it, and rewrite the file.
- **New additions**: If the content introduces new chapters, papers, or separate sections, create **new chapter summary files** under `chapters/`. Start numbering these files after the highest existing chapter number (e.g. if the existing chapters stop at `ch12`, create `ch13-*.md`, `ch14-*.md`, etc.).

### 3. Generate or Update Chapter Summary Files
For each new or revised chapter:
- Read the corresponding section of the extracted new text.
- Follow the formatting guidelines in **Step 7** to build the summary.
- Write/update the file in `$SKILLS_HOME/<skill_name>/chapters/`.
```

New (only the three `chapters/` occurrences change):
```markdown
### 1. Read Existing Skill Structure
Read and parse the existing skill's files:
- Read `$SKILLS_HOME/<skill_name>/SKILL.md` to parse the existing **Chapter Index**, **Topic Index**, metadata (author, total chapters), and **Core Frameworks**.
- List all files in `$SKILLS_HOME/<skill_name>/capitulos/` to find the highest chapter number (e.g. `ch12`).
- Read `$SKILLS_HOME/<skill_name>/glossary.md`, `$SKILLS_HOME/<skill_name>/patterns.md`, and `$SKILLS_HOME/<skill_name>/cheatsheet.md` to see what terms and frameworks are already indexed.

### 2. Match Content & Identify Revisions vs. Additions
Analyze the new extracted text in `<tempdir>/book_skill_work/full_text.txt` to identify if the new content represents:
- **Updates/Revisions to existing chapters**: If a section of the new content directly updates or expands an existing chapter's topic, read the existing chapter file, merge the new details into it, and rewrite the file.
- **New additions**: If the content introduces new chapters, papers, or separate sections, create **new chapter summary files** under `capitulos/`. Start numbering these files after the highest existing chapter number (e.g. if the existing chapters stop at `ch12`, create `ch13-*.md`, `ch14-*.md`, etc.).

### 3. Generate or Update Chapter Summary Files
For each new or revised chapter:
- Read the corresponding section of the extracted new text.
- Follow the formatting guidelines in **Step 7** to build the summary.
- Write/update the file in `$SKILLS_HOME/<skill_name>/capitulos/`.
```

- [ ] **Step 5: Commit**

```bash
git add SKILL.md
git commit -m "rename generated chapters/ folder to capitulos/"
```

---

### Task 5: Add the Cortex pointer-note step and update the final report

**Files:**
- Modify: `SKILL.md` (insert new `## Step 9.6 — Generate the Cortex pointer note` between Step 9 and Step 9.5; update Step 10's report text)

**Interfaces:**
- Consumes: `<Full Title>`, `<Author(s)>`, `<skill_name>` (book slug) from Step 9; `SKILLS_HOME` from Task 3.
- Produces: `03-recursos/tecnologia/{YYYY-MM-DD}-{skill_name}.md` in the Vault, and an updated `_index.md` row for it.

- [ ] **Step 1: Insert the new step right after Step 9 (master `SKILL.md`) and before Step 9.5 (scan)**

Insert this whole section into `SKILL.md`, right before the existing `## Step 9.5 — Scan the generated skill` heading:

```markdown
## Step 9.6 — Generate the Cortex pointer note

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

- [ ] **Step 2: Update the Step 10 final report**

Current (`SKILL.md:554-585`, relevant excerpt):
```markdown
Then report to the user:

```
✅ Skill created: $SKILLS_HOME/<skill_name>/

📚 Book: <Full Title> — <Author>
📄 Pages: ~<N> | Chapters: <N>

Files generated:
  SKILL.md         — core frameworks + index   (~X tokens)
  chapters/        — <N> chapter summaries     (~X tokens each, ~X total)
  glossary.md      — key terms                 (~X tokens)
  patterns.md      — techniques & patterns     (~X tokens)
  cheatsheet.md    — quick reference           (~X tokens)
  ─────────────────────────────────────────────────────
  Total skill size: ~X tokens (loaded on-demand, not all at once)

💡 Tip: check your agent's session cost/usage command to see actual token usage.

Usage:
  Ask for <skill_name>                  → load core frameworks
  Ask <skill_name> about <topic>        → find and explain a topic
  Ask <skill_name> for ch<N>            → dive into a specific chapter

Reload (if your agent doesn't auto-detect new skills):
  GitHub Copilot CLI:  /skills reload
  Claude Code:         restart the session
  Amp:                 restart the session

Share this skill (Copilot ecosystem, optional):
  gh skill publish $SKILLS_HOME/<skill_name>
```
```

New:
```markdown
Then report to the user:

```
✅ Skill created: 99-system/skills/custom/livros/<skill_name>/
✅ Pointer note: 03-recursos/tecnologia/<YYYY-MM-DD>-<skill_name>.md

📚 Book: <Full Title> — <Author>
📄 Pages: ~<N> | Chapters: <N>

Files generated:
  SKILL.md         — core frameworks + index   (~X tokens)
  capitulos/       — <N> chapter summaries     (~X tokens each, ~X total)
  glossary.md      — key terms                 (~X tokens)
  patterns.md      — techniques & patterns     (~X tokens)
  cheatsheet.md    — quick reference           (~X tokens)
  ─────────────────────────────────────────────────────
  Total skill size: ~X tokens (loaded on-demand, not all at once)

💡 Tip: check your agent's session cost/usage command to see actual token usage.

Usage:
  Ask for <skill_name>                  → load core frameworks
  Ask <skill_name> about <topic>        → find and explain a topic
  Ask <skill_name> for ch<N>            → dive into a specific chapter

If the slash command doesn't appear yet, restart the Claude Code session
(or run `sync-commands` in PowerShell if the skill still doesn't show up —
see the note in Task 1's staging doc about nested skill discovery).
```
```

- [ ] **Step 3: Commit**

```bash
git add SKILL.md
git commit -m "add Cortex pointer-note generation step and update final report"
```

---

### Task 6: Rename the hardcoded `chapters` folder check in the security scanner

**Files:**
- Modify: `tools/scan_generated_skill.py:137-141`
- Test: `tests/test_scan_generated_skill.py`

**Interfaces:**
- Consumes: a skill directory path (unchanged signature: `scan_generated_skill(path: Path) -> list[Finding]`).
- Produces: same `Finding` list shape; now looks for `capitulos/*.md` instead of `chapters/*.md`.

- [ ] **Step 1: Read the existing test file to see current fixture layout**

Run: `grep -n "chapters" tests/test_scan_generated_skill.py`
Expected output (from earlier mapping):
```
19:    chapters = root / "chapters"
20:    chapters.mkdir(parents=True)
33:    (chapters / "ch01.md").write_text(
112:    (skill / "chapters" / "ch01.md").write_text(
131:    (skill / "chapters" / "ch01.md").write_text(
158:    (skill / "chapters" / "ch01.md").write_text(
181:    escaped = scanner._terminal_safe("chapters/ch01\x1b[31m.md")
```

- [ ] **Step 2: Update the test fixtures to use `capitulos` and verify they fail against the current (unpatched) scanner**

Edit every one of the six lines above in `tests/test_scan_generated_skill.py`, replacing the literal `"chapters"` folder name with `"capitulos"` (the `_terminal_safe("chapters/ch01\x1b[31m.md")` line at 181 is just exercising the escaping helper on an arbitrary string — change it too for consistency: `_terminal_safe("capitulos/ch01\x1b[31m.md")`).

Run: `python -m pytest tests/test_scan_generated_skill.py -v`
Expected: the tests that create a `capitulos/ch01.md` fixture and then assert `scan_generated_skill` reports its findings will FAIL, because `scan_generated_skill.py:137` still looks for a folder literally named `chapters`.

- [ ] **Step 3: Fix `tools/scan_generated_skill.py`**

Current (`tools/scan_generated_skill.py:137-141`):
```python
    chapters = root / "chapters"
    if chapters.exists():
        if chapters.is_symlink() or not chapters.is_dir():
            raise ScanError("chapters must be a real directory, not a symbolic link")
        candidates.update(chapters.glob("*.md"))
```

New:
```python
    chapters = root / "capitulos"
    if chapters.exists():
        if chapters.is_symlink() or not chapters.is_dir():
            raise ScanError("capitulos must be a real directory, not a symbolic link")
        candidates.update(chapters.glob("*.md"))
```

(The local variable name `chapters` can stay — it's an internal identifier, not user-facing; only the literal folder name string and the error message text change.)

- [ ] **Step 4: Run the tests again to confirm they pass**

Run: `python -m pytest tests/test_scan_generated_skill.py -v`
Expected: PASS, all tests green.

- [ ] **Step 5: Run the full test suite to make sure nothing else broke**

Run: `python -m pytest -v`
Expected: PASS.

- [ ] **Step 6: Commit**

```bash
git add tools/scan_generated_skill.py tests/test_scan_generated_skill.py
git commit -m "rename scanner's expected chapter folder from chapters/ to capitulos/"
```

---

### Task 7: Update `README.md` — fork notice, attribution, Cortex-specific usage

**Files:**
- Modify: `README.md`

**Interfaces:**
- None (documentation only).

- [ ] **Step 1: Add a fork notice right after the title/badges block, before the "How it works, in 3 steps" line**

Insert this new section right before `README.md:46` (`**How it works, in 3 steps:**`):

```markdown
> **This is a personal fork.** [`cortex-book-skill`](https://github.com/custodioneto/cortex-book-skill)
> adapts the upstream [`book-to-skill`](https://github.com/virgiliojr94/book-to-skill)
> project (MIT-licensed, © 2025 virgiliojr94) for one specific setup: Claude Code
> plus a personal Obsidian "Cortex" Vault organized with the PARA method. The
> extraction engine and skill-generation logic are unchanged; what's different is
> *where* the generated skill lands and *what command* triggers it — see
> [Usage](#-usage) below. If you don't use that exact setup, use the upstream
> project instead — it supports GitHub Copilot CLI, Amp, and any filesystem
> layout, none of which this fork does anymore.
```

- [ ] **Step 2: Replace the "3 steps" summary and slash-command examples**

Current (`README.md:46-51`):
```markdown
**How it works, in 3 steps:**

1. **Point** it at a file, folder, or glob — `/book-to-skill ./my-book.pdf`
2. **It distills** the book into a skill — frameworks, decision rules, anti-patterns, and per-chapter files. Structure, not a summary.
3. **Your agent loads it on demand** — ask `/my-book replication` and it reads the right chapter and answers from the real content, no hallucination.
```

New:
```markdown
**How it works, in 3 steps:**

1. **Point** it at a file, folder, or glob — `/vault-livro-to-skill ./my-book.pdf`
2. **It distills** the book into a skill — frameworks, decision rules, anti-patterns, and per-chapter files. Structure, not a summary. The skill lands in `99-system/skills/custom/livros/<slug>/` in the Cortex Vault, plus a short pointer note in `03-recursos/tecnologia/`.
3. **Your agent loads it on demand** — ask `/my-book replication` and it reads the right chapter and answers from the real content, no hallucination.
```

- [ ] **Step 3: Update the "What it generates" section**

Current (`README.md:71-84`):
```markdown
## 📦 What it generates

Running `/book-to-skill your-book.pdf` (or a folder, glob, or list of files) creates a full skill in your agent's skills directory (`~/.copilot/skills/<slug>/` for Copilot CLI, `~/.agents/skills/<slug>/` for Amp or cross-agent, `~/.claude/skills/<slug>/` for Claude Code):

| File | Purpose | Size |
|------|---------|------|
| `SKILL.md` | Core mental models + chapter index | ~4,000 tokens |
| `chapters/ch01-*.md` … | One file per chapter, loaded on-demand | ~1,000 tokens each |
| `glossary.md` | Every key term, alphabetically sorted with chapter refs | ~1,500 tokens |
| `patterns.md` | All techniques, algorithms, and design patterns | ~2,000 tokens |
| `cheatsheet.md` | Decision tables and quick-reference rules | ~1,000 tokens |

**Chapter files are loaded on-demand** — they don't count against the skill budget until you ask about that topic.
```

New:
```markdown
## 📦 What it generates

Running `/vault-livro-to-skill your-book.pdf` (or a folder, glob, or list of files) creates two artifacts in the Cortex Vault:

**a) A full skill** at `99-system/skills/custom/livros/<slug>/`:

| File | Purpose | Size |
|------|---------|------|
| `SKILL.md` | Core mental models + chapter index | ~4,000 tokens |
| `capitulos/ch01-*.md` … | One file per chapter, loaded on-demand | ~1,000 tokens each |
| `glossary.md` | Every key term, alphabetically sorted with chapter refs | ~1,500 tokens |
| `patterns.md` | All techniques, algorithms, and design patterns | ~2,000 tokens |
| `cheatsheet.md` | Decision tables and quick-reference rules | ~1,000 tokens |

**b) A pointer note** at `03-recursos/tecnologia/<YYYY-MM-DD>-<slug>.md` — a lean, dated note linking back to the skill above, with a manually-maintained section for logging where the knowledge got applied in real projects.

**Chapter files are loaded on-demand** — they don't count against the skill budget until you ask about that topic.
```

- [ ] **Step 4: Trim the "How it works" pipeline diagram's multi-host destination lines**

Current (`README.md:203-208`):
```
          Skill written to one of:
            ~/.copilot/skills/<slug>/   (GitHub Copilot CLI)
            ~/.agents/skills/<slug>/    (Copilot CLI or Amp, cross-agent)
            ~/.claude/skills/<slug>/    (Claude Code)
          /tmp/book_skill_work/         🗑️  cleaned up
```

New:
```
          Skill written to:
            99-system/skills/custom/livros/<slug>/   (Cortex Vault)
          Pointer note written to:
            03-recursos/tecnologia/<date>-<slug>.md  (Cortex Vault)
          /tmp/book_skill_work/         🗑️  cleaned up
```

- [ ] **Step 5: Replace the "Install" section**

Current (`README.md:350-406`) covers GitHub Copilot CLI, the cross-agent path, Claude Code, and the standalone pip CLI. Replace the whole section (from `## 📥 Install` through the end of the `### Standalone CLI (pip)` subsection, i.e. `README.md:350-406`) with:

```markdown
## 📥 Install

This fork only supports **Claude Code**, cloned into your personal skills folder:

```bash
git clone https://github.com/custodioneto/cortex-book-skill.git ~/.claude/skills/vault-livro-to-skill
```

Then, in any Claude Code session:

```bash
/vault-livro-to-skill ~/path/to/your-book.pdf
# or
/vault-livro-to-skill ~/path/to/your-book.epub
```

The generated skill and pointer note land directly in the Cortex Vault (see [What it generates](#-what-it-generates)) — there's no separate `SKILLS_HOME` to choose.

### Standalone CLI (pip)

The extraction engine can still be installed and used standalone — this part is untouched from upstream and has nothing Cortex-specific about it:

```bash
pip install "book-to-skill[pdf,epub,docx]"   # engine + optional extractors
book-to-skill ~/path/to/book.pdf --mode text  # or: python -m book_to_skill ...
book-to-skill --check                          # report which extractors are installed
```

This does **not** register the `/vault-livro-to-skill` agent skill — use the `git clone` above for that.
```

- [ ] **Step 6: Verify no stale references remain**

Run: `grep -n "book-to-skill" README.md`
Expected: only mentions of the upstream project name/URL (attribution, standalone pip package name) remain — no more `/book-to-skill` slash-command examples and no more Copilot CLI / Amp install instructions.

- [ ] **Step 7: Commit**

```bash
git add README.md
git commit -m "update README for the Cortex-specific fork"
```

---

### Task 8: Final verification pass

**Files:** none modified; read-only checks.

- [ ] **Step 1: Run the full test suite**

Run: `python -m pytest -v`
Expected: PASS.

- [ ] **Step 2: Validate the rebranded `SKILL.md` against the Claude Code lens**

Run: `python tools/validate_skill.py --lens claude SKILL.md`
Expected: `✓ SKILL.md [Claude Code]: no Claude Code-breaking issues (N warning(s))` — the `name: vault-livro-to-skill` frontmatter must pass the `[a-z0-9-]+` / ≤64-char / no-reserved-word checks in `tools/validate_skill.py:140-146`.

- [ ] **Step 3: Confirm no leftover `chapters/` or old command references anywhere in the changed files**

Run: `grep -rn "chapters" SKILL.md tools/scan_generated_skill.py README.md`
Expected: zero matches (the word `chapters_detected` in `book_to_skill/utils.py` is untouched and out of scope — it's a book-structure-detection field name, unrelated to the output folder name).

Run: `grep -n "/book-to-skill" SKILL.md README.md`
Expected: zero matches for the slash-command form; upstream project name/URL mentions are fine.

- [ ] **Step 4: Report to the user**

Summarize: staging doc awaiting approval (Task 1), `SKILL.md` rebranded and pointed at the fixed Vault path, `capitulos/` rename done in both prose and the scanner, README updated. Ask whether to proceed to a real end-to-end test run (`/vault-livro-to-skill` against a real book) once the staging doc is approved.
