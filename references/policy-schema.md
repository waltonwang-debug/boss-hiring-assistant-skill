# Policy Schema

Use two YAML files:

- `company-policy.yaml`
- `approval-policy.yaml`

The examples in `assets/` are the recommended starting point.

## `company-policy.yaml`

Purpose:

- Encode stable hiring standards that should not drift with the model
- Define role-independent company talent principles
- Define role-specific hard requirements and preferred signals

Recommended shape:

```yaml
version: "1"
company:
  name: "Example Co"
  timezone: "Asia/Shanghai"
talent_principles:
  required_traits:
    - "ownership"
    - "clear communication"
  avoid_traits:
    - "blame shifting"
roles:
  - role_id: "backend-engineer"
    role_title: "Backend Engineer"
    jd_summary: "Build SaaS backend systems"
    must_have:
      years_experience_min: 3
      skills:
        - "java"
        - "spring boot"
      cities:
        - "Shanghai"
      education_min: "bachelor"
    preferred:
      industries:
        - "saas"
      companies:
        - "enterprise software"
      signals:
        - "to_b"
        - "microservices"
    hard_reject:
      missing_skills:
        - "java"
      excluded_backgrounds:
        - "pure internship only"
      max_job_changes_recent_years: 3
    review_focus:
      - "stability"
      - "depth of system design"
      - "business ownership"
    messaging:
      attachment_resume_required_by_default: true
      approved_salary_range_note: "30-45k x 14"
      template_pack: "default-zh-cn"
```

Field guidance:

- `talent_principles.required_traits`: soft standards that should influence screening and interview planning
- `roles[].must_have`: hard entry conditions
- `roles[].preferred`: positive ranking signals
- `roles[].hard_reject`: direct elimination criteria
- `roles[].review_focus`: points to verify in human review or interview
- `roles[].messaging`: bounded facts that may appear in chat templates
- `roles[].messaging.template_pack`: choose a built-in template pack or a user-defined pack name

## `approval-policy.yaml`

Purpose:

- Decide what may be automated
- Decide what requires approval
- Decide what must be reviewed manually
- Control Feishu scheduling defaults

Recommended shape:

```yaml
version: "1"
automation:
  allow_auto_reject: true
  allow_auto_pass_to_chat: true
  allow_auto_request_attachment: true
  allow_auto_schedule_feishu_interview: true
thresholds:
  auto_pass_min_score: 0.82
  auto_pass_min_confidence: 0.80
  human_review_score_floor: 0.55
  approval_required_for:
    - "salary_exception"
    - "level_exception"
    - "location_exception"
    - "headcount_exception"
  always_review_for:
    - "high_value_candidate"
    - "contradictory_resume"
    - "sparse_evidence"
chat_policy:
  template_mode: "builtin_with_optional_overrides"
  allowed_auto_templates:
    - "request_attachment_resume"
    - "attachment_followup"
    - "interview_intent_probe"
    - "collect_availability"
    - "confirm_interview_time"
    - "send_feishu_schedule_link"
    - "faq_role_intro"
    - "faq_interview_process"
  escalate_topics:
    - "salary_negotiation"
    - "benefits_exception"
    - "complaint"
    - "hostile_language"
  allow_bounded_generation_for_unmapped_faq: true
  generation_rules:
    max_sentences: 4
    keep_tone: "professional_warm"
    must_not_request_private_contact: true
    must_not_make_offer_promises: true
feishu:
  lobster_capability_required: true
  create_todo_items: false
  default_add_user_self: true
  ask_once_for_extra_default_attendee: true
  required_permissions:
    - "calendar.event:create"
    - "calendar.event:write"
  extra_default_attendee:
    enabled: false
    open_id: ""
    email: ""
```

Field guidance:

- `automation.allow_auto_schedule_feishu_interview`: enable only after the user is comfortable with the workflow
- `thresholds.auto_pass_*`: control when the skill may proceed without approval
- `thresholds.always_review_for`: protect against costly false negatives
- `chat_policy.template_mode`: recommended value is `builtin_with_optional_overrides`
- `chat_policy.allowed_auto_templates`: hard allowlist for automatic messages
- `chat_policy.allow_bounded_generation_for_unmapped_faq`: allow short controlled replies only for low-risk questions that do not map to a fixed template
- `chat_policy.generation_rules`: constraints for any fallback generation path
- `feishu.lobster_capability_required`: require Lobster's existing Feishu capability instead of a separate client inside this skill
- `feishu.default_add_user_self`: always keep enabled in this version
- `feishu.ask_once_for_extra_default_attendee`: ask the user once during onboarding, not on every candidate
- `feishu.required_permissions`: the Feishu permissions Lobster should request from the user
- `feishu.extra_default_attendee`: store one default extra attendee if the user requests it

## Decision Contract

Every candidate decision should be serializable into:

```json
{
  "decision": "review_human",
  "score": 0.73,
  "confidence": 0.79,
  "reasons": [
    "Relevant SaaS experience",
    "Evidence for system design is limited",
    "Recent tenure length needs verification"
  ],
  "risk_tags": [
    "sparse_evidence"
  ],
  "next_action": "request_attachment_resume"
}
```

## Onboarding Questions

During setup, gather these values before enabling automation:

1. Which role policy should apply to each BOSS job?
2. Which message templates are approved for automatic use?
3. Should Feishu scheduling be automatic after a confirmed time?
4. Should the user's own Feishu account be the only default attendee?
5. If not, which one extra Feishu attendee should be added by default?
6. Should the built-in message templates be used as-is, or should specific templates be overridden?

Keep onboarding concise and store the answers in the policy files.
