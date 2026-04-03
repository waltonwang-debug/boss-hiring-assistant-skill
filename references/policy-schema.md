# 策略文件结构

本 skill 使用两份 YAML：

- `company-policy.yaml`
- `approval-policy.yaml`

`assets/` 里的示例文件是推荐起点。

## `company-policy.yaml`

用途：

- 固化不会随模型漂移的招聘标准
- 定义公司层面的人才观
- 定义岗位层面的硬条件和偏好信号

推荐结构：

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

字段说明：

- `talent_principles.required_traits`：软性人才标准
- `roles[].must_have`：硬门槛
- `roles[].preferred`：加分信号
- `roles[].hard_reject`：直接淘汰条件
- `roles[].review_focus`：人工复核或面试重点
- `roles[].messaging`：聊天里允许使用的有限事实

## `approval-policy.yaml`

用途：

- 定义哪些动作可以自动执行
- 定义哪些动作必须审批
- 定义哪些情况必须人工复核
- 定义汇报节奏和飞书日程默认逻辑

推荐结构：

```yaml
version: "1"
automation:
  allow_auto_reject: true
  allow_auto_pass_to_chat: true
  allow_auto_request_attachment: true
  allow_auto_schedule_feishu_interview: true
reporting_window:
  enabled: true
  local_timezone: "Asia/Shanghai"
  active_days:
    - "monday"
    - "tuesday"
    - "wednesday"
    - "thursday"
    - "friday"
  start_time: "09:00"
  end_time: "17:00"
reporting:
  scan_interval_minutes: 60
  summary_interval_minutes: 60
  summary_format: "group_by_decision_with_reasons"
  include_recommended_next_step: true
  ask_user_to_confirm_chat_progression: true
  ask_user_to_confirm_interview_progression: true
thresholds:
  auto_pass_min_score: 0.82
  auto_pass_min_confidence: 0.80
  human_review_score_floor: 0.55
```

字段说明：

- `reporting_window`：默认运行时段，用户可覆盖
- `reporting`：扫描和汇报节奏
- `thresholds.auto_pass_*`：自动推进阈值
- `approval_required_for`：哪些异常必须审批
- `always_review_for`：哪些高风险情况必须人工复核
- `chat_policy`：自动消息范围和生成边界
- `feishu`：飞书 bot 相关默认逻辑

## 决策结果格式

每个候选人的决策都应可序列化为：

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

## onboarding 需要采集的值

在开启自动化之前，至少要明确：

1. 当前岗位用哪套岗位策略
2. 哪些模板允许自动发送
3. 是否启用飞书约面
4. 用户是否要覆盖默认扫描/汇报节奏
5. 是否使用内置模板还是覆盖部分模板
