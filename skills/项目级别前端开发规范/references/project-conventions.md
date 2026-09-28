# 项目专属规范 — 智慧检测大屏系统

> 固化自《开发注意事项.md》（2026-08-14 版本），以项目活文档为最终权威。

## 1. 技术栈红线

| 项 | 约定 | 说明 |
| --- | --- | --- |
| Vue | **锁定 3.2.47** | 禁 3.5+ 新 API（`useId` 等），唯一 ID 用模块级计数器 `let seq = 0; const id = ++seq` |
| TypeScript | 强制 | 新增代码/类一律 TS，禁止裸 JS |
| GIS 引擎 | **只用 Cesium** | OpenLayers 已全部移除，禁止再引入 `ol` 相关依赖 |
| Cesium 访问 | **走 `src/cesiumConfig/` 封装** | 页面禁止 `import ... from 'cesium'`，一律通过 `CesiumMap` |
| UI 库 | Ant Design Vue `^4.0.0-rc.6` | 全局暗色主题由 `App.vue` 的 `a-config-provider` 配置 |
| 样式 | Less + Scss | `color-common.less` 最先加载，`lc.less` 最后加载兜底覆盖 |
| 路由 | hash 模式 | `createWebHashHistory()`，带登录守卫 |

## 2. 目录与命名

### 2.1 目录结构要点

- **板块主页面**：`src/views/<板块>/<板块>.vue`（主页面在板块同名目录内）
- **板块子组件**：扁平放在板块根级，不套 `components/` 子目录（仅 `overview/`、`progress/` 例外）
- **公共组件**：一律 `src/components/` 根级（禁 `common/` 子目录）
- **API**：`src/api/modules/<业务>.ts` → 注册到 `src/api/index.ts` 汇总导出
- **Mock**：`src/mock/data/` + `src/mock/routes.ts` 路由表
- **图表组件**：`src/views/progress/components/`（ProgressPieChart/BarChart/LineChart）
- **图表色板**：`src/views/progress/type/chart.ts`（CHART_THEME 等 + cssVar() 工具）

### 2.2 命名规则

| 对象 | 规则 | 示例 |
| --- | --- | --- |
| Vue 组件文件 | 大驼峰 | `CategoryTabs.vue`、`ProgressLineChart.vue` |
| 工具/类型文件 | 小驼峰或短横线 | `chart.ts`、`constant.ts` |
| API 模块文件 | 业务名小驼峰 | `sys.ts`、`device.ts` |
| 变量/函数 | 小驼峰，函数动词开头 | `getMenuTree()`、`activeCategory` |
| 常量 | 全大写下划线 | `COLOR_ROADBED`、`MANAGEMENT_OBJECT_COLORS` |
| 布尔 | is/has 开头 | `isVisible`、`hasToken` |

### 2.3 【强制】import 一律 `@/` 绝对路径

- 所有组件/工具/样式引入用 `@/` 绝对路径（`@/views/...`、`@/components/...`、`@/api/...`）
- **禁止 `./` 或 `../` 相对引用**（2026-08-12 起全项目执行）

## 3. 颜色统一规范（强制）

- 所有页面颜色在 `src/assets/styles/color-common.less` 的 `:root` 注册为 `--color-*` 变量；**组件内禁止硬编码颜色**
- **canvas 图表（echarts）不能用 `var(--x)`**，必须走 `chart.ts` 的 `cssVar(name, fallback)` 工具
- SVG 内联样式可用 `fill="var(--color-*)"`（DOM 上下文可解析）
- 组件 `<style>` 中一律 `var(--color-*)`

### 关键色板速查（color-common.less）

| 用途 | 变量 |
| --- | --- |
| 最深背景 / 面板 | `--color-bg-deep:#050810` / `--color-panel-bg:#0c1828` |
| 科技青强调 | `--color-cyan:#3bf6ff`（含 -deep/-border/-scrollbar 系列） |
| 文本层级 | `--color-text-primary/-secondary/-dim/-faint` |
| 状态色 | `--color-green:#22c55e`、`--color-red:#ff7a7a`、`--color-red-strong:#ef4444` |
| 五大对象 | roadbed 路基`#f59e0b`、pavement 路面`#d4af37`、bridge 桥梁`#3b82f6`、tunnel 隧道`#a855f7`、underground 地下`#ef4444` |
| 预警等级 | `--color-alert-red/orange/yellow/normal` |
| 进度状态 | `--color-status-done/doing/todo/overdue` |
| 图表专用 | `--color-chart-tooltip-bg/-axis/-split/-pie-border/-line-border/-area-start/-area-end` |
| 设备/车辆 | `--color-device-working:#36d399`、`-idle:#facc15`、`-offline:#8494ad`、`-alert:#ef4444`、`-track`、`-panel-border` |

