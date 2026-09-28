---
name: figma-tailwind-review
description: Figma to Tailwind CSS v4. Reviews, fixes or builds styling so it matches the Figma file's variables, modes and text styles. Use when checking a built design against Figma, re-checking for regressions, writing or fixing the global CSS from Figma variables, or building a component from a Figma link in a Tailwind v4 project.
---

# Figma Tailwind Review

**Figma is the source of truth.** Every name, value and mode in the code traces back to Figma, copied as Figma has it. When something is missing in Figma, tell the user. Report only mismatches with Figma; design choices belong to the designer.

**Think first, ask only when needed.** Obvious from Figma → do it. Fairly sure → say what you think and ask yes or no. Can't tell → ask. Nothing to decide → say nothing.

## Review an implementation

1. Get the Figma link. You need a tool that runs code inside Figma (with the Figma MCP server: `use_figma`); without one, say so and stop.
2. Check the project uses Tailwind v4, and find the CSS file with `@import "tailwindcss";`.
3. Read Figma, then say what you found: *"I found 3 collections (one with Mode A and Mode B), 248 variables and 10 text styles."*
4. Compare the global CSS with Figma and the rules below, then each component with its Figma source component: the variable Figma attaches to each element, and whether the code uses exactly that. Done when every Figma variable has been checked in every mode, and every element of every component in scope has been compared.
5. List every problem (see "Reporting problems") and change nothing yet. If `CLAUDE.md` and/or `AGENTS.md` don't mention this skill yet, propose adding:
   ```markdown
   ## Figma styling
   Styling follows this Figma file: <link>. Use the figma-tailwind-review skill for any styling work.
   ```
6. Suggest committing first, then apply only what the user approves. Edit existing files in place, touching only the lines a fix needs. When a fix renames or removes a variable, update every component that uses it in the same change.
7. Check: every Figma variable has the right name and value in every mode, and the project builds. A passing build doesn't prove styling is right: if a browser is available, compare the affected rendered styles with Figma; otherwise, say this wasn't checked.

No global CSS yet (only `@import "tailwindcss";`)? In step 4, write it from Figma following the rules below, and ask before writing.

