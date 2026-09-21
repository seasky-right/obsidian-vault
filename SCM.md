> **面向安全关键场景的任务条件化流式视频理解与风险事件检测**
> 
> Task-conditioned Streaming Video Understanding for Safety-Critical Monitoring

实验室是一个非常合适的**验证场景**，而不是方法本身的边界。🫥

---

## 一、行业里现在并不是“视频 → 每帧扔给 VLM”

现在比较成熟的工业视频智能系统，大体上已经形成了一种**分层架构**：

|层|做什么|常见技术|
|---|---|---|
|视频采集|连摄像头、RTSP、录像、多路流|GStreamer / DeepStream / FFmpeg|
|高频轻感知|每帧/高频检测人、物体、区域状态|YOLO / RT-DETR / Grounding 模型|
|对象连续性|谁是谁、去了哪里|ByteTrack / BoT-SORT / NvDCF|
|事件规则|进入禁区、持续未佩戴、越线、停留|ROI + tracker + FSM/规则|
|视频筛选|找“值得认真看”的片段|差分、embedding、keyframe、trigger|
|重语义理解|判断复杂关系、动作、原因、风险|Video VLM / MLLM|
|长时状态|同一事件持续多久、是否解决|Event store / memory|
|输出|告警、证据、报告|DB / MQ / Web|

