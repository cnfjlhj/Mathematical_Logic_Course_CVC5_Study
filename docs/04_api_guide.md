# cvc5 Python API 完整指南

本文档详细介绍 cvc5 的 Python API 使用方法，包括基础 API 和 Pythonic API。

---

## API 概览

cvc5 提供两种 Python API：

| API类型 | 特点 | 适用场景 |
|---------|------|---------|
| **基础 API** | 完整功能，接近 C++ API | 需要精细控制 |
| **Pythonic API** | 简洁优雅，Python 风格 | 快速开发，日常使用 |

**推荐**：大多数情况下使用 **Pythonic API**。

---

## 安装

```bash
pip install cvc5
```

**验证安装**：
```python
import cvc5
print(f"cvc5 version: {cvc5.__version__}")
```

---

## Pythonic API 快速入门

### 1. 导入

```python
from cvc5.pythonic import *
```

### 2. 创建变量

#### 整数变量
```python
x = Int('x')              # 单个变量
x, y, z = Ints('x y z')   # 多个变量
```

#### 实数变量
```python
a = Real('a')
a, b = Reals('a b')
```

#### 布尔变量
```python
p = Bool('p')
p, q, r = Bools('p q r')
```

#### 位向量
```python
bv = BitVec('bv', 8)      # 8 位位向量
```

### 3. 构建约束

#### 算术约束
```python
constraint1 = x + y == 10
constraint2 = x > 0
constraint3 = 2*x + 3*y < 50
```

#### 逻辑约束
```python
constraint4 = And(p, q)
constraint5 = Or(p, Not(q))
constraint6 = Implies(p, q)
```

#### 组合约束
```python
combined = And(x > 0, y > 0, x + y == 10)
```

### 4. 求解

#### 方式 1：快速求解
```python
result = solve(x + y == 10, x > 0, y > 0)
# 自动打印结果
```

#### 方式 2：使用 Solver 对象
```python
solver = Solver()
solver.add(x + y == 10)
solver.add(x > 0, y > 0)

result = solver.check()
if result == sat:
    model = solver.model()
    print(f"x = {model[x]}")
    print(f"y = {model[y]}")
elif result == unsat:
    print("无解")
```

---

## 数据类型详解

### 1. 整数（Int）

```python
x = Int('x')

# 约束
solver.add(x >= 0)
solver.add(x < 100)
solver.add(x % 2 == 0)  # 偶数

# 获取值
if solver.check() == sat:
    val = solver.model()[x].as_long()
```

### 2. 实数（Real）

```python
x = Real('x')

# 约束
solver.add(x > 0.5)
solver.add(x < 1.5)

# 获取值（可能是分数）
if solver.check() == sat:
    val = solver.model()[x]
    print(val)  # 例如：1/2
```

### 3. 布尔（Bool）

```python
p, q = Bools('p q')

# 逻辑运算
solver.add(Or(p, q))
solver.add(Not(And(p, q)))  # XOR

# 获取值
if solver.check() == sat:
    p_val = solver.model()[p]
    print(is_true(p_val))  # True/False
```

### 4. 位向量（BitVec）

```python
x = BitVec('x', 8)  # 8 位

# 位运算
solver.add(x & 0xFF == 0x42)
solver.add(x >> 4 == 4)

# 获取值
if solver.check() == sat:
    val = solver.model()[x].as_long()
```

### 5. 数组（Array）

```python
arr = Array('arr', IntSort(), IntSort())

# 操作
solver.add(Select(arr, 0) == 10)
arr2 = Store(arr, 1, 20)
solver.add(Select(arr2, 1) == 20)
```

### 6. 字符串（String）

```python
s = String('s')

# 操作
solver.add(Length(s) > 3)
solver.add(Concat(s, " world") == "hello world")
solver.add(PrefixOf("hel", s))
```

---

## 高级功能

### 1. 量词

```python
# 存在量词
x = Int('x')
formula = Exists([x], x > 10)

# 全称量词
formula = ForAll([x], Implies(x > 0, x*x > 0))
```

### 2. Distinct 约束

```python
x, y, z = Ints('x y z')

# 所有变量互不相同
solver.add(Distinct(x, y, z))
```

### 3. 条件表达式（If-Then-Else）

```python
x = Int('x')
y = If(x > 0, x + 10, x - 10)

solver.add(y == 20)
# 求解：x = 10 或 x = 30
```

### 4. Push/Pop（状态管理）

```python
solver = Solver()

solver.add(x > 0)
solver.check()  # sat

solver.push()          # 保存状态
solver.add(x < -10)    # 矛盾约束
solver.check()         # unsat
solver.pop()           # 恢复状态

solver.check()         # 又变回 sat
```

### 5. 增量求解

```python
solver = Solver()

# 第 1 次求解
solver.add(x > 0)
solver.check()

# 第 2 次求解（基于第 1 次）
solver.add(x < 10)
solver.check()

# 第 3 次求解
solver.add(x == 5)
solver.check()
```

---

## 逻辑运算符速查

| 运算 | Python 表达式 | 含义 |
|------|--------------|------|
| 否定 | `Not(p)` | ¬p |
| 合取 | `And(p, q)` | p ∧ q |
| 析取 | `Or(p, q)` | p ∨ q |
| 蕴含 | `Implies(p, q)` | p → q |
| 等价 | `p == q` | p ↔ q |
| 异或 | `Xor(p, q)` | p ⊕ q |

---

## 算术运算符速查

