# 龙虾能力边界

## 总原则

这个 skill 假设龙虾已经有一些通用能力，但在 Boss 任务里必须遵守本 skill 的场景边界。

## 浏览器能力边界

Boss 招聘任务中，龙虾必须：

1. 先告诉用户本任务必须使用 `web-access`
2. 禁用或移除会抢任务的浏览器 skill，例如 `browser-use`
3. 安装 `web-access`
4. 切到 `web-access`
5. 验证接管成功后再继续

这一步是阻断式前提，没有完成就不能继续。

### 浏览器最低原语

龙虾必须具备以下等价能力：

- inspect active tab
- read visible DOM text
- read currently opened detail panel or page
- click specific low-risk controls
- remain in the same logged-in browser session

如果做不到，就必须停。

## 飞书能力边界

飞书这部分只允许使用 **已配置好的飞书 bot**。

### 单一路径原则

- bot 已配置：直接用
- bot 未配置：停下来，引导用户自己去开放平台配置
- 禁止同时乱试 API / OAuth / 网页版

### 需要的权限

- `calendar.event:create`
- `calendar.event:write`

### 本 skill 提供什么

- 什么时候该建面试日程
- 参会人默认逻辑
- 日程标题、时间、描述 payload

### 龙虾负责什么

- 告诉用户缺什么
- 接收用户给的 `App ID` 和 `App Secret`
- 完成本地 bot 配置
- 调用 bot 创建日程并返回链接

### 用户负责什么

- 自己去飞书开放平台
- 自己创建应用或 bot
- 自己拿到 `App ID` 和 `App Secret`
- 交给龙虾配置
- 当龙虾提示权限不足时，自己回开放平台补权限
- 当龙虾要求做第一次日程测试时，自己去飞书 bot 聊天里完成授权

### 明确禁止

龙虾不得：

- 替用户打开飞书开放平台网页
- 替用户在开放平台点配置
- 替用户创建 bot
- 替用户完成开放平台配置流程

龙虾只能描述步骤，等待用户手动完成。

## 任务记忆边界

龙虾的通用记忆不足以承载这个长流程任务，所以本 skill 要求维护 Boss 任务专属记忆。

这部分只应覆盖：

- 候选人发现
- 流程状态
- 决策状态
- 动作历史
- 汇报节奏状态

它是 **任务记忆**，不是通用 CRM，也不是替代龙虾整体记忆系统。
