# Design Mojo

**AI skills for designers.**

A skill is a short set of instructions your AI assistant (Claude, Codex, Cursor…) follows for one kind of job. Install a skill once, and the AI does that job the right way every time.

| Skill | What it does |
|---|---|
| [figma-tailwind-review](#figma-tailwind-review) | Checks that the code matches your Figma file: variables, modes, text styles, components. |
| [figma-components-tokens-description](#figma-components-tokens-description) | Goes through your Figma files and writes clear descriptions for your components and color tokens, so other AI agents build with the right ones. |

More skills coming.

---

## figma-tailwind-review

### Why

AI tools write code that looks right but slowly drifts away from your Figma file: the right number taken from the wrong variable, a text style that silently stops working, a border that isn't what you designed. This skill helps you find those bugs and keep the code and Figma consistent, without knowing Tailwind yourself.

### What it does

- Reads your Figma file: variables, modes and text styles.
- Checks the project's global CSS against Figma.
- Checks each component against its source component in Figma: which variable Figma attaches to each element, and whether the code uses exactly that one.
- Lists every problem with the file, the line, what's wrong and the exact fix, and changes nothing until you approve.
- Fixes what you approve, without breaking other screens.
- No global CSS yet? It writes one from Figma.

### What you need

- A project using **Tailwind CSS v4**.
- The **Figma MCP server** connected to your AI tool ([Figma's guide](https://help.figma.com/hc/en-us/articles/32132100833559-Guide-to-the-Figma-MCP-server)).
- A **Dev or Full seat on a paid Figma plan**. View and Collab seats only get a few Figma MCP calls a month, not enough for a review.

### Install

**Automatic (Claude Code, Codex, Cursor and others)**

In your project folder, run:

```
npx skills add mondycz/design-mojo --skill figma-tailwind-review
```

**Manual: Claude Code**

1. Download [`SKILL.md`](skills/figma-tailwind-review/SKILL.md).
2. Create the folder `~/.claude/skills/figma-tailwind-review/` (every project) or `.claude/skills/figma-tailwind-review/` inside one project.
3. Put `SKILL.md` in that folder and restart Claude Code.

**Manual: Claude app (claude.ai)**

1. Download [`figma-tailwind-review.skill`](https://github.com/mondycz/design-mojo/raw/main/downloads/figma-tailwind-review.skill).
2. In Claude, open **Settings → Capabilities → Skills**.
3. Upload the file.

### Try it

> Review my global CSS and components against this Figma file: [your Figma link]

> I changed a few components. Re-check them against Figma for regressions.

> My project has no styling yet. Write the global CSS from this Figma file: [your Figma link]

### Tested

Tested on a real Next.js and Tailwind v4 project with its Figma file: it caught every styling mistake that had earlier been fixed by hand, and found new ones nobody had noticed. In side-by-side tests, an AI with the skill passed 34 of 34 checks; the same AI without it passed 16.

---

## figma-components-tokens-description

With this skill, the AI goes through all your Figma files: the component library, your foundations (colors, icons) and your app screens. Based on what it finds there, it writes proper descriptions for your components and color tokens: what each one is for, which options matter, and when *not* to use it. Other AI agents read these descriptions, so they pick the right component and color when they implement your designs. You approve the plan and the wording before anything is written.

### Why

Component descriptions are what designers, Dev Mode and AI tools like the Figma agent read to decide which component to use. Most libraries have none, or one-liners like "row component", so people and AI guess, and often pick the wrong one. This skill writes descriptions that say what a component is for, which options matter, and, most importantly, when *not* to use it and what to use instead.

### What it does

- Asks you a few short questions first: which files make up your design system, what to skip, and whether to describe **components only** or **components and colors**.
- Reads your library: every page, component, property and existing description. It flags old leftover text, imported components it can't edit, and pages it couldn't read.
- Shows you a full list of what it plans to write, change or skip, and waits for your approval.
- Drafts every description and shows you the old and new text side by side. Nothing is written until you approve the wording.
- For colors: if you have primitive and semantic colors, it describes only the semantic ones. If you don't, it describes all of them.
- Writes in small batches, then checks that nothing was missed and that search actually finds the right component.
- Lists library problems it notices along the way (duplicate names, typos, default property names) without changing them.
- Never publishes your library. That stays your decision.

### What you need

- The **Figma MCP server** connected to your AI tool ([Figma's guide](https://help.figma.com/hc/en-us/articles/32132100833559-Guide-to-the-Figma-MCP-server)).
- **Edit access** to your library file (and to the file with your color variables, if you want colors described).
- A **Dev or Full seat on a paid Figma plan**, for the same reason as above: a full library needs many Figma MCP calls.

### Install

**Automatic (Claude Code, Codex, Cursor and others)**

In your project folder, run:

```
npx skills add mondycz/design-mojo --skill figma-components-tokens-description
```

**Manual: Claude Code**

1. Download [`SKILL.md`](skills/figma-components-tokens-description/SKILL.md).
2. Create the folder `~/.claude/skills/figma-components-tokens-description/` (every project) or `.claude/skills/figma-components-tokens-description/` inside one project.
3. Put `SKILL.md` in that folder and restart Claude Code.

**Manual: Claude app (claude.ai)**

1. Download [`figma-components-tokens-description.skill`](https://github.com/mondycz/design-mojo/raw/main/downloads/figma-components-tokens-description.skill).
2. In Claude, open **Settings → Capabilities → Skills**.
3. Upload the file.

### Try it

> Add descriptions to all components in this Figma library: [your Figma link]

> Document our design system so the Figma agent understands it. Do components and colors.

> Audit the existing descriptions in this library and improve the weak ones: [your Figma link]

---

Made by **Michał Ondycz** · [mondycz.com](https://mondycz.com) · [x.com/michalondycz](https://x.com/michalondycz)

MIT License
