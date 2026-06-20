# python4 Dict 和 Set 知识点梳理

> 原文档:`python/Article/PythonBasis/python4/Dict.md`、`Set.md`
> 整理对象:字典 dict 与 集合 set 的本质、操作、性能特性

---

## 一、Dict 字典

### 1. 本质

- 键值对(key-value)的可变映射,**Python 3.7 起保留插入顺序**(语言级保证)。
- 键必须可哈希(`__hash__` 不为 `None` 且实现了 `__eq__`),所以 list、dict、set 不能作键;tuple、frozenset 元素全部可哈希时可作键。
- 值无任何限制。
- 平均查找/插入复杂度 **O(1)**(基于哈希表),代价是占用更多内存。

### 2. 创建方式

```python
d1 = {'a': 1, 'b': 2}                   # 字面值
d2 = dict(a=1, b=2)                     # 关键字
d3 = dict([('a', 1), ('b', 2)])         # 键值对序列
d4 = dict.fromkeys(['a', 'b'], 0)       # 同一默认值
d5 = {k: v for k, v in zip('ab', [1,2])}# 字典推导
```

### 3. 读、写、删

```python
d = {'a': 1}

# 读
d['a']                  # KeyError 如果不存在
d.get('x')              # 不存在返回 None,不抛错
d.get('x', 0)           # 不存在返回默认值
d.setdefault('x', [])   # 不存在则写入并返回该默认值
d['x']                  # 现在能拿到 []

# 写/改
d['b'] = 2
d.update({'c': 3, 'a': 99})    # 批量改

# 删
del d['a']
d.pop('a', None)        # 不存在不抛错
d.popitem()             # 删除并返回最后插入的键值对(3.7+)
d.clear()
```

### 4. 遍历

```python
for k in d: ...              # 遍历 key,等价 d.keys()
for v in d.values(): ...     # 遍历 value
for k, v in d.items(): ...   # 同时拿 key 和 value
```

> 注意:`d.keys() / d.values() / d.items()` 返回 **视图对象(view)**,不是 list。视图随 d 变化而变化。需要列表时显式 `list(d.keys())`。

### 5. dict 合并(Python 3.9+ `|` / `|=`)

```python
a = {'x': 1, 'y': 2}
b = {'y': 99, 'z': 3}

c = a | b           # 返回新 dict:{'x':1,'y':99,'z':3}    右覆盖左
a |= b              # 原地更新 a
```

3.9 之前用 `{**a, **b}`(等价)或 `a.update(b)`(原地)。

### 6. 与原文档差异

> ⚠️ 原文错误一(`Dict.md` 第 168 行)
> 原文写"dict 内部存放的顺序和 key 放入的顺序是没有任何关系"。
> **正确说法**:
> - CPython 3.6 起以实现细节方式保留插入顺序;
> - **Python 3.7 起作为语言级规范**(PEP 468 / 排序协议)正式保证。
> 所以 **现代 Python 普通 dict 就是有序的**,`collections.OrderedDict` 现在主要用其专属的 `.move_to_end()` 等方法。

> ⚠️ 原文错误二(`Dict.md` 第 189 行)
> 原文 `str(dict)|输出字典可打印的字符串表示` 放在了 dict 专属方法表,容易让人误以为是 `dict.str()` 方法。
> **正确说法**:`str()` / `len()` / `type()` 都是 **内建函数**,不是 dict 的方法。建议在文档里区分"内建函数 / 字典方法"两类。

> ⚠️ 原文错误三(`Dict.md` 第 192 行)
> 原文 `dict.values()|以列表返回字典中的所有值`。
> **正确说法**:`dict.values()` 返回 **dict_values 视图对象**,不是 list。需要列表要 `list(d.values())`。

> ⚠️ 原文错误四(`Dict.md` 第 193 行)
> 原文 `popitem()|随机返回并删除字典中的一对键和值`。
> **正确说法**:Python 3.7+ 起 `popitem()` **按 LIFO(后进先出)弹出**,不是"随机"。这条在 3.6 之前是"随机"的描述,3.7+ 已修订。

