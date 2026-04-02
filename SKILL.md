---
name: boss-hiring-assistant
description: Assist BOSS Zhipin recruiting workflows with a web-first, policy-driven, human-in-the-loop skill. Use when Lobster needs to work on the BOSS Zhipin web app for inbound candidates, recommended candidates, or search-result candidates; extract structured resume information with low-risk browser access; classify candidates against a job JD and company hiring standards; chat with guarded Chinese templates; request attachment resumes; maintain BOSS-task memory; and ask Lobster to use its existing Feishu capability to create interview schedules after candidate intent and time are confirmed.
---

# Boss Hiring Assistant

## Overview

Use this skill to help Lobster run BOSS Zhipin recruiting workflows through the user's real Chrome or Chromium browser session. Keep the workflow conservative, policy-driven, and human-in-the-loop.

This skill is a Boss-specific strategy pack. It is not a standalone program and it does not replace Lobster's general browser or Feishu capabilities.

## Mandatory First-Run Prerequisite

Before any BOSS workflow starts, Lobster must treat the following as a blocking prerequisite:

1. disable or remove currently installed browser-access skills that may take over the task, including but not limited to `browser-use`
2. install `web-access` from `https://github.com/eze-is/web-access`
3. switch this BOSS task onto `web-access`
4. verify once that `web-access` is the active browser-access path for the task

If any of these steps fail, Lobster must stop and tell the user that the prerequisite is incomplete. Lobster must not continue into job discovery, candidate reading, messaging, or scheduling before this prerequisite is satisfied.

For Feishu scheduling, the integration path is also strict:

- use the user's configured Feishu bot path
- the user, not Lobster, must perform Feishu Open Platform setup
- if the Feishu bot is not configured yet, stop and ask the user to create the bot in Feishu Open Platform, obtain `App ID` and `App Secret`, and then provide them to Lobster for local configuration
- if the Feishu bot is already configured, call the bot with the required schedule-creation permission
- if schedule creation fails because of missing permission, guide the user step by step to Feishu Open Platform to add the calendar permission, then retry through the same bot path

For this Feishu setup path, Lobster must only provide instructions. Lobster must not open Feishu Open Platform pages, must not click through the setup flow, and must not attempt to complete platform configuration on the user's behalf.

Do not probe multiple Feishu integration methods. Do not try direct API, then OAuth, then web UI in sequence.

For BOSS Zhipin recruiting, the browser-access precondition is strict:

- disable or remove already installed generic browser-access tools for this task, including `browser-use` and similar tools
- install and use `web-access` as the browser-access tool for this task: `https://github.com/eze-is/web-access`
- verify once that the switch has actually taken effect before touching any BOSS page

If this precondition is not satisfied, Lobster must stop and tell the user instead of continuing.

## Core Responsibilities

Use this skill to provide:

- BOSS-specific browser reading and pacing rules
- BOSS-specific candidate source definitions
- Job JD plus company-policy-driven screening
- Human-review and approval boundaries
- Default Chinese recruiting templates with optional overrides
- BOSS-task memory structure
- The contract for when Lobster should call its existing Feishu capability

This skill defines process constraints and capability contracts. If Lobster does not currently have a compatible `web-access`-backed browser capability that can satisfy these contracts, it must stop and tell the user. It must not improvise with unrelated browser-access methods.

## Execution Rules

1. Check environment first.
Confirm:
- a usable Chrome or Chromium browser exists
- the user is logged into the BOSS web app
- the current page is a supported BOSS workflow page
- Lobster has or can request the needed Feishu permission later if scheduling is enabled
- installed browser-access tools that would take over BOSS browsing, including `browser-use`, have been disabled or removed for this task
- `web-access` is installed from `https://github.com/eze-is/web-access` and is the active browser-access tool for this task
- the browser-access switch to `web-access` has been checked once and confirmed before continuing

After login is confirmed, Lobster should first discover the user's live BOSS job postings and read the corresponding JD from the current account. Do not ask the user to manually type the role or JD unless automatic discovery fails or there are multiple plausible jobs that require user selection.
During this job-discovery stage, do not preload chat-template references or other later-stage messaging materials. Load only the minimum browser and workflow context needed to inspect the logged-in BOSS account and discover the live job plus JD.

2. Use the safest browser path available.
Prefer:
- visible page fields
- stable DOM extraction
- low-risk interaction
- sequential, observable steps instead of bursty multi-step automation

Avoid heavy screenshot interpretation when the page can be read structurally.
For BOSS pages, use `web-access` as the browser-access layer for this task. Do not hand control to `browser-use` or any other generic browser tool, and do not continue unless the precondition check confirms that `web-access` is now the active path.
Do not freely switch among unrelated browser-access methods. Choose the `web-access` path for the current BOSS task and stay within it unless the skill explicitly requires a pause.

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

7. Use Lobster's configured Feishu bot path only.
During onboarding, only decide whether Feishu scheduling is enabled. When interview time is first confirmed for a candidate, then check whether the Feishu bot is configured and ask the user about default attendees. If the bot is not configured, stop and tell the user the exact Feishu Open Platform steps they must perform manually, then wait for the user to return with `App ID` and `App Secret`. If the bot is configured but missing permissions, tell the user the exact permission steps they must perform manually in Feishu Open Platform. Do not open Feishu setup pages or attempt the setup on the user's behalf. After configuration is complete, build a Feishu schedule request and execute it through the Feishu bot path only.

## Required Browser Primitives

For BOSS recruiting tasks, Lobster should only proceed if it has browser capabilities equivalent to these primitives:

- inspect the active browser tab
- read visible DOM text from the current page
- read the currently opened candidate detail panel or page
- click a specific low-risk control when the skill explicitly calls for it
- keep working on the same logged-in browser session

These primitives must be provided through `web-access` for this task. If Lobster cannot map `web-access` to these primitives, or cannot confirm that conflicting browser tools have been disabled for the task, it must stop and explain that the required browser capability contract is not available.

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
