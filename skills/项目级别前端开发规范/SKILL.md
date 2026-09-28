---
name: 项目级别前端开发规范
description: |
  系统（Vue 3.2.47 + TS + Vite + Ant Design Vue + Cesium）专属前端开发技能。
  将项目《开发注意事项.md》规范与「编程专家 / 前端开发规范 / 代码简化 / 前端UI工程 / 前端设计 / Matt Pocock Skills」六套技能规范融合固化，
  用于本项目的页面/组件/图表/接口/地图功能开发、需求对齐、代码审查、重构简化、Bug 诊断、UI 打磨、跨会话交接。

  触发场景（命中任一即使用本技能）：
  - 新增/修改本项目任何 Vue 页面、公共组件、板块子组件、进度图表组件
  - 开发/接入接口（api/modules 新建、mock 路由注册、axios 调用）
  - 地图功能（CesiumMap 封装调用、WMS/WFS、图层交互、点击弹窗）
  - 代码审查 / 重构 / 简化 / 性能优化（限本项目代码）
  - UI 视觉调整 / 换色 / 排版对齐（暗色 GIS 大屏风格）
  - 需求对齐：对齐一下、帮我理清、想清楚再动手、讨论设计、确认方案、grill
  - Bug 诊断：修 Bug、跑不通、报错、抛异常、卡顿、排查、定位问题、故障、崩溃、debug
  - 测试/开发：写测试、TDD、先写测试、红绿重构、test-first
  - 跨会话交接：会话太长了、换新会话、交接一下、handoff、上下文满了、新开一个对话继续
  - 文档沉淀：写 PRD、出需求文档、整理需求、写成 spec、形成文档
  - 关键词：项目规范、按项目风格、板块开发、@/ 路径、--color 变量、cssVar、DataDetailPopup、mapcardNew、CesiumMap、ant-design-vue 组件注册、`<a-button>` 失效、PascalCase 标签
version: 2.1.0
displayName: 大屏前端开发规范
category: dev
agent_created: true
tags:
  - 研发
  - 前端
  - Vue
  - Cesium
  - 项目规范
  - TDD
  - Bug诊断
  - 需求对齐
---

# 智慧检测大屏系统 — 前端开发规范（私有技能）

## 1. 技能定位

本技能是本项目前端开发的**唯一规范入口**，融合了：

| 来源 | 贡献内容 |
| --- | --- |
| 《开发注意事项.md》（项目活文档） | 技术栈红线、目录命名、颜色变量、页面排版、接口规范、Cesium 封装、弹窗交互预设、构建约定、踩坑表 |
| 编程专家.Skill | 六步闭环工作流（分析→方案→执行→验证→交付→复盘）、意图三分法、审计修复分离 |
| 前端开发规范 | 企业级命名/Git/ESLint/安全/性能通用规范 |
| 代码简化 | 简化五原则（保行为/随约定/重清晰/守平衡/限范围）+ 四步流程（理解→识别→增量应用→验证）+ TS/JS 示例，见 `references/frontend-quality.md` 第 1 节 |
| 前端UI工程 | 生产级 UI：状态管理选型、规避 AI 审美、WCAG 可访问性、响应式 |
| 前端开发（impeccable） | 设计上下文先行（目标用户/场景/品牌调性）+ AI Slop Test + DO/DON'T 速查 + 场景化导航（视觉/布局/质量保障），避免泛 AI 审美，见 `references/frontend-quality.md` 第 4 节 |
| Matt Pocock Skills (Kimi) | grilling 需求对齐（一次一问+推荐答案）、grill-with-docs（CONTEXT.md+ADR 同步维护）、tdd 垂直切片、diagnosing-bugs 六阶段、handoff 跨会话交接、to-prd 对话转 PRD |

> 本项目规范文档（`开发注意事项.md`）若更新，以最新内容为准；本技能是对其的**固化提炼**，两者冲突时以项目文档为准。

## 2. 触发词（明确使用时机）

用户输入命中以下任一关键词/场景时，**必须**按本技能执行：

