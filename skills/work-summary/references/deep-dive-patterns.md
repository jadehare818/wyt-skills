# Deep Dive Patterns

Reusable explanation patterns for technical writeups. Each pattern helps explain a specific type of design decision or debugging insight in a structured way that demonstrates understanding.

## Pattern: Context detour vs data structure passing

Use when: A piece of data is being threaded through layers, and you need to explain why one method was chosen over another.

### Structure

```
方案 A (ctx detour):
    Writer: ctx.Set("key", value)
    ... intermediate layers unaware ...
    Reader: ctx.Get("key") → value

方案 B (data structure):
    Producer: result.Field = value
    ... intermediate layers explicitly handle ...
    Consumer: use result.Field
```

### When to recommend which

| Approach | Use when |
|----------|----------|
| ctx (context) |横切关注点: trace ID, request metadata, deadlines — things all layers might need but that aren't business output |
| Struct field | Business data: the value is a direct product of the layer's work and belongs in its return type |

### Explanation template

> 之前的做法把 [数据] 塞进 ctx，中间的 [层名] 对这个数据完全无感知。读和写通过 key 字符串隐式耦合，IDE 追踪不到。改为挂在 [数据结构] 上后，函数签名就能看到依赖关系，数据流清晰可追踪。

---

## Pattern: Lock necessity analysis

Use when: Concurrency code was written with locks, and you need to explain whether locks are actually needed.

### Analysis steps

1. Draw the concurrency model (fan-out? pipeline? serial?)
2. Identify what's being shared and how
3. Check if the sharing pattern is inherently safe (e.g., each goroutine writes to its own index of a pre-allocated slice)

### Explanation template

> 调用图的并发模型是 [fan-out + wg.Wait / serial / pipeline]。各 goroutine 写 [具体描述 — 如"不同 index 的固定地址元素"]。Go memory model 保证 [安全条件]。锁 [需要/不需要] 因为 [原因]。

---

## Pattern: Gate + strategy (higher-order function)

Use when: Multiple variants share identical "should we do X?" logic but differ in "how to do X".

### Structure

```go
type strategyFunc func(ctx, inputs) *Result

func withGate(ctx, config, strategy strategyFunc) *Result {
    // shared gate logic: nil checks, config lookup, whitelist check
    if !shouldProceed(ctx, config) { return nil }
    return strategy(ctx, inputs)
}

// Variant A
func variantA(...) *Result {
    return withGate(ctx, config, func(...) *Result { /* A-specific logic */ })
}

// Variant B  
func variantB(...) *Result {
    return withGate(ctx, config, func(...) *Result { /* B-specific logic */ })
}
```

### Explanation template

> [Variant A] 和 [Variant B] 的门控逻辑完全一样（[列举: nil 检查、读配置、判断白名单]），差异只在"命中后怎么 [动作]"。把共性提取成 `[函数名]`，差异通过 [builder/strategy] 函数注入，避免重复代码，新 variant 接入成本降到只写一个 [函数名]。

---

## Pattern: Two-phase validation

Use when: Input goes through a transformation (e.g., URI resolution), and validation must happen relative to the transformed form.

### Structure

```
Phase 1 (before transform):
    → validate inputs that don't need transformation
    → skip inputs that will be transformed, record their indices

... transformation happens (e.g., asset URI → real URL) ...

Phase 2 (after transform):
    → re-validate the previously skipped indices
    → now operating on the transformed (valid) form
```

### Explanation template

> 原来的顺序是先校验再 [变换名称]，导致 [下游服务] 收到 [未变换的格式] 直接报错。修改后分两阶段：Phase 1 跳过需要 [变换] 的输入，Phase 2 在 [变换] 之后重新校验。

---

## Pattern: Parameter lifecycle in distributed systems

Use when: A parameter flows through multiple services/persistence layers, and you need to explain the full path.

### Structure

```
用户请求 (param: value)
    │
    ▼
Service A (API entry): 解析 → 写入 task
    │   ├─ Storage 1: FieldName1
    │   └─ Storage 2: FieldName2
    │
    ▼
Service A (worker): 消费 task → 构造下游请求
    │   注意: N 种 worker 路径可能独立
    │   注意: 存储层可能有独立的 convert 函数
    │
    ▼
Service B: 使用参数 → 产生效果
```

### Key debugging insight

> 分布式系统中参数的"静默丢失"不会报错——值变成零值后下游正常处理，只是效果不对。排查方法：从链路中间切一刀，先确认"存进去了没有"（查 DB），再确认"读出来了没有"（加日志），最后二分定位是 [读取环节/传递环节/convert 环节] 的问题。

### Multi-worker trap

> 如果 service 有 N 种 worker 类型（如 worker / multiworker / mw_minmax），代码结构相似但路径独立。改了一个不代表改全了。新增字段时必须检查所有 worker 的 controller + 存储层的 convert 函数。

---

## Pattern: Configuration-driven behavior (nacos + asynccache)

Use when: Explaining a feature that's controlled by runtime configuration rather than hardcoded logic.

### Structure

```
Nacos config:
    group/dataID → JSON config
        │
        ▼
asynccache (refresh every N minutes):
    cache.Get(key) → parsed config struct
        │
        ▼
Business logic:
    ResolveXxx(cache, dimensions...) → value or nil
        │
        ▼
    if nil: no-op (fail-open) — behavior same as before
    if value: apply the configured behavior
```

### Explanation template

> 配置驱动设计的好处：[维度] 变了只改配置，代码逻辑（查 `map[string]T`）不变。asynccache [N] 分钟刷新，fail-open（缓存失败则 [默认行为]，不阻断 [核心流程]）。

---

## Pattern: Header passthrough (sync vs async)

Use when: Explaining why async paths can't simply forward HTTP headers.

### Key insight

> 同步链路（用户 → API → 下游服务）可以直接从入口 request header 透传到下游。但异步链路（用户 → 入口服务 → 队列持久化 → worker → 下游服务）经过了队列，原始 HTTP header 在入队时已经丢失。所以必须在 middleware 层从 [元数据服务] 重新获取值写入 ctx，worker 再从 ctx 取出构造下游 header。

---

## Pattern: Iteration narrative

Use when: A requirement went through multiple rounds of deployment and fixing.

### Table format

| 轮次 | 部署后发现 | 根因 | 修复 |
|------|-----------|------|------|
| 第 1 轮 | [现象] | [根因分析] | [修复措施] |
| 第 2 轮 | [现象] | [根因分析] | [修复措施] |

### Debugging methodology to highlight

> 排查方法论：从链路中间切一刀 — 先确认 [检查点 A]，再确认 [检查点 B]，最后二分定位是 [环节 X] 还是 [环节 Y] 的问题。

---

## Pattern: Honest attribution annotation

Use when: Writing self-review sections that need to honestly credit mentor, self, and AI.

### Template

```markdown
#### 诚实标注

- 需求由 [谁] 提出并拆解（[拆解方式]）
- [方案/bug] 是 [mentor/自己/AI] [定位/设计/指导] 的
- [具体子任务] 是我自己 [主动/合作] 完成的
- [遇到的困难] 说明 [当时的理解不足之处]
```

### Why this matters

> 诚实标注不是自贬——它展示自我认知的清晰度。面向自己的文档要完全诚实；面向 leader 的文档突出自己主动的部分；面向面试官的文档把 mentor 指导淡化为"团队协作中确认"。同一事实的不同呈现角度。
