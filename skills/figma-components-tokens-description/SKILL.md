---
name: figma-components-tokens-description
description: Writes and improves descriptions in a Figma design-system library — component sets, variants, private components and, optionally, color variables — through a gated, owner-approved workflow (inventory, scope approval, wording review, write, completeness sweep). Use whenever the user wants to add, audit or fix descriptions in a Figma library, or prepare a library for AI and the Figma agent — e.g. "add descriptions to my components", "document my design system", "prepare my design system for AI agents", "dodaj opisy komponentów" — even without the word "description" ("make my library understandable for the Figma agent"). Not for creating variants (use pre-variant-check).
---

# Writing component descriptions for a Figma library

**How to use this skill:** start at **§6 Workflow** and follow its steps in order — each step ends with a **Done when** line, and the next step starts only once it holds. Sections 1–5b are the rules you check while drafting; §7–11 are reference.

**TL;DR:** A component description answers three questions: *what is this*, *what can I configure*, *is this the right pattern for this use case*. The highest-value section is **"Not for"** — it says what NOT to build with the component. Describe sets and private components always; variants only when the name doesn't explain itself. No screen names, no property dumps, no guessing — mark `Needs owner input` and ask. The workflow is inventory → scope approval → drafting → wording review → injection → sweep — no step is optional. At the start, ask whether to describe components only or components + colors; colors follow the same workflow with their own short rules (§5b).

Purpose: produce component descriptions that work triple duty — teach designers, feed Dev Mode, and give AI agents (`search_design_system`) the context they need. Distilled from documenting a production mobile design system (~80 components), verified against Figma's post-Config-2026 documentation.

---

## 1. What a description is (and is not)

A description communicates **intended use** and doubles as **search keywords** — Figma includes description text in search results for both people and AI. It answers three questions in order: *what is this?*, *what can I configure?*, *is this the correct pattern for this type of use case?*

It is NOT: a spec sheet (anatomy, tokens, versions → external docs via the link field), a changelog, or a map of the product.

## 2. Structure template

Write on the **component set**, in English, using this fixed skeleton (omit empty sections). Settle the description language with the owner in workflow step 1 (ecosystem map): English is the default — best compatibility with `search_design_system`, Dev Mode and AI agents parsing descriptions as context. If the team requires another language: write the prose (purpose, rules, not-for) in that language, but keep prop names, variant values and cross-referenced component names in English — they are the component's API, not documentation, and must match what developers see and what `componentPropertyDefinitions` returns. Backticked references (`ListRow`, `Behavior=Navigate`) stay English regardless of prose language.

```
One-sentence purpose. Composition note if the component wraps others.

**Options:**
- `Prop=Value` — meaning (bullet list when 2+ options; single line when trivial)

**Rules:** constraints, traps, non-obvious behaviors.

**Not for:** the wrong use case — use `CorrectComponent` instead.
```

- **"Not for" is the highest-value section** and the most often skipped. Add it wherever a confusable pair exists (read-only vs interactive row, modal vs toast, empty vs error state, inline vs docked CTA, section tabs vs in-section segmented control).
- **Options is not a property dump.** Enumerate values that represent meaningful usage choices (e.g. `Behavior=Navigate/Select/Expand`); short self-explanatory axes may stay as a single compact line, but do not exhaustively list purely visual states, mechanical values or full property combinations (State × Size × Theme matrices). *Explain* selectively — only where meaning, interaction or constraint isn't obvious from the name. `ShowIcon=True/False` needs no gloss; `Dismissible=False — prevents closing the modal without completing the required decision` earns its words. Skip trivial booleans entirely.
- Explain the **why**, not just the what — articulate the knowledge designers carry in their heads.
- Cross-references to other components are stable library facts: encouraged.
- **Editing rule for length:** no hard limits anywhere — length follows complexity, not component type or visibility. Instead, remove any sentence that repeats the component name, restates an obvious property, or does not change a usage decision.

## 3. Hard prohibitions

Each rule leads with what to write; the prohibition in brackets is the guardrail.