## 4. 页面统一排版

### 4.1 布局骨架（LandingImplementation 为基准）

- 顶栏 64px（App.vue：logo + 顶部菜单 + 标题）+ 左面板 320px / Cesium 地图 / 右面板 340px + 底栏 36px 状态栏或底部 Tab
- 面板容器 `.panel-overlay`：`display: flex; justify-content: space-between; padding: 12px`
  - ⚠️ 不要用 `grid-template-columns: 320px 1fr 340px` 装两个子节点（右面板会被排进中间列，已踩坑）
- 面板区块统一 `<mapcardNew title="xxx" describe="yyy">` + `<template v-slot:content>`
- 地图容器 `#cesiumContainer<页面名>` 占满中间区域（`position: absolute` 铺底）

### 4.2 通用组件

- `mapcardNew.vue`：公共面板卡片（标题白字 + 渐变背景 + 四角装饰），所有面板统一用它
- `MapClickPopup.vue`：图层点击弹窗（坐标 + WFS 反查、可拖拽、可复制坐标），封装层只回传数据
- `FloatingTool.vue`：地图悬浮绘制/测量工具（传入 `map-ins`）
- `LogoutButton.vue`：退出登录按钮（含确认弹窗）

### 4.3 图表统一

- 进度类图表（饼/柱/线）复用 `progress/components/`，色板走 chart.ts，均支持 `@chart-click` 联动
- 轻量 SVG 图表（gauge/severity/donut）手绘无依赖，色值用 `var(--color-*)` 或 chart.ts 常量
- 新图表二选一：SVG 手绘 / ECharts + cssVar，禁止裸写颜色
- **【强制】数值默认可见，禁止仅靠 tooltip 悬停显示**（2026-08-15 全局规范）：
  - ECharts 所有 series 必须配 `label: { show: true, ... }`，数值直接标注在图元上（柱顶 `position: 'top'`、折点上方 `top`、雷达拐点 `top`、环形图引导线外侧 `outside`）
  - 环形图扇区用引导线（`labelLine`）在扇区**外侧**展示「重点文字（名称 `{b}`）+ 数值（`{c}`）」：`label: { show:true, position:'outside', formatter:'{b}  {c}' }` 并启用 `labelLine: { show:true, length, length2 }`；文字超长换行：`label` 设 `width`+`overflow:'break'`（如 `width:64`）；**中心总数保留**，用第二层 silent pie 承载（`radius: ['0%','0%']` + `label position: 'center'`），与外侧引导线标签共存
  - SVG 手绘图表（gauge/severity/donut）数值/百分比必须直接渲染在图形上，不得依赖 hover
  - 已达标基线：ProgressPieChart / KeyPointCard / Reclassification RightPanel；新图表照抄以上写法
- **【强制】饼图必须用南丁格尔玫瑰图（roseType），并用引导线展示文字+数值**（2026-08-15 全局规范）：
  - 饼图（`type: 'pie'`）series 必须设 `roseType: 'radius'`，圆心角相同、扇区半径随数值变化，直观对比占比差异（区别于保留中心总数的「环形图」，环形图不强制转玫瑰）
  - 引导线外侧标签同时展示「重点文字（名称 `{b}`）+ 数值（`{c}`）」：`label: { show: true, position: 'outside', formatter: '{b}  {c}%' }`，并启用 `labelLine: { show: true, length, length2 }`
  - 文字超长换行：`label` 设 `width`（如 92）+ `overflow: 'break'`（或 `overflow: 'truncate'`），保证长名称/长数值自动折行，不溢出、不遮挡
  - 已达标基线：ProgressPieChart（进度占比）；新饼图照抄该写法，禁止退化成普通平铺饼图或仅靠 tooltip 显示

