# save-link-to-notion

A Skill that turns a Notion database into a searchable second brain for links.

Paste a URL, with or without a note about why you saved it, and Claude will:

1. Actually fetch and read the page — never guess from the URL alone.
2. Create a structured entry in a **Saved Links** Notion database with a real title, an inferred category, a specific 1–2 sentence description, and a handful of practical use-case tags.
3. Skip duplicates automatically.

Later, instead of digging through bookmarks, just ask:

> "What's a good site for making mockups?"
> "Find that article about pricing freelance work."
> "Do I have anything saved for color palettes?"

Claude searches the database by description and use-case tags — not just title — and tells you which saved link fits and why.

## Why a database instead of a bookmarks page

Most "save this link" setups dump everything into one page of bullet points, which is fine until you have 50 links and no way to find the one you need. This skill creates real Notion properties (Title, URL, Category, Description, Use Case Tags, Date Added), so the database stays filterable and sortable in Notion itself, and searchable by Claude on top of that.

## Requirements

- A Notion workspace connected via MCP.
- A URL-fetching tool available (e.g. web search/fetch), so descriptions are written from the real page content.

## Installation

**Claude.ai / Claude Desktop / Claude Code:** drop the `save-link-to-notion/` folder (containing `SKILL.md`) into your skills directory, or use your client's "add skill" flow and point it at this repo or the raw `SKILL.md` file.

**Claude Code specifically:** place the folder at `.claude/skills/save-link-to-notion/` in your project, or in your global skills directory for use across projects.

No build step — it's a single instructions file plus this README.

## Database schema

| Property | Type | Purpose |
|---|---|---|
| Title | Title | The site/tool/article's real name |
| URL | URL | The link |
| Category | Select | `Design tool`, `Article`, `Inspiration`, `Reference`, `Course`, `Job board`, `Community`, `Other` |
| Description | Text | 1–2 specific sentences — what it does and who it's for |
| Use Case Tags | Multi-select | Practical search phrases, not vague labels |
| Date Added | Date | Set automatically |

The skill creates this database itself on first use — nothing to set up by hand in Notion.

## License

MIT — see [LICENSE](./LICENSE).
