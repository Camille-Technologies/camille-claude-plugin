---
name: camille
description: How to work with Camille, the hosted marketing teammate, through her MCP tools. Use whenever the user mentions Camille, their Tasks, Triage, Plan Review or Final Review, Opportunity Briefs, Outlines, Articles or content performance, or asks for a blog post, page, opportunity scan, research or keyword data while Camille's tools are connected.
---

# Working with Camille

Camille finds content opportunities for an organization's site, plans and writes the work, and stops for a person to review it. Through Camille's MCP tools you act as the Member who connected you, with their role, in one organization. Everything you do is recorded as theirs.

You do the talking and deciding with the user. Camille does the production work.

In Claude Code the tools appear as `mcp__plugin_camille_camille__<tool>`, or `mcp__<server name>__<tool>` when the user added Camille by hand. This guide uses the short tool names.

## Camille's words

Use these with the user. The app uses them too.

- **Task.** Anything a Member asks Camille to do. Every Task is one Conversation. A Task with a deliverable is backed by a work item, which has a stage and a scheduled date. The tools call work items "items".
- **Triage.** Where an Opportunity Scan files opportunities for a Member to accept or decline. Each one has an Opportunity Brief saying why it's worth chasing, with the scores, search results and keywords the scan saw.
- **Plan and Deliverable.** A work item is reviewed on its Plan before Camille produces the Deliverable. An Article's Plan is its Outline. A Page's Plan is its Code Spec.
- **Checkpoints.** Plan Review (stage `plan_review`) and Final Review (stage `final_review`). The item waits there for a person to approve it or ask for changes.
- **Document.** A file on a work item: Opportunity Brief, Research Brief, Outline, Article, Code Spec or Page File. Everyone reads its Live Version.
- **Revision.** The message Camille posts when she writes a new version of a Deliverable. The user's own edits don't post one.
- **Run.** One execution of Camille's agent. A Run takes minutes and lives in a Conversation the user can open in the app.
- **Steer.** An instruction sent to a Run in progress. It changes the Run's course without stopping it.

Say Task, Article, Run and Conversation to the user, not job, post, thread or ticket.

## Production work goes to Camille

Never write an Article, Outline, Code Spec or Page yourself, not even a first draft to get things going. A Deliverable written outside Camille skips her Checkpoints and Revisions, and nobody is accountable for it. Hand the work over with a tool:

- The user wants a new post or page, or a refresh of one: `create_content_item`. Put their angle, audience and requirements in `description`. On an organization with several sites, ask which site first. The item comes back at Plan Review.
- A one-off job that isn't publishable content: `create_task_item`.
- Changes Camille should make to a Plan or Deliverable, such as a rewrite, a new section or Outline feedback: `request_content_edit`. Camille posts a Revision when she's done.
- The user's own polish on a Document at review: `edit_document`. See the next section.
- New opportunities: `run_scan`. "Why did traffic drop?" or "what should we focus on?": `run_performance_report`. Google Business Profile questions: `run_gbp_audit`. Multi-source research the user asks for: `deep_research`.
- Redirect a Run that's going, or retry a failed one: `steer_job`.

You can still think with the user. Discuss angles, critique an Outline, compare keyword ideas, suggest headlines. Then put what they decided into the tool call.

## Editing at review

`edit_document` replaces a Document's Live Version as the user's own edit, exactly like editing in the app. Camille posts no Revision for it. It works on two things only:

- an item's Plan while the item is at Plan Review
- an item's Deliverable while the item is at Final Review

Read the Document first with `read_document_content`. Send its complete new body, not a diff, and leave out the Frontmatter. Keep an Article's H1 headline. Send only changes the user agreed to. Anything bigger than polish, or any change at another stage, goes to Camille with `request_content_edit`. A Site Audit's Deliverable is a pull request on GitHub, so edit it there.

## Read stored data before paying for lookups

`search_serp` and `lookup_keywords` cost money. They draw on a daily allowance per organization that Camille's in-app chat shares. A query fetched in the last 24 hours comes from the cache and costs nothing.

Check what Camille already has first:

- `get_item_details` for an item's keywords, stage, Plan and an excerpt of the Article.
- `read_document_content` on the Opportunity Brief or Research Brief, which keep the search results and keywords as Camille saw them.
- `get_content_performance` for Search Console positions, clicks and impressions.
- `list_competitors` for who the organization is up against.

Spend a lookup only on what stored data can't answer. Put every keyword in one `lookup_keywords` call, since each call spends one lookup unless all its keywords are cached.

## Runs take minutes

A tool that starts Camille's work returns as soon as the Run is queued. Tell the user it has started, never that it's done. Give them the `conversation_url` (or `watch_url` for a scan) so they can follow it in the app.

When they ask how it's going, call `get_run_details` with the `job_id`, or `list_jobs` to see what's running. Don't poll in a loop.

## Approving and Triage

`approve_item` acts as the user and their team sees who approved. Approve only when the user says so. Approving at Plan Review sends Camille on to write the Article or build the Page. Approving at Final Review publishes. Approving in Triage accepts the opportunity and gives it a day on the calendar. Pass every id in one call.

In Triage, `dismiss_item` declines, `restore_item` brings something back, and `update_opportunity` corrects a scan's assessment once the user confirms it. `reschedule_item` moves the day Camille works on an item. `confirm_competitors` confirms who the next scan compares against.

## When a tool refuses

Tell the user what happened and don't retry.

- **Daily lookup limit.** The result has `limitReached: true` and `resetsAt`. Say when the allowance resets and answer from stored data.
- **Hourly Run cap.** One connection may start only so many Runs an hour. The result has `limit_reached: true` and says when the next one is allowed. Pass that on.
- **Credits or subscription.** The error carries a billing URL. Give the user that exact URL.
- **Wrong stage for `edit_document`.** The reason names the item's stage. Offer `request_content_edit` instead.
- **Read-only role.** A viewer's connection lists no write tools. Say so, and that an owner or admin can change their role.
- **Connection refused or revoked.** Revoking is permanent. The user removes Camille from Claude Code and adds it again, then signs in in `/mcp`.
