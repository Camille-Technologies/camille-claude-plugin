# Camille for Claude Code

A Claude Code plugin marketplace with one plugin, `camille`. It connects Claude Code to [Camille](https://hirecamille.com) and teaches Claude how to work with her.

The plugin adds:

- **An MCP server.** It points at `https://hirecamille.com/mcp`. You sign in with your Camille account and pick an organization. Claude then acts as you in that organization, with your role.
- **The `camille` skill.** Camille's terms (Tasks, Triage, Plan Review, Final Review, Documents, Runs, Conversations) and how to use her tools. Claude hands production work to Camille instead of writing Articles or Outlines itself, and reads the data Camille already stored before spending paid lookups.
- **The `camille-review` skill.** Goes through the items waiting on you at Plan Review and Final Review. Claude reads each Plan or Deliverable, suggests edits, applies the ones you agree to, and approves when you say so.

## Install

In Claude Code:

```
/plugin marketplace add Camille-Technologies/camille-claude-plugin
/plugin install camille@camille-technologies
```

The second command opens the plugin in the `/plugin` panel, where you confirm the install. From a shell, the same two steps are:

```bash
claude plugin marketplace add Camille-Technologies/camille-claude-plugin
claude plugin install camille@camille-technologies
```

Start a new session, or run `/reload-plugins`, to load it.

## Sign in

1. Run `/mcp` and select `plugin:camille:camille`.
2. Choose to authenticate. Your browser opens Camille's sign-in page.
3. Sign in, pick the organization this connection is for, and approve.

The connection shows up in Camille under Settings, Connect Claude. A viewer gets the read tools only. Editors and above get the rest.

Then ask Claude something like "what needs me in Camille?" or "ask Camille for a post about winter tyre storage". You can also run the skills directly as `/camille:camille` and `/camille:camille-review`.

## A second organization

Each connection belongs to the organization you picked when you signed in, and it can't move. For another organization, add Camille again under another name. That registers a new client, which you then sign in to and point at the other organization.

Claude Code treats servers that share a URL as one, and a server you add yourself wins over the plugin's. So once you add Camille by hand, the plugin's own server stops loading. Add one server per organization:

```bash
claude mcp add --transport http --scope user camille-acme https://hirecamille.com/mcp
claude mcp add --transport http --scope user camille-globex https://hirecamille.com/mcp
```

Sign in to each one in `/mcp` and pick the matching organization. The skills keep working, since they refer to Camille's tools by name. The plugin's old connection is no longer used, so revoke it in Settings, Connect Claude.

## Revoking and reconnecting

You can revoke a connection in Settings, Connect Claude. An org owner or admin can revoke anyone's. It stops working on Claude's next call.

Revoking is permanent for that assistant. Signing in again with the same server won't bring it back. To reconnect, remove Camille from Claude Code and add it again. Removing it deletes the client Claude Code registered, so adding it registers a new one:

```bash
claude mcp remove camille-acme --scope user
claude mcp add --transport http --scope user camille-acme https://hirecamille.com/mcp
```

If you were using the plugin's server, add one by hand as in the previous section. It takes over from the plugin's server.

## Paid lookups and limits

Live Google results (`search_serp`) and keyword volume and difficulty (`lookup_keywords`) count against a daily allowance per organization, shared with Camille's in-app chat. A query fetched in the last 24 hours is free. When the allowance runs out, Claude tells you when it resets.

Asking Camille for a post, a scan, a report or research starts a Run, which uses your organization's credits like any other Run. One connection can start a limited number of Runs an hour.

## For maintainers

Validate after every change:

```bash
claude plugin validate .
claude plugin validate ./plugins/camille
```

To try a local change, add this directory as a marketplace (`claude plugin marketplace add ./`) and install from it.

To test against a local Camille, keep production out of the session. A local server has a different address, so it doesn't replace the plugin's production server. Both stay connected, and a test request could start real Runs or spend real credits.

1. Add the local server with your dev server's address, for example `claude mcp add --transport http camille-local http://localhost:3000/mcp`.
2. In `/mcp`, select `plugin:camille:camille` and disable it.
3. Still in `/mcp`, authenticate `camille-local` and pick a test organization.
4. Tell Claude to use `camille-local` before you ask for anything. Check in `/mcp` that it's the only Camille server connected.

Turn the plugin's server back on in `/mcp` when you're done.

`plugins/camille/.claude-plugin/plugin.json` sets `version`, so installed copies stay on that version until it changes. Bump it with every release.
