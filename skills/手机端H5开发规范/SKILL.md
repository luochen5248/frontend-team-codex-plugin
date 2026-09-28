---
name: 手机端H5开发规范
description: |
  移动端 H5 / WebView 项目专属前端开发技能（Vue 3 + TS + Vite + Ant Design Vue）。
  沉淀自「移动端 H5 项目」的实战重构，覆盖移动端最容易踩坑的六个维度：
  布局契约、视口与安全区、点击区、软键盘输入、滚动容器、localStorage 配额保护，
  并包含「对话式表单（槽位状态机）」设计要点与真机验证清单。

  触发场景（命中任一即使用本技能）：
  - 移动端 H5 / WebView / 混合 App 内嵌页面的开发与改造
  - 页面高度算不准、内容被底部导航遮挡、出现双滚动条、软键盘顶飞输入框
  - 100vh / 100dvh / calc(100vh - N) 相关布局问题
  - 安全区适配：刘海屏、iPhone Home 指示条、env(safe-area-inset-*)
  - 点击区过小、移动端点击延迟、:active 无反馈
  - localStorage 写入失败 / 会话恢复失效 / 图片 base64 撑爆配额
  - 对话式交互、多步表单、槽位填充（slot filling）状态机设计
  - 移动端表单提交重复触发（连点生成重复数据）
  - Ant Design Vue 在移动端的用法差异与导出名陷阱
  - 关键词：移动端、H5、手机端、响应式、dvh、安全区、safe-area、点击区、软键盘、
    TabBar、视口高度、会话恢复、localStorage 配额、防重复提交、槽位、对话下单

  不触发：PC 大屏 / GIS / Cesium 可视化项目（那类请用「项目级别前端开发规范」）。
version: 1.0.0
agent_created: true
---

# 手机端 H5 开发规范

移动端 H5 的绝大多数布局 Bug，根因都是**「视口高度被多处各自计算」**与**「固定定位元素互相叠加」**。
本技能给出的解法是收敛为**一套 flex 布局契约**，让所有页面遵守同一约定。

---

## 一、布局契约（最重要，改动布局前必读）

### 1.1 结构

```
.app-shell   height:100dvh（回退 100vh）；flex 列；overflow:hidden   ← 整屏，自身不滚动
  ├── Header      flex:0 0 auto
  ├── main        flex:1; min-height:0; overflow:hidden             ← 唯一内容区
  │     └── 页面根元素  ← 页面自己决定怎么滚
  └── TabBar      flex:0 0 auto                                    ← 流内元素，不用 fixed
```

### 1.2 可直接抄用的骨架

```vue
<!-- App.vue -->
<template>
  <div class="app-shell">
    <PageHeader />
    <main class="app-main">
      <RouterView />   <!-- 注意：不要在这里写 class -->
    </main>
    <TabBar v-if="showTabBar" />
  </div>
</template>

<style scoped lang="less">
.app-shell {
  display: flex;
  flex-direction: column;
  height: 100vh;        /* 老浏览器回退 */
  height: 100dvh;       /* iOS 15.4+ / Chrome 108+：排除地址栏 */
  overflow: hidden;
}
.app-main {
  flex: 1;
  min-height: 0;        /* 必须！否则内容撑破容器 */
  display: flex;
  flex-direction: column;
  overflow: hidden;
}
</style>
```

```less
/* 每个页面的根元素 */
.page {
  flex: 1;
  min-height: 0;
  overflow-y: auto;
  -webkit-overflow-scrolling: touch;
}
```

### 1.3 三条硬性红线

1. **不要在 `<RouterView>` 上写 class**。
   Vue 会把 class **合并到子组件根元素**，导致父级 `.page` 与子级 `.page` 同时命中同一个 div，
   两套高度计算叠加（历史上「双重减法」的成因）。

2. **禁止 `calc(100vh - N)` / `min-height: 100vh`**。
   视口高度只在 `.app-shell` 出现一次，其余一律用 flex。
   页面内写 `min-height:100vh` 会被外层 `overflow:hidden` 裁掉底部内容（表单底部尤其危险）。

3. **不要用 `padding-bottom` 给底部导航让位**。
   TabBar 作为流内 flex 子项，天然不遮挡内容。固定定位 + padding 避让是最脆弱的做法。

### 1.4 底部导航（TabBar）正确写法

```less
.tabbar {
  flex: 0 0 auto;
  height: calc(var(--tabbar-h) + var(--safe-bottom));
  padding: 0 0 var(--safe-bottom);
  /* 不要 position: fixed */
}
```

**高亮必须由路由派生，不能用本地 ref**：

```ts
// 正确：后退 / router.replace 后高亮都同步
const activeTab = computed(() => (route.path === '/' ? '/TaskList' : route.path))

// 错误：ref + onMounted，浏览器后退后高亮与页面不一致
```

---

## 二、视口与安全区

### 2.1 viewport 声明（缺一不可）

