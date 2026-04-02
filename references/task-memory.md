# Task Memory

## Purpose

Keep BOSS recruiting task state separately from Lobster's broader memory because this workflow is long-running and stateful.

## Storage Expectation

Lobster should maintain a small local store for this skill. It may use:

- local SQLite
- local JSON files
- any equivalent local structured store

The user should not need to install or manage a database manually.

## Minimum Candidate Record

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

## Minimum Action Record

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

## Required Memory Dimensions

- candidate source
- current process state
- last decision
- action history
- last hourly summary inclusion state
- pending user confirmation for next chat step
- pending user confirmation for interview progression
- whether attachment resume was requested
- whether attachment resume was received
- whether interview intent was confirmed
- whether interview time was confirmed
- whether a Feishu schedule request was already sent to Lobster

## Reporting Cadence Memory

The task store should also keep a small reporting state, for example:

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

This lets Lobster avoid repeated summary spam and keeps the hourly review cycle stable.

## Retention Principle

Keep enough history to avoid duplicate outreach and lost state, but do not turn this into a general CRM.