| 类别 | 触发词 / 场景 |
| --- | --- |
| 页面开发 | 新增板块、写页面、开发 `<板块>.vue`、布局调整 |
| 组件开发 | 新建组件、写子组件、公共组件、图表组件 |
| 接口 | 接接口、写 api、mock、请求封装、axios |
| 地图 | Cesium、图层、WMS、WFS、打点、飞行、地图弹窗 |
| 规范关键词 | 项目规范、按项目风格、@/ 路径、--color、cssVar、暗色大屏 |
| 审查重构 | 审查、review、简化、重构、优化、性能 |
| 视觉 | 换色、排版、对齐 Reclassification、风格统一 |
| 需求对齐（grilling） | 对齐一下、帮我理清、想清楚再动手、讨论设计、确认方案、grill、方案、需求、思路 |
| Bug 诊断（diagnosing-bugs） | 修 Bug、跑不通、报错、抛异常、卡顿、慢查询、排查、定位问题、故障、崩溃、debug |
| 测试开发（tdd） | 写测试、TDD、先写测试、红绿重构、test-first、单元测试、集成测试 |
| 跨会话交接（handoff） | 会话太长了、换新会话、交接一下、handoff、上下文满了、新开一个对话继续 |
| 文档沉淀（to-prd） | 写 PRD、出需求文档、整理需求、写成 spec、形成文档、记下来 |

**不触发**：与本项目无关的通用编程问题（走通用技能）、纯信息查询。

## 3. 执行逻辑（六步闭环）

### Step 1 加载上下文（每次必做）

1. 读取项目规范：`开发注意事项.md`（若存在且未在当前上下文）。
2. 按需读取：`src/assets/styles/color-common.less`（颜色变量注册处）、`src/views/progress/type/chart.ts`（cssVar 色板）、目标页面/组件源码。
3. 跳过无关内容：`node_modules/`、`dist/`、`src/assets/image/`（资源不入上下文）。

### Step 2 意图三分法

| 类型 | 判定 | 动作 |
| --- | --- | --- |
| 信息查询 | 只问不改 | 直接回答，不写文件 |
| 简单任务 | 单文件小改动、目标明确 | 直接执行 + 最小验证 |
| 复杂任务 | 多文件/跨模块/新板块/新接口/地图能力 | **先输出方案与影响范围，确认后再动手** |

**需求模糊时进入 Grilling 对齐门控**（Matt Pocock）：当需求含模糊词（"优化一下/弄好/看着办"）、< 20 字无上下文、或决策存在多个分支时，按 `references/mattpocock-workflow.md` 的 grilling 流程**逐分支对齐**：
- 一次只问一个问题，等用户回答再继续（禁止一次抛多个问题）
- 每个问题**先给出推荐答案**再等用户确认（推荐答案展示理解方向，不代表替用户决定）
- 能通过查代码回答的问题，直接查代码，不麻烦用户
- 有代码库时，对齐过程同步维护项目术语/决策（grill-with-docs：更新《开发注意事项.md》或 ADR）
- 完成标准：决策树所有分支已解析，没有未解决的依赖。不要急于结束

### Step 3 制定方案（复杂任务）

- 新板块：按「新板块开发 Checklist」12 步逐项打勾（见 references/project-conventions.md 第 7 节）。
- 新接口：按「新接口接入 6 步」（建文件→引 axios→定义类型→写方法→注册 index→页面调用）。
- 涉及点击弹窗交互：直接复用 4.6 预设（DataDetailPopup + showDetail 模式），**无需重新设计**。

### Step 4 执行（强制规范）

编码时**必须**遵守 references 中的全部红线，逐条自检：

