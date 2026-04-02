# Browser Execution

## Goal

Use Lobster's existing browser capability on the user's real Chrome or Chromium session while minimizing the chance of triggering BOSS risk controls.

## Environment Checks

Before doing anything on BOSS, Lobster should check:

1. A usable Chrome or Chromium browser exists locally
2. The user is already logged into the BOSS web app
3. The page is not showing verification, login expiry, or risk interruption
4. The current tab belongs to a supported BOSS workflow surface

If Chrome or Chromium is missing, then and only then ask the user to install one.

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
