# 现在划分的模块

核心模块：

1. **Backend**：仿真后端，负责和 AirSim / Mock / 以后 Isaac 之类交互。
2. **Scenario**：场景，定义环境、初始状态、目标、资源、truth。
3. **Task**：任务规则，定义“要做什么、什么时候算完成”。
4. **Agent**：智能体，负责根据 observation 决定下一步 action。
5. **Evaluator**：评价器，负责对任务结果打指标和分数。

另外还有 5 个偏“平台扩展/实验基础设施”的模块：

6. **Scenario Generator**：批量生成不同 seed / 参数的场景。
7. **Runtime Provider**：管理运行资源，比如仿真进程、GPU、远端实例。
8. **Training Driver**：反复跑 episode，用于训练/RL。
9. **Result Processor**：汇总和后处理多次实验结果。
10. **Benchmark**：定义一组标准测试案例，用来公平比较不同 Agent。


# 项目架构抽象

```
                    ┌──────────── Core ────────────┐
                    │                              │
配置 / Registry ──→ │ Episode / Session / Record  │
                    │ Benchmark / Result / Plugin  │
                    │                              │
                    └───────────┬──────────────────┘
                                │ 公共 Contract
          ┌─────────────────────┼──────────────────────┐
          ↓                     ↓                      ↓
   Simulator Plugin       Spatial Task Pack        Agent / Training
   AirSim / Isaac         Search / Mapping         Rule / RL / VLM
   Sensor / Runtime       GIS / Evaluator          Multi-Agent
```

项目一轮运行逻辑：
```
运行配置
   ↓
插件发现 + Preflight
   ↓
Scenario
   ↓
Backend.reset()
   ↓
初始 Observation
   ↓
Agent
   ↓
Action
   ↓
Backend.execute()
   ↓
Backend.observe()
   ↓
新的 Observation
   ↓
Task 判断是否完成
   ↓
未完成 → Agent 再决策
   ↓
完成 / 超时 / 失败 / 步数耗尽
   ↓
保存 trajectory + result + events
   ↓
清理 Backend / Agent / Task / Scenario / Runtime
```


# 进一步开发方向

| 能力线                          | 包含什么                                                       | 后面主要做什么                                                                    |
| ---------------------------- | ---------------------------------------------------------- | -------------------------------------------------------------------------- |
| **① Simulator / World**      | Backend + Runtime Provider + Scenario + Scenario Generator | AirSim / Isaac / Project AirSim 等适配；UE/场景启动关闭；reset；传感器；场景资源；目标生成；天气/位置随机化 |
| **② Task**                   | Task                                                       | Search、Mapping、Inspection、Emergency Search 等遥感任务规则                         |
| **③ Evaluation / Benchmark** | Evaluator + Benchmark + Result Processor                   | 遥感指标、任务评分、标准测试集、多 seed、多 Agent 比较和最终报告                                     |
| **④ Training**               | Training Driver + RL Adapter + Reward                      | Gymnasium 接口、PPO 等 RL 训练；以后也可以是 imitation learning 等                       |
| **⑤ Agent 接入**               | Agent                                                      | **平台接口基本已经有，不是当前平台研发主线**；只保留 baseline Agent 验证关卡即可                         |
