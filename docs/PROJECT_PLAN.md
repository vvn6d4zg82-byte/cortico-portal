# 项目规划

> 状态：UI-only 路线确认  
> 更新时间：2026-10-12  
> 关联文档：[框架契约](./CONTRACT.md) · [Tabbit 跟踪契约](./TABBIT_TRACKING.md)

---

## 0. 结论先行

本项目当前不再定位为独立的 Agent Runtime、事件总线、Workflow Engine 或管理平台。

新的定位是：

```text
Cortico = Agent 宿主与 Runtime
外部后端 = 事件、规则、动作、安全边界与实体执行
本项目 = Cortico 上的管理 UI 扩展
```

因此最终主要交付物应是一个 Cortico World 扩展包：

```text
cortico-portal
```

它不重新实现 Cortico Core，也不重建管理控制台。它只负责：

1. 作为 Cortico World 提供管理页入口；
2. 通过 `console()` 声明配置、提示词、面板与数据接口；
3. 通过面板 UI 展示和操作外部后端；
4. 把事件、请求、Agent 决策、安全裁决和外部操作映射成可视化时间线。

---

## 1. 当前仓库现实

### 1.1 代码规模

| 项目 | 当前规模 |
|---|---:|
| TypeScript/Node workspace 包 | 10 |
| 应用入口 | 2（CLI / Web UI） |
| TypeScript 源文件（排除 dist/node_modules） | 37 |
| 测试文件 | 9 |
| Java 归档文件 | 389 |
| Java 源文件 | 50 |
| Java 归档体积 | 约 29.5 MiB |

当前活跃包：

```text
packages/contract
packages/core
packages/gateway
packages/safety
packages/brain
packages/locale
packages/hass-adapter
packages/config
packages/llm
packages/runtime
```

当前活跃应用：

```text
apps/cli
apps/web-ui
```

### 1.2 当前主栈

| 层 | 技术 |
|---|---|
| 语言 | TypeScript / JavaScript |
| 运行时 | Node.js >= 20 |
| 测试 | Vitest |
| CLI | Node 原生命令行 |
| 旧 Web 控制台 | Node HTTP + SSE + 原生 JS |
| Vue 控制台 | Vue 3 + Composition API + Pinia + Vite |
| 类型检查 | TypeScript + vue-tsc |
| 分发 | 纯 Node dist bundle |
| 外部实体 | Mock Lamp / Home Assistant adapter |

### 1.3 当前框架重心的变化

`docs/CONTRACT.md` 已经把项目收敛为四层：

```text
Core       无语义事件内核：事件池、水位闸门、主循环、工具派发
World      外部环境插件：产事件、声明工具、执行外部动作
Persona    语义与认知流程
Memory     持久状态
```

新的 `packages/core` 是这条路的 P0：

| 文件 | 作用 |
|---|---|
| `packages/core/src/event.ts` | 可核验事件契约 |
| `packages/core/src/wake.ts` | WakeBus 水位闸门 |
| `packages/core/src/index.ts` | Core 对外导出 |

这意味着后续工作应围绕“Cortico World 扩展 + UI 管理面板”展开，而不是继续扩展旧的 Java Web 控制台或自定义独立工作流引擎。

---

## 2. 技术路线收纳统计

### 2.1 活跃主线

| 路线 | 状态 | 用途 | 是否继续 |
|---|---|---|---|
| TypeScript | 活跃 | 主语言 | 是 |
| Node.js 20+ | 活跃 | CLI、Web、测试、扩展运行时 | 是 |
| Vitest | 活跃 | 单元与规则测试 | 是 |
| Vue 3 / Composition API | 活跃 | 管理 UI 主栈 | 是 |
| Pinia | 活跃 | 前端状态适配 | 视 Cortico 面板协议调整 |
| Vite | 活跃 | 前端开发与构建 | 是 |
| `packages/core` | 活跃 | 无语义事件池与 WakeBus | 是 |
| `packages/gateway` | 活跃 | 事件校验、节流、反射接线 | 是 |
| `packages/safety` | 活跃 | 能力矩阵、人审、kill switch | 是 |
| `packages/brain` | 活跃 | 意图到原子动作 | 是 |
| `packages/locale` | 活跃 | 人话层 | 是 |
| `packages/hass-adapter` | 活跃 | 真实实体 adapter | 是 |
| `packages/config` | 活跃 | 配置加载、校验、补丁 | 是 |
| `packages/llm` | 活跃 | Prompt 与意图源 | 是 |
| `packages/runtime` | 活跃 | 运行时装配 | 是 |
| `apps/cli` | 活跃 | 开发与自检入口 | 是 |
| `apps/web-ui` | 过渡 | 旧控制台 | 被 Cortico Console 取代 |

