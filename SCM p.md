# 一、baseline实现

先实现这样的baseline：

```
一次配置：
Task A = “检查人员 PPE”
Task B = “检查热源附近易燃物”
Task C = “检查危险区域入侵”

                     ↓

视频自动进入
                     ↓
轻量候选采样
                     ↓
对 Task A/B/C 分别做
query-frame relevance
                     ↓
relevance + diversity + coverage
                     ↓
挑代表帧
                     ↓
自动调用 SAGE
                     ↓
记录结果
```

整个运行期间**不需要人不断 prompt**。自然语言只是把任务写成机器可以复用的配置。

## 特殊规则

- 强制周期采样
	- 变化触发不能替代周期必检，否则长期静态隐患会因为画面没变化而消失在采样器视野里。
- 轻量 detector
	- 检测到危险对象组合、敏感区域、人员操作状态，即进入采样评分
- procedure-dependent
	- 改为可复用任务模版，使视频 sampling 本身与任务要求相关联
- 漏检代价不对称
- 可追溯证据

## 当前计划

- **先固定 3–5 个检查任务，不要写“检查所有实验室安全隐患”。** 比如“PPE 是否佩戴”“易燃物是否靠近热源”“实验台是否存在明显杂乱/危险堆放”。把它们保存成 `TaskTemplate`，以后系统自动加载，不需要人每次 prompt。先选那些**单帧就能判断**的任务，动作、流程、持续时间先不碰。
- **写一个完全独立的 `VideoSampler`。** 第一版只处理本地 MP4，不接摄像头、不接 RTSP。它负责解码视频，并为每个候选帧保存 `video_id / frame_index / timestamp / image`。这一层一定和 SAGE 解耦，因为以后视频文件、摄像头、RTSP 都只是不同输入适配器。LightScan 可以参考它“视频输入→后台抽帧”的工程思路，但不要继承它缺失完整视频时间字段的问题。
- **先实现三个采样 baseline。** 最基础的 `Uniform`：每 2 秒一帧；然后 `Relevance`：用现成 CLIP/SigLIP 计算“帧 ↔ TaskTemplate”的相似度，取 top-K；最后是你现在说的 `Relevance + Diversity/Coverage`：既选和任务相关的帧，又避免 10 张全挤在同一个时间段。这里**全部不训练**。暂时也别做 risk-aware，先证明这条链本身工作。
- **把选出的帧自动送给 SAGE。** pipeline 最初就长这样：
    
    ```
    video.mp4
        ↓
    candidate frames
        ↓
    Uniform / Query-aware Selector
        ↓
    selected FrameRecords
        ↓
    TaskTemplate
        ↓
    SAGE 单图分析
        ↓
    每帧结果 + timestamp
        ↓
    简单 JSON / HTML 汇总
    ```
    
    这里每一帧最好创建独立 run/session，至少在你修掉当前 SAGE 的跨运行旧证据污染之前不要复用状态；这个问题已经在审计中确认存在。
    
- **自己录一个很小的实验视频集来评估。** 不需要训练集，先要的是测试集。比如自己控制场景录几十段短视频：正常场景、某个静态隐患一直存在、隐患中途出现、人在画面里走动、光照变化。然后比较 `Uniform / Relevance / Relevance+Diversity` 在相同“送给 SAGE 的帧数”下，谁更容易保留真正有用的证据。主要先看 `事件/隐患有没有被至少一张选中帧覆盖`、SAGE 调用次数和发现延迟，而不是急着算复杂 benchmark。
- **baseline 跑完再决定下一步。** 如果发现“相关帧总集中在一个地方”，再做 coarse-to-fine；如果发现大量稳定画面浪费计算，再加变化筛选；如果发现短动作老漏，再加风险触发高频采样；如果发现规则策略已经很复杂，再考虑 agentic controller。交接材料本身也强调了，周期必检不能完全被变化触发替代，因为静态长期隐患可能几乎没有画面变化。

## 参考

**coverage 参考** 
CVPR 2025 的 **AKS** 就是很典型的一步：它不只追求 prompt–frame relevance，还显式考虑 **coverage**，避免所有选中的帧都挤在同一小段时间里。它是 training-free、plug-and-play 的，而且作者就是从 1 fps 候选池中选有限关键帧。

