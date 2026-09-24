# 首先确定研究问题

面向实验室长视频的 task-conditioned event discovery 与 budget-aware content selection。

# 之前的规划

对，结合刚才查到的流式语义理解工作，我会把 **Agent 之前**正式收束成下面这条线。核心不是再“堆 selector”，而是先搭出一个**不用 Agent 也能真正实时工作的 streaming semantic pipeline**。

1. **改成 Streaming Contract + 冻结 Scorer V1**
    
    输入从完整视频改为连续 `frame/clip + timestamp + task + state`；输出允许 `DROP / KEEP / TRIGGER / UPDATE_MEMORY`。现有 1.5 s / 8-frame Scorer V1 暂时保留，作为较重的 task-conditioned relevance detector；AKS / MDP / FOCUS 留在 `offline/`，只作为完整视频条件下的参考上限，不再是主线。
    
2. **建立三类最基本的 Streaming Baseline**
    
    不是随便复现 N 篇论文，而是覆盖三种真正不同的机制：
    
    - **在线动作语义**：复现一个轻量 OAD baseline，优先 MiniROAD；之后视需要参考 CMeRT。它们解决的是“只看过去，当前正在发生什么”。MiniROAD 本身就是很简洁的 causal RNN OAD；CMeRT进一步引入短期/长期历史。([CVF Open Access](https://openaccess.thecvf.com/content/ICCV2023/html/An_MiniROAD_Minimal_RNN_Framework_for_Online_Action_Detection_ICCV_2023_paper.html?utm_source=chatgpt.com "ICCV 2023 Open Access Repository"))
        
    - **视觉变化/冗余**：做 TimeChat-Online 的 DTD-like baseline，新帧和旧帧变化不大就丢，建立 novelty signal。它只处理新到帧，不重算历史，非常符合 streaming。([TimeChat-Online](https://timechat-online.github.io/?utm_source=chatgpt.com "TimeChat-Online: 80% Visual Tokens are Naturally Redundant in Streaming Videos"))
        
    - **有限历史记忆**：先实现一个 bounded memory baseline，历史不断压缩/替换，而不是无限保存。MA-LMM 已经验证了“online processing + memory bank”这套基本结构；Flash-VStream进一步拆成长时 context memory 和细节 memory。([CVF Open Access](https://openaccess.thecvf.com/content/CVPR2024/html/He_MA-LMM_Memory-Augmented_Large_Multimodal_Model_for_Long-Term_Video_Understanding_CVPR_2024_paper.html?utm_source=chatgpt.com "CVPR 2024 Open Access Repository"))
        
3. **研究三个信号的边界，然后做我们自己的 Strong Streaming Baseline**
    
    重点就研究：
    
    task relevance+novelty+history\text{task relevance}+\text{novelty}+\text{history}
    
    哪些情况下各自有用、哪些情况下会失败。例如“人走过画面” novelty 很高但 task relevance 低；“试剂瓶开始轻微倾斜” novelty 可能低但 relevance 高。
    
    然后组成第一版自己的固定策略：
    
    ```text
    live stream
        ↓
    streaming feature
        ↓
    relevance + novelty + temporal state
        ↓
    short-term buffer + bounded memory
        ↓
    fixed policy
        ↓
    drop / keep / increase sampling / invoke Scorer V1 / commit event
    ```
    
    StreamFormer说明视觉特征本身以后也可以进一步变成 causal、逐帧增量计算，而不是每次重跑一个完整 clip；但这可以放到 baseline 跑通之后再决定是否改 Scorer V2。([CVF Open Access](https://openaccess.thecvf.com/content/ICCV2025/html/Yan_Learning_Streaming_Video_Representation_via_Multitask_Training_ICCV_2025_paper.html?utm_source=chatgpt.com "ICCV 2025 Open Access Repository"))
    
4. **把“帧 memory”进一步升级成“事件状态”，到这里 Agent 前置工作结束**
    
    我们最终不应该长期记：
    
    `frame 132 / 184 / 227`
    
    而应该逐渐形成：
    
    `拿起试剂瓶 → 开始倾倒 → 倾倒结束`
    
    OpenHOUSE 已经在 streaming 条件下研究 action boundary + hierarchical event understanding；StreamForest 则直接把长期历史组织成 persistent event memory。([CVF Open Access](https://openaccess.thecvf.com/content/ICCV2025/html/Kang_Open-ended_Hierarchical_Streaming_Video_Understanding_with_Vision_Language_Models_ICCV_2025_paper.html?utm_source=chatgpt.com "ICCV 2025 Open Access Repository"))
    
    因此 Agent 前最后一个版本应该能够维护：
    
    ```text
    current semantic state
    + short-term detailed buffer
    + long-term event memory
    + task description
    + current compute/budget state
    ```
    
    **但所有决策还是规则写死的。**
    

到这里才进 Agent：

```text
前面：
semantic state + memory + task
        ↓
   hand-written policy
        ↓
      action

Agent：
semantic state + memory + task
        ↓
     learned/reasoning policy
        ↓
      action
```

所以可以把当前开发路线压缩成一句：

> **Streaming 接口/Scorer V1 → OAD + Novelty + Memory 三类 baseline → 研究边界 → Task-conditioned Strong Streaming Baseline → Event Memory → Agent。**

其中真正值得我们自己创新的第一处，是 **Task-conditioned Strong Streaming Baseline**；第二处才是后面的 **Agent policy**。这比之前“复现六个 selector → 选最好一个 → Agent”要顺得多。 🌚


# Scorer结束后

**马上把真实视频接进 Streaming Foundation。**  
不用等 RTSP。直接把 train/val MP4 当“伪直播”按 PTS 顺序 replay：

```
mp4 decoder
→ frame stream
→ real novelty extractor
→ scorer
→ policy
→ event memory
```

你现有 replay/policy/memory 已经做好了，但之前 novelty 是外部假数据、policy 也没有主动调 scorer。 这一步正好把假 observation 换成真的。


# Scorer的意义
训练一个第一代 learned semantic scorer，看看“专门学实验室 task relevance”这件事到底有没有价值，并给后面的 streaming 实验提供一个可用的语义信号。

第一，它可以作为 **task relevance signal** 给 streaming baseline。
第二，它是一个非常清楚的 **learned baseline**。以后即使我们换成 OAD、streaming encoder 或 VLM，也可以比较：

> 一个专门训练的小 scorer 到底能做到多少？

第三，它可以给 Agent 当一个 observation。

# 现在目标

把现有 Streaming Foundation 从“能 replay scorer decisions”升级成“能从顺序到达的视频产生 signal，并真正运行几种 causal fixed selector”。

？