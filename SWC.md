可以。现在回头看，平台本身出现过的问题其实不少，而且其中有几类已经被我们反复验证。最好把它们和“我们调度器自己的 bug”严格分开。

| 类别           | 平台问题                                                 | 证据 / 表现                                                                                               | 对开发的影响                                                                  |
| ------------ | ---------------------------------------------------- | ----------------------------------------------------------------------------------------------------- | ----------------------------------------------------------------------- |
| **严重缺陷**     | **计分引擎不校验真实完成数**                                     | 曾实测即使 0 件完成，也能按类似 `D×3000/(T+60)` 算出很高分，甚至约 69.47                                                     | **score 不能证明任务完成**；必须自己检查 8/8、30/30                                     |
| **严重缺陷**     | **实时日志服务 fsnotify/inotify 资源耗尽**                     | 持续出现 `failed to create fsnotify watcher: too many open files`；早期还能看十几秒，后来甚至一打开日志就 fail                | 最关键的 30–50s 尾部经常丢失，严重阻碍调试                                               |
| **严重缺陷**     | **Run 结束时状态 API 消失过快**                               | 最后一件货刚进入 placing / done 附近，下一 poll 直接变成 `run=null, cargoes=[]`                                        | 经常只能得到本地 7/8，然后根据 run 消失推断最后一件完成                                        |
| **接口问题**     | **AGV 调试接口容器外不可达**                                   | 4 台 AGV 域名和 backend 内网域名公网 DNS 都无法解析；公网也没有可用代理路径                                                      | 官方 AGV `/move_to /grab /release /status` 基本只能在集群内部使用                    |
| **接口问题**     | **Cargo API Token 门禁配置异常**                           | 外部请求无认证/Bearer/teamId/各种 token 都是 `401 invalid_sim_token`；真正需要的 `X-Sim-Token` 只在运行容器里注入               | 无法方便地从本地调 Cargo 官方 API                                                  |
| **后端缺陷**     | **错误请求曾直接泄露 PostgreSQL 错误**                          | 某 CI token 请求打出 HTTP 500，并暴露数据库语法错误                                                                   | 后端错误处理和安全隔离明显有问题                                                        |
| **状态一致性**    | **“新 run”不等于“物理状态已完全 reset”**                        | Cargo 已经全部 `pending/unassigned`，但 AGV 仍可能在上一局返航，甚至 `arm=holding`                                      | clean batch 里也可能出现 stale arm / dirty position，必须做 startup normalization |
| **状态一致性**    | **逻辑 reset 与 AGV reset 不同步**                         | `policy_init_reset` 已发生，但 attach 时 AGV 还在从 shelf 返回 home                                              | 不能假定 `(run detected)` 时四车都已经 home+idle                                  |
| **控制权问题**    | **平台默认 controller 与参赛 scheduler 会同时控制同一辆 AGV**       | 我们接管以后仍不断出现平台写新的 target、assignment、grab/phase advance                                                 | 形成“双控制器 race”，大量 `TARGET_AUTHORITY_LOST`                                |
| **控制权问题**    | **没有明确的 motion ownership / lease 机制**                | 我们发了 `move_to` 后，平台仍可立刻把 target 改走；我们又得 HOLD/reassert                                                 | 必须自行实现 late takeover、safe handoff、observed-target arbitration           |
| **控制权问题**    | **assignment 本身不能证明安全接管边界**                          | Cargo 被 assign 时，AGV 可能还带着上一件货，或者 arm 状态与新任务不一致                                                       | `assigned_agv==X` 不能直接视作 ownership                                      |
| **控制权问题**    | **平台 assignment 可在我们 traffic reservation 之外直接让车动起来** | v7.1 中平台 assignment 后 AGV 在本地 grant 之前就开始运动，最后发生 A2/A3 collision                                      | 安全层必须看真实 `/status current→target`，不能只看自己的 reservation                   |
| **API 语义问题** | **Cargo state POST ≠ 物理动作**                          | 把 Cargo POST 成 `picking` 返回 200，但 AGV 完全不动                                                            | 官方 Cargo state API 只是逻辑 metadata，不是运动控制接口                               |
| **API 语义问题** | **HTTP 200 不一定代表动作真正执行成功**                           | `/grab` 曾返回 `200 {"status":"grabbing","message":"busy"}`；`release` 也出现 `200 ... "nothing to release"` | 不能按 HTTP status 推进 FSM，只能继续观察 `/status` + Cargo state                   |
| **状态机问题**    | **平台会自行推进某些 Cargo lifecycle**                        | 我们还没发下一步时，Cargo 可能已经从 `picking→picked→delivering`，target 也被平台写到 shelf                                 | FSM 必须接受 external advance，不能认为所有状态变化都来自我们                               |
| **可观测性问题**   | **Cargo、arm、target 三套状态不是原子更新**                      | 经常会短暂出现 `picking + grabbing`、`picked + grabbing`、target 已改但 arm 还没变等组合                                | 单次 snapshot 不可靠，必须按多帧/强证据判断                                             |
| **部署稳定性**    | **部分角色 Deployment 曾出现 0/1 NotReady**                 | 早期多次看到 `policy-pathplan / safety-guard / scheduler-global` 未 Ready                                    | 会让测试结果混进部署异常，难以区分算法问题                                                   |
| **可重复性问题**   | **相同 demo-hard 不一定具有完全相同 startup**                   | 有时四车 clean home；有时 attach 时仍在返航；有时甚至存在 holding residue                                                | 单次成绩波动不能全归因于算法                                                          |