**relevance + diversity 参考**
ICCV 2025 **MDP³**、**AdaRD-Key**

**coarse-to-fine 参考**
CVPR 2025 的 **VideoTree** 就构建层级式视频表示，先粗后细，根据 query 逐层缩小感兴趣范围。
同年的 **T*** 更极端：它把长视频搜索定义成一个 _Long Video Haystack_ 问题——几万帧里面真正支持答案的可能只有 **1–5 帧**，然后通过 temporal + spatial adaptive zoom-in 去找那几根“针”。而且他们的 benchmark 暴露了一个挺吓人的事实：现有 keyframe selection 在精确 temporal search 上其实还很差。
到 ICLR 2026 的 **FOCUS**，又把这个做成了一个 exploration–exploitation 问题。它处理不到 2% 的视频帧，并且对长视频 QA 有明显提升。
# 二、Agentic实现

把固定规则改为由controller决定。

演化过程：

```
固定采样
↓
Query-aware relevance + diversity
↓
coarse-to-fine
↓
动态 coarse-to-fine
↓
agentic controller
```

## 参考
ICLR 2026 的 **A.I.R.** 不再只靠一个便宜的 CLIP score 排序。它先让强 VLM 理解复杂 query，然后实现迭代的selection。
2026 的 **VideoBrain** 则更进一步，它让 VLM 自己学习什么时候调用两类采样工具：
- CLIP agent：全局语义搜索，“哪里可能有我要的东西”；
- Uniform agent：对某个区间进行密集采样，“这里发生了什么细节”。

# 三、轻量 Frame selector实现

reward：
$$R=Accuracy−λNselected$$​
## 参考
CVPR 2025 的 M-LLM Frame Selection 用 MLLM/LLM 产生空间和时序监督，让一个轻量 selector 学会挑和 query 有关的帧。
ACL 2025 的 **GenS** 更是专门造了带 dense frame relevance 标注的数据，让轻量 VideoLLM 学习 frame retrieval。
而 CVPR 2026 已经直接做到 **RL frame selection**：先学 query-frame relevance，再用强化学习奖励“这组帧是否真的改善了最终下游答案”，而不是只奖励单帧相似度。


# 初步实现方案

- **先打通视频 → SAGE**
    - 本地 MP4 输入
    - 自动抽帧
    - 每帧保存 `video_id / frame_index / timestamp`
    - 自动调用现有 SAGE
    - 输出结构化 JSON  
        目标只是先把人工挑图、上传、手动输入要求这段接起来。
- **加入预设 TaskTemplate**
    - 检查要求提前配置一次
    - 运行时自动加载，不需要人重复 prompt
    - 先只选几个“单帧可判断”的安全任务
- **做三个采样 baseline**
    - `Uniform`
    - `Query relevance`
    - `Relevance + Diversity/Coverage`
    - 全部先用现成 CLIP/SigLIP，一律不训练
- **做小规模视频测试集并比较**
    - 正常场景
    - 静态持续隐患
    - 隐患中途出现
    - 人员移动/遮挡/光照变化
    - 比较：隐患覆盖率、SAGE 调用次数、发现延迟
- **根据 baseline 的真实失败再升级**
    - 时间上太集中 → `coarse-to-fine`
    - 短暂事件漏掉 → 风险触发加密采样
    - 静态隐患漏掉 → 周期强制检查
    - 规则越来越复杂 → 再考虑 agentic controller
    - 有足够视频数据后 → 再考虑 learned selector / RL  
        不要现在就训练 Video LLM。周期必检不能被变化检测完全替代。

>在现有 SAGE-Lab 外新增一个独立 `video_pipeline`，不要大改现有 Agent。第一阶段只实现本地 MP4 → FrameRecord → 自动 SAGE 分析闭环。FrameRecord 至少包含 video_id、frame_index、timestamp、image_path、task_id、selected_reason。加入可持久化 TaskTemplate，避免每次人工输入 prompt。实现三个 selector：Uniform、QueryRelevance、Relevance+Diversity/Coverage；优先使用现成 CLIP/SigLIP embedding，不训练模型。所有模块解耦，方便后续加入 coarse-to-fine、risk-triggered sampling 和 agentic controller。当前先保证可运行、可比较、可记录，不实现 RTSP、实时监控、RL 或 Video VLM 训练。