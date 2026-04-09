# Boss Hiring Assistant Skill

一个面向中文用户的 GitHub Skill，用于让本地龙虾在 Boss 直聘招聘场景中辅助完成：

- 简历筛选
- Boss 站内沟通
- 飞书 bot 约面

这个仓库当前聚焦的是 **主线 skill 方案**，不是独立程序，也不是通用 RPA 平台。

## 使用前说明

需要先说明一个现实边界：

**skill 这种形态，对 agent（例如龙虾）的约束力并不是强控制。**

这不是本项目独有的问题，而是 skill 形态本身决定的：

- skill 更像“规则包 / 操作手册 / 约束说明”
- 它可以显著减少 agent 的自由发挥
- 但不能像固定执行器那样 100% 强制每个动作都按预期执行

所以在尝试这个 skill 时，建议：

1. **先从部分工作开始**
   例如先只跑 `boss-screening`，不要一上来同时开启筛选、沟通、约面。

2. **先从短时间运行开始**
   例如先让龙虾跑一小段时间、处理一小批候选人，再逐步扩大范围。

3. **先观察真实执行效果，再逐步放开**
   特别是在 Boss 这种有风控的平台上，要避免因为 agent 自由发挥导致：
   - 误读页面
   - 重复触达
   - 异常浏览器行为
   - 触发 Boss 风控或登录强退

如果你是第一次尝试，强烈建议先用最保守的方式验证：

- 只筛选
- 小批量
- 短时间
- 先看汇报结果，再决定是否继续推进沟通和约面

## 能做什么

当前 skill 拆成三类独立服务：

1. `boss-screening`
用于读取在线岗位和 JD、筛选候选人、汇总进度。

2. `boss-chat`
用于 Boss 站内沟通，例如索要附件简历、确认意向、收集面试时间。

3. `boss-scheduling`
用于候选人确认时间后的飞书 bot 约面。

默认建议先从 `boss-screening` 开始。

## 当前主线已经固化的关键规则

除了三类服务之外，当前主线还已经固化了几条很重要的执行边界：

1. **先复用已登录 Boss tab**
   不默认新开 Boss 页面，避免丢招聘者登录态。

2. **policy 文件缺失时，先读 JD**
   如果没有 `company-policy.yaml` / `approval-policy.yaml`，会先读取在线 JD，再自动生成默认筛选标准，而不是让用户从零口述规则。

3. **筛选服务优先处理未读 / 未处理候选人**
   不是反复重扫所有可见候选人，而是优先覆盖当前真正有业务价值的增量对象。

4. **Boss 站内发消息前做候选人身份校验**
   避免把给候选人 A 的消息发给候选人 B。

5. **索要附件简历后会继续跟进闭环**
   不只是发一条“请发送简历”的消息，还会继续跟进候选人是否发送附件、是否出现接收按钮、是否完成接收。

## 当前方案特点

这套 skill 不是按“每小时机械扫一次”来做招聘，而是按：

- **每个 JD 每天要筛选出多少个符合要求的候选人**

来推进筛选。

同时，当前主线已经把一些真实踩坑经验固化进去了，比如：

- 浏览器主路径统一只走 `web-access`
- 默认禁止截图 / OCR 读取 Boss 页面
- 必须复用用户当前已登录的 Boss tab，不能随便新开 tab
- 在线简历很多时候在聊天页右侧面板中读取，而不是新开 URL
- 缺少 policy 文件时，先读在线 JD，再生成默认筛选标准
- 新一轮筛选默认优先处理“未读 / 未处理候选人”
- Boss 站内消息发送前必须先校验当前线程身份
- 附件简历请求后必须继续跟进到“已接收”状态

## 使用方式

把这个 GitHub 链接发给本地龙虾，让它安装并使用这个 skill 即可。

建议首次使用时，用户先在 **Chrome / Chromium** 中登录 **Boss 直聘招聘者页面**，再让龙虾接管。

## 仓库结构

- [SKILL.md](./SKILL.md)
  总入口与顶层规则

- [agents/openai.yaml](./agents/openai.yaml)
  agent 默认提示配置

- [references/boss-screening-service.md](./references/boss-screening-service.md)
  简历筛选服务规则

- [references/boss-chat-service.md](./references/boss-chat-service.md)
  Boss 站内沟通服务规则

- [references/boss-scheduling-service.md](./references/boss-scheduling-service.md)
  飞书约面服务规则

- [references/browser-routing.md](./references/browser-routing.md)
  浏览器访问与 `web-access` 路由规则

- [references/boss-send-recipe.md](./references/boss-send-recipe.md)
  Boss 站内消息发送固定路径

- [references/task-memory.md](./references/task-memory.md)
  本地任务记忆模型

- [execution-lessons-summary.md](./execution-lessons-summary.md)
  真实测试踩坑与经验总表

## 依赖与前置条件

当前主线 skill 默认依赖：

- 本地龙虾
- `web-access`
- 用户真实 Chrome / Chromium 会话
- 用户已登录的 Boss 招聘者页面

飞书约面路径默认依赖：

- 飞书 bot

注意：

- 飞书 bot 的创建和开放平台配置由用户自己完成
- 龙虾负责接入 bot 和后续调用

## 使用时建议注意

1. 先登录 Boss，再让龙虾接管  
2. 默认先从筛选服务开始  
3. 先告诉龙虾每个 JD 的每日目标人数  
4. 推荐页和搜索页默认靠本地记忆判断“本轮是否未处理”  
5. Boss 站内发消息前要确认当前线程身份正确  
6. 对“索要附件简历”的候选人，后续还要跟进是否已发送并完成接收  
7. 不要默认允许截图读取页面  

## 开源协议

本仓库使用 [MIT License](./LICENSE)。
