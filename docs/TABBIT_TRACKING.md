# Tabbit Agent 操作跟踪契约

> 来源：Tabbit Browser 1.16.27.0 已安装 skill 与 `TabbitDance/runtime/src/*`
> 状态：提取记录，不表示本项目要复制 Tabbit Runtime

---

## 1. 任务标识

```text
taskId
taskName
reused
groupId
```

用途：

- `taskId`：任务内部稳定身份；
- `taskName`：用户可读任务名；
- `groupId`：tab group 恢复标识；
- `reused`：区分新建与复用。

Tab inventory 顶层字段：

```text
currentTabId
focusedWindowId
total
truncated
nextCursor
groups
groupsScope
complete
tabs
```

---

## 2. Tab 描述

```text
tabId
windowId
index
title
url
active
state: available | owned | claimed
group: {groupId, title} | null
```

`tabId` 是运行时 inventory ID，和 Playwright page identity 不同。

---

## 3. Request / Receipt

```text
taskId
requestId
status: queued | running | succeeded | failed | interrupted
mutationState: none | possible
submittedAt
startedAt
completedAt
result
outcome
error
errorName
errorCode
```

结果形态：

```text
result: {
  type: "inline",
  value: ...
}
```

或：

```text
result: {
  type: "resource",
  resourceId,
  byteLength
}
```

超时与中断字段：

```text
taskTerminated
tabsRetained
generationRestarting
retryAfterMs
terminationError
```

语义：

- `requestId` 是幂等键；
- 同一 `requestId` 再次提交只返回原 receipt；
- `mutationState: possible` 表示结果不确定时必须先查外部状态；
- `codeDispatched` 只表示代码已派发，不表示已执行；
- `taskTerminated` 只表示任务执行器被终止，tab 可能保留。

---

## 4. Acquisition

```text
acquisition.tabIds
acquisition.groupId
acquisition.status
acquisition.codeDispatched
```

Workspace 结果：

```text
groupId
ownedPageCount
```

约束：

- `tabIds` 与 `groupId` 互斥；
- 所有权变化即使只读也可能产生 mutation；
- acquisition 失败是原子拒绝或 outcome unknown，不等于页面未变化。

---

## 5. Runtime Status

```text
taskId
quarantined
idle
activeRequestId
queuedCount
receiptCount
closing
```

浏览器连接层：

```text
generation
browserSessionId
requestId
kind
payload
```

`generation` 是浏览器服务代次，不等于任务身份。

---

## 6. Observation

`tabbit.observe()`：

```text
page
pages
frames
framesTruncated
revision
snapshot
snapshotTruncated
focus
hitTest
```

`revision` 只用于观察新鲜度。

---

## 7. DOM / Interaction Diagnostics

`tabbit.focusInfo()`：

```text
page
chain
active
```

`tabbit.hitTest()`：

```text
kind
x
y
frame
target
shadowDepth
hitTestConflict
```

`tabbit.actionability()`：

```text
page
frame
box
element
visible
enabled
receivesEvents
viewportIntersection
occludedBy
blockers
actionable
```

---

## 8. Transition

`tabbit.triggerAndObserve()`：

```text
kind
url
pageId
activated
frame
```

可能的 staleness 字段：

```text
targetEpoch
documentGeneration
```

它们不是持久 ID。

---

## 9. Issue Ledger

位置：

```text
result.issuesDelta
```

常见字段：

```text
code
source
severity
recoverable
operation
timestamp
origin
details
```

已知 code：

```text
SITE_RATE_LIMITED
AUTHENTICATION_REQUIRED
SITE_ACCESS_BLOCKED
BROWSER_ERROR_PAGE
BLOCKING_DIALOG
SCREENSHOT_CAPTURE_FAILED
SCREENSHOT_FULL_PAGE_FALLBACK
DUPLICATE_SCREENSHOT
```

`SITE_RATE_LIMITED`：

```text
details.status
details.retryAfterMs
timestamp
origin
```

---

## 10. Artifact / Screenshot

```text
type: browser_screenshot
captureId
path
mimeType
requestedFullPage
capturedFullPage
width
height
bytes
sha256
requiresImageLoad
nextAction
screenshotCount
recommendedLimitExceeded
duplicateOf
warnings
issuesDelta
```

`nextAction`：

```text
type: load_image
path
mimeType
```

---

## 11. Resource Paging

```text
resourceId
offset
offsetUnit: byte
encoding: utf-8
data
nextOffset
eof
byteLength
```

`nextOffset` 是下一段唯一可信的起点。

---

## 12. Runtime Endpoint

管理/诊断层：

```text
version
kind: browser-runtime-service
transport: named_pipe | unix_socket
address
browserPid
generation
instanceId
token
```

`token` 是 secret，不得渲染到 UI、日志或导出包。

实例注册：

```text
version
instanceId
product
cliPath
browserPath
userDataDir
endpointPath
```

---

## 13. Capability Metadata

```text
capabilityVersion
playwrightCore
playwrightTestExpect
testRunner
foregroundPolicy
inlineResultThresholdBytes
maximumSerializedResultBytes
cliCommands
playwrightHelpers
helperContracts
boundedReceiptWaitMs
defaultEvaluationTimeoutMs
maximumEvaluationTimeoutMs
evaluationTimeoutOption
resourcePaging
tabOwnership
frameInclusiveObservation
navigationHelper
inputHelper
networkHelper
artifactBoundary
automaticOutcome
issueLedger
rateLimitReporting
issueReportingHelper
reportableIssueCodes
downloadLedger
screenshot
visualRoutes
```

---

## 14. 软件 UI 建议字段

列表主列建议：

```text
时间
Task
Request
状态
Mutation
获取页
操作
结果
问题
证据
```

详情页建议分组：

```text
身份：taskId / requestId / groupId / generation
时间：submittedAt / startedAt / completedAt / duration
执行：status / mutationState / codeDispatched
对象：tabId / windowId / pageId / frame
结果：result / outcome / error
证据：issuesDelta / screenshots / artifacts / resources
```

不要展示：

```text
token
完整认证信息
无业务意义的内部地址
```

不要当永久身份：

```text
revision
targetEpoch
documentGeneration
pageId
frame ID
snapshot ref
```
