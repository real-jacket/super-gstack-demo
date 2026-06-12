# Jira 任务日历 — 周末原型 Spec

## 背景和目标

### 问题

开发者在 Jira 中管理任务时，无法直观看到工作量分布和时间超载情况。当前依赖：

- 口头估算（"我这周能完成多少？"）
- 手动数任务（打开 Jira 逐个查看）
- 感觉判断（"好像挺多的"）

这导致：

- PM/同事无法快速了解你的工作负载
- 容易承诺过多任务导致延期
- 缺乏可视化依据进行工作量协商

### 目标

**主要目标：验证"同事愿意用吗"**

这不是做一个功能完整的工具，而是验证两个假设：

1. 工作量可视化 + 拖拽排期对开发者有价值
2. 同事愿意用这个工具（能传播）

**次要目标：** 如果验证成功，提供一个可升级的基础原型

### 非目标（明确不做）

- ❌ 双向同步回 Jira（不修改 Jira due date）
- ❌ 实时分享链接（不做后端）
- ❌ 多用户协作（不看同事的容量）
- ❌ AI 排优先级
- ❌ 新任务通知
- ❌ 多浏览器兼容性（只保证 Chrome）
- ❌ 生产级部署

## 用户和场景

### 核心用户

**Solo 开发者**（你自己），次要验证对象是 2-3 个同事

### 使用场景

**场景 1：每周规划（你自己用）**

1. 周一早上打开工具
2. 从 Jira 拉取未完成任务
3. 拖拽任务分配到本周各天
4. 看到周三超载（10h），把一个任务推到周四
5. 开始执行

**场景 2：工作量沟通（分享给同事）**

1. PM 问："你这周能接新任务吗？"
2. 你点"分享"按钮，生成 HTML 文件
3. 发给 PM（Slack / Email）
4. PM 打开 HTML，看到你本周已排满，周五有 2h 余量
5. PM 决定："那这个 3h 的任务推到下周"

**场景 3：决策点（验证假设）**

- 周一：你自己用 1 天，判断是否比直接看 Jira 好
- 下周五：2-3 个同事看了分享 HTML，判断他们是否愿意用

## 功能范围

### 必须做（8-9 小时）

#### 1. Jira 任务同步（3h）

**输入：** Jira Cloud domain + API token + email
**输出：** 未完成任务列表

**技术要求：**

- JQL: `assignee = currentUser() AND statusCategory != Done AND updated >= -90d`
- Vite proxy 解决 CORS
- 认证：`Authorization: Basic base64(email:api_token)`
- Story Points：从 `customfield_10016` 提取，缺失时默认 8h
- 错误处理：连不上就报错（不做重试）

**验收：**

- ✅ 能在浏览器 console 看到拉取的任务 JSON
- ✅ 任务包含：key, summary, story points (或默认 8h), priority

#### 2. 日历拖拽 + 本地存储（2h）

**输入：** Jira 任务列表
**输出：** 可拖拽的周视图日历

**技术要求：**

- FullCalendar daygrid 插件
- 只显示周一到周五（禁用周末）
- 拖拽后存 localStorage：`{taskId: date}` 映射
- 刷新后从 localStorage 恢复排期

**验收：**

- ✅ 任务显示在日历上
- ✅ 拖拽任务到另一天，松手后位置更新
- ✅ 刷新页面，排期不丢失
- ✅ 任务标题截断到 50 字符，hover 显示完整标题

#### 3. 工作量可视化（1h）

**输入：** 每天的任务 + story points
**输出：** 每天的工作量进度条

**技术要求：**

- 每天汇总 story points（假设 1 SP = 1h）
- 进度条颜色：<8h 绿色，8-10h 黄色，>10h 红色
- 显示"已排 Xh / 8h 可用"
- 优先级色条：红色（Highest/High），灰色（其他）

**验收：**

- ✅ 周三排了 3 个任务（2h + 4h + 4h），显示"已排 10h / 8h"，进度条红色
- ✅ 周四排了 1 个任务（5h），显示"已排 5h / 8h"，进度条绿色

#### 4. 分享快照生成（2h）⭐

**输入：** 当前日历状态
**输出：** 自包含的 HTML 文件

**技术要求：**

