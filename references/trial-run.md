# Trial Run

## Goal

Run a safe first real-world test after the skill is installed from GitHub.

## Suggested Order

1. Ask the user to open and log into the BOSS web app in Chrome
2. Confirm Lobster can access the active browser session
3. Start with one target JD
4. Load or create the company policy and approval policy
5. Use default Chinese templates unless the user explicitly wants overrides
6. Start with `inbound_chat` or a small subset of `recommended_feed`
7. Run in dry-run mode first
8. Show:
- extracted candidate summary
- decision result
- message preview
- Feishu bot schedule request preview, if applicable
9. Only after the user confirms the previews, allow automatic message sending

## What To Verify

- candidate source is identified correctly
- profile extraction is structurally correct
- decisions align with the user's hiring standard
- message tone is acceptable
- Feishu attendee defaults are correct

## Dry-Run Success Criteria

The first test is successful if:

- Lobster can read candidates from the target BOSS page
- Lobster can classify them consistently
- Lobster can draft the next message correctly
- Lobster can prepare the Feishu capability request when time is confirmed
- Lobster pauses instead of guessing when the page enters a risky state

## Recovery Checklist

If the first run fails, check these in order:

1. BOSS login state
- confirm the user is still logged into the BOSS web app
- confirm the page is not stuck on a login or verification screen

2. Page stability
- confirm Lobster is reading the expected BOSS page
- confirm the candidate detail is already visible before asking Lobster to extract more deeply
- retry with a single candidate rather than a larger batch

3. Risk-control interruption
- if BOSS shows verification, pause immediately
- let the user solve the interruption manually
- resume only after the page is back to a normal recruiting state

4. Feishu capability
- if no candidate has reached confirmed interview time yet, do not force Feishu bot configuration during onboarding
- when the first schedule is actually needed, confirm the Feishu bot is already configured
- when the first schedule is actually needed, confirm the default attendee choice then instead of during onboarding
- if the bot is not configured, ask the user to create it in Feishu Open Platform and provide `App ID` plus `App Secret`
- after Lobster configures the bot locally, then confirm Lobster has the required Feishu bot permissions
- confirm the default attendee settings are correct
- generate the Feishu bot schedule request preview before executing
- do not retry by switching to API, OAuth, or web UI paths
- if the bot lacks calendar permission, guide the user back to Feishu Open Platform to add it, then retry through the same bot path
- do not open Feishu Open Platform pages on the user's behalf during this process

5. Policy mismatch
- confirm the selected role policy actually matches the current JD
- confirm salary note, template mode, and approval thresholds are not too strict
