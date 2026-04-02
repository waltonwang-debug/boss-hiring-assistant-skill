# Lobster Capability Contract

This skill assumes Lobster already has these general-purpose capabilities:

## Browser capabilities

- Use the user's normal logged-in Chrome session
- Read the active BOSS page
- Click low-risk controls when the skill explicitly asks for it

This skill adds the BOSS-specific extraction strategy and pause rules. It does not replace Lobster's general browser automation layer.

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
