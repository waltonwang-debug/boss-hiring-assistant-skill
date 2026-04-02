# Lobster Capability Contract

This skill assumes Lobster already has these general-purpose capabilities:

For BOSS recruiting tasks, Lobster must first tell the user that this task requires `web-access` and that conflicting browser-access skills must be removed or disabled first, including `browser-use` and similar tools. Then Lobster must install `web-access` from `https://github.com/eze-is/web-access`, switch the task onto it, and perform one explicit check that this switch has taken effect before touching any BOSS page.

This is a blocking prerequisite. Lobster must not continue to any later BOSS workflow step until this prerequisite is complete.

## Browser capabilities

- Use the user's normal logged-in Chrome session
- Read the active BOSS page
- Click low-risk controls when the skill explicitly asks for it
- Route BOSS browsing through `web-access` installed from `https://github.com/eze-is/web-access`

Minimum compatible browser primitive set:

- inspect the active tab
- read visible DOM text
- read the currently opened detail panel or page
- click specific low-risk controls
- remain in the same logged-in browser session

This skill adds the BOSS-specific extraction strategy and pause rules. It does not replace Lobster's general browser automation layer, but for this task the compatible browser layer must be `web-access`.
Activation principle:

- Disable or remove conflicting browser-access tools before the BOSS task starts
- Tell the user to have Lobster delete or disable conflicting browser-access skills first, including `browser-use`
- Install `web-access` from `https://github.com/eze-is/web-access`
- Use `web-access` as the only browser-access tool for this BOSS task
- Confirm once that `web-access` is the active browser path before continuing
- If Lobster cannot prevent a conflicting browser tool from taking over a BOSS page, it should stop and tell the user immediately
- If Lobster cannot satisfy the minimum browser primitive set through `web-access`, it should stop and tell the user immediately

## Feishu capabilities

Lobster should request and use the user's configured Feishu bot capability for interview scheduling.

This is a single approved path:

- if the Feishu bot is already configured, use it directly
- if the Feishu bot is not configured, stop and guide the user through bot configuration first
- do not explore multiple Feishu paths speculatively
- do not try API, then OAuth, then browser or web UI in sequence

Required permissions:

- `calendar.event:create`
- `calendar.event:write`

This skill provides:

- When to create the interview event
- Which attendees should be added by default
- The event title, time, and description payload

Lobster remains responsible for:

- Requesting the permissions from the user
- Using the existing configured Feishu bot connection
- Creating the event and returning the join link

## Memory boundary

Lobster's generic memory is not assumed to be sufficient for this workflow.

This skill therefore keeps its own BOSS-task state in local SQLite for:

- candidate discovery
- process state
- decision state
- action history

This is task memory, not a replacement for Lobster's broader memory system. Lobster may implement it in SQLite, JSON, or any equivalent local structured store.