其中我觉得最值得单独强调的是下面五个。

### 1. 最大的平台逻辑 bug：**计分器不验证完成数**

这个非常严重。

我们已经发现过：

```text
0 cargo complete
```

也可以通过时间/距离项算出异常高分。

所以从那以后我们的验收标准才一直是：

```text
0 violation
0 collision
8/8 或 30/30
然后才看 score
```

而不是：

```text
score 高
⇒ 算法好
```

这也是为什么早期那些异常高分不能直接算有效结果。

---

### 2. 最大的工程问题：**平台没有真正定义“控制权交接”**

目前实际架构很像：

```text
        ┌─ 平台 Policy controller
AGV  ←─┤
        └─ 我们 Scheduler.Global
```

两边都能写：

```text
target
grab
release
```

但没有：

```text
acquire_control()
lease
owner
lock
```

这种明确机制。

所以我们才不得不自己发明：

```text
Safe Handoff
Late Loaded Handoff
Stationary Takeover
Observed Target Arbiter
Traffic Hold
Route Reassert Interlock
Cooperative Target Adoption
```

从比赛平台设计角度说，这个接口契约非常不友好。

而现在 v7.13 做的事情，本质上甚至不是“彻底夺权”，而是开始：

> 平台动作如果与我们的计划一致，就顺着它走；只有冲突时才覆盖。

某种程度上是在给平台补一个**协作式 ownership protocol**。

---

### 3. `policy_init_reset` 并不意味着真正 reset

这个坑我们踩了很多次。

理论想象：

```text
new run
→ cargo reset
→ AGV home
→ arm idle
→ scheduler start
```

实际可能是：

```text
cargo 已经 8 个 pending
↓
new run 已经存在
↓
但是：
A1 正在回 home
A2 arm=holding
A3 还在上一局位置
A4 已经 home
```

也就是说：

> **逻辑 run 生命周期与物理 AGV 生命周期不是同步事务。**

v7.7 的 `CLEAN_BATCH_RESIDUE_NORMALIZATION` 就是专门给平台补这个洞的。

---

### 4. 日志服务已经明显是平台事故

现在这个：

```text
failed to create fsnotify watcher:
too many open files
```

已经不是偶发了。

它经历过：

```text
能看到 20s
→ 只能看到 3s
→ 一打开就 fail
→ 偶尔又能抓到完整几十秒
```

这种状态非常像服务端 fsnotify/inotify watcher 泄漏或限额耗尽。

而且 Passive 的：

```text
PathPlan
SafetyGuard
TelemetryReporter
```

一样会报。

所以它跟我们的 C++ 日志实现基本没关系。

这其实已经属于应该向比赛平台报障的程度。

---

### 5. 平台 API 的“状态”不能按传统 REST 思维理解

例如：

```text
POST cargo state=picking
→ 200
```

不意味着：

```text
AGV 开始去 pickup
```

甚至：

```text
POST /grab
→ 200
```

也不意味着：

```text
已经 grab 成功
```

平台更接近：

```text
命令请求
↓
异步模拟器
↓
若干 tick 后
Cargo / Arm / Target 分别更新
```

因此我们后来才形成现在这条原则：

> **FSM 是计划，`/status` 里的 position/target/arm 才是物理事实；Cargo state 是另一条异步逻辑事实。**

这个认识其实是整个 v5→v7 演化里最重要的东西之一。

---

如果压缩成一句话，现在这个平台最大的特点就是：

> **计分、生命周期、运动控制、Cargo 状态和日志系统彼此都不是严格同步的，而且平台默认 controller 与参赛代码之间没有正式的控制权仲裁协议。**

所以我们这套代码后来越来越像是在平台之上再实现一层：

```text
transaction reconciliation
+
control ownership
+
traffic arbitration
+
liveness recovery
```

而不只是普通的“AGV 路径规划”。

如果后面要做最终交接文档，我建议直接加一个章节叫 **《平台缺陷与非理想行为清单》**，把这些和我们的版本 bug 分开写；这部分对之后解释为什么代码这么复杂非常重要。