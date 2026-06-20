# python19 异常处理与异常组 · 知识点梳理

> 原文档:`python/Article/PythonBasis/python19/1.md`
> 整理对象:`try/except/else/finally`、异常链、自定义异常、`ExceptionGroup`(3.11+)、`add_note`(3.11+)、traceback、warnings、EAFP vs LBYL

---

## 一、异常处理基础

### try/except/else/finally 完整结构

```python
try:
    result = risky_operation()
except ValueError as e:
    print(f"值错误: {e}")
except (TypeError, KeyError):
    print("类型或键错误")
except Exception as e:
    print(f"未知错误: {e}")
else:
    print(f"成功，结果为 {result}")   # 仅在无异常时执行
finally:
    cleanup()                         # 无论如何都执行
```

| 子句 | 执行条件 |
|------|---------|
| `try` | 监控的代码块 |
| `except` | 匹配到指定异常时执行 |
| `else` | try 块无异常时执行(不应放在 try 中以避免被捕获) |
| `finally` | 无论是否有异常都执行(清理资源) |

## 二、异常层次结构(核心分支)

```
BaseException
 ├── KeyboardInterrupt      (Ctrl+C)
 ├── SystemExit              (sys.exit())
 ├── GeneratorExit           (生成器关闭)
 └── Exception
      ├── ValueError
      ├── TypeError
      ├── KeyError
      ├── IndexError
      ├── AttributeError
      ├── RuntimeError
      ├── OSError
      │    ├── FileNotFoundError
      │    ├── PermissionError
      │    └── ...
      ├── ImportError
      │    └── ModuleNotFoundError   (3.6+)
      ├── StopIteration
      ├── StopAsyncIteration
      └── ... (更多子类)
```

> ⚠️ **关键规则**:自定义异常应继承 `Exception`(不继承 `BaseException`)，否则不会被 `except Exception` 捕获，也不会被大部分框架正确追踪。

## 三、异常链 `raise ... from`

```python
try:
    value = int("abc")
except ValueError as e:
    raise RuntimeError("转换失败") from e

# 输出两个 traceback:
#   RuntimeError: 转换失败
#   The above exception was the direct cause of the following exception:
#   ValueError: invalid literal for int() with base 10: 'abc'

# 禁止链:
raise RuntimeError("转换失败") from None   # 隐藏原始异常
```

`__cause__`:显式 `from e` 设置该属性。
`__context__`:隐式(在 except 块中 raise 新异常时自动设置)。

## 四、自定义异常

```python
class OrderError(Exception):
    """订单相关错误基类"""
    def __init__(self, message: str, order_id: str):
        super().__init__(message)
        self.order_id = order_id

class InsufficientStockError(OrderError):
    """库存不足"""

raise InsufficientStockError("库存不足", order_id="ORD-001")
```

## 五、ExceptionGroup 与 `except*`(Python 3.11+)

```python
# 同时抛出多个异常
def validate(data):
    errors = []
    if "name" not in data:
        errors.append(ValueError("缺少 name"))
    if data.get("age", 0) < 0:
        errors.append(ValueError("age 不能为负"))
    if errors:
        raise ExceptionGroup("验证失败", errors)

# 捕获和处理异常组
try:
    validate({"age": -5})
except* ValueError as eg:
    for exc in eg.exceptions:
        print(f"→ {exc}")
```

**`ExceptionGroup` 是树状结构**:一个 `ExceptionGroup` 可以嵌套另一个 `ExceptionGroup`。

```python
ExceptionGroup("顶层", [
    ValueError("a"),
    ExceptionGroup("子组", [
        TypeError("b"),
        ValueError("c"),
    ])
])
```

`except* ValueError` 会提取树中 **所有** `ValueError`(包括嵌套在内层组中的)，汇聚到一个新的 `ExceptionGroup`。

## 六、`add_note()`(Python 3.11+)