NVIDIA DeepStream 这种工业视频基础设施，以及`nvdsanalytics` 直接提供 ROI、拥挤、方向、越线等基于轨迹的分析说明，**“视频流进来然后持续处理”这个工程问题已经相当成熟，不需要你自己从 VideoCapture 一点点造。** ([NVIDIA Docs](https://docs.nvidia.com/metropolis/deepstream/dev-guide/text/DS_ref_app_deepstream.html?utm_source=chatgpt.com "DeepStream Reference Application - deepstream-app — DeepStream documentation"))

真正还在快速发展的，是上面那块：

**这么长的视频，我到底什么时候看、看什么、看多少，以及怎么把零散帧变成一个语义事件？**

这才是你可以研究的地方。

---

# 二、现在学术前沿其实可以分成 5 条路线

### 1. Streaming Video VLM：让模型真正“边看边理解”

传统 Video VLM 的思路通常还是：

`一个已经录好的 clip → 抽若干帧 → 一次性喂进去 → 回答`

但真实监控不是这样的：

`frame1 → frame2 → frame3 → …… → 永远没有结束`

**online / streaming video understanding**

[VideoLLM-online](https://arxiv.org/abs/2406.11816?utm_source=chatgpt.com) 专门研究连续视频流，设计了所谓 LIVE 框架，让模型一边接收视频、一边维护上下文、一边对当前时刻作答，而不是等整个视频结束。

[StreamingBench](https://arxiv.org/abs/2406.11816?utm_source=chatgpt.com) 就专门测这种能力：实时视觉理解、持续上下文以及多模态流。它发现当时最强的 MLLM 与人类的流式理解之间依然存在明显差距。

这条路线跟你的需求非常接近：

> 摄像头一直开着 → AI 不断获得新视觉证据 → 发现风险时形成事件。

问题：**贵。**

你不可能 30 fps × 24h 全送大模型。

所以自然就产生第二条路线。

---

### 2. Adaptive Sampling：不是“怎么理解所有帧”，而是“什么时候值得看”

现在长视频理解的一个核心问题就是：

> 给我 100 万帧，我只有资源认真看 32 帧，哪 32 帧？

传统做法是：

`每 N 秒抽一张`

**自适应采样**。

例如 LongVU 会根据帧间冗余和文本 query 对长视频进行时空压缩；高度相似的帧会被删掉，而与问题相关的帧保留更多信息。([arXiv](https://arxiv.org/abs/2410.17434?utm_source=chatgpt.com "LongVU: Spatiotemporal Adaptive Compression for Long Video-Language Understanding"))

InternVideo2.5 也把长时视频的核心问题定义成在有限 token 下保留“长时间结构 + 细粒度视觉信息”，使用自适应层级压缩。([arXiv](https://arxiv.org/abs/2501.12386?utm_source=chatgpt.com "InternVideo2.5: Empowering Video MLLMs with Long and Rich Context Modeling"))

Adaptive Keyframe Sampling 同时考虑“这帧与问题有多相关”和“这些帧是否覆盖了整段视频”。([arXiv](https://arxiv.org/abs/2502.21271?utm_source=chatgpt.com "Adaptive Keyframe Sampling for Long Video Understanding"))

2026 年的 VideoBrain已经开始把“选择下一批应该看的帧”本身变成一个 agent policy，让 VLM 在觉得信息不够时主动请求更多帧。([arXiv](https://arxiv.org/abs/2602.04094?utm_source=chatgpt.com "VideoBrain: Learning Adaptive Frame Sampling for Long Video Understanding"))

所以可以想象一个非常适合 SAGE 的结构：

```text
Camera / RTSP
      ↓
连续轻量视觉流
      ↓
Sampling / Trigger Agent
  ├─ 没变化 → 暂时跳过
  ├─ 周期必检 → 取一帧
  ├─ 新人/新物体 → 取帧
  ├─ 高风险目标 → 提高采样率
  └─ 动作变化 → 截取 5~10 秒 clip
               ↓
             SAGE
```

---

### 3. Detection + Tracking + Temporal Rules：工业界目前最靠谱的一层

有些事情其实根本不需要 VLM。

例如：

> 某人进入激光器危险区。

如果已经检测到 person：

```text
person bbox
    ↓
tracker ID=17
    ↓
进入 ROI_A
    ↓
持续 > 5 s
    ↓
hazard
```

就够了。

甚至“某个人持续十分钟没穿实验服”也可以：

```text
person #17
↓
lab_coat = false
↓
持续存在
↓
超过 threshold
↓
alert
```

DeepStream 的方向检测和越线检测甚至明确要求 tracker ID，因为这两个问题本身依赖历史状态。([NVIDIA Docs](https://docs.nvidia.com/metropolis/deepstream/7.1/text/DS_plugin_gst-nvdsanalytics.html?utm_source=chatgpt.com "Gst-nvdsanalytics — DeepStream documentation"))

学术上对应的则是：

- Multi-Object Tracking
    
- Temporal Action Localization
    
- Action Segmentation
    
- Temporal Grounding
    
- Open-Vocabulary Action Localization
    

例如 OVFormer 已经开始研究：

> 训练集里没有出现过这个动作类别，能否通过自然语言在视频里定位它？

这就是 **Open-Vocabulary Temporal Action Localization**。([arXiv](https://arxiv.org/abs/2406.15556?utm_source=chatgpt.com "Open-Vocabulary Temporal Action Localization using Multimodal Guidance"))

这对你们很关键，因为实验室隐患类别显然不可能提前枚举完整。

---

### 4. Video VLM：处理“规则写不出来”的东西

规则非常适合：

> 人进入禁区。

但下面这种就麻烦了：

> 这个人正在以不安全的方式转移化学试剂。

此时 Video VLM 才真正有价值。

当前前沿已经开始从普通 VideoQA 走向：

**temporal grounding + reasoning**：

> “危险操作发生在哪里？”
> 
> → `01:34–01:41`

而不只是：

> “视频里发生危险了吗？”
> 
> → Yes.

这其实是安全系统特别需要的，因为结论必须回溯到证据。现在 Video Temporal Grounding + MLLM 已经形成相当独立的一条研究方向。([arXiv](https://arxiv.org/abs/2508.10922?utm_source=chatgpt.com "A Survey on Video Temporal Grounding with Multimodal Large Language Model"))

但这里有个很重要的现实：

**Video VLM 现在远没有可靠到可以直接接管安全判断。**

2026 年新的 Video-MME-v2 专门加强了对视觉信息聚合、时间动态和复杂多模态推理的测试，结果仍然显示最强模型和人类专家之间存在明显距离，而且前面的视觉和时间理解错误会继续传播到高级推理。([arXiv](https://arxiv.org/abs/2604.05015?utm_source=chatgpt.com "Video-MME-v2: Towards the Next Stage in Benchmarks for Comprehensive Video Understanding"))

所以更合理的是：

**VLM 是语义层，不是整个系统。**

---

### 5. Agentic Video Understanding：这个反而非常像你现在的 SAGE 思路

还有一条越来越有意思的路线：

> 不让 VLM 一口吞完整视频，而让 LLM/VLM 当 Agent，决定下一步去哪找证据。

VideoAgent 的逻辑就是：

```text
Question
   ↓
Agent
   ↓
“先看看开头”
   ↓
“好像在操作设备”
   ↓
“检索设备出现区间”
   ↓
“再取 03:20 附近几帧”
   ↓
“不确定，再看 03:18–03:26”
   ↓
Answer
```

最早的 VideoAgent 工作就发现，用平均只有约 8 帧的主动检索，可以完成相当多长视频问题，而不是均匀读取整个视频。([arXiv](https://arxiv.org/abs/2403.10517?utm_source=chatgpt.com "VideoAgent: Long-form Video Understanding with Large Language Model as Agent"))

VideoAgent2 又把“不确定性”引进来：如果当前证据可信度低，就继续找；工具输出也可能错，因此 Agent 要评估工具可靠性。([arXiv](https://arxiv.org/abs/2504.04471?utm_source=chatgpt.com "VideoAgent2: Enhancing the LLM-Based Agent System for Long-Form Video Understanding by Uncertainty-Aware CoT"))

这和你现在已经有：

**Planner → Detector → Scene → Advisor → Reviewer**

这种结构其实非常容易接起来。

```text
视频时间轴
        ↓
Video Observer
        ↓
发现疑点
        ↓
SAGE Planner
        ↓
决定需要：
   再看一帧？
   看局部？
   看前5秒？
   看后5秒？
   跟踪某个人？
        ↓
收集证据
        ↓
判断
```

这就已经是比较正经的**Agentic Video Reasoning**了。

---

# 实验室安全特别在哪里？

### 第一、实验室危险很多是“状态 + 关系”，而不是简单对象类别

“有瓶子”不是危险。

而可能是：

> 易燃物 + 热源 + 距离过近。

所以它天然是：

$$ Hazard=f(Object, Relation, State, Context)Hazard = f(Object,\ Relation,\ State,\ Context)$$

这正是 SAGE 原来 Scene Graph / Advisor 思路能发挥作用的地方。

---

### 第二、很多实验室安全判断具有**程序依赖性**

同一个动作可能：

- 在步骤 A 是正确操作；
    
- 在步骤 B 就是违规操作。
    

比如 PPE 要求、设备状态、试剂顺序、等待时间等，很可能依赖当前实验阶段。

**2026 年刚好已经有专门针对实验室视频安全的工作把这点作为核心问题。**

Xu 和 Zhao 的工作直接把实验室安全描述为 **procedure-dependent**，其方案是：

```text
实验视频
↓
识别当前实验步骤
↓
和 protocol 对齐
↓
检索当前步骤的安全要求
↓
提取设备/操作视觉证据
↓
Video VLM 判断
```

而且他们报告说加入 procedure context 后，相比 video-only baseline，安全判断明显改善。([DOI](https://doi.org/10.69997/sct.104078?utm_source=chatgpt.com "PSE Community.org"))

这个方向跟 SAGE 的“检查要求”非常接近。

甚至可以把你现在的：

> 用户输入检查要求

升级成：

> **动态 Task/Protocol Context**

于是系统不是问：

> “这幅画面危险吗？”

而是问：

> “按照当前实验步骤和对应规范，这段操作有什么风险？”

这个学术味道就完全不一样了。

---

### 第三、实验操作是典型的细粒度 hand-object interaction

这一点 FineBio 已经非常清楚地证明了最难的恰恰是**细粒度时间边界和被操作对象**。([arXiv](https://arxiv.org/abs/2402.00293?utm_source=chatgpt.com "FineBio: A Fine-Grained Video Dataset of Biological Experiments with Hierarchical Annotation"))

比如：

> “拿起试管”

容易。

但：

> “把 A 液体加入 B 容器”

就变成：

```text
hand
↓
manipulated object = A
↓
action = pour
↓
affected object = B
```

这就是时空关系理解。

---

### 第四、小目标特别多

护目镜、手套、瓶盖、针头、标签、细管……

已有实验室 PPE 工作甚至直接把**小目标**作为实验室视觉检测的一项核心困难。([科学直达](https://www.sciencedirect.com/science/article/pii/S2405844024122955?utm_source=chatgpt.com "Recognition algorithm for laboratory protective equipment based on improved YOLOv7 - ScienceDirect"))

所以普通 Video VLM 大幅压缩视频帧时，很可能把你最想看的东西压没了。

这给“安全感知采样”留下了一个很有意思的问题：

> **采样不能只保证语义覆盖，还必须保证安全关键小目标的空间细节。**

这比普通长视频 QA 多了一个约束。

---

### 第五、实验室安全是极端 long-tail

真事故不能天天制造来训练模型🌚。

安全数据天然呈：

```text
99.99% 正常
0.01% 真正重要
```

而且最重要的那 0.01% 还最难收集。

这就是为什么异常检测、弱监督、合成数据、仿真以及 open-vocabulary reasoning 都特别有意义。

Chemist Eye 今年在 self-driving laboratory 中就是通过真实实验室采集的数据进行可重复 replay，而不是为了评估直接制造危险事故；它还把 RGB、Depth、IR 和 VLM 组合起来用于 PPE、人员事故和火灾风险。([皇家化学会出版物](https://pubs.rsc.org/en/content/articlehtml/2026/dd/d6dd00062b?utm_source=chatgpt.com "Chemist Eye: a visual language model-powered system for safety monitoring and robot decision-making in self-driving laboratories - Digital Discovery (RSC Publishing) DOI:10.1039/D6DD00062B"))

你们 PPT 本身也已经把夜间弱光、可见光失效以及红外+可见光双模态作为重要问题提出。

所以“多模态”并不是纯包装，确实是实验室场景的合理延伸。

---

# 问题的界定

不要滑成：

> YOLOvX + 改 backbone + 做实验室 PPE 数据集

这条线已经很多，而且学术上相对拥挤。比如 2024 年已经有实验室 PPE YOLOv7，2026 年也还有针对 laboratory unsafe behavior 的轻量 YOLO 工作。([科学直达](https://www.sciencedirect.com/science/article/pii/S2405844024122955?utm_source=chatgpt.com "Recognition algorithm for laboratory protective equipment based on improved YOLOv7 - ScienceDirect"))

更有价值的是把问题抽象一层。

三个等级：

### A. 面向连续视频的安全关键自适应采样与多智能体语义分析

最容易落地。

核心问题：

> 在有限算力下，如何从长视频中选择足够的视觉证据，同时尽量不漏掉安全事件？

这是：

$$max⁡Hazard Recall−λ1VLM Cost−λ2Latency\max \text{Hazard Recall} - \lambda_1 \text{VLM Cost} - \lambda_2 \text{Latency}$$

这条最容易直接接 SAGE。

---

### B. 任务条件化的时空安全事件理解

输入：

$$(Video, Task/Rule)(Video,\ Task/Rule)$$

输出：

$$Event=(type, object, region, ts, te, evidence, confidence)Event = (type,\ object,\ region,\ t_s,\ t_e,\ evidence,\ confidence)$$

例如：

```text
Video:
实验室摄像头流

Task:
“检查人员在操作强腐蚀性试剂时是否佩戴护目镜”

Output:
event_type: PPE_violation
person_id: 17
time: 12:37:21–12:37:46
evidence_frames: [...]
status: persistent
confidence: ...
```

于是它就不再是实验室专用问题。

---

### C. Procedure/Policy-Grounded Video Safety Reasoning

更强、也更难。

即：

```text
视频
+
当前任务/实验流程
+
安全规则/法规
+
设备状态
        ↓
时空证据提取
        ↓
风险推理
```

这里真正研究的是：

> **行为本身不一定危险，行为与“当前任务规则”的不一致才构成危险。**

这个思想能自然迁移到：

- 化工厂；
    
- 制造流水线；
    
- 仓储；
    
- 医院；
    
- 手术室；
    
- 食品加工；
    
- 建筑施工；
    
- 电力检修；
    
- 自动化实验室。
    

因为它们全部具有：

$$Video+Procedure+Safety Rules→RiskVideo + Procedure + Safety\ Rules \rightarrow Risk$$

所以应用范围一下就打开了。

---

# “论文级”架构

我会把它画成：

```text
                     ┌─ RGB Camera
Video Acquisition ───┼─ RTSP / recorded video
                     └─ optional IR / Depth
                              │
                              ▼
                  Streaming Observation Layer
                 detector / tracker / change
                              │
                   ┌──────────┴──────────┐
                   │                     │
            periodic inspection    event trigger
                   │                     │
                   └──────────┬──────────┘
                              ▼
                Safety-aware Frame/Clip Selector
                   “现在应该看哪些证据？”
                              │
                   ┌──────────┴──────────┐
                   ▼                     ▼
               Single Frame         Short Video
                 SAGE               Video VLM
                   │                     │
                   └──────────┬──────────┘
                              ▼
                  Task / Protocol Context
                              │
                              ▼
                    Evidence Grounding
                  object + region + time
                              │
                              ▼
                      Hazard Event State
             NEW → ACTIVE → UNKNOWN → RESOLVED
                              │
                              ▼
                  Reviewer / Human Review
                              │
                              ▼
                   Alert / Report / Archive
```

这里面**真正有论文价值的模块不是 Video Acquisition**。

采集直接用成熟技术即可。

真正能写东西的是中间三个：

**① Safety-aware adaptive sampling  
② task-conditioned temporal grounding  
③ evidence-grounded hazard event reasoning**

---

## SAGE 的功能

SAGE 可以继续充当：

> **重型语义分析器**

Detector / Scene / Advisor / Reviewer 都不用扔。

你只是把它从：

```text
用户上传图
→ SAGE
```

升级成：

```text
连续视频
→ 自动决定什么时候、看什么
→ 需要的时候调用 SAGE
→ 必要时补视频时序分析
→ 形成持续 hazard event
```

这样它才真正从一个“图片分析 Demo”变成“自主视觉巡检系统”**。

而且最有意思的是，你们现在甚至已经恰好处在这个分界线上：SAGE 有单图多 Agent 语义能力，但没有采集、跟踪、跨时间事件；你现在缺的这一层，恰好就是当前视频理解研究里仍然非常活跃的问题。

如果让我选，我会优先把问题收敛成 **“任务条件化 + 自适应采样 + 时空事件证据”** 这一条。它比“给 SAGE 加视频”宽得多，又没有宽到“我要解决所有 Video Understanding”这种完全做不动的程度。

下一步其实应该先把**实验室安全事件按「单帧可判断 / 持续状态 / 必须时序 / 必须结合流程」分成四类**，然后我们就能非常具体地判断哪些该交给 YOLO/Tracker，哪些给 SAGE，哪些才真的值得上 Video VLM。这个表一做出来，研究问题基本就会自己浮出来。