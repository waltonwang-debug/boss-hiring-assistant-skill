---
name: boss-hiring-assistant
description: Assist BOSS Zhipin recruiting workflows with a web-first, policy-driven, human-in-the-loop skill. Use when Lobster needs to work on the BOSS Zhipin web app for inbound candidates, recommended candidates, or search-result candidates; extract structured resume information with low-risk browser access; classify candidates against a job JD and company hiring standards; chat with guarded Chinese templates; request attachment resumes; maintain BOSS-task memory; and ask Lobster to use its existing Feishu capability to create interview schedules after candidate intent and time are confirmed.
---

# Boss Hiring Assistant

## Overview

Use this skill to help Lobster run BOSS Zhipin recruiting workflows through the user's real Chrome or Chromium browser session. Keep the workflow conservative, policy-driven, and human-in-the-loop.

This skill is a Boss-specific strategy pack. It is not a standalone program and it does not replace Lobster's general browser or Feishu capabilities.

When the task is about BOSS Zhipin recruiting, this skill should take precedence over any generic browser skill. Do not default to a general browser workflow such as `browser-use` for BOSS pages unless this skill explicitly says a fallback is needed.

## Core Responsibilities

Use this skill to provide:

- BOSS-specific browser reading and pacing rules
- BOSS-specific candidate source definitions
- Job JD plus company-policy-driven screening
- Human-review and approval boundaries
- Default Chinese recruiting templates with optional overrides
- BOSS-task memory structure
- The contract for when Lobster should call its existing Feishu capability

## Execution Rules

1. Check environment first.
Confirm:
- a usable Chrome or Chromium browser exists
- the user is logged into the BOSS web app
- the current page is a supported BOSS workflow page
- Lobster has or can request the needed Feishu permission later if scheduling is enabled

After login is confirmed, Lobster should first discover the user's live BOSS job postings and read the corresponding JD from the current account. Do not ask the user to manually type the role or JD unless automatic discovery fails or there are multiple plausible jobs that require user selection.

2. Use the safest browser path available.
Prefer:
- visible page fields
- stable DOM extraction
- low-risk interaction
- sequential, observable steps instead of bursty multi-step automation

Avoid heavy screenshot interpretation when the page can be read structurally.
For BOSS pages, do not hand control to a generic browser skill by default. Apply the BOSS-specific execution rules in this skill first, and only use a generic browser fallback if the BOSS-specific path is blocked and the user still wants to continue.

3. Keep BOSS behavior conservative.
- reuse the user's normal logged-in browser session
- do not export or import cookies
- do not behave like a bursty bot
- prefer one-candidate-at-a-time reading and progression
- pause immediately on verification, login loss, risk prompts, or repeated extraction instability

4. Decide with policy first.
Load:
- the target job JD discovered from the logged-in BOSS account when possible
- [company-policy.example.yaml](./assets/company-policy.example.yaml) or the user's derived company policy
- [approval-policy.example.yaml](./assets/approval-policy.example.yaml) or the user's derived approval policy

Use rules first and only use model judgment for boundary cases.

5. Advance with guarded messaging.
Use the built-in Chinese templates by default. Allow the user to override only the templates they care about.

6. Keep BOSS-task memory.
Maintain candidate state and action history in Lobster-managed local structured storage so the workflow can resume safely.

7. Use Lobster's Feishu capability instead of a separate Feishu client.
When interview time is confirmed, build a Feishu schedule request and ask Lobster to execute it with its existing Feishu capability.

## Supported Candidate Sources

- `inbound_chat`
- `recommended_feed`
- `search_results`

## Decision Outcomes

- `reject_hard`
- `reject_soft`
- `review_human`
- `approval_required`
- `pass_to_chat`

## Process States

- `new`
- `card_scanned`
- `profile_opened`
- `profile_extracted`
- `decision_pending`
- `attachment_requested`
- `attachment_received`
- `attachment_forwarded`
- `interview_intent_checking`
- `interview_intent_confirmed`
- `time_slot_collecting`
- `time_confirmed`
- `feishu_event_created`
- `schedule_link_sent`
- `candidate_declined`
- `no_response_followup`
- `process_completed`
- `process_paused`

## Human Escalation

Escalate when:

- the candidate is valuable and a false negative would be costly
- the evidence is incomplete or contradictory
- compensation, level, location, or headcount needs an exception
- the candidate asks something outside approved FAQ boundaries
- BOSS shows any risk-control or verification interruption
- Lobster does not yet have the required Feishu permission

## References

- Use [browser-execution.md](./references/browser-execution.md) for Chrome/Chromium checks, extraction order, selector guidance, and pacing rules.
- Use [boss-workflow.md](./references/boss-workflow.md) for the recruiting state machine and progression logic.
- Use [chat-templates.md](./references/chat-templates.md) for the template strategy.
- Use [lobster-capabilities.md](./references/lobster-capabilities.md) for the boundary between this skill and Lobster's own browser and Feishu abilities.
- Use [onboarding-flow.md](./references/onboarding-flow.md) for the first-run setup path.
- Use [policy-schema.md](./references/policy-schema.md) for hiring policy and approval policy schemas.
- Use [task-memory.md](./references/task-memory.md) for the BOSS-task memory model.
- Use [trial-run.md](./references/trial-run.md) for the first real dry-run checklist.
