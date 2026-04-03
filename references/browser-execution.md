# 浏览器执行规则

## 目标

通过 `web-access` 在用户真实的 Chrome/Chromium 会话中完成 BOSS 招聘流程，并尽量降低触发风控的概率。

## 前提

在任何 Boss 页面动作之前，必须先完成：

- 停用 `browser-use` 等冲突浏览器 skill
- 安装并启用 `web-access`
- 明确验证 `web-access` 已接管当前任务

做不到就停。

## 环境检查

开始前必须确认：

1. 本机有可用 Chrome 或 Chromium
2. 用户已登录 BOSS 网页端
3. 页面没有登录失效、风控验证或异常提示
4. 当前标签页是受支持的 Boss 页面
5. 冲突浏览器 skill 已移除或禁用
6. `web-access` 已激活
7. `web-access` 能满足本 skill 需要的浏览器原语

如果 `web-access` 可用，就不允许静默退回截图理解或别的浏览器工具。

## 支持页面

- 聊天列表 / `inbound_chat`
- 推荐流 / `recommended_feed`
- 搜索结果 / `search_results`
- 候选人详情面板或详情页

## 读取顺序

1. 先读卡片可见字段
2. 再用 `web-access` DOM 读取
3. DOM 不够时，再用 `web-access` CDP 读取
4. 再读已经打开的详情面板
5. 只在结构化读取失败时才用截图兜底

额外原则：

- 能一次合并读取，就不要拆成很多小读取
- 线程已在列表里可见时，优先点开，不优先做整页 URL 导航
- 选择器优先用短且稳定的版本

## 工具纪律

### 允许的模式

- 始终走 `web-access`
- 停留在用户真实登录的浏览器上下文里
- 每一步只做最少必要读写
- 优先 DOM/CDP，后截图
- 常规消息发送、切线程、开详情都在主线程内用短时路径完成

### 禁止的模式

- 在一个 Boss 任务里乱切多套浏览器方法
- 让 `browser-use` 或其他通用浏览器工具接管
- 页面还能结构化读取时改走截图/OCR
- 为常规单步动作起子代理
- 针对一条消息、一次切线程、一次开详情做长时间探索

## 建议选择器

### 候选人卡片

- `[data-geek-id]`
- `.geek-item`
- `.candidate-card`
- `.job-card-wrapper`

字段常见位置：

- name: `.name`, `.candidate-name`, `h3`, `h4`
- title: `.job-name`, `.expect-position`, `.title`
- city: `.city`, `.location`, `.expect-city`
- education: `.education`, `.degree`
- experience: `.experience`, `.exp`
- salary: `.salary`, `.expect-salary`

### 候选人详情

- `.resume-detail-wrap`
- `.geek-resume-wrap`
- `.candidate-detail`
- `.resume-detail`

### 聊天发送

- 线程项：`.geek-item.selected`, `#_{candidate_id}`
- `candidate_id` 来源：`.geek-item[data-id]`
- 优先打开线程的方式：点击 `#_{candidate_id}`
- 输入框：`[contenteditable]`
- 发送按钮：`.submit`
- 一级成功判定：`[contenteditable]` 变空
- 二级成功判定：最新消息包含送达标记
- 三级成功判定：左侧预览出现送达标记

## 发消息的标准路径

常规发消息按以下顺序：

1. 获取或复用目标 `candidate_id`
2. 目标线程未激活时，点击 `#_{candidate_id}`
3. 确认左侧高亮线程就是目标候选人
4. 定位 `[contenteditable]`
5. 写入最终消息文本
6. 点击 `.submit`
7. 优先检查输入框是否已清空
8. 如果一级判定不够，再看送达标记

唯一备用路径：

- 对同一个输入框派发一次 Enter

如果这两条都失败，就进入可恢复暂停态，不允许继续发散。

## 稳定性优先级

在多个可行路径中，优先：

- 串行而不是并行
- 已打开视图而不是强跳转
- 便宜验证而不是重度验证
- DOM/CDP 而不是截图/OCR

## 立即暂停条件

出现以下任一情况必须暂停：

- 风控验证
- 登录失效
- 异常重定向
- 结构化提取连续不稳定
- 线程/输入框状态异常

## 可恢复暂停要求

如果强约束动作失败，暂停时必须告诉用户：

- 哪一步失败了
- 龙虾已经按短时快路径尝试过，没有继续乱试
- 最可能原因
- 用户现在最小要做什么
- 做完后如何让龙虾继续

适用动作包括：

- 开详情
- 切线程
- 发消息
- 索要附件简历
- 收集时间
- 发飞书日程链接