- **Describe where to use it by domain, not by file location (no artboard, frame, page or screen names).** They rot on the first rename and map the library to one product's IA. Wrong: "used on the Checkout screen". Right: "for order-summary views".
  - The line: **file artifacts** (artboard/page/node names, usage counts) = never; **domain context** ("payment schedule", "order status", "trip attribution") = encouraged — that IS "where to use it" and it's searchable.
- **Explain variants by what they contain or do (no variant→screen lists)** in set descriptions. Variant names should carry their own meaning; explain them functionally (by what they contain/do), not by where they appear.
- **Write only what is specific to this component (no generic platform guidance)** duplicated into descriptions. It dilutes component-specific search context, goes stale independently of the library, and belongs in linked platform documentation. Exception: a one-line **pattern anchor** compresses known behavior cheaply ("follows the iOS swipe-actions pattern"). Anchor, don't paste. For deeper platform docs use the description **link field**, not the text.
- **Work search terms into full sentences (no keyword lists or stuffing).** Figma itself recommends adding relevant keywords to descriptions because they are used in search — this rule agrees with that, it only governs the form. Include common domain terms and aliases naturally within complete sentences. Never append standalone keyword lists or repeat synonyms solely to influence search ranking — descriptions must stay readable prose, not SEO metadata.
- **Write only finished, approved text into the library (no placeholders).** `Needs owner input`, TODOs and unresolved questions are review statuses that live in the review report, never description content — do not inject them into `descriptionMarkdown`.

## 4. Three levels: set, variant, private

- **Set description** = purpose, composition, prop semantics, rules, not-for.
- **Variant descriptions** = only where the variant name doesn't explain itself: hidden logic, functional (not just visual) difference, non-obvious configuration effects. Do NOT describe every combination (`Variant=Primary, State=Default, Size=Medium` needs nothing). Expect an AI reviewer to flag "most variants undocumented" — that's by design; document the *criterion*, not every node.
- **Private (dot-prefixed) components**: every private component must have a description — no exceptions; one undescribed private makes the convention ambiguous (intentional or accidental?). Keep it concise when its only purpose is to identify the parent and prevent direct instantiation. Add further context whenever it has non-obvious behavior, configuration constraints or an important role in the parent's composition. Length follows complexity, never the dot prefix.

## 5. Formatting (descriptionMarkdown)

The Plugin API field `descriptionMarkdown` supports rich text that renders in the panel:
- Inline code (backticks) on every prop, variant and referenced component name — renders as chips.
- Bulleted lists for Options with 2+ entries.
- Bold section labels (`**Options:**`, `**Rules:**`, `**Not for:**`). No headings — the panel is narrow.
- Links: markdown links (`[text](url)`) render as clickable; raw URLs render as plain text — if a link belongs inline, use markdown syntax. For external documentation prefer the description's separate **Link field** (`documentationLinks` in the Plugin API — writable programmatically) over inline links. The API currently supports setting only **one** documentation link per component — if several sources matter, link the main one and mention the rest inline.
- Line breaks: a single newline is ignored (standard Markdown); a double newline starts a paragraph. Use bullet points for lists, never manual line breaks.
- Soft readability threshold: ~150 words. Beyond that the panel scrolls and the description stops being scannable. Not a hard limit — complex components (5+ behavioral variants) may exceed it — but when an ordinary component runs longer, check it for property dumps or repetition first.
- Verify rendering on ONE component first, then batch (verified: bold, inline code, bullets and markdown links all render in the panel).

## 5b. Color variable descriptions (when the owner chose components + colors)

Colors are described in the same pass, with the same gates (inventory, scope approval, wording review, sweep). Only the writing rules differ, because a color's job is simpler than a component's.

**Which colors get a description:**
- **The system splits primitives and semantics** → describe the semantic colors; skip the primitives. Primitives are raw values nobody should pick directly, so a description adds nothing. List them in the inventory as `Skip — approved reason: raw palette values`.
- **No split** → every color is in direct use, so every color gets a description.
- **How to tell which case you are in:** semantic colors usually alias another variable; primitives usually hold a raw value. Check this during the crawl, report the result in the inventory, and let the owner correct it before anything is written — collection names and naming habits vary too much to decide silently.

