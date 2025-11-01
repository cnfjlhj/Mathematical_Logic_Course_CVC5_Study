# cvc5 SMT 求解器：介绍与使用演示

> 西安电子科技大学 数学逻辑课程 - cvc5 深度教学项目

## 📚 项目简介

本项目系统性地介绍了 **cvc5** SMT（Satisfiability Modulo Theories，可满足性模理论）求解器，包括：

- 🎓 **理论基础**：SMT 核心概念、cvc5 架构原理
- 💻 **Python API**：从基础到高级的完整 API 使用教程
- 🚀 **实战应用**：涵盖经典问题、约束求解、形式化验证等多个领域
- 📊 **可视化演示**：Jupyter Notebook 交互式展示

### 与同类项目的差异化优势

相比传统的 cvc5 教程，本项目特点：

✅ **系统化教学路径**：从零基础到高级应用的递进式学习
✅ **统一 Pythonic API**：使用更简洁优雅的 Python 风格代码
✅ **详细中文注释**：每一步都有清晰的原理说明
✅ **真实应用场景**：包含有界模型检查、程序验证等实际案例
✅ **可视化展示**：数独、N 皇后等问题的图形化输出

---

## 🗂️ 项目结构

```
cvc5/
├── README.md                           # 项目总览（本文件）
├── requirements.txt                    # Python 依赖
│
├── docs/                               # 📖 理论文档
│   ├── 01_introduction.md              # cvc5 背景介绍
│   ├── 02_smt_basics.md                # SMT 核心概念
│   ├── 03_cvc5_architecture.md         # cvc5 架构原理
│   └── 04_api_guide.md                 # Python API 完整指南
│
├── examples/                           # 💻 代码示例
│   ├── basics/                         # 基础篇
│   │   ├── 01_hello_cvc5.py           # 最简示例
│   │   ├── 02_boolean_logic.py        # 布尔逻辑
│   │   ├── 03_linear_arithmetic.py    # 线性算术
│   │   └── 04_data_types.py           # 数据类型
│   │
│   ├── intermediate/                   # 进阶篇
│   │   ├── 05_quantifiers.py          # 量词逻辑
│   │   ├── 06_arrays_theory.py        # 数组理论
│   │   ├── 07_bitvectors.py           # 位向量
│   │   └── 08_unsat_core.py           # 不可满足核心分析
│   │
│   └── advanced/                       # 高级应用篇
│       ├── 09_sudoku_solver.py        # 数独求解器（带可视化）
│       ├── 10_n_queens.py             # N 皇后问题
│       ├── 11_graph_coloring.py       # 图着色问题
│       ├── 12_program_verification.py # 程序验证
│       └── 13_bounded_model_checking.py # 有界模型检查 ⭐
│
├── notebooks/                          # 📊 Jupyter 演示
│   └── cvc5_presentation.ipynb        # 课堂展示主 Notebook
│
└── utils/                              # 🛠️ 工具函数
    ├── visualizer.py                  # 可视化工具
    └── helpers.py                     # 通用辅助函数
```

---

## 🚀 快速开始

### 1. 安装依赖

```bash
pip install -r requirements.txt
```

主要依赖：
- `cvc5` - cvc5 Python API
- `matplotlib` - 可视化
- `numpy` - 数值计算
- `jupyter` - Notebook 支持

### 2. 运行示例

```bash
# 运行基础示例
python examples/basics/01_hello_cvc5.py

# 运行数独求解器
python examples/advanced/09_sudoku_solver.py

# 运行有界模型检查
python examples/advanced/13_bounded_model_checking.py
```

### 3. 查看交互式演示

```bash
jupyter notebook notebooks/cvc5_presentation.ipynb
```

---

## 📖 学习路径

### 初学者
1. 阅读 `docs/01_introduction.md` 了解 cvc5 背景
2. 阅读 `docs/02_smt_basics.md` 理解 SMT 概念
3. 运行 `examples/basics/` 下的所有示例
4. 查看 `notebooks/cvc5_presentation.ipynb` 前半部分

### 进阶学习
1. 阅读 `docs/03_cvc5_architecture.md` 深入理解原理
2. 运行 `examples/intermediate/` 下的示例
3. 尝试修改示例代码，解决自己的问题

### 高级应用
1. 阅读 `docs/04_api_guide.md` 掌握完整 API
2. 研究 `examples/advanced/` 下的实战案例
3. 特别关注有界模型检查（BMC）示例

---

## 🎯 核心示例介绍

### 基础示例

#### 1. Hello cvc5 - 最简约束求解
```python
from cvc5.pythonic import *

x, y = Ints('x y')
solve(x + y == 5, x > 0, y > 0)
# 输出: [x = 1, y = 4] (或其他满足条件的解)
```

#### 2. 数独求解器
使用 cvc5 求解任意数独谜题，支持可视化输出。

#### 3. N 皇后问题
经典约束满足问题，演示如何用 SMT 求解器处理组合优化。

### 高级应用

#### 🔥 有界模型检查（Bounded Model Checking）
本项目的**核心亮点**，展示 cvc5 在形式化验证中的实际应用：