### 4.4 移动端 H5 视口基线（移动端 H5 项目，2026-09-06 新增）

- **【强制】整屏高度只消费 `--app-vh`，禁止裸写 `100vh` 撑整屏**
  - `--app-vh` 由 `src/utils/viewport.ts` 的 `initViewportHeight()` 用 `window.innerHeight` 实测写入（rAF 节流监听 `resize` / `orientationchange`），在 `main.ts` 的 `createApp` 之前调用
  - 标准回退链（App.vue `.app-shell`）：`height: 100%` → `height: 100vh` → `height: var(--app-vh, 100dvh)`
  - 原因：老 WebView（微信 X5 / iOS 15.4 以下）不支持 `dvh`，回退 `100vh` 取的是「地址栏收起后的大视口」，底部 TabBar 被浏览器工具栏挡住，只有滚动才看得到全高
  - 刻意不用 `visualViewport.height`：软键盘弹出时整屏高度会反复抖动
- **【强制】全局视口基线在 `src/assets/styles/base.less`**，必须在 `lc.less` 之后引入才能覆盖组件库样式：
  - `*,*::before,*::after { box-sizing: border-box }`
  - `html, body, #app { height:100%; margin:0; padding:0 }` —— 项目未引 `ant-design-vue/dist/reset.css`，body 默认 8px margin 会撑出 16px 多余滚动区
  - `body { overflow:hidden }` —— 整屏不滚动，滚动交给各页面 `.page` 自己（`flex:1; min-height:0; overflow-y:auto`）
- 新增页面不要自行 `calc` 视口高度；TabBar 保持 `.app-shell` 流内末尾子项（`flex:0 0 auto` + `box-sizing:border-box`），不要改回 fixed，也无需 padding 占位
- 全屏浮层（如 `SignatureImage` 签名层）复用同一基线：`height: var(--app-vh, 100dvh)`
- **【注意】全局 `border-box` 之后的点击区**：`min-height` + 上下 `padding` 的组合不再被 padding 撑高（原 content-box 下 36px 实际渲染 52px，现在只有 36px）。凡是可点元素（`.opt` / `.chip` / 各类按钮）必须显式写 `min-height: var(--tap-min)`；只有左右 padding（`padding: 0 12px`）的元素不受影响，无需改

## 5. 点击数据/图表弹窗交互预设（4.6 节，已固化）

> 新页面直接照抄，**无需重新沟通设计**。已应用：LandingImplementation / Reclassification。

### 交互三步：点击 → 聚焦 → 弹窗

1. 显示对应图层（先模拟 GeoJSON，后台就绪换 `addWMSLayer`），隐藏其余非聚焦图层
2. `mapIns.flyTo({ lng, lat, height, duration: 1.6 })` + `addPointAt` 打点
3. 弹出 `DataDetailPopup`

### 强制规则

- **标题格式**：`板块名称 * 模块名称`（星号两侧各一个空格），如 `路面 * 预警等级`
- 板块映射：pavement→路面 / bridge→桥梁 / tunnel→隧道 / underground→地下
- 图表 key → chartTitle 中文映射（trend→趋势分析 / donut→分布图 / radar→分项指数 / iri→IRI 平整度 / bar→深度分布 / env→环境荷载 / rate→发展速率 / gauge→仪表盘）
- 弹窗组件：`src/components/DataDetailPopup.vue`（导出类型 `DataDetailInfo`/`DetailBadge`，主题色 `--detail-color`）
- 定位：有底部状态栏 → `bottom: 44px`；有底部 Tab → `bottom: 76px`
- 可点击项必须 `cursor: pointer`（CSS）或 `series.cursor: 'pointer'`（ECharts）
- **页面专属弹窗复用边界（强制）**：一页定制的弹窗只在本页复用；他页需要时**复制一份到自己的板块目录**再微调，禁止跨页 import 别家页面的专属弹窗（真正跨板块的通用弹窗才放 `src/components/` 根级）
- **嵌入面板内的弹窗须 `<Teleport to="body">`**：中栏/侧栏悬浮面板位于 `.panel-overlay`（`z-index:10`）层叠上下文中，弹窗若渲染在面板组件内会被 `FloatingTool` / `LayerMenu` / 警铃压住；Teleport 到 body 后与页面顶层专属弹窗（`z-index:1200`）同级。参考 `SensorDetailPopup.vue`

