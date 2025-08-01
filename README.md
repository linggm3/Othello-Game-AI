# 🤖 Othello-Game-AI: 基于 MiniMax 的黑白棋智能棋手  



这是一个用 **纯 Python** 实现的黑白棋人机对战项目。详细报告参见 `report.pdf`。

* 核心算法采用 **MiniMax + Alpha-Beta 剪枝**，实现自动博弈
* 通过 **遗传算法（GA）** 与 **模拟退火（SA）** 自动进化评估函数权重，让 AI 越下越强！


> “你能战胜你自己亲手训练出的 AI 吗？”  

---

## 📑 目录

- [🤖 Othello-Game-AI: 基于 MiniMax 的黑白棋智能棋手](#-othello-game-ai-基于-minimax-的黑白棋智能棋手)
  - [📑 目录](#-目录)
  - [🎲 项目简介](#-项目简介)
  - [🧠 算法亮点](#-算法亮点)
  - [🛠️ 项目结构](#️-项目结构)
  - [🎮 黑白棋简介](#-黑白棋简介)
  - [🧮 MiniMax 博弈树搜索原理](#-minimax-博弈树搜索原理)
  - [✂️ Alpha-Beta 剪枝原理](#️-alpha-beta-剪枝原理)
  - [🧬 遗传算法和模拟退火算法](#-遗传算法和模拟退火算法)

## 🎲 项目简介
黑白棋（Othello）是一款经典的完全信息零和博弈。本项目通过 Minimax 搜索 + Alpha-Beta 剪枝 + 启发式评估函数 实现了一个高性能 AI 棋手，并提供了：
* 人机对战（支持命令行 & GUI）
* AI 参数自学习（遗传算法 / 模拟退火）
* 可视化棋盘（matplotlib 动态渲染）


## 🧠 算法亮点
| 模块       | 技术方案                               |
| -------- | ---------------------------------- |
| **搜索算法** | Minimax + Alpha-Beta 剪枝（支持自定义搜索深度） |
| **评估函数** | 棋盘位置权重 + 子数权重 + 行动力 + 稳定子（参数可学习）   |
| **参数优化** | 遗传算法（锦标赛选择） vs 模拟退火（对比实验）          |
| **性能优化** | Python 多进程并行锦标赛（加速 60 倍）           |
| **可视化**  | matplotlib 实时绘制棋盘 & 落子动画           |


## 🛠️ 项目结构
```
Othello-AI/
├── Code/                 # 所有源代码
│   ├── Class.py              # 核心棋类 + AI 逻辑
│   ├── Main.py               # 人机对战入口
│   ├── GA.py                 # 遗传算法训练
│   └── SA.py                 # 模拟退火训练
├── figs/                 # README 图片
├── report.pdf            # 项目报告，包含项目详细说明
├── requirements.txt
└── README.md
```


## 🎮 黑白棋简介

黑白棋（Reversi / Othello）是 19 世纪末由英国人发明的两人零和完全信息博弈。  
- **棋盘**：8×8 共 64 个格子。  
- **棋子**：双面，一面黑一面白，双方各执一色。  
- **初始布局**：中心 2×2 区域交叉放置 4 枚棋子。  
- **落子规则**：必须夹住对方至少一条连续直线（横、竖、斜），并将该直线上对方棋子全部翻转。  
- **胜负**：棋盘填满或双方均无合法落子时，棋子多者获胜；若数量相同则为平局。  

黑白棋的魅力在于 **局势瞬息万变**，一步落子可能翻转十几枚棋子，因此需要强大的搜索与评估能力来预判局面。


## 🧮 MiniMax 博弈树搜索原理

Minimax 是专为 **零和博弈** 设计的递归搜索算法，其核心思想是：

1. **构建博弈树**  
   - 根节点：当前局面。  
   - 子节点：当前玩家所有合法落子后的局面。  
   - 叶节点：游戏结束或达到预设搜索深度的局面。  

2. **节点类型**  
   - **Max 节点**（我方回合）：选择子节点中 **价值最大** 的走法。  
   - **Min 节点**（对手回合）：选择子节点中 **价值最小** 的走法（对我方最不利）。  

3. **价值回传**  
   自底向上计算每个节点的 minimax 值，直至根节点，从而选出最佳落子。  

4. **评估函数**（当搜索无法到达终局时使用）  
   在本项目中，评估函数综合考虑：  
   - 棋盘位置权重（角、边、中心）  
   - 子数差  
   - 行动力（可落子数）  
   - 稳定子（无法被翻转的棋子）  

Minimax 伪代码：
```python
def minimax(node, depth, maximizing):
    if depth == 0 or node.is_terminal():
        return evaluate(node)

    if maximizing:
        value = -inf
        for child in node.children():
            value = max(value, minimax(child, depth-1, False))
        return value
    else:
        value = +inf
        for child in node.children():
            value = min(value, minimax(child, depth-1, True))
        return value
```

## ✂️ Alpha-Beta 剪枝原理
Alpha-Beta 剪枝是 Minimax 的优化技巧，通过维护两个边界值 α 和 β，剪掉不可能影响最终决策的分支，从而将搜索树规模从指数级降为 近线性。
* α（max 的下界）：Max 节点已确认的最小收益。
* β（min 的上界）：Min 节点已确认的最大损失。

剪枝规则：

* Max 层：若某子节点的值 ≥ β，则停止搜索该节点的剩余子节点（剪枝）。
* Min 层：若某子节点的值 ≤ α，则停止搜索该节点的剩余子节点（剪枝）。

伪代码：
```python
def alphabeta(node, depth, α, β, maximizing):
    if depth == 0 or node.is_terminal():
        return evaluate(node)

    if maximizing:
        value = -inf
        for child in node.children():
            value = max(value, alphabeta(child, depth-1, α, β, False))
            α = max(α, value)
            if α ≥ β: break  # β剪枝
        return value
    else:
        value = +inf
        for child in node.children():
            value = min(value, alphabeta(child, depth-1, α, β, True))
            β = min(β, value)
            if α ≥ β: break  # α剪枝
        return value
```

## 🧬 遗传算法和模拟退火算法
本项目中的 AI 强度 取决于评估函数的权重。由于权重维度高，人工调参效率低，因此我们引入 元启发式算法 自动寻优。

| 算法            | 角色     | 实现细节                                  | 优缺点               |
| ------------- | ------ | ------------------------------------- | ----------------- |
| **遗传算法 (GA)** | 全局搜索   | 种群规模 64 → 锦标赛选择 → 交叉 & 变异 → 200 代收敛   | 探索能力强，收敛快；局部微调弱   |
| **模拟退火 (SA)** | 局部精细搜索 | 单解迭代 → 温度 T 递减 → 以概率 `exp(-Δ/T)` 接受劣解 | 易跳出局部极值；收敛慢，对初值敏感 |

在本项目的流程
1. 编码：将 10 维位置权重 + 行动力权重 + 子数权重编码成染色体/解向量。
2. 适应度：让两个权重对弈 30 步，净胜子数作为适应度。
3. 训练：
     * GA：多进程并行锦标赛（64 场同时跑），24 核 CPU 加速 60 倍。
     * SA：单进程串行，