# 中文话术模板策略

## 推荐策略

推荐采用混合模式：

- 内置默认模板，保证即装即用
- 用户只覆盖自己在意的模板
- 只有低风险 FAQ 才允许有限生成

不要要求用户先把所有模板都写完。  
也不要允许“首次触达、索要附件、约面、拒绝”这些关键节点完全自由生成。

## 为什么这样最合适

如果要求开源 skill 用户先配置完全部模板，会大幅增加 onboarding 成本。  
如果完全让模型自由发挥，又会带来：

- 语气漂移
- 事实漂移
- 招聘策略不一致
- 平台投诉或用户不信任风险

最合理的平衡是：

1. 内置一套强默认中文模板
2. 允许用户局部覆盖
3. 仅对低风险 FAQ 开放受控生成

默认模板文件在：

- `assets/default-message-templates.zh-CN.yaml`

## 模板模式

推荐默认值：

- `builtin_with_optional_overrides`

可支持的概念模式：

- `builtin_only`
- `builtin_with_optional_overrides`
- `user_templates_required`

公开 skill 场景下，不建议默认 `user_templates_required`。

## 内置模板 ID

这些 ID 应保持稳定：

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

## 变量约定

模板应当参数化，而不是每次从零生成。

示例：

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

## 受限生成

只有满足以下全部条件，才允许 fallback 生成：

- 话题低风险
- 没有命中固定模板
- 话题在允许自动回复的 FAQ 范围内
- 生成可以被明确约束

约束要求：

- 句子短
- 专业、友好
- 默认不索要私人联系方式
- 不承诺 offer 或薪资
- 不捏造公司政策

## 覆盖策略

用户覆盖应当是局部的，而不是全量替换。

例如：

- 如果用户只提供 `request_attachment_resume`，就只覆盖这一条
- 没提供 `confirm_interview_time`，就继续使用内置版本

变量名在内置模板和用户模板之间必须保持一致。