```python
try:
    value = int(user_input)
except ValueError as e:
    e.add_note(f"用户输入: {user_input!r}")
    e.add_note("应该在调用前做格式校验")
    raise
```

`add_note` 是给异常附加上下文信息的标准方式，会追加到 traceback 中。替代了之前 `raise ... from` 或字符串拼接的 hack 做法。

## 七、常用辅助

### assert vs raise

```python
# assert — 可被 python -O 跳过，仅用于调试/不该发生的情况
assert x > 0, "x 必须大于 0"

# raise — 生产代码的输入校验必须用它
if x <= 0:
    raise ValueError("x 必须大于 0")
```

### traceback 模块

```python
import traceback
traceback.print_exc()           # 打印当前异常到 stderr
traceback.format_exc()          # 返回 str
```

### logging

```python
import logging
logger = logging.getLogger(__name__)

try:
    ...
except Exception:
    logger.exception("操作失败")   # 自动附带 traceback
```

### warnings

```python
import warnings
warnings.warn("此函数将废弃", DeprecationWarning)
```

## 八、EAFP vs LBYL

| 风格 | 全称 | 含义 | 示例 |
|------|------|------|------|
| **EAFP** | Easier to Ask Forgiveness than Permission | 先尝试，出错再处理 | `try: d["key"] except KeyError: ...` |
| **LBYL** | Look Before You Leap | 先检查条件再执行 | `if "key" in d: d["key"]` |

Python 推崇 **EAFP**，原因是:
1. 减少竞态条件(多线程/多进程下检查与执行之间可能变化)
2. 主流程不受检查代码干扰
3. 异常在 Python 中成本可控

> ⚠️ 原文差异1
> 原文未明确区分 `BaseException` 与 `Exception` 的继承差异。自定义异常务必继承 `Exception`(不继承 `BaseException`)，否则 `except Exception` 不会捕获。

---

## 九、异常执行流(ASCII 图)

```
try/except/else/finally 执行流:

    进入 try 块
        │
        ├── 无异常 ──→ 执行 else 块 ──→ 执行 finally 块 ──→ 继续
        │
        ├── 有异常 ──→ 匹配 except? ──┬── 是 ──→ 执行匹配的 except 块 ──→ 执行 finally 块 ──→ 继续
        │                              │
        │                              └── 否 ──→ 执行 finally 块 ──→ 向上层传播异常
        │
        └── return/break/continue 在 try/else 中
                │
                └── finally 仍会执行！(finally 的 return 会覆盖前面的)

注意: finally 中的 return 会吞掉任何未处理的异常——这是一个常见陷阱。
```

```
ExceptionGroup 处理流 (3.11+):

    raise ExceptionGroup("root", [ValueError, TypeError, ExceptionGroup([ValueError])])
        │
        ▼
    except* ValueError as eg:
        │
        ├── 从异常树中提取所有 ValueError(含嵌套)
        ├── 打包为新的 ExceptionGroup 绑定到 eg
        └── 未匹配的 TypeError 继续向上传播
```

---

## 十、与原文档差异速查

| 编号 | 出处 | 原文描述 | 正确说明 |
|------|------|----------|----------|
| 1 | 1.md | 未区分 `BaseException` vs `Exception` | 自定义异常必须继承 `Exception` 而非 `BaseException` |
| 2 | 1.md | 缺少 `ExceptionGroup` 和 `except*` | 3.11+ 核心新特性，并发场景(如 TaskGroup)批量收集异常的关键工具 |
| 3 | 1.md | 缺少 `add_note()` | 3.11+ 给异常附加上下文的官方方法 |
| 4 | 1.md | 未明确 `except Exception` 和裸 `except:` 的区别 | 裸 `except:` 捕获 `BaseException`(含 KeyboardInterrupt/SystemExit)，几乎永远不该用 |
| 5 | 1.md | 未提 finally 中的 return 会吞异常 | 这是一个难以调试的陷阱 |
