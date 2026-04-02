# Browser Execution

## Goal

Use `web-access` on the user's real Chrome or Chromium session while minimizing the chance of triggering BOSS risk controls.

Before any BOSS action starts, Lobster must tell the user that this task requires `web-access`, ask the user to have Lobster remove or disable conflicting browser-access skills including `browser-use`, and then install `web-access` from `https://github.com/eze-is/web-access`. If that switch cannot be enforced and verified once, stop and report it to the user before touching the BOSS page.

This is a blocking prerequisite, not a suggestion. Do not continue to job discovery or candidate inspection before it is complete.

## Environment Checks

Before doing anything on BOSS, Lobster should check:

1. A usable Chrome or Chromium browser exists locally
2. The user is already logged into the BOSS web app
3. The page is not showing verification, login expiry, or risk interruption
4. The current tab belongs to a supported BOSS workflow surface
5. Conflicting browser-access tools such as `browser-use` have been disabled or removed for this task
6. `web-access` installed from `https://github.com/eze-is/web-access` is the active browser-access tool for this task
7. The currently available `web-access` path can satisfy the minimum browser primitives defined by this skill

If Chrome or Chromium is missing, then and only then ask the user to install one.
If the browser capability contract is missing, or the switch to `web-access` is not confirmed, stop instead of improvising.

## Supported Surfaces

- Chat list / inbound candidates
- Recommended candidate feed
- Search results, only if the account already has search rights
- Candidate detail panel or detail page

## Extraction Order

1. Read visible card fields first
2. Read the currently opened candidate detail if it is already visible
3. Use stable selectors where possible
4. Avoid screenshots unless the page cannot be parsed structurally

At the job-discovery stage, avoid loading message-template context. Message-template material belongs to the later communication stage, not the initial BOSS job and JD discovery stage.

## Tool Discipline

Do not hop across unrelated browser-access methods during one BOSS task.

Allowed pattern:

- disable conflicting browser tools for the task
- choose the `web-access` path
- stay on the user's existing logged-in browser session
- perform the minimum reads and clicks needed for the current step

Disallowed pattern:

- trying multiple unrelated browser-access approaches speculatively
- letting `browser-use` or another generic browser tool silently take over the BOSS task
- falling back to broad web exploration without first reporting the limitation
- changing execution mode silently in the middle of a BOSS step

## Suggested Selectors

These are heuristic selectors for Lobster to try in order and adapt as the site changes:

### Candidate cards

- `[data-geek-id]`
- `.geek-item`
- `.candidate-card`
- `.job-card-wrapper`

Fields:

- name: `.name`, `.candidate-name`, `h3`, `h4`
- title: `.job-name`, `.expect-position`, `.title`
- city: `.city`, `.location`, `.expect-city`
- education: `.education`, `.degree`
- experience: `.experience`, `.exp`
- salary: `.salary`, `.expect-salary`

### Candidate detail

- `.resume-detail-wrap`
- `.geek-resume-wrap`
- `.candidate-detail`
- `.resume-detail`

Fields:

- name: `.name`, `h1`, `h2`
- city: `.city`, `.location`
- education: `.education`, `.degree`
- years: `.experience`, `.exp`
- target role: `.expect-position`, `.job-name`
- salary: `.expect-salary`, `.salary`
- skill tags: `.tag`, `.label`, `.skill-tag`
- work history: `.experience-item`, `.work-item`, `.resume-item`

## Pacing Rules

Recommended feed does not need a business-side total browsing cap, but the interaction pattern should still be low-risk:

- Prefer sequential, one-candidate-at-a-time progression
- Avoid machine-like rapid opening and closing of profiles
- Avoid bursty multi-action sequences
- Prefer reading before clicking
- Prefer opening promising candidates instead of every card
- Add natural pauses between detail views and outbound messages

## Stability Preference

When there are multiple ways to complete the same BOSS task, prefer the most observable and conservative path:

- sequential over parallel
- already-open detail views over forced navigation
- stable visible content over deep page traversal
- explicit step confirmation over chained hidden actions

This reduces the chance of unexpected redirects, login instability, or abnormal behavior patterns.

## Immediate Pause Conditions

Pause the workflow and ask the user to step in if any of these appear:

- verification prompt
- login expiration
- risk warning
- unusual redirect loop
- extraction instability across multiple candidates

## Execution Principle

The skill should treat BOSS as a high-sensitivity site.

The goal is not to bypass anti-automation systems.
The goal is to behave conservatively enough that Lobster can assist real recruiting work without creating obviously abnormal browser patterns.

## Execution Boundary

For this skill, the browser-access path is `web-access`.

If Lobster cannot get the task onto `web-access`, or cannot verify that conflicting browser tools have been disabled for the task, it must pause and report the limitation instead of continuing with another browser tool.