- [ ] 技术栈：Vue 3.2.47（禁 3.5+ API，禁 `useId`）、TS 强制、只用 Cesium（禁 ol）
- [ ] import 一律 `@/` 绝对路径，禁止相对引用
- [ ] ant-design-vue 组件必须本文件显式 `import` 并以 **PascalCase 标签**使用（`<Button>`），**禁止 `<a-button>` 等 kebab 标签**——项目仅全局注册 `ConfigProvider`，其余组件未注册，kebab 标签不渲染会导致 `@click` 全部失效（详见《开发注意事项.md》1.1 节）
- [ ] 颜色一律 `var(--color-*)`，echarts 走 `cssVar()`，禁止硬编码
- [ ] 地图能力走 `CesiumMap` 封装层，禁止裸 import cesium；禁止绕过封装改 `viewer.scene` 属性
- [ ] 页面底图用 `addDefaultBasemap()`，禁止硬编码瓦片 URL
- [ ] 接口统一 `api.模块.方法()`，不直接 import request 发请求（新接口除外，见规范 5.4）
- [ ] 弹窗标题格式：`板块名称 * 模块名称`（星号两侧空格）
- [ ] 可点击项必须 `cursor: pointer`
- [ ] 图表复用 `progress/components/`，色板走 chart.ts
- [ ] **图表数值必须默认可见**：所有 ECharts series 配 `label: { show: true }`（柱顶/折点/雷达拐点/扇区内侧），禁止仅 tooltip 悬停显示（详见 references 4.3）
- [ ] **饼图强制南丁格尔玫瑰图**：`type:'pie'` 须设 `roseType:'radius'`，引导线外侧标签展示「名称 `{b}` + 数值 `{c}%`」并 `labelLine` 可见；`label` 设 `width`+`overflow:'break'` 保证超长换行（详见 references 4.3；环形图保留中心总数不强制转玫瑰）
- [ ] **环形图用引导线展示文字+数值**：`type:'pie'` 环形图（保留中心总数）主扇区 `label.position:'outside'` + `formatter:'{b}  {c}'` 并经 `labelLine` 引导线引出；`label` 设 `width`+`overflow:'break'` 保证超长换行；中心总数由第二层 silent pie 承载（详见 references 4.3）
- [ ] 代码简化五原则：保行为 / 随约定 / 重清晰 / 守平衡 / 限范围
- [ ] 规避 AI 审美：用项目真实调色板（暗色大屏），不堆渐变/圆角/发光
- [ ] 新代码带类型定义，禁 any 泛滥；函数动词开头、布尔 is/has 开头、常量全大写下划线
- [ ] 复杂功能若拆多步实现，按**垂直切片**推进（tdd 精神）：一次完成一个「行为」，验证通过再进下一个，禁止水平切片（先写全部再补全部）
- [ ] 遇到 Bug 时按**六阶段诊断**走（先建反馈循环，再假设，绝不跳过第一阶段），见 `references/mattpocock-workflow.md`

### Step 5 验证（遵循项目约定）

- **【强制】不跑构建/类型验证**：由用户本地 `npm run dev`（端口 8098）验证。
- 需要检查语法时可只做静态自查（括号配对、import 路径存在、类型引用存在）。
- 沙箱内确需构建时：`npm run build-only -- --emptyOutDir false`（跳过 dist 清理保护）。
- 交付时列出：改了哪些文件、为什么改、需要用户验证的点、遗留事项。
- 涉及 Bug 修复时：交付前确认原始复现场景不再复现（重跑复现 loop），回归测试通过或说明原因。

### Step 6 复盘沉淀

- 新踩坑 / 新约定 → 同步更新 `开发注意事项.md` 对应章节 + 项目工作日志。
- 若发现本技能 references 过时 → 直接更新本技能文件。
- **跨会话交接（handoff）**：当会话很长（接近上下文上限）或用户要求换新会话继续时，生成 `handoff-<时间戳>.md` 保存到工作区（见 `references/mattpocock-workflow.md` 第 5 节），内容只含「会话中新产生、尚未写入任何文件」的信息，已有文档只引用路径不复制。
- **需求已对齐、需要沉淀文档** → 按 to-prd 模板输出 PRD（本项目无 Issue Tracker，默认回退本地 markdown）。

## 4. 资源索引

| 文件 | 内容 | 何时读取 |
| --- | --- | --- |
| `references/project-conventions.md` | 项目专属规范全文：技术栈红线、目录命名、颜色变量清单、页面排版、接口/mock、Cesium 封装与性能、4.6 弹窗预设、新板块 Checklist、构建约定、踩坑表 | **每次开发前必读** |
| `references/frontend-quality.md` | 通用前端质量规范：TS/Vue 写法、代码简化原则、UI 工程/可访问性/响应式、性能与安全 | 编码与审查时参考 |
| `references/mattpocock-workflow.md` | Matt Pocock 方法论（项目适配版）：grilling 需求对齐、grill-with-docs 文档维护、tdd 垂直切片、diagnosing-bugs 六阶段诊断、handoff 交接文档模板、to-prd 文档模板 | 需求对齐 / Bug 诊断 / 长会话交接 / 写 PRD 时必读 |

## 5. 输出规范

- 交付内容 = 改动文件清单 + 变更摘要 + 需要用户验证的点 + 已知限制/遗留。
- 遵循项目约定：**不执行 build**；改动增量式、隔离式，不影响其他页面。
- 中文沟通；代码/路径保持原始格式。
