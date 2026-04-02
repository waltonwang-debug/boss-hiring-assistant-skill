# Chat Template Strategy

## Recommendation

Use a hybrid strategy:

- Built-in default templates for immediate usability
- Optional user overrides for company tone and special workflows
- Bounded generation only for low-risk FAQ replies that do not match a fixed template

Do not require users to author all templates before the skill is usable.
Do not allow unrestricted free-form generation for first contact, attachment requests, interview scheduling, or rejection messages.

## Why This Strategy Fits Best

For an open GitHub skill, forcing users to configure every template creates onboarding friction and lowers adoption.

For BOSS chat safety, relying on fully free-form generation creates these risks:

- Tone drift across candidates
- Accidental policy violations
- Inconsistent asks around resumes and interview links
- Higher chance of triggering user distrust or platform complaints

The best balance is:

1. Ship a strong default template pack
2. Let users override only what they care about
3. Keep fallback generation constrained and auditable

The recommended default Chinese pack for this skill lives in `assets/default-message-templates.zh-CN.yaml`.

## Template Modes

Recommended mode:

- `builtin_with_optional_overrides`

Supported conceptual modes:

- `builtin_only`
- `builtin_with_optional_overrides`
- `user_templates_required`

Recommendation:

- Default to `builtin_with_optional_overrides`
- Avoid `user_templates_required` for public adoption

## Built-in Template IDs

Use stable IDs so policy files and tools can reference them safely:

- `request_attachment_resume`
- `attachment_followup`
- `interview_intent_probe`
- `collect_availability`
- `confirm_interview_time`
- `send_feishu_schedule_link`
- `candidate_reject_polite`
- `faq_role_intro`
- `faq_salary_range`
- `faq_interview_process`
- `uncertain_answer_escalate`

## Variable Contract

Templates should be parameterized rather than regenerated from scratch.

Example variables:

```json
{
  "candidate_name": "张三",
  "job_title": "高级产品经理",
  "company_name": "示例公司",
  "salary_note": "30-45k x 14",
  "interview_time": "周三下午 3:00",
  "feishu_link": "https://..."
}
```

## Bounded Generation

Allow fallback generation only when all of the following are true:

- The candidate message is low risk
- No fixed template applies
- The topic is inside the approved FAQ scope
- The reply can be constrained by explicit generation rules

Required constraints:

- Keep it short
- Keep it professional and warm
- Do not request private contact information by default
- Do not make compensation or offer promises
- Do not invent company policy

## Override Model

User overrides should be partial, not all-or-nothing.

Recommended behavior:

- If a user supplies `request_attachment_resume`, use their version
- If they do not supply `confirm_interview_time`, keep the built-in version
- Always preserve the same variable names across built-in and overridden templates

## Suggested Onboarding Question

Ask one concise question during setup:

`Do you want to use the default BOSS chat templates, or override specific templates with your own company wording?`

If the user chooses overrides, collect only the templates they want to change.