**What a color description says** — one or two sentences:
- What the color is for (its role), and when *not* to use it, with the right alternative. Example: `Background for cards and sheets that sit on top of the page. Not for the page background itself — use bg/page.`
- Domain context is welcome ("for overdue payment states"); file artifacts are not — the §3 prohibitions apply to colors too (no screen names, no keyword lists, no placeholders).
- Do not restate the value or the mode values (hex, light/dark) — Figma already shows them, and they go stale.

**Format:** plain text. Variables expose only a plain `description` (there is no Markdown variant), so the bold labels, bullets and chips from §5 do not apply.

## 6. Workflow

1. **Ecosystem map — ask the owner before touching any file.** A design system usually spans several files, and the library cannot be documented safely knowing only one of them. Ask five short questions:
   1. *Which files make up this design system? Paste links to all of them* — typically: a component library ("App Name Components" — the app's components, where descriptions are written), a foundations file ("App Name Core" — icons, variables, text styles, logo), and a product file ("App Name Designs" — the app's artboards/screens).
   2. *Which files am I documenting, and which are reference-only?* (e.g. "write descriptions in the library; foundations read-only; product file only as usage context")
   3. *What is the product and who is it for — one sentence.* (also: target platform)
   4. *Is there anything to exclude?* — e.g. an Archive page, experiments/concepts in progress, another team's components, external UI kits (Apple/Material). Everything named here lands in the inventory as an explicit, owner-approved skip.
   5. *Do you want descriptions for components only, or for components and colors?* If colors are included, the file that holds the color variables (usually the foundations file) becomes writable **for color descriptions only** — everything else in it stays reference-only.

   Any file the descriptions will reference (e.g. an icon library named in swap-slot text) must be either inspected or listed in the inventory as *referenced, out of scope* with a reason — never silently assumed.

   **Done when:** all five questions have an answer — or, if the owner cannot be asked, an assumed answer written into the report — and every file the descriptions will reference is either inspected or listed as out of scope.
2. **Acquire domain context from cheap sources before asking domain questions.** In order of cost: (a) the structural crawl described in step 3 — always first, everything else builds on it; (b) usage crawl of the product file — real text overrides teach the product's vocabulary faster than anything, and instance contexts give "where to use it"; (c) screenshots of key product screens — rendered UI is the fastest way to understand what the app does; (d) whatever documentation already exists in the files: old descriptions, Dev Mode annotations, README/Guidelines/cover pages; (e) public sources if the product is public (website, app store listing, help center). Owner questions come last and only for what none of the above resolved.

   **Done when:** each source (a)–(e) has been used or marked unavailable, and the only open domain questions left are ones none of them could answer.
3. **Structural crawl before writing.** Read `componentPropertyDefinitions` (variants, booleans, TEXT props, INSTANCE_SWAP slots) and internal composition (nested instances) per component. Never infer meaning from names alone — names are historical, abbreviated, inherited and sometimes wrong. Usage crawl of the product file is secondary context, not the source of truth. During the crawl also check whether the library has `_example`-suffixed components or an `Examples` page — record presence/absence for the inventory.

   Technical traps that silently corrupt a crawl:
   - **Load all pages first** (`figma.loadAllPagesAsync()`). Only the current page is guaranteed to be loaded; a file-wide search without it misses components on other pages and the inventory looks complete when it is not.
   - **Read property definitions from the set or a standalone component, never from a variant.** Definitions belong to the set; reading them per variant fails or gives a partial picture.
   - **Property keys carry a hidden suffix.** Boolean, text and instance-swap properties are named like `ShowIcon#123:4`. Use the full key whenever you call the API, but strip the `#…` part in anything a human reads — descriptions, inventory, reports.
   - **Imported (remote) components are read-only** — writing to them throws. List them in the inventory as `Skip — approved reason: imported from another library`, with the owning library named.
   - **Look for leftover variant descriptions, not only empty ones.** When separate components are combined into a set, Figma moves their old descriptions onto the variants. These are usually outdated; flag them in the inventory (`Revise` or clear, owner decides).
   - **If colors are in scope**, crawl the local color variables too: name, collection, whether each one aliases another variable or holds a raw value, and its current description. Imported variables are read-only — skip them the same way as imported components.

   **Done when:** every page is loaded, and every set, standalone component (and, if in scope, color variable) has its properties, composition and current description recorded — with variant-level descriptions and the Examples check recorded too. Anything the crawl could not read is listed as a gap, not silently left out.
