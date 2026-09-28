# Design Mojo

**Just AI skills for designers.**

A skill is a short set of instructions your AI assistant (Claude, Codex, Cursor…) follows for one kind of job. Install a skill once, and the AI does that job the right way every time.

| Skill | What it does |
|---|---|
| [figma-tailwind-review](#figma-tailwind-review) | Checks that the code matches your Figma file: variables, modes, text styles, components. |

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

Made by **Michał Ondycz** · [mondycz.com](https://mondycz.com) · [x.com/michalondycz](https://x.com/michalondycz)

MIT License