| 运算 | 整数 | 实数 | 示例 |
|------|------|------|------|
| 加法 | ✓ | ✓ | `x + y` |
| 减法 | ✓ | ✓ | `x - y` |
| 乘法 | ✓ | ✓ | `x * y` |
| 除法 | ✓ | ✓ | `x / y` |
| 取模 | ✓ | ✗ | `x % y` |
| 乘方 | ✓ | ✓ | `x ** 2` |

---

## 比较运算符速查

| 运算符 | 示例 | 含义 |
|--------|------|------|
| `==` | `x == 10` | 等于 |
| `!=` | `x != 10` | 不等于 |
| `>` | `x > 10` | 大于 |
| `<` | `x < 10` | 小于 |
| `>=` | `x >= 10` | 大于等于 |
| `<=` | `x <= 10` | 小于等于 |

---

## 模型操作

### 获取模型

```python
solver = Solver()
x, y = Ints('x y')
solver.add(x + y == 10, x > 0)

if solver.check() == sat:
    model = solver.model()

    # 方式 1：直接访问
    x_val = model[x]
    print(f"x = {x_val}")

    # 方式 2：转换为 Python 类型
    x_val = model[x].as_long()  # 整数
    print(f"x = {x_val}")

    # 方式 3：求值表达式
    expr_val = model.eval(x + y)
    print(f"x + y = {expr_val}")
```

### 模型评估

```python
model = solver.model()

# 评估表达式
result = model.eval(x * 2 + y)
print(result)

# 评估布尔表达式
result = model.eval(x > 0)
print(is_true(result))
```

---

## 错误处理

### 检查可满足性

```python
result = solver.check()

if result == sat:
    print("可满足")
    model = solver.model()
elif result == unsat:
    print("不可满足")
    # 可选：获取 unsat core
elif result == unknown:
    print("未知（可能超时）")
```

### 超时设置

```python
solver = Solver()
solver.set("timeout", 10000)  # 10 秒

result = solver.check()
if result == unknown:
    print("求解超时")
```

---

## 性能优化技巧

### 1. 设置逻辑

```python
solver = Solver()
solver.set_logic("QF_LIA")  # 无量词线性整数算术
```

**常见逻辑**：
- `QF_LIA`：无量词线性整数算术
- `QF_LRA`：无量词线性实数算术
- `QF_BV`：无量词位向量
- `ALL`：所有理论（最灵活，但可能较慢）

### 2. 简化表达式

```python
# 不好：重复计算
for i in range(100):
    solver.add(x + y + z > i)

# 好：预计算
sum_xyz = x + y + z
for i in range(100):
    solver.add(sum_xyz > i)
```

### 3. 使用 Push/Pop

```python
# 不好：每次创建新求解器
for config in configurations:
    solver = Solver()
    solver.add(base_constraints)
    solver.add(config)
    solver.check()

# 好：重用求解器
solver = Solver()
solver.add(base_constraints)
for config in configurations:
    solver.push()
    solver.add(config)
    solver.check()
    solver.pop()
```

---

## 常见模式

### 模式 1：枚举所有解

```python
solver = Solver()
x, y = Ints('x y')
solver.add(x + y == 10, x >= 0, y >= 0)

solutions = []
while solver.check() == sat:
    model = solver.model()
    x_val = model[x].as_long()
    y_val = model[y].as_long()

    solutions.append((x_val, y_val))

    # 排除这个解
    solver.add(Or(x != x_val, y != y_val))

print(f"找到 {len(solutions)} 个解")
```

### 模式 2：优化（找最小值）

```python
solver = Solver()
x, y = Ints('x y')
solver.add(x > 0, y > 0, x + y < 20)

# 迭代查找最小的 x*x + y*y
best = None
for target in range(1000):
    solver.push()
    solver.add(x*x + y*y == target)

    if solver.check() == sat:
        best = target
        solver.pop()
        break
    solver.pop()

print(f"最小值：{best}")
```

### 模式 3：验证性质

```python
# 验证：对于所有 x > 0，都有 x*x > 0

solver = Solver()
x = Int('x')

# 假设性质不成立
solver.add(x > 0)
solver.add(Not(x*x > 0))

result = solver.check()
if result == unsat:
    print("性质成立！（找不到反例）")
else:
    print("性质不成立，反例：", solver.model())
```

---

## 基础 API vs Pythonic API

### 基础 API 示例

```python
import cvc5

# 创建 TermManager 和 Solver
tm = cvc5.TermManager()
solver = cvc5.Solver(tm)

# 创建类型
int_sort = tm.getIntegerSort()

# 创建变量
x = tm.mkConst(int_sort, 'x')
y = tm.mkConst(int_sort, 'y')

# 创建约束
sum = tm.mkTerm(cvc5.Kind.ADD, x, y)
constraint = tm.mkTerm(cvc5.Kind.EQUAL, sum, tm.mkInteger(10))

# 添加约束
solver.assertFormula(constraint)

# 求解
result = solver.checkSat()
```

### Pythonic API 示例（相同功能）

```python
from cvc5.pythonic import *

x, y = Ints('x y')
solve(x + y == 10)
```

**结论**：Pythonic API 更简洁！

---

## 下一步

- 👉 [查看实际代码示例](../examples/)
- 👉 [学习高级应用](../examples/advanced/)
- 👉 [阅读官方文档](https://cvc5.github.io/)

---

## 参考资料

1. [cvc5 Pythonic API 文档](https://cvc5.github.io/docs/cvc5-1.2.1/api/python/pythonic/pythonic.html)
2. [cvc5 基础 API 文档](https://cvc5.github.io/docs/cvc5-1.2.1/api/python/python.html)
3. [SMT-LIB 标准](http://smtlib.cs.uiowa.edu/)