4. **Pre-write inventory report — scope approval before any drafting at scale.** From the crawl, produce a read-only inventory the owner can verify before reviewing any wording:
   - Counts: total `COMPONENT_SET` + standalone `COMPONENT` nodes discovered; public sets / standalone / private / variants selected for description; already-described vs empty; entries needing owner input; deliberately skipped nodes grouped by approved reason; duplicate names, trailing-whitespace names and other matching risks; nodes that could not be classified confidently. Report total variants discovered separately from variants selected for description — variants excluded because their names are self-explanatory count as an approved criterion-based skip, not missing documentation.
   - The file-level scope from step 1: files in scope, reference-only files, and referenced-but-out-of-scope files with reasons.
   - Examples check result: `_example` components / `Examples` page present or absent.
   - A table, one row per node in scope, **grouped by library page**, without node IDs in the rows (IDs go to an appendix at the end of the report — they serve the injection script, not the reader). For large libraries deliver the table as a file, keep aggregates in the conversation:

     | Component | Parent | Type | Current description | Planned action | Confidence | Issues |
     |---|---|---|---|---|---|---|

     Allowed `Planned action` values: `Write` / `Revise` / `Keep existing` / `Needs owner input` / `Skip — approved reason` / `Blocked — collision or ambiguity`.
   - **If colors are in scope:** a separate *Colors* section with the same columns (Parent = collection, Type = Semantic / Primitive / Single-tier), plus the detected split (primitives + semantics, or none) stated in plain words for the owner to confirm.
   - A publication scope summary: descriptions to add / revise / leave unchanged, unresolved entries, skipped entries, detected library issues, and **other pending file changes that a library publish would also push**.

   Gating: inventory approval confirms the **scope**, not the wording — drafting starts after scope approval, injection only after the separate wording review (step 7).

   **Done when:** every node in scope has a `Planned action`, and the owner has approved the scope.
5. **Do not guess.** If intended use cannot be determined from properties, composition and verified usage, do not write a plausible-sounding description to close the task. Mark the entry `Needs owner input` in the review output and ask one precise question. Resolve uncertainty by reading data or asking — never by theorizing (placeholder rules: see §3).

   **Done when:** every entry has either a draft backed by evidence or `Needs owner input` with one precise question.
6. **Assign confidence per entry, backed by evidence:**
   - `High` — properties, composition and usage all agree.
   - `Medium` — structure is clear, domain meaning needs confirmation.
   - `Low` — name, properties or usage contradict each other; never inject without review.
   - Always list the evidence behind the label (`properties / composition / verified usage / owner input`) — a `High` with no verified usage on a domain-specific component is really a `Medium`.

   **Done when:** every entry has a confidence label and the evidence behind it.
7. **Owner review loop before injecting** — expect ~10% of entries to be materially wrong in domain terms the structure can't reveal (business intent, historical exceptions, product constraints). Deliver drafts in a fixed **output contract**; all fields are mandatory — use `None` when a field has no entries, never omit fields:
   ```
   Component: [name] · Node ID: [id] · Type: Set / Standalone / Private / Variant
   Confidence: High / Medium / Low
   Evidence: properties / composition / verified usage / owner input
   Current description: [... | None]
   Proposed descriptionMarkdown: [...]
   Open questions: [... | None]
   Detected library issues: [... | None]
   ```
   `Current description` matters for every `Revise` entry: writing replaces the old text completely, so the owner must see old and new side by side to keep anything valuable the old one said. For colors, use the same contract with `Type: Color` and `Proposed description` (plain text).

   **Done when:** the owner has approved the wording of every entry that will be written, and none of those entries has an open question left.