### 2.2 归档路线

| 路线 | 状态 | 说明 |
|---|---|---|
| Java Spring Boot | 已归档 | 位于 `archive/java/`，不再演进 |
| Maven | 已归档 | 随 Java 平行实现一起归档 |
| Checkstyle | 已归档 | 旧 Java 规范链 |
| ArchUnit | 已归档 | 旧 Java 架构约束 |
| Spring Web / SSE | 已归档 | 旧 Java Web 控制台 |
| JUnit / AssertJ | 已归档 | 旧 Java 测试链 |

归档原因：

```text
单栈深入 TypeScript，避免 Java / TS 双栈长期漂移。
```

### 2.3 外部宿主路线

| 项目 | 许可 | 状态 | 作用 |
|---|---|---|---|
| [Cortico](https://github.com/Pal-AI-Lab/Cortico) | MIT | 主宿主 | Agent Harness、事件流、Persona、Memory、World、Console |
| Cortico 扩展 API | MIT 项目契约 | 目标接口 | `world` API v5 |
| Cortico Console | MIT 项目契约 | 管理 UI 宿主 | World 页、配置组、面板、提示流 |
| Cortina | 独立开源项目 | 参考 | 辅助生成 Cortico 扩展 |

本项目最重要的集成对象是 Cortico，而不是另起 Workflow Engine。

### 2.4 外部工具路线

| 工具 | 状态 | 用途 | 是否进入核心架构 |
|---|---|---|---|
| Tabbit Browser | 外部宿主 | Browser Agent 操作、任务、回执、跟踪 | 是，作为 UI 跟踪数据源 |
| Tabbit Skill | 外部契约 | Agent 操作协议 | 是，作为跟踪语义参考 |
| Tabbit CLI | 外部工具 | `create/tabs/claim/resume/nodejs/receipt/resource/diagnose` | 是，作为操作接口参考 |
| Playwright / CDP | 外部运行时 | Browser 自动化 | 是，作为观察与执行属性来源 |
| ComfyUI | 外部工具 | 图像工作流 | 旁路，不属于 Agent Core |
| Z-Image 工作流 | 外部资产 | 图像生成实验 | 旁路 |
| SD1.5 工作流 | 外部资产 | 图像生成实验 | 旁路 |

### 2.5 已评估但不采用为主干的路线

| 项目 | 结果 | 原因 |
|---|---|---|
| Temporal | 备选 | 持久执行很强，但会引入独立 Server/DB 和管理面 |
| Conductor OSS | 备选 | 事件驱动 Agent 工作流很贴合，但本项目已被 Cortico 接管宿主层 |
| Node-RED | 备选 | 适合可视化事件规则，但不应取得最终动作执行权 |
| Kestra | 不采用主干 | 偏事件编排和调度，管理语义与 Cortico World 重叠 |
| Apache Camel + Hawtio | 不采用主干 | 适合 JVM 集成与 JVM 管理，不符合当前 TS/Cortico 单栈 |
| Flowable | 不采用主干 | BPMN/人工流程强，事件 Agent 宿主重叠 |
| n8n | 排除 | fair-code / Sustainable Use License，不是标准 OSI 开源许可 |
| Windmill | 谨慎 | 许可证为 Other，需逐条确认 |
| Activepieces | 谨慎 | 许可证为 Other |
| ToolJet | 谨慎 | AGPL-3.0，网络服务分发需要评估 |
| Appsmith | 仅 UI 备选 | 可做内部工具，但不能承担 Agent 语义层 |
| Kafka UI / Redpanda Console | 仅观测 | 只管理事件流，不管理 Agent 决策与执行 |

---

## 3. 目标架构

### 3.1 总图

```text
Cortico
├─ Core
│   ├─ EventEnvelope
│   ├─ WakeBus
│   ├─ Session / Run
│   └─ Tool Dispatch
├─ Persona
│   ├─ 上下文
│   ├─ 认知循环
│   └─ 提示词
├─ Memory
│   └─ 持久状态
└─ World
    ├─ Portal Runtime Adapter
    ├─ Tabbit Adapter
    └─ 其他外部环境

本项目
└─ cortico-portal
    ├─ WorldDefinition
    ├─ Config Group
    ├─ Prompt Doc
    └─ Console Client Panels
```

### 3.2 UI-only 边界

本项目可以做：

- 展示状态；
- 展示事件；
- 展示 Agent 决策；
- 展示请求与回执；
- 展示安全裁决；
- 发起管理动作；
- 编辑配置；
- 调用外部后端接口。

本项目不做：

- 重新实现事件总线；
- 重新实现 SafetyGate；
- 重新实现 Agent 决策；
- 重新实现 Workflow Engine；
- 修改 Cortico Core；
- 直接执行外部副作用。

### 3.3 数据流

```text
外部后端
  → Cortico World invoke
    → Console Panel ctx.invoke
      → UI 只读快照

UI 操作
  → ctx.invoke
    → World 服务端 handler
      → 外部分类/安全/执行接口
        → 返回 receipt
          → UI 更新
```

面板实时数据：

```text
World.console().stream(panel, socket)
  → ConsolePanelContext.stream({
      message,
      open,
      close
    })
```

---

## 4. 管理工具规划

扩展包建议：

```text
cortico-portal/
  package.json
  tsconfig.json
  src/
    index.ts
    definition.ts
    config.ts
    world.ts
    ENV_PROMPT.md
    console/
      client.ts
      style.css
  tests/
    world.test.ts
    console-contract.test.ts
```

### 4.1 页面

| 页面 | 主要数据 |
|---|---|
| `overview` | 运行状态、事件速率、Agent 状态、队列、安全状态 |
| `events` | 原始事件、核验状态、来源、游标、时间线 |
| `agent` | 规则、决策、冷却、暂停/恢复、请求追踪 |
| `safety` | 能力、人审、kill switch、审计 |
| `entities` | 实体、adapter、工具表、可执行动作 |

### 4.2 统一追踪模型

UI 需要统一以链路为核心，而不是分别展示孤立列表：

```text
Task
  └─ Request / Receipt
       ├─ Event
       ├─ Agent Decision
       ├─ Tool Call
       ├─ Safety Verdict
       ├─ External Action
       └─ Result / Issue / Artifact
```

推荐主追溯键：

```text
taskId
requestId
event.cursor / event.seq
tool.callId
safety.auditId
adapter.operationId
```

### 4.3 Tabbit 跟踪属性

Tabbit 的完整字段契约见：

```text
docs/TABBIT_TRACKING.md
```

UI 至少要收纳：

- `taskId`
- `taskName`
- `groupId`
- `requestId`
- `status`
- `mutationState`
- `submittedAt`
- `startedAt`
- `completedAt`
- `acquisition`
- `result`
- `outcome`
- `issuesDelta`
- `transition`
- `generation`

---

## 5. Cortico 扩展契约

### 5.1 包声明

```jsonc
{
  "name": "cortico-portal",
  "type": "module",
  "keywords": ["cortico-world", "cortico"],
  "cortico": {
    "kind": "world",
    "api": 5,
    "consoleClient": "dist/console.js",
    "consoleStyle": "dist/console.css"
  }
}
```

### 5.2 WorldDefinition

```ts
import type { WorldDefinition } from 'cortico/world.ts';

export const PORTAL: WorldDefinition<PortalConfig> = {
  id: 'portal',
  label: 'Portal',
  defaults: () => ({ ...DEFAULTS }),
  create: (ctx) => new PortalWorld(ctx),
};
```

### 5.3 面板契约

```ts
export default {
  panels: {
    overview: { mount(ctx) {} },
    events: { mount(ctx) {} },
    agent: { mount(ctx) {} },
    safety: { mount(ctx) {} },
    entities: { mount(ctx) {} },
  },
};
```

约束：

- 只用 `ctx.ui`；
- 不直接 `fetch('/api/...')`；
- 不碰 `document.body`；
- 不绕过 `ctx.invoke` / `ctx.stream`；
- 不自定义主题系统；
- 不把长期状态放在模块全局。

---

## 6. 阶段路线

### P0：契约冻结

- [x] 确认 UI-only 定位；
- [x] 确认 Cortico World 扩展方向；
- [x] 提取 Tabbit 跟踪属性；
- [x] 归档 Java 平行实现；
- [x] 建立 Core WakeBus 原型。

### P1：Cortico 扩展骨架

- [ ] 创建 `cortico-portal` 包；
- [ ] 实现 `WorldDefinition`；
- [ ] 实现最小 Console Client Bundle；
- [ ] 实现 `console().config`；
- [ ] 实现 `console().promptDocs`；
- [ ] 通过 `pnpm check:extension`。

### P2：管理面板

- [ ] `overview`；
- [ ] `events`；
- [ ] `agent`；
- [ ] `safety`；
- [ ] `entities`；
- [ ] 统一 trace 数据结构；
- [ ] 实时流与断线重连。

### P3：后端桥接

- [ ] 定义后端管理 API；
- [ ] 定义请求与回执 DTO；
- [ ] 接入事件流；
- [ ] 接入 Agent 决策；
- [ ] 接入 Safety；
- [ ] 接入实体执行结果。

### P4：测试

- [ ] `FakeHost` 干装载；
- [ ] Config schema 校验；
- [ ] Console invoke 契约测试；
- [ ] Stream 重连测试；
- [ ] Zero-diff 控制台契约测试；
- [ ] 面板构建产物检查。

### P5：交付

- [ ] `dist/console.js`；
- [ ] `dist/console.css`；
- [ ] npm 包元数据；
- [ ] 安装说明；
- [ ] 本地 link 安装；
- [ ] Cortico 重启后加载验证。

---

## 7. 不变量

1. **UI 不执行业务动作。**
2. **UI 不解释业务语义。**
3. **UI 不保存权威状态。**
4. **UI 不绕过安全边界。**
5. **请求必须带 `requestId`。**
6. **操作必须可还原为 receipt。**
7. **事件必须保留 `source / cursor / ts`。**
8. **变化必须有 before / after 或审计证据。**
9. **`generation` 只用于代次标识，不用于业务身份。**
10. **`targetEpoch / documentGeneration / revision` 只用于新鲜度。**

---

## 8. 当前技术路线统计

### 8.1 按状态

| 状态 | 数量 | 路线 |
|---|---:|---|
| 活跃 | 12 | TS、Node、Vitest、Vue、Pinia、Vite、Core、Gateway、Safety、Brain、Locale、Runtime |
| 过渡 | 1 | 旧 Web UI |
| 归档 | 6 | Java、Maven、Checkstyle、ArchUnit、Spring Web、JUnit/AssertJ |
| 外部宿主 | 1 | Cortico |
| 外部工具 | 5 | Tabbit、Playwright/CDP、ComfyUI、Z-Image、SD1.5 |
| 备选 | 3 | Temporal、Conductor OSS、Node-RED |
| 排除/谨慎 | 7 | Kestra、Camel/Hawtio、Flowable、n8n、Windmill、Activepieces、ToolJet |

### 8.2 最终主路线

```text
TypeScript
  + Node.js
    + Cortico World API v5
      + Cortico Console Client
        + Portal backend
          + Tabbit operation tracking
```

### 8.3 旁路路线

```text
ComfyUI
  └─ Z-Image / SD1.5 workflow

Browser QA / Agent operation
  └─ Tabbit
      └─ Playwright / CDP
```

---

## 9. 待确认事项

1. 后端管理 API 由谁提供，是否沿用现有任何实体 runtime。
2. Cortico World 扩展是否只做 UI 代理，还是允许提供只读工具。
3. 面板实时流采用 SSE 还是 WebSocket。
4. Trace 存储是内存、JSONL，还是外部数据库。
5. 是否将 Tabbit 操作作为 UI 的核心轨迹页，还是作为独立来源。
6. `docs/CONTRACT.md` 与本规划冲突时，以契约文档为准；规划不得反向修改不变量。
