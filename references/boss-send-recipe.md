# BOSS 站内消息发送固定方案

## 定位

这个文档不是“建议方案”，而是 BOSS 站内消息发送的唯一主路径。
当页面结构没有明显变化时，不得重新设计发送实现。

## 固定元素

线程确认：

- 候选人线程节点：`#_{candidate_id}`
- 也可以用 `.geek-item[data-id]` 获取 `candidate_id`

输入框：

- 首选 selector：`[contenteditable]`
- 更稳妥 selector：`.chat-container-private [contenteditable]`

发送按钮：

- selector：`.submit`
- 更稳妥 selector：`.chat-container-private .submit`

## 唯一主路径

1. 点击 `#_{candidate_id}` 打开目标候选人线程
2. 等待输入框出现
3. 对 `.chat-container-private [contenteditable]` 写入消息文本
4. 点击 `.chat-container-private .submit`
5. 做一级成功校验：输入框内容清空
6. 做二级成功校验：最后一条消息或会话预览出现“送达”标记

## 唯一备用路径

如果点击 `.submit` 无效，只允许一次备用路径：

- 在同一个输入框节点上派发一次 Enter

不允许继续扩展出更多发送变体。

## 优化要求

发送动作要优先遵守这些低 token 规则：

1. 少 eval，多合并
2. selector 尽量短
3. 成功验证优先看输入框是否清空
4. 优先点击候选人打开线程，不要优先走 URL 导航

## 禁止项

以下行为不允许作为常规发送路径：

- 研究 Vue 事件系统
- 起子代理处理普通发送
- 连续尝试多套前端实现
- 默认走截图/OCR
- 发送失败后长时间试错

## 失败后的处理

如果主路径和唯一备用路径都失败，立即进入 `paused_for_send_message`，并告诉用户：

1. 当前线程是否已打开
2. 输入框是否可编辑
3. 已经试过主路径和 Enter 备用路径
4. 用户现在需要做什么才能恢复