### 标准接入代码（照抄）

```html
<div v-if="detailVisible && detailInfo" class="detail-wrap">
  <DataDetailPopup :info="detailInfo" :visible="detailVisible" @close="detailVisible = false" />
</div>
```

```ts
import DataDetailPopup from '@/components/DataDetailPopup.vue'
import type { DataDetailInfo } from '@/components/DataDetailPopup.vue'

const detailInfo = ref<DataDetailInfo | null>(null)
const detailVisible = ref(false)

function showDetail(detail: DataDetailInfo, center: [number, number], height = 30000) {
  const map = mapIns.value
  if (!map) return
  map.flyTo({ lng: center[0], lat: center[1], height, duration: 1.6 })
  map.removeLayer('clickMarker')
  map.addPointAt('clickMarker', center[0], center[1], { color: detail.color, pixelSize: 14 })
  detailInfo.value = detail
  detailVisible.value = true
}
```

```less
.detail-wrap {
  position: absolute;
  left: 50%;
  bottom: 44px; /* 有底部 Tab 用 76px */
  transform: translateX(-50%);
  z-index: 30;
  pointer-events: auto;
}
@media (max-width: 767px) {
  .detail-wrap { left: 12px; right: 12px; transform: none; }
}
```

## 6. 接口调用规范

### 6.1 统一请求封装（src/axios.ts）

- 超时 15s；自动带 token（`Authorization: Bearer <token>`）；请求去重；统一响应 `{ code, message, data }`
- `code === 200` → 返回 `response.data`；`code === 401` → 提示登录过期跳登录页；其他 → `message.error` + reject
- 不需 token 的接口加 `meta: { isToken: false }`

### 6.2 API 组织与调用

- 页面统一 `import api from '@/api/index'`，通过 `api.<模块>.<方法>(params)` 调用
- **不要**在页面直接 `import request from '@/axios'` 发请求

### 6.3 新接口接入 6 步

1. 新建 `src/api/modules/<业务名>.ts`
2. `import request from '@/axios'`
3. interface 定义参数/返回类型（禁 any 泛滥）
4. 写方法：url 带前缀 `/dbxq_yzt`；查询/提交一般 `method: 'post'`；分页拼 query
5. 注册到 `src/api/index.ts`
6. 页面调用 `api.xxx.getXxxList(...)`，`res.code === 200` 取 `res.data`

### 6.4 地图空间数据

- WMS：`/cesiumAPI/gzkj/wms`；WFS：`/cesiumAPI/gzkj/wfs`；工作区 `gzkj`；常用图层 `azf`（安置房）、`jdsc`（道路修复）
- WFS-T 事务：`CesiumMap.updateFeature / insertFeature / deleteFeature`（WFS 1.1.0 + GML3，几何列 `geom`，EPSG:4326 lat,lng 轴序）
- SHP 上传：`shpjs` 解析，结果直接为 GeoJSON

### 6.5 Mock（无后台时使用，后台就绪无缝切换）

- 工作原理：`axios-mock-adapter` + `onNoMatch: 'passthrough'`（未命中走真实后端）
- 无缝切换：`.env.development` 中 `VITE_USE_MOCK=false` 全关；或 `src/mock/routes.ts` 单条 `enabled: false`
- 新增 mock：`src/mock/data/` 建数据文件 → `routes.ts` 注册路由 → api 模块定义方法 → 页面照常调用
- mock 数据颜色必须用 chart.ts 的 cssVar 常量（禁止硬编码）

## 7. Cesium 地图接入规范

### 7.1 页面接入五步

```ts
const mapIns = ref<CesiumMap | null>(null)

onMounted(async () => {
  mapIns.value = new CesiumMap()
  await mapIns.value.init('cesiumContainerLanding')
  mapIns.value.addDefaultBasemap({
    clickable: true,
    onLayerClick: (info) => openPopup(info),   // 弹 MapClickPopup
    clickQuery: [{ typeName: 'gzkj:azf' }]      // 可选：点击反查 WFS 属性
  })
})

onUnmounted(() => mapIns.value?.destroy())
```

