---
name: embedded-project-memory
description: >-
  Owns the structure memory of embedded projects: creates the team-standard
  project skeleton from a project name (numbered dirs 1.代码/Main_project,
  2.原理图PCB, 3.APP或上位机, 4.相关资料 with 需求或功能/相关技术资料/技术分析
  subdirs, plus git init), and maintains lightweight pointer-based project
  memory (.agents/project-map.md, current-state.md) for incremental work in
  existing STM32/Keil/Raspberry Pi/embedded repositories. Use when the user
  says 新建工程、创建工程、初始化工程、建个工程、工程模板、开新项目、new
  project, or mentions 项目记忆、项目结构、子工程、目录结构、.agents 记忆, or
  when starting work in a repo where future tasks should not re-scan the
  whole repo. Do NOT use for pure hardware analysis, debugging, or
  tool-building in a repo that already has memory — those tasks just read
  the two memory files and proceed.
metadata:
  short-description: Create & maintain pointer-based project memory for embedded repos
  agent_created: true
---

# Embedded Project Memory

Own the structure memory of an embedded project for its whole lifecycle:
create the team-standard skeleton from a project name (Mode 1), then keep
small, pointer-based memory maintained across tasks (Mode 2). Memory stores
WHAT and WHERE; facts stay in code and project docs — except the few
high-frequency facts that get short, dated copies (see Pointer Rule).

**Lightweight path (most tasks).** Triggering this skill does NOT mean
running its full procedures. Once memory exists, an ordinary task only reads
the two memory files, does the work, and updates them if (and only if) an
update trigger from **Memory Maintenance** fires. Never re-run the
first-entry survey for routine debug/hardware/tool tasks.

## Standard Layout (team convention)

ALWAYS use this exact structure when creating:

```
<工程名>/
├── 1.代码/
│   └── Main_project/
├── 2.原理图PCB/
├── 3.APP或上位机/
└── 4.相关资料/
    ├── 1.需求或功能/
    ├── 2.相关技术资料/
    └── 3.技术分析/
```

Dir names are fixed: Chinese + numeric prefix, exactly as above. Do not
translate, reorder, or "improve" them.

Known legacy variants in older projects (recognize, never rename):
`2.原理图和PCB`, host tools under `1.代码/`, top-level `4.技术分析文档/`.

## Mode 1 — Creating a New Project