Re-checking after changes (for example, for regressions)? Start with the files changed since the last check (using git; ask which commit or branch to compare with, or use the main branch if the user doesn't know), plus every component that uses a variable those changes touched. Say which commit or branch you compared with. Check the rest only if asked.

## Reading Figma

- Read variables by running code inside Figma (the Plugin API). Design-summary tools return only what the current selection uses, usually only the default mode, and often final values instead of links.
- **Library variables:** a variable can point to one in a Figma library file. Read that one too, or ask for the library's link, so every link in the CSS has a target.
- A failed or partial read is not a deletion. Stop and say what couldn't be read.
- **Units:** check each Figma number's unit before writing it into CSS:
  - sizes → `px`
  - line height in percent → a plain number (150% → `1.5`); "Auto" → `normal`
  - letter spacing in percent → `em` (2% → `0.02em`)
  - a number variable bound to line height or letter spacing → `px`
  - font weight → a number without a unit; a style name like "Semi Bold" becomes `600`
  - opacity → Figma stores 0–100, CSS needs 0–1 (50 → `0.5`)
- When a link to the project's Figma file comes up later (not an unrelated link), compare Figma with the global CSS first. List what's new, changed or gone, and ask before changing anything. A name that vanished while a new one appeared with the same value may be a rename: ask.

## Global CSS

- **File order:**
  ```css
  @import "tailwindcss";      /* 1. Tailwind */
  @theme { … }                /* 2. Tailwind settings: defaults off, breakpoints */
  :root { … }                 /* 3. Figma collections, then text styles */
  [data-mode="mode-b"] { … }  /* 4. Mode blocks, always after :root */
  @utility name { … }         /* 5. Helpers classes can't express */
  @layer base { … }           /* 6. Base rules for every page */
  ```
- **Names:** the Figma path in lowercase, with hyphens: `Group/Token Name` → `--group-token-name`. If Figma's code syntax field is filled in, use it, taking only the `--token-name` part (it often holds `var(--token-name)`). Change the format, keep the meaning. If two variables would get the same name, ask, and keep them separate.
- **Collections and links.** A file may have one, two or more collections, named anything. Work out the structure from what each variable holds, and copy it exactly as Figma has it:
  - One section per Figma collection, with the collection's name, in Figma's order.
  - A variable that **holds a value** in Figma holds that value in the CSS.
  - A variable that **links to another variable** in Figma links to it in the CSS with `var()`, even when the value is the same, so the link to Figma stays.
  - Check this **per mode**: one variable can link to another in Mode A and hold a plain value in Mode B.
  - **Chains stay chains:** if A links to B and B links to C, the CSS does the same.
  - Copy every variable, including ones no screen uses yet, with the same links Figma has.
  - Values Figma can't store as variables (like animation timings, easings or stacking layers) live in their own section marked `/* ---- Code only (not in Figma) ---- */`. The review leaves them out.

  The blueprint:
  ```css
  :root {
    /* ---- Collection name 1 ---- */
    --group-token-a: raw-value;           /* holds a value in Figma */
    --group-token-b: raw-value;

    /* ---- Collection name 2 ---- */
    --group-token-c: var(--group-token-a); /* links to token-a in Figma */

    /* ---- Collection name 3 ---- */
    --group-token-d: var(--group-token-c); /* links to token-c: the chain stays */

    /* ---- Text styles ---- */
  }
  ```
- **Variables go in `:root`, not `@theme`.** Inside `@theme`, Tailwind gives names like `--text-*` its own meaning.
- **Tailwind's own values** (its colours, font sizes, radius, breakpoints…):
  - **Writing a new global CSS:** switch off each group Figma replaces, in `@theme`, so they can't be used by accident: `--color-*: initial;`, `--breakpoint-*: initial;` and so on. Keep Tailwind's spacing scale (`--spacing`) on, even when Figma has spacing variables: number classes like `size-6` and `p-0` depend on it.
  - **Reviewing an existing project:** leave the settings as they are. List every class using Tailwind's own values (like `bg-red-500`, `text-sm`, `rounded-lg`, or `md:` when the project has its own breakpoints) as a problem, with the Figma variable to use instead.
- **Calculated values** (like `color-mix()` or `calc()`): only when CSS needs something Figma can't store. Every piece is a Figma variable, a plain value shown in Figma, or a device measurement (like screen height), with a comment explaining the calculation.
- **Text styles:** one variable each, written in this order, which is always valid (italic goes first when the style has it):
  ```css
  --text-style-name: [italic] var(--weight) var(--size)/var(--line-height) var(--family);
  ```
  A wrong order, or a part with the wrong kind of value (like "Semi Bold" instead of `600`), makes the browser silently ignore the whole value. Each part uses the variable the Figma text style is bound to; a plain value in Figma is written as that value (see "Units"). Anything the shorthand can't hold gets its own variable next to it: `--text-style-name-letter-spacing`, and text case or decoration if the style has them (these are always plain values; Figma can't bind them to variables).
- **A font loaded by the framework** (like `next/font`): point to the framework's font variable, add a fallback, and say where it comes from: `--token-name: var(--framework-font-variable), sans-serif; /* loaded by the framework */`.
- **Modes:** modes belong to a collection, so handle each collection's modes separately. No modes → skip. Otherwise look at what changes between them, then say what you think and ask to confirm:
  - *"Only colours change between Mode A and Mode B, so these look like themes. Correct?"*
  - *"Sizes get smaller in Mode B, so this looks like a screen-size mode. Correct? At what screen width does it switch?"*
- **One name in every mode.** A variable keeps its single Figma name across all modes; only its value changes.
- **Theme modes:** the first mode's values go in `:root`. Each other mode gets its own block, named after the Figma mode and listing only the values that change:
  ```css
  :root                { --token-name: var(--token-a); } /* Mode A */
  [data-mode="mode-b"] { --token-name: var(--token-b); }
  [data-mode="mode-c"] { --token-name: var(--token-c); }
  ```
  - Mode blocks come after `:root` in the file: they have the same weight, so the later one wins.
  - Put the `data-mode` attribute on `<html>`. If a mode can apply to just part of a page, its block also repeats every variable that links to a changed one.
  - If a mode should switch on by itself from a device setting, ask which setting, and use that media query instead of the attribute.
- **Screen-size modes:** the smallest screen is the default. Larger screens get a media block listing only the values that change, keeping Figma's names. Ask the user at what width it switches, since Figma doesn't store it. Put that width in `@theme` as a length with a unit (like `48rem`), in the same unit as any other breakpoint:
  ```css
  @theme { --breakpoint-mode-name: raw-value; }
  @media (width >= theme(--breakpoint-mode-name)) { :root { --token-name: raw-value; } }
  ```
  If the width must also be written somewhere else (like JavaScript), add a comment naming the variable it copies.
