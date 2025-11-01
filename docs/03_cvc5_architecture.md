# cvc5 架构与求解原理

## cvc5 的核心架构

cvc5 采用 **DPLL(T)** 架构，这是现代 SMT 求解器的标准框架。

### 整体架构图

```
┌─────────────────────────────────────────────────────┐
│                     用户接口                          │
│  - SMT-LIB 2 语言                                    │
│  - Python API (基础 / Pythonic)                      │
│  - C++ API                                          │
└──────────────────┬──────────────────────────────────┘
                   ↓
┌─────────────────────────────────────────────────────┐
│              前端处理（Frontend）                      │
│  - 语法解析                                          │
│  - 类型检查                                          │
│  - 公式简化                                          │
└──────────────────┬──────────────────────────────────┘
                   ↓
┌─────────────────────────────────────────────────────┐
│           SAT 求解器核心（CDCL）                       │
│  - 决策（Decision）                                   │
│  - 单元传播（Unit Propagation）                       │
│  - 冲突分析（Conflict Analysis）                      │
│  - 学习（Clause Learning）                           │
│  - 回溯（Backtracking）                              │
└──────────────────┬──────────────────────────────────┘
                   ↓
           ┌───────┴────────┐
           ↓                ↓
┌───────────────────┐  ┌──────────────────┐
│   理论求解器        │  │   理论组合        │
│ (Theory Solvers)  │  │  (Theory Combo)  │
├───────────────────┤  └──────────────────┘
│ • 算术 (Arith)    │
│ • 数组 (Arrays)   │
│ • 位向量 (BV)      │
│ • 字符串 (Strings)│
│ • 数据类型 (DT)    │
│ • 量词 (Quant)    │
└───────────────────┘
```

---

## CDCL(T) 算法详解

**CDCL** = Conflict-Driven Clause Learning with Theories

### 算法流程

```
1. 初始化
   ├─ 加载公式
   └─ 初始化理论求解器

2. 主循环：
   │
   ├─ 决策（Decision）
   │  选择一个未赋值的布尔变量并赋值
   │
   ├─ 单元传播（BCP）
   │  根据当前赋值推导出必然的赋值
   │
   ├─ 理论检查（Theory Propagation）
   │  └─ 将当前赋值传递给理论求解器
   │      └─ 理论求解器检查一致性
   │
   ├─ 冲突？
   │  ├─ 否 → 继续（回到 2）
   │  └─ 是 ↓
   │
   ├─ 冲突分析
   │  ├─ 分析冲突原因
   │  ├─ 学习冲突子句（Conflict Clause）
   │  └─ 计算回溯层级
   │
   └─ 回溯（Backtrack）
      └─ 回到之前的状态，继续搜索

3. 结果：
   ├─ 所有变量都赋值 → SAT + Model
   └─ 冲突无法解决 → UNSAT + Core
```

---

## 理论求解器

每个理论求解器负责处理特定类型的约束。

### 1. 算术求解器（Arithmetic Solver）

**支持的理论**：
- **LIA** (Linear Integer Arithmetic)：线性整数算术
- **LRA** (Linear Real Arithmetic)：线性实数算术
- **NIA/NRA**：非线性算术

**核心算法**：
- **Simplex 算法**：处理线性约束
- **切平面法**（Cutting Planes）：处理整数约束
- **Gröbner 基**：处理非线性约束

**示例约束**：
```python
x + 2y <= 10
3x - y >= 5
x, y ∈ ℤ  # 整数域
```

---

### 2. 数组求解器（Array Solver）

**核心理论**：
```
• select(arr, i)：读取数组 arr 的第 i 个元素
• store(arr, i, v)：将数组 arr 的第 i 个元素设为 v

公理：
• select(store(arr, i, v), i) = v
• i ≠ j → select(store(arr, i, v), j) = select(arr, j)
```

**应用**：
- 程序验证中的数组操作
- 内存模型

---

### 3. 位向量求解器（Bitvector Solver）

**支持操作**：
- 位运算：`&`, `|`, `^`, `~`
- 移位：`<<`, `>>`
- 算术：`+`, `-`, `*`, `/`（有符号/无符号）

**应用**：
- 硬件电路验证
- 底层代码验证

---

### 4. 字符串求解器（String Solver）

**支持操作**：
- 拼接：`concat(s1, s2)`
- 长度：`length(s)`
- 子串：`substring(s, start, end)`
- 正则匹配：`str.in.re(s, re)`

**应用**：
- Web 安全（XSS、SQL 注入检测）
- 输入验证