- **层次 1**：简单状态机验证（交通灯系统）
- **层次 2**：硬件电路正确性验证（加法器）
- **原理讲解**：如何将系统建模为符号转换系统（STS）

---

## 📊 性能测试

本项目包含多个性能测试示例，对比不同求解策略的效率：

| 问题类型 | 问题规模 | 求解时间 | 备注 |
|---------|---------|---------|------|
| SAT (布尔逻辑) | 5 变量 7 子句 | < 0.001s | 基础问题 |
| 数独 | 9×9 | ~ 0.05s | 中等约束 |
| N 皇后 | N=8 | ~ 0.02s | 组合优化 |
| 整数规划 | 3 变量 | ~ 0.001s/迭代 | 线性约束 |

---

## 🚧 后续增强计划

本项目正在持续改进中，计划添加以下增强内容：

### Phase 1: 核心可视化 (P0 优先级)

**目标**：为核心深度示例添加可视化，帮助理解算法内部机制

| 示例 | 可视化内容 | 价值 |
|------|----------|------|
| **BMC (有界模型检查)** | 状态转换图、执行路径树 | 理解符号执行过程 |
| **DPLL(T) 算法** | 4 阶段流程图、决策树 | 理解 SMT 求解器核心 |
| **UNSAT Core** | 约束增长条形图、冲突子集 | 理解冲突诊断机制 |
| **程序验证** | Hoare 逻辑三段论图、前后置条件推理 | 理解形式化验证 |

**预期成果**：
- ASCII 艺术风格的流程图
- Matplotlib 性能对比图表
- 推理过程表格化展示

### Phase 2: 多变体扩展 (P0 优先级)

**目标**：为每个核心示例提供多个变体，展示技术的通用性

| 原示例 | 变体 1 | 变体 2 | 变体 3 |
|--------|--------|--------|--------|
| **SAT 求解** | 图 3-着色问题 | 排班系统 | 逻辑谜题 2 |
| **BMC** | 自动售货机验证 | 计数器溢出检测 | 协议状态机验证 |
| **程序验证** | 排序算法验证 | 最大值查找验证 | 数组反转验证 |
| **整数规划** | 背包问题 | 生产计划 | 资源分配 |

### Phase 3: 交互式内容 (P1 优先级)

**目标**：添加 ipywidgets 交互式组件，提升学习体验

**计划内容**：
- **N 皇后交互**：滑块调整 N 值（4-12），实时求解并可视化
- **整数规划交互**：拖动约束边界，观察可行域变化
- **DPLL(T) 演示**：单步执行按钮，逐步展示推理过程
- **数独生成器**：难度选择器，生成不同难度的数独谜题

**技术栈**：
```python
import ipywidgets as widgets
from IPython.display import display

@widgets.interact(n=(4, 12, 1))
def solve_n_queens(n=8):
    # 动态求解并可视化
    ...
```

### Phase 4: 深度可视化 (P1 优先级)

**目标**：添加高级可视化，深入展示算法细节

**计划内容**：
- **表格化推理过程**：每个示例都有详细的步骤表格
- **搜索树可视化**：DPLL 决策树、回溯过程
- **进位传播表**：位向量加法器的逐位计算过程
- **量词实例化图**：E-matching 触发模式的匹配过程

### 实施优先级

**当前版本 (v1.0)**：
- ✅ 17 个完整示例
- ✅ 4 篇理论文档
- ✅ 严格线性顺序
- ✅ 无冗余内容

**下一版本 (v2.0 规划)**：
- 🚧 Phase 1 + Phase 2（预计 5-7 小时工作量）
- 🎯 核心目标：深度内容 + 多样性

**未来版本 (v3.0 规划)**：
- 📅 Phase 3 + Phase 4（预计 5-6 小时工作量）
- 🎯 核心目标：交互性 + 细节展示

---

## 🔗 参考资源

### 官方资源
- [cvc5 GitHub](https://github.com/cvc5/cvc5)
- [cvc5 官方文档](https://cvc5.github.io/)
- [cvc5 在线工具](https://cvc5.github.io/app/)

### 学习资料
- [SMT 初学者教程](https://cvc5.github.io/tutorials/beginners/)
- [cvc5 Python API 文档](https://cvc5.github.io/docs/cvc5-1.2.1/api/python/pythonic/quickstart.html)

### 相关工具
- [Z3 Theorem Prover](https://github.com/Z3Prover/z3) - 微软开发的 SMT 求解器
- [Yosys](https://github.com/YosysHQ/yosys) - 开源 Verilog 综合工具（用于 BMC）

---

## 🤝 贡献

本项目是教学演示项目，欢迎：
- 报告代码错误
- 提出改进建议
- 补充更多示例

---

## 📝 许可证

本项目采用 MIT 许可证，可自由用于学习和教学。

---

## 👥 作者

西安电子科技大学 数学逻辑课程项目组

**联系方式**：[待补充]

---

## 🙏 致谢

感谢 cvc5 开发团队提供如此优秀的 SMT 求解器工具。

---

> **提示**：本项目所有代码均经过测试，可直接运行。如遇到问题，请检查 Python 版本（建议 3.8+）和依赖安装。
