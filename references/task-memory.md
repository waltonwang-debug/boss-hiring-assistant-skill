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
- 上次是否已经进入小时汇报
- 是否等待用户确认下一步沟通
- 是否等待用户确认进入约面
- 是否已请求附件简历
- 是否已收到附件简历
- 是否已确认面试意向
- 是否已确认面试时间
- 是否已生成飞书建会请求

## 汇报节奏状态

还应保存汇报节奏状态，例如：

```json
{
  "reporting_window": {
    "start_hour_local": 9,
    "end_hour_local": 17
  },
  "scan_interval_minutes": 60,
  "summary_interval_minutes": 60,
  "last_scan_at": "2026-04-02T10:00:00+08:00",
  "last_summary_at": "2026-04-02T10:00:00+08:00"
}
```

这样可以避免重复汇报、漏汇报，以及节奏漂移。

## 保留原则

保留足够多的状态，防止重复触达和状态丢失；  
但不要把这部分扩展成通用 CRM。
