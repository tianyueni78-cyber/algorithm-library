# CP：约束规划

## 1. 基本信息
- 中文名称：约束规划
- 英文名称：Constraint Programming
- 英文缩写：CP
- 算法类型：精确优化方法
- 主要用途：组合优化、调度、分配、排班、装箱和路径规划
---

## 2. CP 是什么
约束规划是一种通过定义决策变量、变量取值范围和约束条件来求解组合优化问题的方法。
你负责把现实问题翻译成变量、约束和目标；CP 求解器负责在这些规则下寻找可行解或最优解。
它的基本思想是：
1. 定义需要决定的变量。
2. 规定每个变量可以取哪些值。
3. 添加所有必须满足的约束。
4. 根据需要设置优化目标。
5. 由 CP 求解器搜索可行解或最优解。

基本流程：
```text
问题数据
  ↓
创建决策变量
  ↓
设置变量域
  ↓
添加约束
  ↓
设置目标函数
  ↓
调用求解器
  ↓
提取并验证结果
```
##CP到底做什么
现实问题
   ↓
你进行建模
   ├── 找出决策变量
   ├── 写出约束
   └── 定义目标
   ↓
CP 求解器
   ├── 排除不可能的取值
   ├── 搜索合法组合
   ├── 判断是否无解
   └── 寻找更优方案
   ↓
求解结果

你建立变量
识别约束
说明目标
现在 CP 会帮助你：
尝试各种机器分配；
尝试各种任务顺序；
排除违反约束的方案；
判断问题是否有解；
找到合法方案；
尽可能找到最优方案。
CP 不负责发现现实世界的规则，但能根据你给出的规则推导出更多限制。
---
## 3. CP 的输入与输出
### 3.1 总体输入
CP 模型通常需要以下输入：
- 问题数据
- 决策变量定义
- 变量取值范围
- 约束条件
- 目标函数
- 求解参数
- 可选的初始解

### 3.2 总体输出
CP 求解器通常返回：
- 求解状态
- 决策变量的取值
- 可行解或最优解
- 目标函数值
- 求解时间
- 最优性界
- 搜索统计信息

常见求解状态：
```text
OPTIMAL      找到并证明了最优解
FEASIBLE     找到可行解，但没有证明最优
INFEASIBLE   证明问题没有可行解
UNKNOWN      暂时无法确定结果
```
---
## 4. CP 的模块结构
```text
CP
├── 01_Problem_Data
├── 02_Model
├── 03_Variables
├── 04_Domains
├── 05_Constraints
├── 06_Objective
├── 07_Search_Strategy
├── 08_Constraint_Propagation
├── 09_Solver
└── 10_Solution
```
---
## 5. 模块一：问题数据 Problem Data
### 5.1 作用
保存现实问题中的原始数据，为变量、约束和目标函数提供参数。

### 5.2 输入
- 作业和工序信息
- 机器信息
- 加工时间
- 可选机器集合
- 工序优先关系
- 资源容量
- 时间窗口
- 运输时间
- 交货期

### 5.3 构成
```text
ProblemData
├── jobs
├── operations
├── machines
├── processing_times
├── eligible_machines
├── precedence_relations
├── resource_capacities
└── time_windows
```
### 5.4 输出
结构化的问题数据，例如：
```python
processing_times[job][operation][machine]
eligible_machines[job][operation]
precedence_relations
machine_capacities
due_dates
```
---
## 6. 模块二：模型 Model
### 6.1 作用
保存整个约束规划问题，是变量、约束和目标函数的容器。

### 6.2 输入
- 结构化问题数据
- 模型配置参数

### 6.3 构成
```text
Model
├── Variables
├── Domains
├── Constraints
└── Objective
```

### 6.4 输出
一个可以提交给求解器的完整 CP 模型。
概念示例：
```python
model = CPModel()
```

---
## 7. 模块三：决策变量 Variables
### 7.1 作用
表示求解器需要决定的内容。
### 7.2 输入
- 变量名称
- 变量类型
- 最小值和最大值
- 离散候选集合
- 对应的任务或资源

### 7.3 构成
常见变量类型：
```text
Variables
├── IntegerVariable
├── BooleanVariable
├── IntervalVariable
└── SequenceVariable
```

### 7.4 常见变量
| 变量 | 含义 | 示例 |
|---|---|---|
| `start` | 任务开始时间 | `start_A = 10` |
| `end` | 任务结束时间 | `end_A = 15` |
| `machine` | 任务选择的机器 | `machine_A = 2` |
| `selected` | 是否选择某个方案 | `selected_A_2 = 1` |
| `interval` | 任务执行区间 | `[10, 15)` |
| `sequence` | 机器上的任务顺序 | `A → C → B` |

