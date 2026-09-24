# Spider Lily — Obsidian Dashboard Starter

A dark, rose-accented Obsidian setup with a custom homepage dashboard (greeting, daily quote, editable link pills, project tracker, a pet, and a "recently edited" strip) plus a right-sidebar panel (month calendar, focus/pomodoro timer, and an upcoming-tasks timeline).

![dashboard](00%20Assets/spider-lily.png)

## What's inside

| Piece | File |
| --- | --- |
| Homepage dashboard | `Homepage.md` (DataviewJS) |
| Sidebar panel | `00 Assets/Sidebar.md` (DataviewJS) |
| Dashboard styles | `.obsidian/snippets/dashboard.css` |
| Theme (colors, dark surfaces) | `.obsidian/snippets/spider-lily.css` |
| Accent + enabled snippet | `.obsidian/appearance.json` |

The look is the default Obsidian theme + a `#e0566b` accent + the two CSS snippets, so there's no separate community theme to install.

## Requirements

Two community plugins (install them from **Settings → Community plugins → Browse**):

- **Dataview** — runs the dashboard. After installing, make sure **Enable JavaScript queries** is ON (this repo already ships that setting).
- **Homepage** — opens `Homepage.md` on startup.

## Setup

1. **Download** this repo (green *Code* button → *Download ZIP*, or `git clone`).
2. In Obsidian: **Open another vault → Open folder as vault**, and pick this folder.
3. When prompted about "third-party plugins", trust the vault and enable community plugins.
4. Install **Dataview** and **Homepage** from the community plugin browser, then reload.
5. Open `Homepage.md`. Click **✎ Edit** to change the greeting, tagline, link pills and projects — everything saves back into the note's frontmatter automatically.

If the dashboard shows as code instead of rendering, Dataview isn't installed/enabled yet, or JS queries are off.

## Customizing

- **Accent color:** change `#e0566b` in `appearance.json` and in the two snippet files.
- **Banner image:** replace `00 Assets/spider-lily.png` with any image of the same name, or edit the `BANNER` path near the top of `Homepage.md`.
- **Daily notes:** the calendar and timeline expect dated notes like `Daily/2026-09-30.md`. Add tasks as `- [ ] task 📅 2026-09-30` in any note, or as plain `- [ ] task` inside a daily note.

## Credits

Dashboard and theme by [your name]. Built on [Dataview](https://github.com/blacksmithgu/obsidian-dataview) and [Homepage](https://github.com/mirnovov/obsidian-homepage).