---

## 二、Set 集合

### 1. 本质

- 无序、不重复、元素必须可哈希。
- 基于哈希表,`in` 检测 **O(1) 平均**。
- 字面值:`{1, 2, 3}`;**空集合必须写 `set()`**,**写 `{}` 会得到空 dict**。

### 2. 创建

```python
s1 = {1, 2, 3}
s2 = set([1, 2, 2, 3])      # → {1,2,3} 自动去重
s3 = set('hello')           # → {'h','e','l','o'} 接收任意 iterable
s4 = {x*x for x in range(5)}# 集合推导
empty = set()               # ⚠️ 不能用 {} 表空集合
```

### 3. 增删

```python
s = {1, 2, 3}
s.add(4)
s.update([5, 6])            # 批量加
s.remove(2)                 # 不存在抛 KeyError
s.discard(99)               # 不存在不抛错
x = s.pop()                 # 随机弹出一个(集合无序)
s.clear()
```

### 4. 集合运算(数学语义)

| 写法 | 含义 | 等价方法 |
|------|------|----------|
| `a \| b` | 并集 | `a.union(b)` |
| `a & b` | 交集 | `a.intersection(b)` |
| `a - b` | 差集 A\B | `a.difference(b)` |
| `a ^ b` | 对称差(异或) | `a.symmetric_difference(b)` |
| `a <= b` | A 是 B 的子集 | `a.issubset(b)` |
| `a < b` | A 是 B 的真子集 | — |
| `a >= b` | 超集 | `a.issuperset(b)` |
| `a.isdisjoint(b)` | 是否无交集 | — |

带 `=` 版本均原地修改:`a |= b`、`a &= b`、`a -= b`、`a ^= b`。

### 5. frozenset 不可变集合

- 创建后不可增删。
- **可哈希**,因此能作为 dict 的 key、set 的元素。
- 集合运算返回普通 set(只读输入 → 普通输出,这点要留意)。

```python
fs = frozenset([1, 2, 3])
config = {frozenset({'admin', 'editor'}): '后台权限'}
config[frozenset({'editor', 'admin'})]   # 相等(set 无序),命中同一 key
```

### 6. 何时用 set 而不是 list

- 频繁 `in` 检测 → 用 set / dict(O(1) vs O(n))。
- 仅去重不在乎顺序 → `set(lst)`。
- 既要去重又要保留顺序 → `list(dict.fromkeys(lst))`(3.7+ 利用 dict 有序性)。

---

## 三、dict / set / list / tuple 总览

| 维度 | list | tuple | set | dict |
|------|------|-------|-----|------|
| 字面值 | `[ ]` | `( )` 或裸逗号 | `{x,y}` / `set()` | `{k:v}` / `dict()` |
| 可变 | ✅ | ❌ | ✅ | ✅ |
| 有序 | ✅ | ✅ | ❌ | ✅(3.7+) |
| 允许重复 | ✅ | ✅ | ❌ | key 唯一 |
| 元素可哈希? | 不要求 | 不要求 | **要求** | **key 要求** |
| 索引 | `lst[i]` O(1) | `t[i]` O(1) | 无 | `d[k]` O(1) 平均 |
| `in` 复杂度 | O(n) | O(n) | O(1) | O(1) |

---

## 四、与原文档差异速查

| 编号 | 出处 | 原文描述 | 正确说明 |
|------|------|----------|----------|
| 1 | Dict.md §6 | dict 内部顺序与 key 插入顺序无关 | Python 3.7+ 保证保留插入顺序 |
| 2 | Dict.md §7 | `str(dict)` 列入"字典方法" | `str/len/type` 是内建函数,非 dict 方法 |
| 3 | Dict.md §7 | `dict.values()` 返回"列表" | 返回 `dict_values` 视图对象 |
| 4 | Dict.md §7 | `popitem()` 随机弹出 | 3.7+ 起 LIFO 弹出(后进先出) |

> 本章无多文件调用,流程图不适用。set/dict 的"哈希查找"原理后续在 python10 Magic Method 章节(`__hash__/__eq__`)再画。
