---
name: boss-hiring-assistant
description: 面向 BOSS 直聘招聘流程的中文 skill。默认先做简历筛选，也支持站内沟通和飞书约面。
---

# Boss Hiring Assistant

## 目标

这个 skill 只做三类服务，并且三类服务必须分开理解、分开执行：

1. `boss-screening`
只负责读取岗位、读取候选人、筛选、分类、汇报。
2. `boss-chat`
只负责 BOSS 站内沟通。
3. `boss-scheduling`
只负责候选人已确认时间后的飞书约面。

如果用户没有明确指定服务类型，默认先进入 `boss-screening`。
不要把三类服务混在同一轮里自由串行。

## 首次安装后的介绍

用户首次安装这个 skill 后，必须先用中文向用户做一个简短介绍，再进入任何具体执行。

介绍内容至少包含：

1. 这个 skill 分成三类独立服务：
   - `boss-screening`：简历筛选和周期汇报
   - `boss-chat`：BOSS 站内沟通
   - `boss-scheduling`：飞书 bot 约面
2. 推荐用户先从 `boss-screening` 开始
3. 后两类服务按需启动，不必首次安装时一起配置
4. `boss-screening` 可覆盖的候选人范围：
   - 主动进入聊天列表或主动投递的候选人
   - BOSS 推荐页面里的候选人
   - 搜索页面里的候选人（当账号具备搜索权益时）
5. `boss-screening` 会按固定节奏运行和汇报；如果用户不指定，则采用默认节奏，如果用户指定自己的时间节奏，则使用用户偏好
6. 启动方式：
   - 用户说“先帮我做简历筛选”时进入 `boss-screening`
   - 用户说“帮我联系这些候选人”时进入 `boss-chat`
   - 用户说“帮我给这些候选人约面”时进入 `boss-scheduling`

如果用户没有明确选择，默认建议先从 `boss-screening` 开始。

## 顶层硬规则

### 1. 浏览器访问只允许走 web-access

BOSS 页面上的所有读取、点击、切换线程、打开详情、发送消息，都必须先遵守：

- [browser-routing.md](./references/browser-routing.md)

这是一条硬规则，不是建议。

### 1.1 截图默认彻底禁止

在 BOSS 页面读取中，截图、OCR、图像理解默认彻底禁止。

如果 `web-access` 读取失败：

1. 允许在同一路径内最多再尝试 `2` 次
2. 如果仍失败，不允许自动切换到截图路径
3. 必须先向用户说明：
   - `web-access` 已尝试失败
   - 现在可以改走截图等异常方案
   - 这些方案可能增加 token 消耗，也可能提高 BOSS 风控或网页登录强退风险
   - 默认更建议继续坚持 `web-access`
4. 只有用户明确同意后，才允许临时启用截图等异常方案

### 2. BOSS 站内发消息必须走固定 recipe

只要进入 BOSS 站内消息发送，必须先读取并执行：

- [boss-send-recipe.md](./references/boss-send-recipe.md)

不要把该文档当参考说明。
不要再自行重新设计 DOM 路径、前端事件路径、子代理方案。

### 3. 服务按入口执行

简历筛选：

- [boss-screening-service.md](./references/boss-screening-service.md)

站内沟通：

- [boss-chat-service.md](./references/boss-chat-service.md)

飞书约面：

- [boss-scheduling-service.md](./references/boss-scheduling-service.md)

## 阻断式前置条件

在介绍完三类服务之后，再告诉用户该服务对应的强制前置条件。

所有服务共用的强制前置条件：

1. 禁用或移除会抢占浏览器访问的 skill，包括但不限于 `browser-use`。
2. 由龙虾代为安装 `web-access`。
   - GitHub 链接：`https://github.com/eze-is/web-access`
3. 把当前任务切换到 `web-access`。
4. 验证 `web-access` 已经接管当前 BOSS 任务。
5. 使用真实 Chrome 或 Chromium 浏览器。
6. 用户已登录 BOSS 网页端。

如果这些通用前置条件没有完成，禁止继续：

- 读取在线岗位
- 读取 JD
- 查看候选人
- 发站内消息
- 创建飞书日程

不要自行搜索 `web-access` 的安装来源。
默认由龙虾先尝试代为安装；只有代装失败、权限不足或环境阻塞时，才让用户手动安装，并直接提供上面的 GitHub 链接。

服务专属前置条件必须按需引导，不要一次性全部抛给用户：

- `boss-screening`：只需要通用前置条件
- `boss-chat`：需要通用前置条件，以及用户已明确批准哪些候选人进入沟通
- `boss-scheduling`：需要通用前置条件，以及用户已选择候选人进入约面；飞书 bot 的配置和授权只在第一次真实创建日程时再引导

## 默认运行规则

以下是默认值，用户可以覆盖：

- 工作日：周一到周五
- 工作时段：本地时间 `09:00-17:00`
- 简历扫描频率：每 `60` 分钟一次
- 汇报频率：每 `60` 分钟一次
- 汇报格式：按候选人分类分组，附分类原因和建议下一步

如果用户没有表达自己的节奏，就自动采用这些默认值。
如果用户表达了自己的工作时间、扫描频率或汇报节奏，就采用用户偏好。

## 每小时汇报的固定要求

每次汇报至少要包含：

1. 本周期内新增或更新的候选人分类结果
2. 每位候选人的分类原因
3. 建议继续沟通的候选人
4. 建议进入约面的候选人
5. 需要用户确认的动作

每次汇报都要分别向用户确认：

- 哪些候选人可以继续推进站内沟通，例如索要附件简历
- 哪些候选人可以进入约面路径

首次进入 `boss-screening` 时，也要明确告诉用户：

- 可以筛选哪些来源的候选人
- 默认多久扫描一次、多久汇报一次
- 用户可以随时把节奏改成自己的偏好

## 飞书边界

飞书只允许走 bot 路径。

用户负责：

- 去飞书开放平台创建应用或 bot
- 获取 `App ID` 和 `App Secret`
- 按提示去 bot 聊天里完成真正可用的日历授权

龙虾负责：

- 接收用户提供的 `App ID` / `App Secret`
- 完成本地配置
- 调用 bot 创建日程

龙虾不得：

- 主动打开飞书开放平台网页
- 替用户点击开放平台后台
- 自由尝试 API、OAuth、网页版三套路径

第一次真的要创建飞书日程时，才去确认 bot 是否已配置、授权是否已完成。

## 失败处理

所有强约束动作失败后，都必须进入“可恢复暂停态”，而不是继续长时间试错。

暂停时必须告诉用户：

1. 失败的是哪一步
2. 已经按最小快路径尝试过
3. 最可能原因
4. 用户现在最小需要做什么
5. 用户做完后如何恢复

## 阅读顺序

1. 先读 [browser-routing.md](./references/browser-routing.md)
2. 再按当前服务读取：
   - [boss-screening-service.md](./references/boss-screening-service.md)
   - [boss-chat-service.md](./references/boss-chat-service.md)
   - [boss-scheduling-service.md](./references/boss-scheduling-service.md)
3. 如需发 BOSS 站内消息，再读 [boss-send-recipe.md](./references/boss-send-recipe.md)
4. 其他补充文档：
   - [policy-schema.md](./references/policy-schema.md)
   - [task-memory.md](./references/task-memory.md)
   - [trial-run.md](./references/trial-run.md)