### 7.5 输出
可以被约束和目标函数引用的变量对象。
例如：
```text
start_A
end_A
machine_A
interval_A
selected_A_machine_2
```
---
## 8. 模块四：变量域 Domains
### 8.1 作用
规定每个决策变量允许采用的值。
### 8.2 输入
- 决策变量
- 最小值和最大值
- 离散候选集合
- 问题数据中的限制
### 8.3 构成
```text
Domains
├── ContinuousRange
├── IntegerRange
├── BooleanDomain
└── DiscreteSet
```
### 8.4 示例
```text
start_A ∈ [0, 100]
machine_A ∈ {1, 2, 4}
selected_A ∈ {0, 1}
```
### 8.5 输出
带有合法取值范围的决策变量。
约束传播过程中，变量域可能被缩小：
```text
原始范围：start_A ∈ [0, 100]
传播以后：start_A ∈ [0, 60]
```
---
## 9. 模块五：约束 Constraints
### 9.1 作用
描述一个解必须遵守的规则，排除不合法的变量组合。
### 9.2 输入
- 决策变量
- 问题规则
- 资源容量
- 时间参数
- 工序关系
### 9.3 构成
```text
Constraints
├── AssignmentConstraint
├── PrecedenceConstraint
├── NoOverlapConstraint
├── CapacityConstraint
├── TimeWindowConstraint
├── AlternativeConstraint
└── LogicalConstraint
```

### 9.4 常见约束
#### 9.4.1 分配约束
每道工序只能选择一台机器。
输入：
- 候选机器集合
- 机器选择变量
输出：
- 唯一机器选择约束
示例：
```text
selected_O1_M1
+ selected_O1_M2
+ selected_O1_M3
= 1
```
#### 9.4.2 前序约束
前一道工序结束后，后一道工序才能开始。
输入：
- 前序任务结束时间
- 后续任务开始时间
输出：
- 工序顺序约束
示例：
```text
end(O1) ≤ start(O2)
```
#### 9.4.3 机器互斥约束
同一台机器不能同时加工多个任务。
输入：
- 分配给同一台机器的任务区间
输出：
- 无重叠约束
示例：
```text
NoOverlap(machine_1_intervals)
```
#### 9.4.4 资源容量约束
所有任务使用的资源总量不能超过容量。
输入：
- 任务区间
- 任务资源需求
- 资源容量
输出：
- 累积资源约束

#### 9.4.5 时间窗口约束
任务必须在规定时间范围内执行。
输入：
- 任务开始时间
- 任务结束时间
- 最早开始时间
- 最晚结束时间
输出：
- 时间范围约束
### 9.5 输出
添加到 CP 模型中的全部约束对象。
---

## 10. 模块六：目标函数 Objective
### 10.1 作用
规定求解器应该优化什么。
### 10.2 输入
- 决策变量
- 目标计算表达式
- 最小化或最大化方向
### 10.3 构成
```text
Objective
├── ObjectiveExpression
├── OptimizationDirection
└── ObjectivePriority
```
### 10.4 常见目标
- 最小化最大完工时间
- 最小化总延误时间
- 最小化机器空闲时间
- 最小化运输时间
- 最小化能源消耗
- 最大化设备利用率
示例：
```text
makespan = max(所有工序的结束时间)
minimize makespan
```
### 10.5 输出
求解器需要优化的目标表达式。
---

## 11. 模块七：搜索策略 Search Strategy
### 11.1 作用
决定求解器先选择哪个变量，以及先尝试哪个值。
### 11.2 输入
- 当前变量
- 当前变量域
- 变量选择规则
- 值选择规则
- 随机种子
- 初始解
### 11.3 构成
```text
SearchStrategy
├── VariableSelection
├── ValueSelection
├── BranchingRule
├── RestartStrategy
└── InitialSolution
```
### 11.4 输出
下一次搜索需要尝试的分支。
例如：
```text
machine_A = 1
├── start_A = 0
└── start_A > 0
```
---
## 12. 模块八：约束传播 Constraint Propagation
### 12.1 作用
根据现有约束不断缩小变量域，提前排除不可能的取值。
### 12.2 输入
- 当前变量域
- 全部约束
- 当前搜索决定
### 12.3 构成
```text
ConstraintPropagation
├── DomainReduction
├── ConstraintChecking
├── ConflictDetection
└── FixpointIteration
```
### 12.4 输出
- 缩小后的变量域
- 新确定的变量
- 冲突信息
- 当前分支是否可行
如果某个变量的取值范围变为空：
```text
domain(variable) = {}
```
说明当前搜索分支不可行，需要回溯。
该模块通常由求解器内部实现，不需要用户自己编写。
---