- 导出 JSON：tasks + schedule + exportDate
- 生成 HTML：内联 FullCalendar CSS/JS（不用 CDN）
- 只读模式：`editable: false`
- 浏览器下载：`blob + URL.createObjectURL`

**验收：**

- ✅ 点"分享"按钮，下载 `jira-calendar-{timestamp}.html`
- ✅ 同事双击 HTML，浏览器打开，看到你的日历（不需要 Jira token）
- ✅ 快照显示生成时间："📅 快照生成时间: 2026-06-12 14:30"
- ✅ 快照只读（不能拖拽）

**关键修复：**

- `file://` 协议无法加载 CDN，必须内联完整 bundle
- Day 2 第一小时优先测试此功能

#### 5. 基础 UI（1h）

**输出：** 可用的界面

**技术要求：**

- FullCalendar 默认样式
- 顶部：刷新按钮 + 分享按钮
- 任务卡片：标题 + SP + 左侧优先级色条

**验收：**

- ✅ 界面不丑（能给同事看）
- ✅ 刷新按钮：重新拉取 Jira 任务
- ✅ 分享按钮：生成 HTML 下载

#### 6. 测试（1h）

**验收：**

- ✅ 拖拽任务 → 刷新 → 排期不丢失
- ✅ 生成分享 HTML → 打开 → 看到日历
- ✅ 排 10h 任务到周三 → 进度条红色

### 明确砍掉的功能

| 功能 | 理由 | 何时考虑 |
|------|------|---------|
| 实时分享链接 | 需要后端 + 数据库 + 部署 | 验证成功后的 v2 |
| 双向同步回 Jira | 需要写权限 + 冲突处理 | 验证成功后的 v2 |
| 新任务通知 | 需要 webhook + 浏览器 API | 验证成功后的 v2 |
| 团队协作 | 需要多用户系统 | 验证成功后的 v3 |
| AI 排优先级 | 需要数据积累 + 模型 | 验证成功后的 v3 |
| 未排期列表 | 增加复杂度 | 强制所有任务分配到某天 |
| 5 色优先级 | 过度设计 | 红/灰两色足够 |
| 多浏览器测试 | 时间成本高 | 推广时再测 |

## 核心流程

### 流程 1：首次使用（技术验证）

```
1. 用户打开工具
2. 输入 Jira domain + API token + email
3. 点"连接"按钮
4. 工具拉取任务（JQL 查询）
   → 成功：显示日历 + 任务列表
   → 失败：显示错误信息（CORS / 认证 / 权限）
5. 用户拖拽任务到日历
6. 刷新页面，排期保留
```

**失败路径：**

- CORS 错误 → 检查 Vite proxy 配置
- 认证失败 → 检查 API token 是否有效
- 无任务 → 显示空日历（不报错）

### 流程 2：分享快照（价值验证）

```
1. 用户点"分享"按钮
2. 工具导出当前状态为 JSON
3. 生成自包含的 HTML 文件
4. 浏览器下载 `jira-calendar-{timestamp}.html`
5. 用户发给同事（Slack / Email / 共享文件夹）
6. 同事双击 HTML，浏览器打开
7. 同事看到只读日历（不能编辑）
```

**失败路径：**

- HTML 生成失败 → 降级到 Markdown 导出
- 同事打开空白 → 检查浏览器 console（可能是 JS 错误）
- 同事看不懂 → 说明工具 UX 有问题

### 流程 3：刷新数据（日常使用）

```
1. 用户点"刷新"按钮
2. 工具重新拉取 Jira 任务
3. 读取 localStorage 中的排期
4. 合并逻辑：
   - 任务在 Jira 已删除 → 从 localStorage 移除
   - 任务是新增的 → 显示为"未排期"（用户手动拖拽）
   - 任务仍存在 → 保留本地排期
5. 更新日历显示
```

## 技术架构

### 技术栈

- **前端框架：** Vite + React（或 Vue，根据熟悉度）
- **日历库：** FullCalendar v6（MIT 许可的 daygrid 插件）
- **存储：** localStorage（不需要后端）
- **认证：** Jira Cloud API token

### 关键依赖

```json
{
  "@fullcalendar/core": "^6.1.0",
  "@fullcalendar/daygrid": "^6.1.0",
  "@fullcalendar/interaction": "^6.1.0"
}
```

