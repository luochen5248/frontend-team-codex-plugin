# 通用大屏前端专家团（frontend-team）

一个面向 **大屏 GIS / 移动端 H5** 场景的前端专家团 Codex 插件：把大屏 GIS 前端、移动端 H5、UI 设计、质量与治理（代码评审 / 代码简化 / 交付门控）组织成一个多角色协作团队，按「立项对齐 → 实现 → 质量治理 → 交付」四阶段推进，并内置路由规则决定每一次该谁上场。

## 特性

- **多角色编排**：一个编排技能（`frontend-team`）统领，按任务类型自动路由到对应角色技能。
- **平台分流**：PC 大屏 GIS 与移动端 H5 使用互不混用的两套规范。
- **质量门控**：交付前强制过「交付自检 → 代码评审 → 代码简化」三道闸门。
- **零依赖**：纯 Markdown 技能与元数据，无需安装任何运行时依赖。

## 目录结构

```
frontend-team/
├── .codex-plugin/
│   └── plugin.json            # 插件清单（名称、版本、界面元数据）
├── LICENSE                    # MIT 许可
├── README.md
├── assets/                    # 插件图标（icon.png 512×512 / logo.png 1024×1024）
└── skills/                    # 全部技能
    ├── frontend-team/         # 专家团编排层（入口）
    ├── 项目级别前端开发规范/    # PC 大屏 GIS 前端规范
    ├── 手机端H5开发规范/       # 移动端 H5 / WebView 规范
    ├── ui-new__skillhub/      # UI 设计
    ├── wenwei-code-review__skillhub/  # 代码评审
    ├── code-simplification__skillhub/ # 代码简化
    └── delivery-no-pseudoblock/       # 交付前自检门控
```

## 技能清单

| 技能 | 职责 |
| --- | --- |
| `frontend-team` | 专家团编排：团队构成、四阶段工作流、路由规则、强制规则 |
| `项目级别前端开发规范` | 大屏 GIS 前端规范（Vue 3.2.47 + TS + Vite + Ant Design Vue + Cesium）：技术栈红线、目录命名、颜色变量、页面排版、接口规范、Cesium 封装、弹窗交互预设 |
| `手机端H5开发规范` | 移动端 H5 / WebView 规范：布局契约、视口与安全区、点击区、软键盘、滚动容器、localStorage 配额保护、对话式表单状态机 |
| `ui-new__skillhub` | UI 设计与生产级界面打磨（设计系统、色彩体系、布局模板、组件模式） |
| `wenwei-code-review__skillhub` | 开发后代码评审：结合功能点 / 需求 / 概要设计文档，核对需求完成度、BUG 与设计偏离 |
| `code-simplification__skillhub` | 代码简化：在不改变行为的前提下降低复杂度、提升可读性 |
| `delivery-no-pseudoblock` | 交付前全自动自检：拦截责任转嫁、半途交付、伪阻断提问等形态 |

## 安装

### 方式一：作为 Codex 插件（推荐）

1. 将本目录放到本地插件目录，例如 `~/plugins/frontend-team`。
2. 在个人市场文件 `~/.agents/plugins/marketplace.json` 中登记：

```json
{
  "name": "personal",
  "interface": { "displayName": "Personal" },
  "plugins": [
    {
      "name": "frontend-team",
      "source": { "source": "local", "path": "./plugins/frontend-team" },
      "policy": { "installation": "AVAILABLE", "authentication": "ON_INSTALL" },
      "category": "Productivity"
    }
  ]
}
```

3. 重启 Codex 客户端，在市场列表中启用 `frontend-team`。

### 方式二：作为技能直接使用

把 `skills/` 下的任意技能目录复制到 `~/.codex/skills/` 即可，其中的 `SKILL.md` 即技能入口。

## 使用

安装后，用触发词唤起专家团，例如：

- 用前端专家团帮我做一个 GIS 大屏页面
- 前端专家团，评审一下我这次的代码改动
- 移动端 H5 页面开发，按手机端 H5 规范来

`frontend-team` 会根据任务自动路由到对应角色技能；具体触发词见各技能 `SKILL.md` 的 frontmatter `description`。

## 许可

本项目基于 [MIT License](./LICENSE) 开源。