# 任务记忆模型

## 目的

Boss 招聘流程是长流程任务，不能只依赖龙虾通用记忆，所以需要单独维护任务记忆。

## 存储要求

龙虾应维护一个轻量本地结构化存储，可以是：

- 本地 SQLite
- 本地 JSON
- 其他等价的本地结构化存储

用户不需要手动安装或管理数据库。

## 最小候选人记录

```json
{
  "candidate_id": "boss_xxx",
  "source_type": "recommended_feed",
  "process_state": "card_scanned",
  "decision": "pending",
  "last_snapshot_at": "2026-04-02T12:00:00+08:00",
  "payload": {
    "name": "张三",
    "title": "高级后端工程师",
    "city": "Shanghai",
    "profile": {}
  }
}
```

## 最小动作记录

```json
{
  "candidate_id": "boss_xxx",
  "action_type": "classification",
  "created_at": "2026-04-02T12:05:00+08:00",
  "payload": {
    "decision": "pass_to_chat",
    "next_action": "request_attachment_resume"
  }
}
```

## 必须记住的维度

- 候选人来源
- 当前流程状态
- 上次决策
- 动作历史
- 上次是否已经进入本轮进度汇报
- 是否等待用户确认下一步沟通
- 是否等待用户确认进入约面
- 是否已请求附件简历
- 是否已收到附件简历
- 是否已确认面试意向
- 是否已确认面试时间
- 是否已生成飞书建会请求
- 是否本轮发现为未读候选人
- 是否本轮已成功更新简历
- 是否本轮仍待补齐
- 本轮未补齐原因

## 建议增加的候选人状态

建议至少记录：

- `unread_discovered`
- `resume_read`
- `classification_updated`
- `has_unread_message`
- `replied_by_candidate`
- `pending_user_confirmation`
- `resume_read_failed`
- `needs_rescan`

## 来源相关判断建议

任务记忆不仅用于防止重复触达，也用于区分不同来源下“哪些候选人本轮应该处理”：

- 对 `inbound_chat`，页面未读信号优先，本地记忆负责去重与补漏
- 对 `recommended_feed`，本地记忆优先，页面信号只做辅助
- 对 `search_results`，本地记忆优先，页面信号只做辅助

不要把推荐页和搜索页误当成自带稳定“已读/已处理”标记的页面。

## 汇报与目标状态

还应保存进度与汇报状态，例如：

```json
{
  "reporting_window": {
    "start_hour_local": 9,
    "end_hour_local": 17
  },
  "summary_interval_minutes": 60,
  "daily_target_count": 10,
  "current_qualified_count": 6,
  "current_unread_discovered_count": 12,
  "current_unread_updated_count": 9,
  "current_unread_missing_count": 3,
  "last_scan_at": "2026-04-02T10:00:00+08:00",
  "last_summary_at": "2026-04-02T10:00:00+08:00"
}
```

这样可以避免重复汇报、漏汇报，以及人数口径混乱。

## 保留原则

保留足够多的状态，防止重复触达和状态丢失；  
但不要把这部分扩展成通用 CRM。