### 文件结构

```
/src
  /api
    jira.ts          # Jira API 调用
  /components
    Calendar.tsx     # 日历组件
    ShareButton.tsx  # 分享按钮
  /utils
    storage.ts       # localStorage 操作
    export.ts        # HTML 导出
  App.tsx            # 主入口
/vite.config.ts      # Vite proxy 配置
/.env.local          # Jira credentials（不 commit）
```

### 数据模型

**任务（Jira）：**

```typescript
interface JiraTask {
  key: string;              // "PROJ-123"
  fields: {
    summary: string;        // "修复登录 bug"
    customfield_10016?: number; // Story Points
    priority?: {
      name: string;         // "Highest" / "High" / ...
    };
  };
}
```

**排期（localStorage）：**

```typescript
interface Schedule {
  [taskId: string]: string; // { "PROJ-123": "2026-06-15" }
}
```

**导出状态：**

```typescript
interface ExportState {
  tasks: JiraTask[];
  schedule: Schedule;
  exportDate: string;       // ISO 8601
}
```

### Vite Proxy 配置

```typescript
// vite.config.ts
export default {
  server: {
    proxy: {
      '/api/jira': {
        target: 'https://your-domain.atlassian.net',
        changeOrigin: true,
        rewrite: (path) => path.replace(/^\/api\/jira/, '')
      }
    }
  }
}
```

### 认证

```typescript
const token = btoa(`${email}:${apiToken}`);
const headers = {
  'Authorization': `Basic ${token}`,
  'Content-Type': 'application/json'
};
```

## 验收标准

### 第一阶段：技术可行（周末）

1. ✅ **Jira 连接**
   - 输入 domain + token，能拉取任务
   - 在 console 看到任务 JSON
   - 错误有明确提示（CORS / 认证 / 权限）

2. ✅ **日历拖拽**
   - 任务显示在日历上
   - 拖拽到另一天，位置更新
   - 刷新页面，排期不丢失

3. ✅ **工作量可视化**
   - 每天显示"已排 Xh / 8h"
   - 进度条颜色正确（绿/黄/红）
   - 高优先级任务有红色色条

4. ✅ **分享快照**
   - 点"分享"按钮，下载 HTML
   - 同事双击 HTML，看到日历（只读）
   - 快照显示生成时间

5. ✅ **基础 UI**
   - 界面不丑（能给同事看）
   - 刷新按钮 + 分享按钮可用

### 第二阶段：价值验证（下周）

1. ✅ **给 2-3 个同事发分享 HTML**
   - 至少 1 个同事说"有用"或"我也想用" → 继续
   - 所有同事说"没啥用" → 停止

2. ✅ **你自己用 1 天**
   - 周一早上用工具规划今天
   - 觉得"比直接看 Jira 好" → 继续
   - 觉得"也就那样" → 停止

### 失败判定（明确停止条件）

❌ **立即停止：**

- 周六调试 Jira 连接 3 小时还没成功
- 周日 HTML 导出测试失败，降级方案也不行
- 周一你自己用 1 天觉得"没用"

❌ **下周停止：**

- 所有同事（2-3 个）都说"没啥用"
- 你用了 3 天就不想用了

✅ **继续投入：**

- 至少 1 个同事说"我也想用"
- 你自己每天都在用

## 风险和开放问题

### 风险 1：Jira 认证调试超时

**概率：** 50%（最可能）
**影响：** 周末原型做不完
**对冲：**

- 第一天优先解决 CORS + 认证（2-3h）
- 如果 3h 还没成功，暂停决策
- 备选：用 mock 数据先把日历跑通

### 风险 2：HTML 导出 file:// 协议失败

**概率：** 30%
**影响：** 分享功能不可用，无法验证价值
**对冲：**

- Day 2 第一小时就测试 HTML 导出
- 如果 CDN 加载 blocked，立刻切换到内联 bundle
- 备选：导出为 Markdown 表格

### 风险 3：同事说"没啥用"

**概率：** 40%
**影响：** 工具没有传播价值
**对冲：**

- 这就是验证的目的 — 8h 损失可接受
- 问他们："你平时怎么规划任务？"
- 问他们："Jira 哪里让你不爽？"
- 写复盘，学到什么

