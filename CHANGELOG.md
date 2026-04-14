# 二次开发日志

基于 [GuDong2003/xianyu-auto-reply-fix](https://github.com/GuDong2003/xianyu-auto-reply-fix) 的二次开发记录。

---

## 已完成功能

### v2.0.0 (2026-04-12)

**在线客服重构**
- 替换原始"在线客服"模块，重构为三栏布局（账号列表 / 会话列表 / 聊天窗口）
- 基于 SSE (Server-Sent Events) + Pub/Sub 事件总线实现实时消息推送
- 支持图片发送（粘贴 / 拖拽上传）
- 新消息红色角标提醒 + 浏览器声音通知
- 快捷回复管理：分组、触发关键词、CRUD 操作

### v2.1.0 (2026-04-13)

**交互与体验优化**
- 快捷回复改为内联表单卡片，替换原来的 `prompt()` 弹窗
- Tab 栏和快捷回复列表样式美化（渐变背景、圆角、悬浮态）
- 点击在线客服自动打开第一条未读会话
- 未读角标点击后即时消除

**群消息过滤**
- 通过 `sessionType == '30'` 识别群聊消息
- 群消息不再写入聊天记录和推送到前端

**账号备注体系**
- 账号编辑弹窗新增"备注名称"字段
- 后端日志全局替换：`【cookie_id】` → `【备注/cookie_id】`（920 处）
- 前端全局备注缓存 `accountRemarkCache` + `getAccountLabel()` 工具函数
- 覆盖所有前端视图：仪表盘卡片、账号管理表格、自动回复 / 默认回复 / 消息通知 / 商品 / 订单列表、在线客服面板、诊断页面、各类弹窗标题
- 备注修改后实时同步缓存，全局生效

**热更新与版本管理**
- 替换为自有 fork 的版本日志和 GitHub 仓库配置
- 通过 `generate_update_manifest.py` 生成 `update_files.json` 并上传至 Release
- Docker 环境通过 `UPDATE_GITHUB_OWNER` / `UPDATE_GITHUB_REPO` 环境变量指向自有仓库

---

## 后续开发方向

### 高优先级

**买家信息增强**
- 会话列表显示买家昵称 / 头像（目前仅显示 user_id）
- 聊天窗口展示买家历史订单、信用等级等辅助信息
- 标记"已购买"、"已退款"等买家状态标签

**消息能力扩展**
- 支持发送商品链接卡片（而非纯文本链接）
- 聊天窗口消息搜索与历史记录翻页
- 消息撤回提示
- 支持发送多张图片 / 视频

**快捷回复增强**
- 快捷回复支持变量模板（如 `{商品名称}`、`{买家昵称}`）
- 按商品或场景自动推荐快捷回复
- 快捷回复使用频率统计，常用的排前面

### 中优先级

**数据统计与分析**
- 每日/每周消息量、响应速度统计面板
- 买家咨询转化率分析（咨询 → 下单比例）
- 热门问题 / 关键词词云
- 各账号业绩对比报表

**自动化流程增强**
- 自动回复规则支持时间条件（如工作时间和非工作时间不同回复策略）
- 长时间未回复自动发送提醒话术
- 自动识别砍价意图并应用预设策略

**多账号协同**
- 跨账号统一消息收件箱（合并所有账号的未读消息）
- 账号分组管理（按店铺类型、业务线分组）
- 批量操作：一键启用 / 禁用 / 修改多个账号的配置

### 低优先级

**移动端适配**
- 响应式布局优化，适配手机浏览器
- PWA 支持，可添加到手机桌面
- 移动端推送通知（结合微信 / 钉钉 / Bark 等渠道）

**运维与安全**
- 操作审计日志（谁在什么时间做了什么操作）
- Cookie 自动刷新 / 续期机制
- 数据定时备份到云存储
- 支持 HTTPS + 自定义域名配置指南

**插件化架构**
- 将自动回复、自动发货、消息通知等功能模块化为可插拔组件
- 提供简单的插件 API，方便扩展自定义逻辑
- 第三方服务集成接口（ERP、进销存系统对接）

---

## 技术栈

| 层面 | 技术 |
|------|------|
| 后端 | FastAPI + Uvicorn + Python 3.11+ async |
| 数据库 | SQLite 3 |
| 前端 | Bootstrap 5 + Vanilla JS |
| 实时通信 | WebSocket (闲鱼) + SSE (前端推送) |
| 浏览器自动化 | Playwright + DrissionPage |
| 部署 | Docker + Docker Compose |

## 本地开发

```bash
# 克隆
git clone https://github.com/979193906/xianyu-auto-reply-fix.git
cd xianyu-auto-reply-fix

# 热更新配置（Docker 环境）
UPDATE_GITHUB_OWNER=979193906
UPDATE_GITHUB_REPO=xianyu-auto-reply-fix
```

发版流程：
1. 修改代码
2. 更新 `static/version.txt`
3. 提交并推送到 main
4. 执行 `python generate_update_manifest.py . <版本号> 979193906 xianyu-auto-reply-fix`
5. `git tag -f <版本号> && git push -f origin <版本号>`
6. `gh release upload <版本号> update_files.json --clobber`