```html
<meta name="viewport" content="width=device-width, initial-scale=1.0, viewport-fit=cover" />
```

> **没有 `viewport-fit=cover`，`env(safe-area-inset-*)` 恒为 0**，所有安全区适配全部失效。

### 2.2 CSS 变量基线

```less
:root {
  --header-h: 44px;
  --tabbar-h: 60px;
  --safe-bottom: env(safe-area-inset-bottom, 0px);
  --tap-min: 44px;
}
@media screen and (max-width: 768px) {
  :root { --tabbar-h: 50px; }
}
```

### 2.3 高度单位

| 场景 | 用法 |
|---|---|
| 整屏容器 | `height: 100vh; height: 100dvh;`（前者回退，后者生效时覆盖） |
| 页面内部 | 只用 flex，不出现视口单位 |

`dvh` 兼容：iOS 15.4+ / Chrome 108+。老设备回退 `100vh`，不会出现布局崩坏，只是滚动时略有抖动。

---

## 三、点击区与可访问性

- **最小点击区 44px**（`--tap-min`）；空间紧张时不低于 36–40px。
- 加 `touch-action: manipulation`，消除移动端 300ms 点击延迟与双击缩放。
- 优先 `<button type="button">`：天然可聚焦、可被读屏识别。
  若必须用 `<div @click>`，补齐 `role="button"` + `tabindex="0"` + `@keydown.enter`。
- 可点项必须有 `cursor: pointer` 与明确的 `:active` 反馈（移动端无 hover）。

```less
.opt {
  min-height: 36px;
  touch-action: manipulation;
  &:active:not(.opt-disabled) { background: var(--color-primary); color: #fff; }
}
@media screen and (max-width: 768px) {
  .opt { min-height: 40px; }
}
```

---

## 四、输入与软键盘

- 多行文本用 `Textarea` + `:auto-size="{ minRows: 1, maxRows: 4 }"`，**不用单行 Input**。
  移动端字段（地址、单位、备注）常需换行。
- 软键盘**不会可靠触发换行的回车语义**。约定：回车 = 发送，`Shift + Enter` = 换行：

```ts
const onPressEnter = (e: Event) => {
  const ev = e as KeyboardEvent
  if (ev.shiftKey) return
  ev.preventDefault()
  sendText()
}
```

- 始终提供**可见的发送按钮**作为主路径，不要把回车当唯一入口。
- 输入区**不要** `position: fixed` + 硬编码 `bottom`（软键盘弹起会被顶飞）。
  作为页面 flex 列的末端子项即可。

---

## 五、滚动

- 滚动容器必须显式：`overflow-y: auto` + `min-height: 0` + `-webkit-overflow-scrolling: touch`。
- **一个页面只允许一个滚动容器**。外层 + 内层同时可滚会导致手势冲突，表现为「滚不动」或「滚两层」。

---

## 六、localStorage 配额保护（高频事故点）

手机直出照片转 base64 通常 **3–5MB**，直接写入会撑爆 localStorage 约 **5MB** 配额，
且失败常被 `try/catch` 吞掉，表现为「会话恢复功能静默失效」。

### 6.1 图片必须先压缩再落盘

```ts
function compressImage(dataUrl: string, maxWidth = 640, quality = 0.7): Promise<string> {
  return new Promise((resolve) => {
    const img = new Image()
    img.onload = () => {
      try {
        const scale = Math.min(1, maxWidth / (img.width || maxWidth))
        const canvas = document.createElement('canvas')
        canvas.width = Math.max(1, Math.round(img.width * scale))
        canvas.height = Math.max(1, Math.round(img.height * scale))
        const ctx = canvas.getContext('2d')
        if (!ctx) return resolve(dataUrl)
        ctx.drawImage(img, 0, 0, canvas.width, canvas.height)
        resolve(canvas.toDataURL('image/jpeg', quality))
      } catch { resolve(dataUrl) }
    }
    img.onerror = () => resolve(dataUrl)
    img.src = dataUrl
  })
}
```

> 原则：**原图送识别（保精度），缩略图进存档（保空间）**。

### 6.2 deep watch 必须防抖

```ts
let timer: ReturnType<typeof setTimeout> | null = null
const schedulePersist = () => {
  if (timer) clearTimeout(timer)
  timer = setTimeout(persistSession, 300)
}
watch([messages, draft, phase], schedulePersist, { deep: true })

onBeforeUnmount(() => {          // 防抖会吞掉最后一次变更
  if (timer) { clearTimeout(timer); timer = null }
  persistSession()
})
```

### 6.3 超限降级 + 历史数据兼容

```ts
const MAX = 1.5 * 1024 * 1024
let payload = JSON.stringify(session)
if (payload.length > MAX) payload = JSON.stringify(stripImages(session))  // 只留文字
// 读取时同样判断，兼容旧版本写入的超大数据
```

---

## 七、对话式表单 / 槽位状态机设计要点

适用于「多步引导收集信息」类交互（对话下单、向导表单）。

### 7.1 状态模型

