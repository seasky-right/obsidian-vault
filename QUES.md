**已经完成的内容**

|模块|当前状态|
|---|---|
|固定基线|已冻结 `fastmapglue_rgb_5x5_v1`，保留无 Agent 回退路径|
|第二阶段基础能力|数据检查、评价 Manifest、防数据泄漏、领域模型、事件/API 已验收|
|语义规划|豆包成对语义分析、SAM3 分割与降级、价值图、512/1024 网格、几何映射已真实运行|
|P3-1 局部内循环|多网格 MapGlue、局部质量判断、单格受控重试、证据绑定已实现并真实运行|
|P3-2 全局外循环|点池、冻结检查点、MAGSAC、DEM/RPC 精化、外检和重规划已在历史任务跑通|
|W4-A 诊断|已确定“局部约 2 px，但锁定外检约 73 px”，属于模型或数据泛化限制，不能宣称配准成功|
|W4-B 后处理|同 Manifest 比较、best/accepted/published 分离、诊断成果发布、报告和恢复状态机已实现|
|后端|Django API 已支持创建、运行、取消、回复、事件、Trace、成果下载和 W5 readiness|
|前端|Vue 工作台已接入任务、网格尝试、全局链、后处理、事件、Trace 和成果展示|
|运行保障|一次性 W5 授权、敏感信息脱敏、下载白名单、worker 租约、异常重启恢复已经落地|

PRD 中第二阶段和历史 P3-2 的验收记录见[项目PRD.md (line 26)](F:/Project/rpc/项目PRD.md:26)，W4-A 的实际误差证据见[W4_A_QUALITY_DIAGNOSIS.md (line 15)](F:/Project/rpc/reports/W4_A_QUALITY_DIAGNOSIS.md:15)。Django 接口定义在[urls.py (line 6)](F:/Project/rpc/rpc_matching_tool/rpc_matching_agent/rpc_matching_agent/matching_agent/django_api/urls.py:6)，Vue 主界面在[App.vue (line 157)](F:/Project/rpc/rpc_matching_tool/rpc_matching_agent/rpc_matching_agent/dashboard/src/App.vue:157)。

## Loop过程

1. **检查目标影像、参考影像、RPC、DEM、坐标系和覆盖范围。**

2. **豆包进行成对语义观察，SAM3分割云等低价值区域；失败时允许明确降级。**
	1. Doubao 做成对语义观察，输出低价值区域。
	2. SAM3 做精细分割。
		- 系统只处理与 Doubao 粗框相交的瓦片。
		- 命中像素后续价值固定为 0，并从网格均值统计中排除。
	3. 检查 Doubao 与 SAM3 一致性
		- 每条提议计算：
			- 提议区域像素数
			- SAM3 命中像素数
			- 命中比例
			- `consistent / suspicious / unavailable`
		- 当前阈值是 20%。低于 20% 标为 `suspicious`，但不会撤销 Doubao 的低价值判断。
	4. 计算视觉基础图
		- 采用加权平均：$visual_base = 0.46 × texture + 0.31 × edge + 0.23 × corner$
	5. 融合语义信息
		- 规则：
			- Doubao 粗云区之外：× 1.0
			- Doubao 粗云区之内：× 0.35
			- SAM3 命中像素：直接设为 0
	6. 生成价值图 artifact
	
3. **构造并评分 512/1024 网格，计算每格分数。**
	- 规则：$matchability_score = 0.55 × 网格平均价值 + 0.45 × 高价值像素比例$
	- 高价值像素目前指价值不低于 `0.65`。

	- 每格同时记录：
		- SAM3 排除比例
		- Doubao 低价值提议覆盖比例，覆盖率低于25%的格标记为```clean```
		- 有效评分比例
		- 原始视觉基础分
		- 语义降权后分数

4. **P3-1 选择最多六个兼顾质量与空间覆盖的格；每格运行 MapGlue，质量不足时只允许一次受控重试。**
	- 先对全图做 $4*4$ 粗分割。
	- 要求：
		- 规划器在最多 6 格内满足：
			- 12/16 空间桶覆盖
			- 四象限覆盖
			- 风险探针不超过队列的 1/3
			- clean 格必须严格多于风险格
	- 优化顺序是：
			风险格数量最少
			→ 总格数最少
			→ SAM3排除更少
			→ Doubao覆盖更少
			→ 有效像素更多
			→ 匹配分更高
	- 达到至少 3 格、50% 的 4×4 空间覆盖和四象限覆盖后，才能进入 P3-2。
		- 具体准入规则：
			- 系统检查接受格和匹配点是否满足：
				- 最低接受格数量
				- 最低控制点数量
				- 至少 8/16 的实际空间覆盖
				- 四象限覆盖
				- 点来源和 `GridCellRef` 身份一致
			- 不可达时停在 `grid_matching`，不会强行进入 RPC 拟合。
	
5. **P3-2 汇总严格绑定文件路径和 SHA-256 的局部匹配点，去重、排冲突，冻结唯一评价 Manifest。**
6. **运行全局 MAGSAC、DEM 采样和 RPC 精化，再用冻结检查点评价。**
7. **外循环可选择接受、从 3 px 调到 5 px、删除一个有害格，或有序停止；最多 3 次全局尝试。**
	- 当前只实现了“同格纠正”，后续可继续实现“换格”。
8. **W4-B 按固定五步执行：进入后处理 → 评价冻结 baseline → 确定性比较 → 受控发布 → 最终化。**
9. **最终只允许 `succeeded`、`quality_failed` 或 `failed`。未过门的结果最多标记为 `best_diagnostic`，不能显示为合格成果。**

W4-B 五步顺序由机器固定，不允许豆包跳步或指定发布对象，见[agent_decision.py (line 42)](F:/Project/rpc/rpc_matching_tool/rpc_matching_agent/rpc_matching_agent/matching_agent/agent_decision.py:42)。状态在每个原子边界写入 `task_state.json`，Trace 单独保存，见[storage.py (line 36)](F:/Project/rpc/rpc_matching_tool/rpc_matching_agent/rpc_matching_agent/matching_agent/storage.py:36)。