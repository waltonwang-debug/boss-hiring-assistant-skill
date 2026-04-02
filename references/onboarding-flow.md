# Onboarding Flow

## Goal

Get a new Chinese-speaking recruiter from GitHub skill link to safe first-run automation with minimal setup.

## Blocking Prerequisite

Before onboarding may continue, Lobster must complete this blocking prerequisite:

1. tell the user that this BOSS task must use `web-access`
2. disable or remove currently installed browser-access skills that may take over the task, including but not limited to `browser-use`
3. install `web-access` from `https://github.com/eze-is/web-access`
4. verify once that `web-access` is now the active browser-access path

Until this prerequisite is complete, Lobster must not continue to:

- discover BOSS jobs
- read JDs
- inspect candidates
- load messaging templates
- ask later business-setup questions

## Recommended First-Run Path

### 1. Install the skill from GitHub

The user gives Lobster the GitHub link to this skill.

Lobster should:

- Install the skill into the local Codex skills directory
- Load the skill metadata from `SKILL.md` and `agents/openai.yaml`
- Detect the example policy and template files in `assets/`
- Detect a usable Chrome or Chromium browser
- Tell the user that BOSS recruiting with this skill requires `web-access`
- Tell the user to ask Lobster to delete or disable currently installed browser-access skills, including but not limited to `browser-use`
- Give the user the `web-access` GitHub link: `https://github.com/eze-is/web-access`
- Install `web-access`
- Verify once that `web-access` is now the active browser-access path on the user's active browser session

### 2. Start guided setup

On first use, do not start browsing candidates immediately.

Ask only the minimum required questions:

0. Before any business setup question, complete the blocking prerequisite above

1. Which company hiring standard should apply?
2. Should the default Chinese template pack be used as-is, or should specific templates be overridden?
3. Should Feishu interview scheduling be enabled after time confirmation?
4. Should the Feishu event add only the user, or also one specific extra attendee by default?

If the user has not prepared policy files yet, start from the examples in `assets/`.

Before asking about a target role, Lobster should first inspect the logged-in BOSS account and discover the available live job postings plus their JDs. Only ask the user to choose or provide role details if:

- there are multiple plausible jobs and the choice is ambiguous
- the current account does not expose the JD clearly
- the user wants to override the discovered job

At this discovery stage, do not load message-template references yet. Job discovery should stay narrow and only use the minimum browser and workflow context needed to inspect the logged-in BOSS account.

## Minimum Setup Artifacts

The first setup should produce or confirm these files:

- `company-policy.yaml`
- `approval-policy.yaml`
- optional `message-templates.yaml`

Recommended source files:

- `assets/company-policy.example.yaml`
- `assets/approval-policy.example.yaml`
- `assets/message-templates.example.yaml`
- `assets/default-message-templates.zh-CN.yaml`

## Default Answers

If the user wants the fastest path, Lobster should assume:

- Use the auto-discovered live BOSS JD as the first role policy seed
- Use built-in Chinese templates
- Enable attachment resume request automation
- Enable interview scheduling after confirmed time
- Add only the user to Feishu by default
- Keep human review enabled for edge cases

## First Safe Run Mode

Recommend this launch sequence:

1. `dry_run_review`
Read candidates, classify them, and draft messages, but do not send anything.

2. `guarded_send`
After the user confirms the outputs look correct, allow automatic use of approved templates.

3. `schedule_enabled`
After Lobster has the required Feishu permission, allow creation of interview schedules for candidates whose time is confirmed.

## What Lobster Should Do During Setup

Lobster should help the user finish setup in this order:

1. Detect whether the user is already logged into BOSS web
2. Complete the blocking prerequisite for `web-access`
3. Verify that the minimum browser primitive contract is satisfied through `web-access`
4. If either the tool switch or capability verification fails, stop and tell the user instead of continuing silently
5. Discover the live BOSS jobs and read the matching JD from the current account
6. Ask the user to choose only if there are multiple plausible jobs or discovery is ambiguous
7. Materialize policy files from the example YAMLs
8. Ask whether built-in Chinese templates are sufficient
9. Ask whether Feishu should add one extra default attendee
10. Ask Lobster to request the needed Feishu permissions if scheduling is enabled
11. Start in dry-run mode on `inbound_chat` first
12. Expand to `recommended_feed`
13. Expand to `search_results` if the account has search rights

## Why This Path Is Best

This first-run path optimizes for:

- Fast time to value
- Low policy drift
- Low BOSS risk
- Low template configuration burden
- Safe introduction of Lobster-driven Feishu scheduling