```ts
phase: 'collecting' | 'preview' | 'done'
editingKey: SlotKey | null   // 非空 = 「修改单字段」模式
slotIndex: number
submitting: boolean          // 提交锁
```

### 7.2 五个必踩的坑

1. **恢复会话必须重建派生状态**。只恢复 `draft` 不恢复 `previewTask`，预览卡会因 `v-if="phase==='preview' && previewTask"` 不成立而不渲染，用户无法确认提交。
2. **提交必须防重**：`if (submitting) return`，否则连点生成多条重复数据。
3. **历史选项必须禁用**。只保留最后一条助手提问的选项可点
   （`latestAskIndex === messages.length - 1 && 该条是 assistant 且带 options`），
   否则点旧选项会被当成**当前**槽位的答案，造成错填。
4. **必须支持单字段修改**。让用户为改一个字段而重填全部，是最常见的体验事故。
   预览区每行可点 → 进入 `editingKey` 模式 → 改完自动回预览。
5. **索引查找必须防 -1**。`SLOTS.findIndex(...)` 无匹配时返回 `-1`，直接 `SLOTS[-1]` 会崩。

### 7.3 接真实大模型

把本地解析函数（如 `callInspectionSkill`）替换为模型调用即可：
入参 = 已填槽位 + 用户语句，出参 = 结构化抽取结果。**页面交互与提交逻辑无需改动**。

---

## 八、Ant Design Vue 移动端注意

| 事项 | 说明 |
|---|---|
| **显式 import + PascalCase** | 项目通常只全局注册 `ConfigProvider`。`<a-button>` 不会渲染且 `@click` **静默失效**。必须 `import { Button } from 'ant-design-vue'` 后用 `<Button>`。 |
| **多行文本导出名是 `Textarea`** | 不是 `TextArea`。要写 `<TextArea>` 请 `import { Textarea as TextArea } from 'ant-design-vue'`。 |
| `LayoutFooter` 默认 padding | 默认 `padding: 24px 50px`，做 TabBar 时需显式清零。 |
| Drawer 宽度 | 移动端用百分比（如 `:width="'92%'"'），不要固定 px。 |

---

## 九、类型检查配置（常见误报来源）

`@vue/tsconfig` 默认 `lib: ["ES2016", "DOM", "DOM.Iterable"]`，会导致以下**误报**
（代码在浏览器中运行正常，仅类型层面报错）：

- `padStart` / `Object.entries`（ES2017）
- `Promise.prototype.finally`（ES2018）

修复（只影响类型检查，运行时转译由 Vite/esbuild 决定）：

```json
{ "compilerOptions": { "lib": ["ESNext", "DOM", "DOM.Iterable"] } }
```

另一类常见误报：`import` 语句写在 `<script setup>` 中部（非文件顶部）时，
`vue-tsc` 会报 TS1232 / TS2307。**运行时正常**——Vue SFC 编译器会 hoist import。

---

## 十、真机验证清单（提测前自查）

- [ ] iOS Safari：地址栏收起/展开时不跳动，无双滚动条
- [ ] iPhone（有 Home 条机型）：底部导航不被 Home 指示条遮挡
- [ ] 安卓 WebView：软键盘弹起时输入框可见、不被遮挡
- [ ] 弱网/断网：提交失败有提示且可重试，不产生重复数据
- [ ] 快速连点提交按钮：只生成一条记录
- [ ] 填写中途刷新：草稿可恢复，派生状态（预览卡等）正常显示
- [ ] 横竖屏切换：布局不破版
- [ ] 长文本（工单号、地址）：不溢出，正确换行或省略

---

## 十一、排查速查表

| 现象 | 根因 | 修法 |
|---|---|---|
| 页面底部内容被裁掉 | 页面写了 `min-height:100vh`，被外层 `overflow:hidden` 裁切 | 改 `flex:1; min-height:0; overflow-y:auto` |
| 高度怎么调都不对 | 父子 `.page` 同时命中同一元素（RouterView 上写了 class） | 去掉 RouterView 的 class |
| 底部导航与内容重叠/留缝 | TabBar 用 fixed，页面用 padding 避让，断点高度不一致 | TabBar 改流内 flex 子项，高度用 CSS 变量 |
| 后退后 TabBar 高亮不对 | 高亮用本地 ref | 改 `computed(() => route.path)` |
| 会话恢复失效且无报错 | base64 图片撑爆 localStorage，被 try/catch 吞掉 | 图片压缩 + 超限剥离 + 防抖落盘 |
| 点旧按钮填错字段 | 历史选项未禁用 | 用 `latestAskIndex` 禁用非最新选项 |
| 连点提交生成多条数据 | 无提交锁 | 加 `submitting` 状态 |
| `env(safe-area-inset-*)` 为 0 | viewport 缺 `viewport-fit=cover` | 补上 |
| 上传图片后页面卡死 | 大 base64 触发 deep watch 同步序列化 | 防抖 + 压缩 + 剥离 |
