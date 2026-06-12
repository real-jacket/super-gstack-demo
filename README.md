# Super GStack Demo

Jira 任务日历周末原型项目。

## 项目简介

这是一个极简的 Jira 任务日历工具，用于验证"工作量可视化 + 拖拽排期"的价值。

**核心目标：** 验证同事是否愿意用这个工具（可传播性）

## 功能特性

- ✅ 从 Jira Cloud 同步未完成任务
- ✅ 日历拖拽排期（周一到周五）
- ✅ 工作量可视化（红黄绿进度条）
- ✅ 分享快照（导出静态 HTML）
- ✅ 本地持久化（localStorage）

## 项目文档

- [完整需求 Spec](docs/jira-calendar-spec.md) - 包含背景、目标、功能范围、验收标准
- [CEO 计划](https://github.com/user/repo) - 产品愿景和决策记录

## 开发计划

**周末原型（8-9 小时）：**
- 周六：Jira 连接 + Mock 日历 + 工作量可视化（4-5h）
- 周日：HTML 导出 + 数据持久化 + UI 美化（4-5h）

**验证阶段：**
- 周一：自己使用 1 天，判断是否有价值
- 下周：分享给 2-3 个同事，收集反馈

## 技术栈

- 前端框架：Vite + React
- 日历库：FullCalendar v6
- 存储：localStorage
- API：Jira Cloud REST API

## 快速开始

```bash
# 克隆项目
git clone https://github.com/YOUR_USERNAME/super-gstack-demo.git
cd super-gstack-demo

# 安装依赖
npm install

# 配置 Jira credentials
cp .env.example .env.local
# 编辑 .env.local，填入你的 Jira domain + API token

# 启动开发服务器
npm run dev
```

## 环境变量

创建 `.env.local` 文件：

```bash
VITE_JIRA_DOMAIN=your-domain.atlassian.net
VITE_JIRA_EMAIL=your-email@example.com
VITE_JIRA_API_TOKEN=your-api-token
```

## 决策点

1. **周日早上：** Jira 连接 + 日历渲染是否成功？
2. **周日下午：** 分享 HTML 能否生成？
3. **周一晚上：** 比直接看 Jira 好吗？
4. **下周五：** 至少 1 个同事说"我也想用"吗？

## 明确不做的功能

- ❌ 双向同步回 Jira
- ❌ 实时分享链接（需要后端）
- ❌ 团队协作功能
- ❌ AI 排优先级
- ❌ 新任务通知

这些功能留待验证成功后的 v2 版本。

## License

MIT
