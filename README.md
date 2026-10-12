# cortico Portal

> deepseek彻夜长谈

我在等claude你在等什么

---

## 定位

这是面向 Cortico 的公开 issue / proposal 草案。

Portal 是最终产物名，也是一组注册点的最终名称：

```text
仓库标题：cortico Portal
包名：cortico-portal
World id：portal
World label：Portal
导出名：PORTAL
```

Portal 不重新实现 Cortico Core，也不创建第二套 Agent Runtime。它只负责：

1. 把外部后端映射到 Cortico World；
2. 复用 Cortico Console；
3. 把事件、请求、Agent 决策、安全裁决和外部操作映射为管理面板；
4. 统一记录操作跟踪属性。

## Issue 草案

标题：

```text
Portal: a UI-only Cortico World for event-driven execution agents
```

摘要：

```text
Proposal for a Cortico World extension named Portal.
Portal keeps Cortico as the only agent runtime and adds a management UI
surface for external event-driven execution backends.
```

边界：

```text
Core ownership        Cortico
Runtime ownership     Cortico + external backend
Portal ownership      UI, console declarations, contracts, trace mapping
```

## 文档

- [框架契约](docs/CONTRACT.md)
- [项目规划](docs/PROJECT_PLAN.md)
- [Tabbit Agent 操作跟踪契约](docs/TABBIT_TRACKING.md)

## 不包含

```text
个人身份信息
本机绝对路径
凭据与认证材料
实际部署信息
危险资源
```