---

### 5. 量词求解器（Quantifier Solver）

**技术**：
- **E-matching**：模式匹配实例化
- **触发器（Triggers）**：控制实例化
- **MBQI**（Model-Based Quantifier Instantiation）：基于模型的实例化

**挑战**：
- 量词求解是不可判定的
- 需要启发式方法

---

## 优化技术

### 1. 预处理（Preprocessing）

在求解前简化公式：

```python
# 示例：常量传播
x = 3 ∧ y = x + 2  →  x = 3 ∧ y = 5

# 示例：子句化
(a ∨ b) ∧ (c ∨ d)  →  (a ∨ b), (c ∨ d)
```

### 2. 增量求解（Incremental Solving）

支持逐步添加约束：

```python
solver = Solver()
solver.add(x > 0)       # 第 1 步
solver.check()          # SAT

solver.add(x < 10)      # 第 2 步（增量）
solver.check()          # SAT

solver.add(x > 20)      # 第 3 步（矛盾）
solver.check()          # UNSAT
```

**优势**：
- 重用之前的求解结果
- 避免从头开始

### 3. Push/Pop 机制

类似栈的操作：

```python
solver.push()           # 保存当前状态
solver.add(x > 100)     # 临时约束
solver.check()          # SAT/UNSAT
solver.pop()            # 恢复到 push 前
```

**应用**：
- 有界模型检查（BMC）
- 符号执行

---

## cvc5 的独特优势

### 1. 强大的字符串理论支持

cvc5 是字符串约束求解的领导者：
- 支持复杂的正则表达式
- 高效的字符串求解算法

### 2. 证明生成

cvc5 可以生成**形式化证明**：
```bash
cvc5 --dump-proofs=full input.smt2
```

**应用**：
- 定理证明
- 可信验证

### 3. 模型生成

SAT 时，cvc5 提供满足所有约束的**模型**：
```python
result = solver.check()
if result == sat:
    model = solver.model()
    print(model)
```

### 4. UNSAT Core

UNSAT 时，cvc5 可以返回**最小冲突集**：
```python
result = solver.check()
if result == unsat:
    core = solver.get_unsat_core()
    print("冲突的约束：", core)
```

---

## 性能调优

### 设置逻辑（Logic）

指定逻辑可以加速求解：

```python
solver.set_logic("QF_LIA")  # 无量词线性整数算术
```

**常见逻辑**：
- `QF_LIA`：无量词线性整数算术
- `QF_UFLIA`：无量词线性整数算术 + 未解释函数
- `ALL`：支持所有理论（但可能较慢）

### 超时设置

```python
solver.set("timeout", 10000)  # 10 秒超时
```

### 详细输出

```python
solver.set("verbosity", 3)  # 详细日志
```

---

## cvc5 vs Z3

| 特性 | cvc5 | Z3 |
|------|------|----|-
| **字符串理论** | ⭐⭐⭐⭐⭐ 强大 | ⭐⭐⭐⭐ 良好 |
| **量词处理** | ⭐⭐⭐⭐⭐ 优秀 | ⭐⭐⭐⭐⭐ 优秀 |
| **证明生成** | ⭐⭐⭐⭐⭐ 完整 | ⭐⭐⭐⭐ 部分 |
| **工业应用** | 形式化验证 | 广泛 |
| **社区活跃度** | ⭐⭐⭐⭐ 高 | ⭐⭐⭐⭐⭐ 极高 |

**选择建议**：
- **字符串约束**：优先 cvc5
- **一般用途**：cvc5 和 Z3 都很好
- **生态系统**：Z3 生态更丰富

---

## 实际应用案例

### 案例 1：硬件验证

cvc5 用于验证 Intel、AMD 的芯片设计：
- 验证浮点运算单元
- 验证缓存一致性协议

### 案例 2：软件验证

- **CBMC**：C 程序的有界模型检查器（使用 cvc5）
- **KLEE**：符号执行工具

### 案例 3：Web 安全

- 检测 XSS 漏洞
- SQL 注入检测

---

## 下一步

- 👉 [学习 Python API 完整指南](04_api_guide.md)
- 👉 [查看实际应用示例](../examples/advanced/)

---

## 参考资料

1. Barbosa et al. "cvc5: A Versatile and Industrial-Strength SMT Solver." TACAS 2022.
2. [cvc5 官方文档](https://cvc5.github.io/)
3. Barrett, Tinelli. "Satisfiability Modulo Theories." Handbook of Model Checking, 2018.
