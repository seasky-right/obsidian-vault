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
- **Simulator / Runtime**
    
    - 保留现在的 Legacy AirSim 当回归基线；
    - 选一个现代 Primary Backend；
    - 再找一个第二 Backend 做兼容性证明，不必把市面上全接一遍；
    - 做对应 `RuntimeProvider`：自动启动、关闭、端口、健康检查、崩溃恢复、Headless、批量运行；
    - 真正验证传感器、reset、真实多机。
    
- **World / Scenario**
    
    - Scene / World Resource；
    - GIS/遥感需要的坐标系、区域 Polygon、障碍物、目标；
    - Scenario Generator + seed；
    - 天气、光照、初始位置、目标位置等随机化；
    - 刚才发现的 `VehicleProfile`：让 Scenario 能指定“这一局用什么飞机配置”。
    
- **遥感 Task + Evaluator + Benchmark**
    
    这应该成为以后**最重要的一条**：
    
    ```
    ReachPoint                基础测试
    SearchTarget              目标搜索/定位
    AreaMapping               区域测绘
    FacilityInspection        设施巡检
    EmergencySearch           应急搜索
    Multi-UAV Mapping/Search  多机任务
    ```
    
    对应真正有遥感味道的指标：
    
    ```
    Coverage
    定位误差
    Recall / False Positive
    有效观测率
    重叠度
    航程
    完成时间
    空间完整性
    ```
    
    再把这些组织成正式 Benchmark。
    
    这一条决定：**“为什么这是无人机遥感空间智能平台，而不是通用插件框架。”**
    
- **Training / RL**
    
    不重做 Core，只补：
    
    ```
    Gym / RL Adapter
    +
    reset / step
    +
    Reward Provider
    +
    Training Driver
    +
    PPO baseline
    ```
    
    然后训练好的 PPOAgent 再像普通 Agent 一样参加 Benchmark。
    
    这一条解决：**“平台怎么训练 Agent，而不只是测 Agent。”**
    
- **Vehicle Configurator**
    
    这个是计划书专门要求、目前架构里还缺接口的一块：
    
    ```
    模板机型
    + 传感器配装
    + 参数配置
    + 工程约束
    + LLM / 知识库辅助设计
         ↓
    VehicleProfile
    ```
    
    第一阶段不用搞自由 CAD。最多做到我们刚才说的“钢四式配装界面”😇：载荷、航时、速度、传感器、功耗这些约束。
    
    它独立开发，只把 `VehicleProfile` 交给平台。
    
- **平台产品化**
    
    最后再做：
    
    ```
    Agent 上传
    manifest / 依赖校验
    隔离执行
    任务队列
    Worker
    Web 前端
    历史结果
    排行榜 / 对比
    ```