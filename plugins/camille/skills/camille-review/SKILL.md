---
name: camille-review
description: Work through the Camille items waiting on the user at Plan Review or Final Review. Reads each Plan or Deliverable, suggests edits, applies the ones the user agrees to with edit_document, then approves or asks Camille for changes. Use when the user asks what needs them in Camille, wants to review or approve Outlines, Articles, Code Specs or Pages, or says "let's do Camille review".
---

# Review with Camille

Go through what's waiting on the user, one item at a time. You read and suggest. The user decides. Read the `camille` skill first if it isn't loaded, for the terms and the rule that Camille does the production work.

## 1. Find what's waiting

Call `list_items` twice, with status `in_progress`:

- stage `plan_review`: Plans (Outlines and Code Specs) waiting for approval
- stage `final_review`: Deliverables (Articles and Pages) waiting before they publish

Show the user one short list: title, which review, and scheduled date. Plan Review comes first, since Camille can't start writing until a Plan is approved. Ask where they want to start, or take them in order.

If nothing is waiting, say so. `get_pipeline_summary` can tell them what Camille is working on and what's coming up. Opportunities in Triage need the user too, but they aren't this workflow. Offer to go through them separately.

## 2. Read the item

For each item:

1. Call `get_item_details`. At Plan Review its `documentId` is the Plan. At Final Review it's the Article, or the first Page File of a Page.
2. Read the whole Document with `read_document_content`. The details only carry an excerpt.
3. Read the Opportunity Brief or Research Brief if you need to know what the piece is meant to win. Don't spend `search_serp` or `lookup_keywords` unless the user asks for something the stored data can't answer.

## 3. Suggest edits

Give the user a short read: what works, what doesn't, and specific changes. Quote the text you'd change and show the replacement. Group the changes so they can say yes or no to each one.

Sort each suggestion into one of two kinds:

- **Polish the user can make now.** Wording, a headline, a sentence, a fact they know, a link. These go through `edit_document`.
- **Rework for Camille.** New or dropped sections, restructuring, a different angle, anything that needs research or rewriting at length. These go through `request_content_edit`, so Camille revises it and posts a Revision.

Don't make an edit the user hasn't agreed to.

## 4. Apply what the user agreed to

For polish, call `edit_document` with the Document's id and its complete new body:

- Start from the text `read_document_content` returned, change only the agreed parts, and send the whole thing. Not a diff.
- Leave out the Frontmatter. Keep an Article's H1 headline.
- It works only while the item is still at the same review. If it refuses because the stage moved or Camille is working on the item, tell the user and read the Document again before trying.

A Site Audit's Deliverable is a pull request on GitHub, so changes there happen on GitHub.

## 5. Approve or ask for changes

Ask the user what they want. Never approve on your own judgment.

- **Approve.** Call `approve_item` only when the user says so. At Plan Review this sends Camille on to write the Article or build the Page. At Final Review it publishes. Relay any `note` in the result. Pass several ids in one call if the user approves several at once.
- **Ask for changes.** Call `request_content_edit` with the rework they want, written clearly enough that Camille can act on it without the conversation. This starts a Run that takes minutes. Say it has started and share the `conversation_url` if the result has one.
- **Leave it.** Fine too. Move on.

Then go to the next item. At the end, summarise in a few lines what was approved, what went back to Camille, and what's still waiting.