## 13. 模块九：求解器 Solver
### 13.1 作用
运行约束传播、搜索、剪枝和优化过程。
### 13.2 输入
- 完整 CP 模型
- 求解时间限制
- 线程数量
- 随机种子
- 搜索策略
- 可选初始解
### 13.3 构成
```text
Solver
├── ModelLoader
├── PropagationEngine
├── SearchEngine
├── ConflictAnalyzer
├── BoundManager
└── SolutionManager
```
### 13.4 输出
```text
SolverResult
├── status
├── variable_values
├── objective_value
├── best_bound
├── solve_time
└── search_statistics
```
---
## 14. 模块十：解提取与验证 Solution
### 14.1 作用
把求解器变量转换成可以理解和使用的业务结果。
### 14.2 输入
- 求解状态
- 决策变量的取值
- 原始问题数据
- 目标函数值
### 14.3 构成
```text
Solution
├── SolutionExtractor
├── ScheduleBuilder
├── ConstraintValidator
├── ObjectiveCalculator
└── ResultExporter
```
### 14.4 输出
- 任务开始时间
- 任务结束时间
- 机器分配结果
- 机器任务序列
- 目标函数值
- 调度表
- 甘特图数据
- 约束验证结果
示例：
```text
机器1：J1-O1 → J3-O2
机器2：J2-O1 → J1-O2

Makespan = 125
```
---
## 15. CP 项目的推荐代码结构
```text
CP_Project
├── data_loader.py
├── model.py
├── variables.py
├── constraints.py
├── objective.py
├── solver.py
├── solution.py
└── main.py
```
各文件职责：
| 文件 | 主要职责 |
|---|---|
| `data_loader.py` | 读取和整理问题数据 |
| `model.py` | 创建并组织 CP 模型 |
| `variables.py` | 创建决策变量 |
| `constraints.py` | 添加约束 |
| `objective.py` | 设置目标函数 |
| `solver.py` | 配置并调用求解器 |
| `solution.py` | 提取和验证结果 |
| `main.py` | 组织完整求解流程 |
对于小型项目，不必拆成这么多文件，可以先写成：
```text
CP_Project
├── problem.py
├── solve.py
└── main.py
```
---
## 16. 用户需要实现的部分
一般需要自己实现：
1. 问题数据读取。
2. 决策变量设计。
3. 约束建模。
4. 目标函数设计。
5. 求解参数配置。
6. 结果提取。
7. 结果验证和可视化。

一般不需要自己实现：
1. 约束传播。
2. 回溯搜索。
3. 分支定界。
4. 冲突分析。
5. 底层剪枝算法。

这些通常由 CP 求解器提供。
---
## 17. CP 与 MILP 的区别
| 对比项 | CP | MILP |
|---|---|---|
| 主要表达方式 | 变量域和逻辑约束 | 线性等式与不等式 |
| 核心求解机制 | 约束传播和搜索 | 线性松弛和分支定界 |
| 调度表达 | 可以直接使用区间和互斥约束 | 通常需要引入额外二进制变量 |
| 适合的问题 | 排序、调度、分配和复杂逻辑 | 线性资源分配和精确数学优化 |
| 建模特点 | 接近问题的业务描述 | 接近统一的数学模型 |
简单记忆：
```text
MILP：这些线性公式必须成立。

CP：这些变量之间的逻辑关系必须成立。
```
---
## 18. CP 适用场景
CP 比较适合：
- Job Shop Scheduling
- Flexible Job Shop Scheduling
- 员工排班
- 课程排课
- 任务分配
- 机器分配
- 装箱问题
- 路径和顺序规划
- 时间窗口问题
- 复杂资源约束问题
---
## 19. 可与其他算法融合的位置
CP 可以与其他算法组合：
```text
GA / NSGA-II
    ↓ 生成候选方案
CP
    ↓ 修复或精确优化
可行调度方案
```
也可以：

```text
RL
    ↓ 选择搜索策略或变量顺序
CP Solver
    ↓
求解结果
```

常见融合方向：

- GA + CP
- NSGA-II + CP
- CP + RL
- CP + Large Neighborhood Search
- CP + Digital Twin

可融合模块主要包括：
- 初始解生成
- 变量选择策略
- 值选择策略
- 邻域选择
- 约束修复
- 动态重新调度

---
## 20. 一句话总结
CP 的核心结构是：
```text
问题数据
+ 决策变量
+ 变量域
+ 约束
+ 目标函数
+ 求解器
= 可行解或最优解
```
CP 最重要的工作不是自己编写搜索算法，而是把现实问题正确地表达成变量和约束。