### 风险 4：同事说"我在用 Akiflow"

**概率：** 30%
**影响：** "极简"优势不成立
**对冲：**

- 问："Akiflow 哪里不好？"
- 如果他们说"挺好的"，那就停止
- 验证"极简 + Jira-only"是否真的是优势

### 开放问题

**Q1: Story Points 字段 ID 因 Jira 实例而异怎么办？**

- A1: 周末原型硬编码 `customfield_10016`（Jira Cloud 标准）
- A1: 如果失败，让用户手动输入字段 ID（v2 功能）

**Q2: 如果 Jira 任务超过 100 个怎么办？**

- A2: JQL 限制最近 90 天（`updated >= -90d`）
- A2: 如果还超过 100，显示前 100 个 + 警告

**Q3: 如果用户想看团队成员的日历怎么办？**

- A3: 明确不做（需要后端 + 权限）
- A3: 如果验证成功，v2 再考虑

**Q4: 如果 FullCalendar 内联太大（>2MB）怎么办？**

- A4: 优先尝试内联（验证可行性）
- A4: 如果太大，降级到截图 + Markdown 导出

## 实施计划

### Day 1（周六，4-5h）

**9:00 - 11:00: Jira 连接（2h）**

1. 初始化 Vite 项目
2. 配置 Vite proxy
3. 实现 Jira API 调用
4. 在 console 看到任务 JSON
   - **决策点：** 如果 3h 还没成功 → 暂停决策

**11:00 - 12:00: Mock 日历（1h）**
5. 安装 FullCalendar
6. 用 mock 数据显示日历
7. 测试拖拽

**12:00 - 13:00: 工作量可视化（1h）**
8. 汇总每天 story points
9. 显示进度条（绿/黄/红）

### Day 2（周日，4-5h）

**9:00 - 10:00: HTML 导出测试（1h）** ⭐ 优先
10. 生成最简单的 HTML（只内联 JSON）
11. 测试 `file://` 协议能否打开
    - **决策点：** 如果 CDN blocked → 立刻切换内联 bundle

**10:00 - 11:00: 连接真实数据（1h）**
12. 把 mock 数据换成 Jira API

**11:00 - 12:00: localStorage 持久化（1h）**
13. 拖拽后存储
14. 刷新后恢复

**12:00 - 13:00: 分享功能完善（1h）**
15. 完善 HTML 模板（内联 FullCalendar）
16. 加只读模式
17. 加生成时间戳

**13:00 - 14:00: UI 美化（1h）**
18. 加刷新按钮 + 分享按钮
19. 加优先级色条
20. 测试完整流程

### Day 3（周一）

**早上：** 自己用 1 天

- **决策点：** 比直接看 Jira 好 → 发给同事；否则 → 停止

**下午：** 发分享 HTML 给 2-3 个同事

### Day 8（下周五）

**决策点：** 至少 1 个同事说"我也想用" → 升级 v2；否则 → 写复盘

## 工作量估算

| 任务 | 时间 |
|------|------|
| Jira 连接 + 认证调试 | 3h |
| 日历拖拽 + localStorage | 2h |
| 工作量可视化 | 1h |
| 分享快照生成 | 2h |
| 基础 UI | 1h |
| 测试 | 1h |
| **总计** | **8-9h** |

**缓冲时间：** 预留 1-2h 处理意外（CORS 调试、HTML 导出失败）

## 成功定义

### 技术成功（周末）

- ✅ 能连接 Jira 并拉取任务
- ✅ 能拖拽任务并保存排期
- ✅ 能生成分享 HTML
- ✅ 工作量可视化正常显示

### 价值验证成功（下周）

- ✅ 你自己每天都在用
- ✅ 至少 1 个同事说"我也想用"

### 失败（写复盘）

- ❌ 你自己用 1 天就不想用了
- ❌ 所有同事都说"没啥用"
- ❌ 周六调试 3h 还连不上 Jira

## 相关文档

- CEO Plan: `/Users/shimo/.gstack/projects/super-gstack-demo/ceo-plans/2026-06-12-jira-calendar.md`
- FullCalendar 文档: <https://fullcalendar.io/docs>
- Jira Cloud REST API: <https://developer.atlassian.com/cloud/jira/platform/rest/v3/>
