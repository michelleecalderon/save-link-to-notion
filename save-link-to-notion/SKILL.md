---
name: save-link-to-notion
description: Save a URL as a structured, searchable entry in a "Saved Links" Notion database, or search that database to recommend a previously saved link for a task. Use this skill whenever the user pastes a bare URL, says "save this link/site/tool," or asks a recommendation question that a saved resource might answer — e.g. "what's a good site for making mockups," "find that article about pricing freelance work," "do I have anything saved for X," "what did I save about Y." Always trigger on a pasted URL even with no other text, and always trigger on requests to recall, find, search, or recommend a saved resource, even if the user doesn't say "Notion," "saved links," or "bookmark."
license: MIT
---

# Save Link to Notion

Turns Notion into a searchable second brain for links. Instead of a flat bookmarks list, this skill maintains one Notion database — **Saved Links** — where every saved URL gets a real title, an inferred category, a specific 1–2 sentence description, and a handful of practical use-case tags. Because those are actual Notion properties and not text buried in a page body, the database stays filterable in Notion itself, and later you can ask a plain-language question ("what's a good site for making mockups?") and get the right saved link back — matched on what it's *for*, not just its name.

This skill has two modes:
1. **Save mode** — the user gives you a URL (with or without a note about why).
2. **Recall mode** — the user asks a question that a saved link might answer.

Decide the mode from the user's message: a bare or embedded URL with no clear question means save mode; a question with no URL means recall mode. If both are present, do both.

## Prerequisites

- A Notion workspace connected via MCP (the `notion-search`, `notion-fetch`, `notion-create-database`, `notion-update-data-source`, `notion-create-pages`, and `notion-query-data-sources` tools, or equivalent).
- A tool that can fetch the content of an arbitrary URL (e.g. `web_fetch`), so descriptions are written from the actual page rather than guessed.

Without the Notion connection this skill has nothing to write to — if it isn't available, tell the user to connect Notion first rather than falling back to a plain-text list.

---

## Setup: does the database exist yet?

The skill creates the database itself — you don't need to make it manually first.

On first use in a workspace, search for a database named **Saved Links** (use Notion search / fetch). If it doesn't exist:

1. Ask the user once where to create it (which page/workspace location) if that isn't obvious, or default to their private root if you have one you've used before.
2. Create it with the exact schema in the **Database Schema** section below.
3. Remember its page ID for the rest of this conversation so you don't search for it again on every call.

If it already exists, just use it — don't recreate or duplicate it.

---

## Database Schema

Create the database with these properties exactly (types matter — they're what makes it filterable later):

| Property | Type | Notes |
|---|---|---|
| **Title** | Title | The site/tool/article's actual name, not the raw URL |
| **URL** | URL | The link itself |
| **Category** | Select | Options: `Design tool`, `Article`, `Inspiration`, `Reference`, `Course`, `Job board`, `Community`, `Other`. Add a new option only if nothing in this list fits. |
| **Description** | Text (rich text) | 1–2 sentences, specific — see Quality Standard |
| **Use Case Tags** | Multi-select | Specific, practical, lowercase search phrases (see below) |
| **Date Added** | Date | Defaults to today |

---

## Save Mode: steps

1. **Fetch the page.** Actually retrieve the URL's content before writing anything. Never write a description from the URL string, domain name, or your prior knowledge of the site alone — if the fetch fails, tell the user and ask them to confirm what the site is rather than guessing.

2. **Check for a duplicate first.** Query the Saved Links database's URL property for an exact or near-exact match (strip trailing slashes/query params/utm parameters before comparing). If found, stop and tell the user: "Already saved — [Title], added [date]." Don't create a second entry. If the user's new note adds real information the old entry lacks, offer to update the existing entry's description/tags instead of creating a new row.

3. **Understand what the page actually is.** Read enough of the fetched content to determine what it does, who it's for, and what's distinctive about it — not just its tagline.

4. **Fill in the fields:**
   - **Title**: the site/tool/article's real name (from the page, not the URL slug).
   - **URL**: as given.
   - **Category**: infer the single best fit from the Select options above. Don't ask the user to categorize — pick the closest match yourself. If the user's note strongly implies a category, weight that over your own read of the page.
   - **Description**: 1–2 sentences. See Quality Standard below — this is the field recall mode leans on most, so specificity here matters more than anywhere else.
   - **Use Case Tags**: 3–6 short, concrete phrases someone would plausibly type later when looking for this — not the category restated. Think in terms of the *job* the resource does: "mockup tool," "color palette generator," "freelance job board," "pricing freelance work," "clickable prototypes," "portfolio inspiration." Avoid vague tags like "useful," "design," or "cool site." Multi-select tags in Notion must exist as options before a page can use them — if a tag you want isn't already an option on the property, add it to the schema first, then set it on the new entry.
   - **Date Added**: today's date.

5. **Create the Notion page as a database entry** with those properties set (not free text in the page body — the properties are what make it queryable).

6. **Confirm briefly** — one line: title, category, and the tags you gave it, so the user can correct anything on the spot ("Saved as [Title] under Design tool — tagged mockup tool, prototyping, figma alternative. Let me know if you'd tweak any of that.").

---

## Recall Mode: steps

1. **Parse the user's need** into the kind of task or problem they're describing (e.g. "making mockups," "pricing freelance work," "finding freelance jobs").

2. **Query the Saved Links database** — search across **Use Case Tags** and **Description** first (these carry the specific, practical language), then Title and Category as a fallback. Don't restrict to exact tag matches only; also look for semantically close entries (e.g. a query about "wireframing" should surface something tagged "mockup tool" if the description supports it).

3. **Pick the best match(es).** If one entry is a clear fit, lead with it. If a few are plausible, offer the top 2–3, ranked.

4. **Answer in one line per result**, naming the resource and *why* it fits the ask — pull the reason from the stored description/tags, don't just restate the category:
   - "Uizard — browser-based tool for building clickable UI mockups from Figma-style components. Tagged: mockup tool, prototyping."

5. **If nothing matches**, say so plainly rather than stretching a loose match to fit — a wrong recommendation is worse than none. Offer to save something new if the user wants to search the web instead.

---

## Quality Standard

- **Never invent a description from the URL alone.** Always fetch and actually read the page first. If a fetch fails or the page is behind a login wall, say so and ask the user to describe it rather than guessing from the URL/title.
- **Be specific, not generic.** Bad: "A design tool." Good: "Browser-based tool for building clickable UI mockups from Figma-style components, aimed at designers who want to prototype without learning Figma." The description should tell a future search whether this resource fits, without the user needing to click through.
- **Tags are search terms, not labels.** Write them the way the user would phrase a need, not the way a catalog would classify the site.
- **No duplicates.** Always check the URL property before creating a new entry.
- **Confirm, don't over-ask.** Infer category and tags yourself; only ask the user something if the fetch genuinely failed or the page's purpose is ambiguous even after reading it.
