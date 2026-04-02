# BOSS Hiring Workflow

## Scope

This skill supports the BOSS Zhipin web app only. It does not assume desktop client automation.

Primary business actions:

1. Read candidates from `inbound_chat`, `recommended_feed`, and `search_results`
2. Open the candidate profile and extract a structured profile
3. Classify the candidate against the JD and company policy
4. Continue with one of:
- reject
- human review
- approval request
- guarded chat progression
5. Request an attachment resume when needed
6. Ask Lobster to create a Feishu interview event after candidate intent and time are confirmed
7. Send the Feishu event link back in the BOSS chat

## Default Operating Cadence

Use a fixed cadence unless the user changes it:

- active screening window: 09:00 to 17:00 local time
- screening cycle: once every hour
- summary cycle: once every hour

Within each hourly cycle, Lobster should:

1. scan the supported candidate sources
2. update candidate classifications and reasons
3. prepare one consolidated summary for the user
4. ask which candidates may continue to the next communication step
5. ask which candidates may proceed toward interview scheduling

Outside the active screening window, do not continue scheduled scanning unless the user explicitly overrides the default.

## Candidate Sources

### `inbound_chat`

Candidates who have already entered the recruiting chat list through direct delivery or direct contact.

Recommended handling:

- Highest priority for MVP
- Safest source for automation
- Read the candidate card before opening full details

### `recommended_feed`

Candidates recommended by BOSS based on the account, behavior, and open roles.

Recommended handling:

- No business-side total browsing cap
- Keep operational pacing controlled
- Open only promising profiles instead of rapidly opening every card

### `search_results`

Candidates returned by BOSS search when the account has the required paid search rights.

Recommended handling:

- Use explicit filters tied to the role
- Treat search as a higher-intent sourcing flow
- Apply the same policy pack as other sources

## Extraction Strategy

Use this extraction order:

1. Visible card fields
2. In-page DOM extraction through Lobster's browser reading capability
3. Light CDP evaluation if DOM access is insufficient
4. Visual fallback only if structured access fails

Extract the candidate profile into this shape:

```json
{
  "candidate_id": "boss_xxx",
  "source": "recommended_feed",
  "basic_info": {
    "city": "Shanghai",
    "education": "Bachelor",
    "years_of_experience": 5,
    "work_status": "open_to_opportunities"
  },
  "target_info": {
    "target_role": "Backend Engineer",
    "target_city": "Shanghai",
    "target_salary_range": "30-45k"
  },
  "career_history": [
    {
      "company": "Example Inc",
      "title": "Senior Engineer",
      "period": "2022-01 to present",
      "industry": "SaaS"
    }
  ],
  "evidence_tags": [
    "java",
    "microservices",
    "to_b",
    "team_lead"
  ],
  "risk_signals": [
    "job_hopping"
  ],
  "chat_context": {
    "has_chat_thread": true,
    "has_attachment_resume": false,
    "last_message_direction": "candidate"
  }
}
```

## Decision Flow

### Rule-first pass

Run hard rules before the model:

- Must-have skill missing
- Hard reject condition hit
- Salary mismatch beyond allowed range
- Location mismatch beyond allowed rule
- Level mismatch beyond allowed rule

### Model-assisted pass

Use the model only for boundary cases. Require:

- `decision`
- `score`
- `confidence`
- `reasons`
- `risk_tags`
- `next_action`

Allowed decisions:

- `reject_hard`
- `reject_soft`
- `review_human`
- `approval_required`
- `pass_to_chat`

## Hourly Summary Format

The default summary should be grouped by candidate decision:

- `pass_to_chat`
- `approval_required`
- `review_human`
- `reject_soft`
- `reject_hard`

For each candidate, include:

- candidate name or stable identifier
- source
- current decision
- concise reasons for the decision
- Lobster's recommended next step

The summary should end with two explicit decision requests to the user:

1. which candidates may continue to the next communication step, such as requesting an attachment resume
2. which candidates may proceed to interview scheduling after time confirmation

If Lobster has a clear recommendation, include it next to the candidate instead of silently waiting for the user to infer the next step.

## Process State Machine

```text
new
-> card_scanned
-> profile_opened
-> profile_extracted
-> decision_pending

decision_pending
-> reject_hard
-> reject_soft
-> review_human
-> approval_required
-> pass_to_chat

pass_to_chat
-> attachment_requested
-> attachment_received
-> attachment_forwarded
-> feishu_review_requested
-> feishu_review_done
-> interview_intent_checking
-> interview_intent_confirmed
-> time_slot_collecting
-> time_confirmed
-> feishu_event_created
-> schedule_link_sent
-> process_completed
```

## Chat Progression

### 1. Request attachment resume

Use when:

- The visible profile lacks enough evidence
- The user wants internal review
- The candidate is promising but details are incomplete

Goal:

- Request a PDF attachment resume
- Avoid asking for private contact details by default

### 2. Internal review sync

After the attachment is received:

- Send the candidate packet to the user's own Feishu account
- Wait for the user's internal decision if review is required

### 3. Confirm interview intent

Use only after:

- The candidate passes automatically
- Or the user marks the candidate as approved for interview

### 4. Collect time availability

Ask for availability first. Confirm the exact time before creating the Feishu event.

### 5. Send Feishu schedule link

Use only after Feishu event creation succeeds.

Message must include:

- Confirmed interview time
- Feishu event or join link
- Simple instructions for attendance
- A fallback instruction if the candidate cannot attend

## FAQ Boundaries

Safe for automatic reply:

- Role summary
- Work location
- Basic interview process
- Base salary range if the user approved a fixed range

Escalate to the user:

- Salary negotiation
- Exceptional benefits requests
- Sensitive company disputes
- Emotional or hostile messages
- Any answer requiring non-public company judgment

## Feishu Scheduling Rules

Use Lobster's existing Feishu capability rather than implementing a separate Feishu client inside this skill.

This skill creates only interview schedules, not todo items.

Default behavior:

- Add the user themself to the Feishu event

When the first interview schedule is actually needed:

- ask whether only the user should be added, or one extra fixed attendee should also be added
- if the Feishu bot is not configured, ask the user to create it in Feishu Open Platform and return with `App ID` plus `App Secret`
- after Lobster configures the bot locally, explicitly tell the user to go to the Feishu bot chat and ask the bot to test-create a schedule there
- use that bot-chat flow to complete the needed calendar authorization
- if event creation still fails because calendar permission is missing, guide the user to add the missing permission and retry through the same bot path

## Tool Boundary

Treat these as the desired tool contracts:

```ts
list_candidate_sources(job_id)
list_inbound_candidates(job_id, cursor)
list_recommended_candidates(job_id, cursor)
list_search_candidates(job_id, filters, cursor)

scan_candidate_card(candidate_id)
open_candidate_profile(candidate_id)
extract_candidate_profile(candidate_id)

classify_candidate(candidate_profile, company_policy, approval_policy)
mark_candidate_decision(candidate_id, decision, reason)

get_chat_thread(candidate_id)
read_recent_messages(thread_id)
send_boss_message(thread_id, template_id, variables)

request_attachment(candidate_id)
sync_attachment_to_feishu(candidate_id)

create_feishu_review(candidate_packet)
call_lobster_feishu_capability(event_payload)

pause_candidate_flow(candidate_id, reason)
request_human_review(candidate_id, packet)
```

## Pause Conditions

Pause immediately when:

- BOSS asks for repeated verification
- The account appears logged out
- The site shows risk-control interruption
- Structured extraction becomes unstable
- Lobster does not have the required Feishu permission during scheduling