- 默认视角：项目所在城市中心区 `104.07, 30.65, 高度 70000`
- 底图一律 `mapIns.addDefaultBasemap(roadOptions?)`，禁止手写高德瓦片 URL（常量在 CesiumMap.ts）
- 标记能力：`addMarker(id, {lng,lat,color,label,labelColor,alert,alertColor,ringRadius,meta})`、`updateMarker`、`removeMarker`、`onMarkerClick`、`focusMarker`

### 7.2 性能优化红线（封装层已内置，页面无需关心）

- **惰性渲染 requestRenderMode**：禁止绕过封装层直接改 `viewer.scene` 下对象属性（不自动重绘，必须走 showLayer/透明度/层级等封装方法）
- 移除默认全球底图（`baseLayer: false`），页面必须自己 `addDefaultBasemap()`
- 瓦片级别限制、MSAA、Cesium.js defer（vite.config.ts 的 `cesium-defer-load` 插件勿删）
- 新增页面路由用 `() => import(...)` 懒加载
- 多图层红线：非聚焦板块图层默认 `show: false` 懒显

## 8. 新板块开发 Checklist（12 步）

- [ ] 1. 建目录 `src/views/<板块>/`，主页面 `<板块>.vue`（目录名=文件名）
- [ ] 2. 子组件扁平放板块根级，公共组件放 `src/components/`
- [ ] 3. 注册路由（hash 模式，`import` 组件 + `name`）
- [ ] 4. `App.vue` 顶部菜单加项（`handleClick('Xxx')` → `router.push('/Xxx')`，`Menu_Active` 存 localStorage）
- [ ] 5. 建 API 文件（按 6.3 六步）并注册到 api/index.ts
- [ ] 6. 页面骨架：`#cesiumContainerXxx` + `.panel-overlay`（flex，左右 320/340px，padding 12px）+ mapcardNew + 底部 Tab/状态栏
- [ ] 7. 地图接入（6.2 五步，CesiumMap 封装，严禁裸 import cesium）
- [ ] 8. 颜色规范：`--color-*` 变量；echarts 走 `cssVar()`
- [ ] 9. import 全 `@/` 绝对路径
- [ ] 10. 点击弹窗交互：按 4.6 预设接入（DataDetailPopup + showDetail + 标题「板块 * 模块」）
- [ ] 11. 交付约定：不跑构建/类型验证，用户本地验证
- [ ] 12. 文件头注释：板块说明 + 布局 ASCII 图（参照 Reclassification.vue）

## 9. 构建与工作流约定

- **【强制】修改代码后不跑构建/类型验证**，由用户本地 `npm run dev`（端口 8098）验证
- 沙箱确需构建：`npm run build-only -- --emptyOutDir false`（跳过 dist 清理保护）
- 常用命令：`npm run dev`（PC 智慧检测 8098；移动端 H5 7321）/ `npm run build` / `npm run type-check` / `npm run lint`
- 登录态存 localStorage（`ACCESS_TOKEN` + `Menu_Active`）

## 10. 常见坑速查

| 坑 | 解决 |
| --- | --- |
| Vue 3.2.47 无 `useId` | 模块级计数器 `let seq = 0; const id = ++seq` |
| ECharts + v-if 切换图表错乱 | 切换前 `disposeAllCharts()`，nextTick 后再 init |
| mapcardNew 高度链 | 高度链 root 100% → flex:1 → content flex:1，图表容器 `height:100%` |
| 列表放 grow 区块被裁剪 | 列表用固定自然高（flex-shrink:0） |
| grid 三栏只有 2 子节点右面板错位 | 用 `flex + space-between` + 左右固定宽 |
| 图层勾选无真实数据 | `ensureLayer` 中 `addGeoJsonLayer` 换 `addWMSLayer` 出真实瓦片 |
| 页面数据是 mock | `fetchDashboardData()` 集中管理，接真实 API 时替换函数内 mock |
| 改图层属性画面不刷新 | 必须走封装层方法（内部已 requestRender），禁直接改 viewer 属性 |
| ECharts 尺寸不随容器变化 | ResizeObserver 监听容器本身（不能只靠 window.resize） |

## 11. 遗留事项

- `package.json` 移除 `ol` 依赖（残留勿用）
- `App.vue` 全局框架导航硬编码颜色待变量化
- 各页面统计图表数据为 mock，待后端接口就绪替换
