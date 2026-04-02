# Lobster Capability Contract

This skill assumes Lobster already has these general-purpose capabilities:

When Lobster also has a generic browser skill such as `browser-use`, that generic skill should not become the default controller for BOSS recruiting tasks. This skill should override the generic route for BOSS pages because the pacing, extraction, and pause rules are site-specific.

## Browser capabilities

- Use the user's normal logged-in Chrome session
- Read the active BOSS page
- Click low-risk controls when the skill explicitly asks for it

This skill adds the BOSS-specific extraction strategy and pause rules. It does not replace Lobster's general browser automation layer.
Fallback principle:

- Use this skill first for all BOSS recruiting pages
- Use a generic browser skill only as a fallback when this skill's BOSS-specific path cannot complete a required read or click
- If a fallback is used, keep the same conservative pacing and pause conditions

## Feishu capabilities

Lobster should request and use the user's existing Feishu capability for interview scheduling.

Required permissions:

- `calendar.event:create`
- `calendar.event:write`

This skill provides:

- When to create the interview event
- Which attendees should be added by default
- The event title, time, and description payload

Lobster remains responsible for:

- Requesting the permissions from the user
- Using the existing Feishu app or bot connection
- Creating the event and returning the join link

## Memory boundary

Lobster's generic memory is not assumed to be sufficient for this workflow.

This skill therefore keeps its own BOSS-task state in local SQLite for:

- candidate discovery
- process state
- decision state
- action history

This is task memory, not a replacement for Lobster's broader memory system. Lobster may implement it in SQLite, JSON, or any equivalent local structured store.
