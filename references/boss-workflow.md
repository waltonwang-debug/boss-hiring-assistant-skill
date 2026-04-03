# BOSS 招聘流程

## 适用范围

本 skill 只支持 BOSS 直聘网页端，不假设桌面客户端自动化。

主要业务动作：

1. 从 `inbound_chat`、`recommended_feed`、`search_results` 读取候选人
2. 打开候选人详情并提取结构化信息
3. 按 JD 和公司策略做分类
4. 根据分类进入：
   - reject
   - human review
   - approval request
   - guarded chat progression
5. 需要时索要附件简历
6. 候选人确认时间后通过飞书 bot 创建日程
7. 把飞书日程链接回发给候选人

## 默认运行节奏

默认节奏如下，用户可覆盖：

- 工作时段：`09:00-17:00`
- 工作日：周一到周五
- 每 `60` 分钟扫描一次
- 每 `60` 分钟汇报一次

每个小时周期内，龙虾应：

1. 扫描候选人来源
2. 更新分类和原因
3. 生成一份汇总给用户
4. 询问哪些候选人继续推进沟通
5. 询问哪些候选人进入约面推进

## 候选人来源

### `inbound_chat`

已进入聊天列表的候选人。

特点：

- MVP 最优先
- 风险最低
- 最适合先跑通

### `recommended_feed`

Boss 根据账号和岗位推荐的候选人。

特点：

- 不设业务总量上限
- 但操作节奏必须保守
- 不要机械地每个都快速打开

### `search_results`

账号具备搜索权益时的搜索结果。

特点：

- 需要更明确的筛选条件
- 仍然使用相同 policy pack

## 信息提取策略

顺序如下：

1. 读卡片可见字段
2. DOM 结构化提取
3. 必要时再用轻量 CDP
4. 只有结构化失败时才允许视觉兜底

## 决策流程

### 规则优先

先跑硬规则：

- 必要技能缺失
- 命中硬淘汰条件
- 薪资越界
- 地点越界
- 职级越界

### 模型辅助

只处理边界样本，并要求输出：

- `decision`
- `score`
- `confidence`
- `reasons`
- `risk_tags`
- `next_action`

允许的决策：

- `reject_hard`
- `reject_soft`
- `review_human`
- `approval_required`
- `pass_to_chat`

## 每小时汇报格式

默认按分类分组：

- `pass_to_chat`
- `approval_required`
- `review_human`
- `reject_soft`
- `reject_hard`

每位候选人应包含：

- 姓名或稳定 ID
- 来源
- 当前分类
- 简短分类原因
- 龙虾建议的下一步

汇报结尾必须明确问用户两件事：

1. 哪些候选人可以继续推进沟通，例如索要附件简历
2. 哪些候选人可以进入约面推进

## 状态机

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
-> interview_intent_checking
-> interview_intent_confirmed
-> time_slot_collecting
-> time_confirmed
-> feishu_event_created
-> schedule_link_sent
-> process_completed
```

## 聊天推进

### 1. 索要附件简历

适用场景：

- 可见简历信息不足
- 用户想做内部复核
- 候选人有潜力但证据不够

### 2. 确认面试意向

只有在以下情况下才推进：

- 自动通过
- 或用户明确批准进入面试

### 3. 收集时间

先收可用时间，再确认具体时间。

### 4. 发送飞书日程链接

只有飞书日程创建成功后才发送。

## 消息发送快速路径

单条 Boss 消息发送固定按下列步骤：

1. 先准备最终消息文本
2. 停留在目标候选人线程
3. 优先点击候选人项打开线程
4. 使用 `[contenteditable]` 写入消息
5. 点击 `.submit`
6. 用输入框清空做一级成功判定
7. 有必要时再用送达标记做二级验证

速度和控制要求：

- 不得为常规单条消息起子代理
- 不得为标准发送长时间探索页面
- 不得因为前端框架复杂就无限切换发送策略
- 失败就进入可恢复暂停态

## 强约束动作集合

以下动作都属于强约束动作：

- 打开候选人详情
- 切换候选人线程
- 索要附件简历
- 发送标准模板消息
- 收集可约时间
- 确认面试时间
- 发送飞书日程链接

统一执行规则：

1. 先准备好 payload
2. 尽量停留在当前页面或线程
3. 只允许一次短时主线程 `web-access` 尝试
4. 用一次直接页面检查验证结果
5. 失败就暂停并汇报，不继续发散探索

## 可恢复暂停格式

强约束动作失败时，暂停信息必须包含：

- 失败动作
- 目标候选人或线程
- 已按快路径尝试且未继续乱试
- 最可能原因
- 用户最小恢复动作

推荐暂停标签：

- `paused_for_open_profile`
- `paused_for_switch_thread`
- `paused_for_send_message`
- `paused_for_attachment_request`
- `paused_for_availability_collection`
- `paused_for_schedule_link_send`

## 飞书日程规则

本 skill 只创建面试日程，不创建 todo。

第一次真正要创建面试日程时：

- 询问默认参会人只加用户，还是再加一个固定账号
- bot 未配置时，让用户去开放平台创建并交回 `App ID`/`App Secret`
- 龙虾完成本地配置后，明确要求用户去飞书 bot 聊天里测试创建一个日程
- 用 bot 聊天里的授权流程完成可用授权
- 如果仍缺权限，再引导用户补权限并沿同一 bot 路径重试

## 工具边界

期望工具契约如下：

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

call_lobster_feishu_capability(event_payload)

pause_candidate_flow(candidate_id, reason)
request_human_review(candidate_id, packet)
```