1. Confirm inputs: project name (keep the user's exact format, e.g.
   SLV-270-人员流量设备) and parent directory (default to the team project
   root when it exists on this machine, e.g. `E:\YQ Project\`; otherwise ask
   the user).
2. Single-level listing of the parent directory (never recursive). Check:
   (a) exact same name exists → **STOP and ask**, never overwrite or merge;
   (b) a near-collision exists (e.g. same number, different spelling:
   SLV-270 vs SLV270) → show the user the similar names and confirm before
   creating.
3. Create the tree exactly as above. Do not create any additional
   directories. Because git does not track empty dirs, put an empty
   `.gitkeep` file in each of the six leaf dirs (Main_project, 2.原理图PCB,
   3.APP或上位机, and the three 4.相关资料 subdirs) so the skeleton survives
   clone/push.
4. By default (no need to ask): run `git init` in the project root (use the
   team's usual default branch name; ask if unsure). Do NOT
   create `.agents/` at skeleton time — there are no real facts yet, and a
   placeholder full of TBD/pending would fake the "memory already exists"
   signal and cause the real first-entry survey to be skipped later. Memory
   is born from the first real task (Mode 2), when there is something true
   to record. The skeleton itself marks that the project was initialized.
5. Report the created tree and where each kind of content belongs.

## Mode 2 — First Entry / Memory Bootstrap (Existing Project)

Run this on the first real task in a project (firmware setup, a code fix, a
hardware review — anything that produces facts). This is where memory is
born. Bounded survey, then write ONLY the two default files:

1. Read project `AGENTS.md` (may not exist — skip silently) and any existing
   `.agents/*.md`. Register existing context docs (HANDOFF.md,
   PROJECT_CONTEXT.md, ...) as pointers — never merge or rewrite them. If
   one has become a dated log, point it out to the user; do not refactor it
   silently.
2. Identify sub-projects: Keil `*.uvprojx`, Make/CMake, `package.json`,
   Python entries, host-tool solutions. One row per sub-project.
3. Identify no-go zones: vendor/data dirs that look huge by name or by a
   single-level listing (e.g. 相关资料/SDK, 厂商资料). Mark the scale as an
   estimate ("~10k files, do not recurse") — do NOT count files recursively
   to get an exact number. Access rule: filename search only.
4. Write `.agents/project-map.md` + `.agents/current-state.md` from the
   templates below, filled with what the survey actually found. Create
   `file-index.md` / `decisions.md` only when genuinely needed and say so.

### Project-Map Template

```markdown
# Project Map

## Root

- Path / type / numbering scheme of top dirs

## Sub-projects

| Sub-project | Purpose (one line) |

## Build & Flash Quick Facts

<!-- High-frequency facts, short dated copies. Source stays the authority;
     update the copy whenever you touch the source. -->

- <sub-project>: <build/flash command or current version>
  (source: <path>; verified: <YYYY-MM-DD>)

## No-Go Zones

| Dir | Contents | Estimated scale | Access rule (filename search only) |

## Hardware Pointers

- PCB version doc / pin map / schematic file: <path>

## Existing Docs

- <doc>: <what it is> (authority: its own)
```

`.agents/current-state.md` (one section per ACTIVE workstream):

```markdown
# Current State

## <workstream name>

- Task: <one line>
- Branch / rollback point: <refs>
- Last validation: <command + result>
- Next step: <one line>
```

Keep each memory file under ~80 lines target, ~150 hard ceiling.

### Pointer Rule (split by frequency × volatility)

- **Low-frequency, stable facts** (pin tables, protocol specs, datasheet
  locations, doc ownership) → pointers only. Never copy them into memory.
- **High-frequency facts that work touches every task** (build command +
  success check, flash/download tool entry, current firmware/algorithm
  version) → short inline copies (~5–10 lines) in **Build & Flash Quick
  Facts**, each marked with its source path and a last-verified date.
  Update the copy whenever you touch the source; the source stays the
  authority.
- **Never write a pointer whose target you have not verified exists.** Many
  sub-projects have no AGENTS.md/README (bare test projects, tool dirs) —
  if no home exists for a fact, inline it in Quick Facts, or ask the user
  where to put it. A dangling pointer is worse than no pointer.

## Document Placement Rules

The three resource subdirs are separated by WHO produced the document:

- Customer-provided requirements / feature overviews → `4.相关资料/1.需求或功能/`
- Existing vendor documents (datasheets, manuals, SDK dumps; whether found
  by engineers or downloaded by AI) → `4.相关资料/2.相关技术资料/`
- Documents the AI or the team GENERATES (analysis, comparison, notes)
  → `4.相关资料/3.技术分析/` — never into `1.代码/`, the project root, or the
  other two resource dirs.

This rule applies for the whole project lifetime, not just creation day.

## Memory Maintenance (do not over-maintain)

Update triggers:

- Code edit, bug fix, parameter tweak → nothing, at most one status line.
- Sub-project or dir added/removed/renamed → one map row.
- Build/flash command or version changed → update its Quick Facts copy +
  last-verified date (you are touching the source anyway).
- PCB/hardware version change → one hardware pointer row.
- Decision on approach/protocol/algorithm version → decisions.md (create it
  then).

`current-state.md` hygiene: one section per ACTIVE workstream. When a
workstream finishes, DELETE its section — never archive finished sections;
accumulated status is how memory files rot.

Compact when any memory file exceeds ~150 lines or 6 KB: merge duplicates,
drop finished state, keep pointers and durable facts. During compaction,
also delete Quick Facts rows of dead sub-projects (finished test projects,
removed tools). Never let a memory
file become a conversation log; report what was removed when compacting.

## Later Tasks

Read the two memory files + project `AGENTS.md` (if present). Select files
by keyword, pointer, and direct dependency. Expand only on evidence: memory
stale, cross-module impact, refactor/review task, or a failing build the
index cannot explain. Never recursively scan a no-go zone; search by
filename there.

**Quick Facts self-heal:** if a build/flash command copied in **Build &
Flash Quick Facts** fails, suspect the copy is stale FIRST — verify against
the source, update the copy and its last-verified date, then retry — before
blaming the code.

## Embedded Rules

- Memory files are UTF-8 (they contain Chinese dir/sub-project names).
  Never write them with ANSI/GBK-default tools.
- Encoding / Keil build rules: follow the user's global MEMORY.md and the
  project's `AGENTS.md` where they exist; ask when neither covers the
  platform.
- Hardware/timing claims: consult datasheet/vendor docs (no-go zone pointers
  say where) before stating conclusions.
- Validation: after edits run the smallest meaningful build/validation for
  the touched sub-project (Keil CLI, py_compile, dataset replay). Update
  current-state only if the result matters for future tasks.

## Do NOT

- Do not overwrite, reuse, or merge into an existing directory. Stop and ask.
- Do not generate chip-specific Keil/IAR/CubeMX project files — firmware
  project setup is a separate task that needs a real MCU part number.
- Do not create `.agents/` placeholder files full of TBD/pending/none —
  memory is born from real facts (Mode 2), never from the skeleton; an empty
  memory fakes the "already surveyed" signal.
- Do not create extra "useful" dirs (docs/, output/, tools/) beyond the
  standard tree, git init, and the six .gitkeep files.
- Do not recursively scan vendor/datasheet dirs or the parent directory.
- Do not copy low-frequency stable facts into memory — point to them, and
  verify the pointer target exists first.
- Do not merge with or rewrite existing HANDOFF/PROJECT_CONTEXT documents.
- Do not write conversation logs into .agents/, and do not produce any
  documentation as a side effect.
- Do not run first-entry procedures when real memory already exists
  (lightweight path applies instead).