8. **Inject via `descriptionMarkdown`.** Match nodes **by ID whenever possible**. If matching by name returns multiple nodes: compare their property definitions, composition and parent context; update all matches **only when structurally and semantically equivalent** — otherwise stop and report a name collision for owner review. Identically-named nodes are not necessarily the same component; blind multi-write creates wrong documentation with no technical error. Use `trim()` to catch trailing-whitespace names, but trimmed equality alone does not prove equivalence. When you write **by ID**, a duplicate name is not a reason to block writing — each node gets its own description, and its "Not for" should tell the two apart. Log the duplicate name as library debt (§9) so the owner can rename it later.

   **Write in chunks** (e.g. one library page per call), and record which entries each chunk updated. A single huge write can time out and stop partway, leaving the file half-written with no clear record of where it stopped; chunks make a failure easy to resume. Colors are written to each variable's plain `description`.

   **Done when:** every approved entry appears in the chunk log as written exactly once, and nothing outside the approved list was touched.
9. **Completeness sweep at the end — non-negotiable.** Query the actual file state for every COMPONENT_SET and standalone COMPONENT with an empty description, **and** verify no unresolved draft markers (`Needs owner input`, `TODO`, `TBC`) remain in injected descriptions. Verifying "did my list get written" is not verification; the questions are "does anything lack a description" and "did any placeholder leak in". This inverse check catches items lost between draft versions, sub-components silently skipped, and components missing from the original crawl. If colors are in scope, run the same check on the color variables that were planned for a description.

   Two notes for reading the sweep:
   - Figma has a known bug where a description can appear missing or outdated when it is not; re-publishing the affected nodes fixes it. Before reporting an entry as empty, re-read it directly from the node — don't report a false gap.
   - **Search check.** The goal is better search for people and AI, so test it: run `search_design_system` with 5–10 plain-language queries a designer would actually type ("row that opens another screen", "warning message that can be dismissed") and check that the intended component ranks near the top. A miss usually means the description lacks the words people use — fix the wording, not the query. Note that search only sees published content, so this check may have to wait until after the publish in step 11; say so in the change report.

   **Done when:** the sweep finds zero empty descriptions outside the approved skip list and zero leftover markers, and the search check has run (or is scheduled right after publishing, noted in the change report).
10. **State skip-decisions explicitly.** Any category deliberately left undocumented must be written down and approved — an unrecorded decision is indistinguishable from an omission. For third-party components and referenced external files, record *why* they are excluded (unmodified / externally maintained / outside the published library scope) and who owns their documentation.

    **Done when:** every skipped node and category in the inventory has a written reason and an owner.
11. **Publishing is a separate, owner-approved step.** Publish pushes ALL pending file changes, not just descriptions. After injection: run the sweep, run the duplicate-name/property-quality audit, verify Markdown rendering, produce a change report — then the owner approves and publishes explicitly. The final change report must reconcile the approved inventory with the actual result: planned, successfully updated, unchanged, blocked, skipped, and newly discovered during the completeness sweep. If the library has no `Examples` page and no `_example` components, include a standing recommendation to add one next to the components — agents learn full compositions from it. Never publish as an automatic side effect of finishing the injection.

    **Done when:** the change report reconciles every planned entry with its result, and the owner has explicitly published (or explicitly decided not to yet).

## 7. Description maintenance

A description is correct at the moment of writing — components keep changing, and a stale description is worse than none because it actively misleads. Maintenance triggers:
- **When property definitions change** (prop added/removed, variant values changed, new instance-swap slot): revise the description. A new variant may extend Options; changed semantics may invalidate "Not for".
- **When components are split or merged**: the old description is stale by definition. Write a new one from scratch instead of editing the old — inherited text after a split/merge is the most common source of nonsense descriptions.
- **When a component is renamed**: check whether other descriptions reference the old name in cross-references ("Not for" and composition notes often point to components by name).
- **When a color is renamed or its role changes**: update its own description, and check other color descriptions that point to it ("use `bg/page` instead").