- **`@utility`:** styling a class can't express (pseudo-elements, layered backgrounds, a calculation used in many places) goes in a named helper in the global CSS, still using Figma variables. Components then use it like a normal class:
  ```css
  @utility bg-name { background-image: linear-gradient(var(--token-a), var(--token-a)); }
  ```
- **Base rules for every page:** set once here, inside `@layer base`, so Tailwind classes can still override them. Tailwind v4 shows the arrow cursor on buttons, and the page's background, text colour and font come from Figma variables:
  ```css
  @layer base {
    button:not(:disabled), [role="button"]:not(:disabled) { cursor: pointer; }
    button:disabled { cursor: not-allowed; }
    body { background-color: var(--token-a); color: var(--token-b); font-family: var(--token-c); }
  }
  ```

## Components

**Which Figma node:** build and check against the source component in Figma, where all variants, states and component properties (on/off toggles, swaps, text) are, not a copy on a screen. If it lives in a library file, ask for that link. If there's no component at all, only frames on screens, compare with those frames and say so. Two exceptions: an override on a copy wins on that screen, and width and height can come from where the copy sits (Hug in the library, Fill on the screen).

**Where each value comes from**, first match wins:
1. A Figma variable is attached → use exactly that variable, even if another has the same value, and even if it links to another variable. If the fill also has its own opacity in Figma, keep it: `bg-(--token-name)/10`.
2. Hug or Fill → reproduce the behaviour in the actual parent layout: Hug fits the content, Fill takes the available space, so the element keeps resizing as Figma intends.
3. A plain value in Figma → copy it. Use a Tailwind number class (like `size-6`) only if the project's spacing scale gives exactly that value; otherwise the exact value in brackets: `w-[343px]`, `bg-[raw-value]`.
4. Figma can't know it (like screen height) → calculate it from variables, with a comment.
5. Nothing → ask.

**Writing classes:**
- Use the short form: `p-(--token)`, not `p-[var(--token)]`. The same for every property: `gap-(--token)`, `bg-(--token)`, `text-(--token)`, `border-(--token)`, `rounded-(--token)`, `w-(--token)`, `h-(--token)`, `z-(--token)`.
- The variable must exist in the global CSS. An unknown one gives no error and silently resets that property to its default.
- Text styles are one class: `[font:var(--text-style-name)]`, plus the style's extra variables (like `tracking-(--text-style-name-letter-spacing)`). Set font size or line height separately only when a copy in Figma overrides them.
- Some classes guess wrong what a variable is, silently. Add a type hint:
  - size variables on `text-`, `border-`, `outline-`, `ring-`, `stroke-`, `decoration-` become colours → `border-(length:--token)`
  - `font-(--token)` becomes a font weight → a font family needs `font-(family-name:--token)`
  - a colour on `shadow-` becomes the whole shadow → `shadow-(color:--token)`
- A border's colour must match Figma. In v4 the default border colour is the text colour, so a border width alone is usually wrong.
- Write whole class names: Tailwind only generates names it finds written in full, so pieces like `` `p-${size}` `` produce nothing.

**States:**
- Build exactly the variants, states and component properties the source component has, and report only what differs from it.
- Mark states for screen readers, and style from that mark, so the look and what a screen reader says can't disagree. The element decides the attribute, not the Figma variant name:
  - selected → `aria-pressed` (toggle button, always `true` or `false`) or `aria-selected` (tab, option)
  - checked → `aria-checked` (checkbox, switch); current page → `aria-current`
  - open → `aria-expanded`
  - disabled → `disabled` on buttons and form fields, `aria-disabled` on anything else

  Style with the matching variant: `aria-pressed:bg-(--token)`, `aria-expanded:…`, `disabled:…`, `aria-disabled:…`.
- Keep everything important visible without hover: in v4, `hover:` doesn't work on touch screens. A Pressed state in Figma → `active:`.
- Focus: keep the browser's focus ring unless Figma has a Focus state. If it does, hide the ring with `outline-hidden`, which stays visible in high-contrast mode (unlike `outline-none`), and build Figma's focus style with `focus-visible:`, so it shows for keyboard users, not on every click.

## Reporting problems

Each problem names the file, line and element, what's wrong, and the exact fix. If there's no single clear fix, list the options. Group by file; show a repeated problem once, with its count.

> `ComponentName.tsx`, line 00: padding uses `--token-a`, but Figma attaches `--token-b`. Fix: `p-(--token-b)`.
>
> `ComponentName.tsx`, line 00: `border-(--token-name)` is missing `length:`, so it sets the colour. Fix: `border-(length:--token-name)`.