Do not schedule periodic "description audits" — that is a dead process. Maintenance works only as part of normal component work, the same way prop updates do.

## 8. Reports are written for humans

Every report (inventory, review batch, final change report) opens with a plain-language summary of at most three sentences: **what I found, what I plan to do, what I need from you.** Machine detail follows below it, never instead of it. Tables carry human-readable values, are grouped by library page, and contain no node IDs in rows — identifiers live in an appendix at the end of the report where the injection script can find them. Plain markdown; enum values and counts support the summary, they are not the message.

## 9. Side effects to harvest

A description pass surfaces library debt — **log it and assign an owner; fix it in the same pass only when explicitly in scope and owner-approved, otherwise move it to a follow-up debt list.** Documenting and refactoring are separate operations — mixing them silently inflates scope and makes review harder. Typical findings:
- Name collisions (two components named identically) — break search for humans and AI.
- Default prop names (`Property 1`) — rename to semantic (`State`).
- Typos in component, prop and variant names — they poison text search.
- Dual-purpose components (one badge serving two unrelated domains, e.g. user role AND object status) — candidates for splitting into semantic sets, with state values named semantically (`Active/Inactive`), never domain-specifically.
- Repeated ad-hoc structures in product files that should be components (recurring layout bugs are the symptom).

## 10. Beyond descriptions (same goal, different mechanisms)

- **Examples**: an `Examples` page in the published library (Figma's own guidance recommends a dedicated page with complete examples; `_example`-suffixed components are a convention some teams use) teaches AI full compositions — how atoms assemble into screens. Highest-leverage complement to descriptions.
- **Annotations** (Dev Mode) carry screen-specific behavior ("this phone number is a tel: link") — product-file knowledge that must never leak into library descriptions.
- **Rules file / agent skill** carries behavioral instructions (search-before-building-custom, switch-variant-never-override-text, platform baseline) — process knowledge, not component knowledge.

## 11. Reference examples

**Anti-example — the typical mistakes in one description:**
> Used on the Settings screen and Profile screen. A versatile, flexible, customizable row component. Props: Size (Small, Medium, Large), State (Default, Hover, Pressed, Disabled, Focused), ShowIcon (True/False), ShowDivider (True/False), IconPosition (Left/Right), TextAlignment (Left/Center). Can be used for navigation, selection, displaying information, and many other use cases. Keywords: list, row, cell, item, table, menu, option, setting, profile.

What's wrong: "Settings screen and Profile screen" — screen names, breaks §3 (rot on rename, tie the library to one product); "versatile, flexible, customizable" — empty adjectives that say nothing about intended use; the full prop list with State × Size — a property dump instead of selective Options (§2), and `ShowIcon`/`ShowDivider` need no documentation; "many other use cases" — no "Not for", so nothing says when NOT to use it; "Keywords: list, row, cell…" — keyword stuffing (§3).

**Public component set** — note the selectivity it models: `Behavior` (non-obvious) gets a full gloss, `Size` is enumerated without one, and the `ShowDivider` boolean is skipped entirely:
> Displays a tappable row that opens another view or selection flow. May include leading media, supporting text and a trailing value or disclosure indicator.
>
> **Options:**
> - `Behavior=Navigate` — tapping pushes another view; shows a trailing chevron
> - `Behavior=Select` — tapping toggles selection in place; never combine with a chevron
> - `Behavior=Expand` — tapping reveals inline content below the row
> - `Size=Compact/Regular/Large`
>
> **Rules:** the trailing slot shows either a value or a disclosure indicator — never both.
>
> **Not for:** static key-value information — use `Information Row` instead.

**Simple private component:**
> Private internal of `List Row`. Do not instantiate directly — configure it through `List Row`. Handles alignment and spacing between the title, supporting text and trailing content.

**Complex private component** (complexity, not the dot prefix, drives length):
> Private internal of `Date Picker`. Do not instantiate directly — configure it through `Date Picker`. Controls the selected-day state and range-edge styling. Its visual state depends on the parent's `SelectionMode` and must not be overridden independently.
