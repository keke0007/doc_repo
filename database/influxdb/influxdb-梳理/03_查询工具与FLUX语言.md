# 03 查询工具与 FLUX 语言

> 覆盖原文档:第 3.2(Data Explorer / Notebook)、第 4 章 FLUX 语法、第 5 章 数据类型、
> 第 6 章 文档使用、第 7 章 FLUX 查询 InfluxDB、第 18 章 FLUX 查询优化
> 前置:已读 `01_概念与架构.md`(尤其 Series / Tag / Field 概念)

---

## 一、Data Explorer:FLUX 的轻量 IDE

### 1.1 页面分区

```
┌────────────────────── Data Explorer ──────────────────────┐
│                                                            │
│   ┌──────────────── 数据预览区 ────────────────┐          │
│   │  图表(折线/柱/饼/Table/Raw Data ...)        │          │
│   └─────────────────────────────────────────────┘         │
│                                                            │
│   ┌──────────────── 查询编辑区 ────────────────┐          │
│   │ [Query Builder] / [Script Editor] 二选一    │          │
│   │ FROM bucket → Filter → Aggregate Window     │          │
│   └─────────────────────────────────────────────┘         │
│                                                            │
│   右上角:[时间范围][SAVE AS][SUBMIT]                      │
└────────────────────────────────────────────────────────────┘
```

### 1.2 两种编辑模式

| 模式 | 本质 | 适用 |
| --- | --- | --- |
| **Query Builder(查询构造器)** | 点击表单 → UI 后台**自动拼一段 FLUX 提交** | 快速探索,90% 的简单查询 |
| **Script Editor(脚本编辑器)** | 手写 FLUX,带补全和函数文档悬浮 | 自定义复杂查询、调用函数包 |

> 💡 在 Builder 模式下点击 `SCRIPT EDITOR` 可看到生成的 FLUX,**这是学 FLUX 最好的入门方式**。

### 1.3 默认行为(容易踩坑)

- **默认窗口聚合**:Builder 一定会带 `aggregateWindow()`,窗口大小由时间范围自动选(1h → 10s 窗)
- **默认聚合函数**:`mean()`(平均值)
- **可视化字段**:默认按返回数据中的 `_value` 列绘图;**如果你的查询输出列不叫 `_value`,折线图会"空白但不报错"**

### 1.4 Data Explorer 的其他出口

```
查询出结果后,右上角 SAVE AS / 其他菜单可触发:
   ├─ Export to CSV(直接下载)
   ├─ Save as Dashboard Cell(挂到仪表盘)
   ├─ Save as Task(转成定时任务,见 第13章)
   └─ Save as Variable(全局变量:Map / CSV / FLUX 三种)
```

---

## 二、Notebook:可拆步骤的工作流

### 2.1 Notebook vs Data Explorer

| 维度 | Data Explorer | Notebook |
| --- | --- | --- |
| 风格 | 一锤子查询 | 多 Cell 流水线,模仿 Jupyter |
| Cell 概念 | 无 | 有,**可调换顺序、可插入** |
| 创建报警(Alert) | ❌ 不支持 | ✅ 支持 |
| 写回 Bucket | ❌ | ✅ |
| 文档(Markdown) | ❌ | ✅ Note Cell 支持 Markdown |

### 2.2 Cell 类型

```
Notebook Cells:
├─ Data Source(数据源)
│   ├─ Query Builder
│   └─ FLUX Script
├─ Visualization(可视化)
│   ├─ Table
│   ├─ Graph
│   └─ Note (Markdown 说明)
└─ Action(行为)
    ├─ Set Alert(报警)
    ├─ Schedule Task(定时任务)
    └─ Output to Bucket(写回存储桶)
```

### 2.3 工作流范式

```
[Query Cell] ──▶ [Visualization Cell] ──▶ [FLUX Cell 加工] ──▶ [Action Cell]
                                          ↑
                          通过 __PREVIOUS_RESULT__ 接前一段输出
```

> 💡 Notebook 中的 FLUX cell 隐式接收上一个 cell 的输出作为输入流,因此后面的 cell 可以直接对前一段查询结果继续 pipe。

---

## 三、FLUX 语言总览

### 3.1 心智模型:数据流处理

```
┌──────────┐    ┌──────────┐    ┌──────────┐    ┌──────────┐
│   from   │───▶│  range   │───▶│  filter  │───▶│ aggregate│
│ (数据源) │ |> │ (时间范围)│ |> │ (维度过滤)│ |> │ /map/... │
└──────────┘    └──────────┘    └──────────┘    └──────────┘
                                                       │
                                                       ▼
                                                ┌──────────┐
                                                │  yield   │
                                                │ (返回结果)│
                                                └──────────┘
```

- **管道符** `|>` — 把左侧表流(table stream)送进右侧函数
- **数据流** — 多个 series(表)在管道里以 **table stream** 流动
- **结果必须是表流** — FLUX 是查询语言,顶层必须返回表流,**纯标量(数字/字符串)需要用 `array.from` 包装**

### 3.2 三种数据源同构示例

```python
# 从 InfluxDB
from(bucket: "example-bucket")
  |> range(start: -1d)
  |> filter(fn: (r) => r._measurement == "cpu")
  |> mean()
  |> yield(name: "_results")

# 从 CSV 文件
import "csv"
csv.from(file: "data.csv")
  |> range(start: -1d)
  |> filter(fn: (r) => r._measurement == "cpu")
  |> mean()
  |> yield(name: "_results")

# 从 PostgreSQL
import "sql"
sql.from(
    driverName: "postgres",
    dataSourceName: "postgresql://user:pwd@localhost",
    query: "SELECT * FROM TestTable",
)
  |> filter(fn: (r) => r.UserID == "123ABC")
  |> mean(column: "purchase_total")
```

> ⚠️ 原文 4.2 写"上面 3 个示例用的函数都是一模一样的"。**严格说不是**:
> - `csv.from / sql.from` 的输出**没有 `_time / _measurement` 这种 InfluxDB 约定列**,直接对 SQL 结果 `range()` 会因没有 `_time` 列而**报错**
> - 范例只是表达"FLUX 想统一所有数据源",**实际跨源用法要做更多列预处理**

### 3.3 版本绑定

`influxd` 是单一二进制,**FLUX 解释器随它一起编译**。所以 InfluxDB 版本和 FLUX 版本是**强绑定**的(见原文表格):

| InfluxDB OSS | FLUX 版本 |
| --- | --- |
| 2.0 | 0.131.0 |
| 2.1 | 0.139.0 |
| 2.2 | 0.162.0 |
| 2.3 | 0.171.0 |
| 2.4 | 0.179.0 |

升级 InfluxDB 才能拿到新版 FLUX 的新函数。

---

## 四、FLUX 基本语法

### 4.1 注释

**只有单行注释** `// ...`,**没有多行注释**。

### 4.2 变量与基本运算

```
s = "foo"      // string
i = 1          // integer
f = 2.0        // float

1 + 1          // 2
10 * 3         // 30
"a" + "b"      // 字符串拼接
(12.0 + 18.0) / (2.0 ^ 2.0) + (240.0 % 55.0)   // 27.5
```

支持 `+ - * / % ^`。

### 4.3 谓词与逻辑

| 运算符 | 用途 |
| --- | --- |
| `==` `!=` `<` `<=` `>` `>=` | 标量比较 |
| `=~` `!~` | **正则**匹配/不匹配(右侧是 regex) |
| `and` `or` `not` | 逻辑(**不是 `&&` / `\|\|`!**) |

```flux
"abcdefg" =~ /abc|bcd/   // true
not (a and b)            // 取反
```

### 4.4 控制语句:**只有三元式条件**

```flux
x = 0
y = if x == 0 then "hello" else "world"
```

**没有 `for / while / try-catch`**,循环靠 map / reduce / 数组函数代替。

---

## 五、FLUX 的数据类型

### 5.1 10 个基本类型

| 类型 | 字面量例子 | 转换函数 | 备注 |
| --- | --- | --- | --- |
| boolean | `true`, `false` | `bool(v: ...)` | 0/1、"true"/"false" 可转 |
| bytes | (无字面量) | `bytes(v: "hi")` | 输出 `[104 101 ...]` |
| **duration** | `1h30m`, `3d12h4m25s` | `duration(v: ...)` | ns/us/ms/s/m/h/d/w/**mo**/**y** |
| **regex** | `/^foo/` | `regexp.compile(...)` | 直接是语言内建类型 |
| string | `"abc"`, `"日本語"` | `string(v: ...)` | 支持 `\x` 十六进制转义 |
| **time** | `2020-01-01T19:22:31Z` | (无字面量构造函数,需 RFC3339) | 纳秒精度 |
| float | `1.23`, `-0.5` | `float(v: ...)` | 科学计数法、Inf、NaN 都靠 `float(v: "...")` |
| int | `1`, `-2` | `int(v: ...)` | int64 范围 |
| uint | (无字面量) | `uint(v: ...)` | 负数会发生整数环绕 |
| null | (无字面量) | `debug.null(type: ...)` | 用 `exists x` 判断非空 |

> ⚠️ 原文 5.1.3.1 漏了一个易错点:**`mo`(月)和 `y`(年)是日历持续时间**,与"30 天 / 365 天"**不等价**,做时间加减时会按日历推进。生产代码里要明确语义。

> ⚠️ 原文 5.1.3.3 写"使用 `int()` 或 `unit()`"以及 5.1.3.5 写"使用 `date.add()` 从时间中减去持续时间" — **应为 `uint()` 和 `date.sub()`**,前者是中文笔误,后者是函数名错。

> ⚠️ 原文 5.1.3.1 注释里说"`01m` 解析为整数 0 和 1 分钟" — 准确说 **FLUX 解析器会将前导 0 作为独立 int 字面量,导致语法歧义/报错**,书写时**不要带前导 0**。

> ⚠️ 原文 5.1.7.4 标题"Not a Number"对应的代码块里第二行 `NaN` 是错误示意,正确做法是 `float(v: "NaN")`。

### 5.2 类型转换"双向表"

```
   string  ◀──string()──▶  bool / int / uint / float / duration / bytes / time
   int     ◀──int()─────▶  string / bool / float / duration / time(返回 ns 时间戳)
   uint    ◀──uint()────▶  同上(负值会环绕)
   float   ◀──float()───▶  string / bool / int / uint
   bool    ◀──bool()────▶  string("true"/"false") / 0|1 / 0.0|1.0
   duration◀─duration()─▶  string("1h30m") / int(纳秒)
   time    ◀─time()─────▶  string(RFC3339)
   bytes   ◀─bytes()────▶  仅 string 可来回
   regex   ◀─regexp.compile()─▶  string
```

### 5.3 4 个复合类型

#### 5.3.1 Record(记录)— 类似 JSON 对象

```flux
c = {name: "John Doe", id: 1123445}
c.name           // 点表示法
c["id"]          // 中括号(key 带空格时必须用)

// with 扩展(返回新 record,旧 record 不变)
{c with name: "Xiao Ming", pet: "Spot"}

// 比较两个 record
{a:1, b:2} == {b:2, a:1}   // true(key 顺序无关)
```

**关键限制**:
- key 是**静态**的:访问不存在的 key **直接抛异常**,不是返回 null
- **嵌套 record 不能放进表流的列里**(列不能是 record 类型);嵌套 record 主要用于 `http.post` / `json.encode`

#### 5.3.2 Array(数组)— 同类型有序序列

```flux
arr = ["one", "two", "three"]
arr[0]                              // "one"
contains(value: "Joe", set: names)  // 是否包含
```

> ⚠️ 原文 5.3.2.2 示例代码注释写错:`arr[2]` 注释为 `Returns two`,**实际应为 `Returns three`**(数组下标 0 起算)。

#### 5.3.3 Dictionary(字典)— **所有 key 同类型,所有 value 同类型**

```flux
import "dict"

positions = ["Manager": "Jane", "Clerk": "John"]
dict.get(dict: positions, key: "Manager", default: "Unknown")  // "Jane"
dict.insert(dict: positions, key: "k3", value: "v3")
dict.remove(dict: positions, key: "Clerk")
```

| 维度 | Record | Dictionary |
| --- | --- | --- |
| key 类型 | 任意,但必须是合法标识符或带引号字符串 | **统一同一种基本类型**(全 string / 全 int) |
| value 类型 | 任意,**每个 key 可不同** | **全部同类型** |
| key 是否动态 | **静态**,编译期固定 | **动态**,运行期可增删 |
| 取值 | `r.k` / `r["k"]` | `dict.get(...)` |

#### 5.3.4 Function(函数)— 一等公民

```flux
// 命名函数
square = (n) => n * n
square(n: 3)              // 9

// 默认参数
chengfa = (a, b=100) => a * b
chengfa(a: 3)             // 300  ← 注意:b 默认 100,300=3*100

// 没有位置参数,调用必须用 name:value
```

> ⚠️ 原文 5.3.4.2 示例 `chengfa(a:3) // Returns 300` 写法正确,**讲义解释里写"默认值"应明确这是关键字参数 + 默认值机制**(不是 Java/Python 的可选参数语法糖)。

### 5.4 FLUX 类型 ≠ InfluxDB 字段类型

**Duration 和 Regex 是 FLUX 独有的**,InfluxDB **不能存这两种类型**作为 field。

InfluxDB field 实际只支持:**float / int / uint / string / boolean**(行协议层面,见附录 2 笔记)。

---

## 六、函数包与文档使用

### 6.1 标准库布局

```
FLUX 标准库:
├─ universe (默认加载,无需 import)
│   from / range / filter / map / mean / aggregateWindow ...
├─ date            日期函数
├─ regexp          正则
├─ json / http     I/O 和序列化
├─ array / dict    集合
├─ math            数学
├─ csv / sql       外部数据源
├─ profiler        查询性能分析
└─ experimental    ← 实验性,生产慎用,API 不稳定
```

**自定义函数可写,但不能自定义包**(截至 2.4)。要复用必须改 InfluxDB 源码。

### 6.2 文档使用三要点

1. 文档地址 `https://docs.influxdata.com/flux/v0.x/`
2. **每个函数标题正下方标注最早可用版本**(如 `Available since: 0.173`)
3. **避免 `experimental` 包**:其中函数可能在后续版本被改名或删除

### 6.3 如何判断"管道函数"还是"普通函数"

打开函数文档,看 **Function type Signature** 一行:

```
管道函数(可接收 |>)的签名以 <-tables: stream[A] 开头
   例:filter: (<-tables: stream[A], fn: (r: A) => bool) => stream[B]

普通函数的签名第一参数不是 <-tables
   例:int: (v: T) => int
```

---

## 七、FLUX 查询 InfluxDB

### 7.1 必须以 `from(bucket:) |> range(start:)` 开头

```flux
from(bucket: "test_init")
  |> range(start: -1h)
  // 后面才能 filter / aggregate 等
```

**`range` 必须紧跟 `from`,中间不能插别的函数,否则解析器报错**。

> 原因:InfluxDB 优化器要把时间范围下推到存储引擎(谓词下推),`range` 不在 `from` 之后无法识别。

### 7.2 表流(Table Stream)的核心概念

```
       Bucket
         │
         ▼
    ┌────────┐
    │ Table  │ ← 多张子表组成一个 stream
    │ Stream │
    └────────┘
         │
   按 _measurement + tag_set + _field 分组
         │
    ┌────┴────┬─────────┬─────────┐
    ▼         ▼         ▼         ▼
 Table 1   Table 2   Table 3   ...
 (一个     (一个     (一个
  series)   series)   series)
    │
    ▼
  rows:每行 = 一个 point = (_time, 各列值)
```

**聚合函数默认按子表聚合,不会跨 series 计算**(除非显式 `group()`)。

### 7.3 filter 与下划线列约定

```flux
|> filter(fn: (r) => r._measurement == "cpu" and r._field == "usage_idle")
```

**以 `_` 开头的列名是 FLUX 约定**:
- `_time` — 时间戳
- `_value` — 默认的数值列
- `_measurement` — 测量名
- `_field` — 字段名(转列后用)
- `_start` / `_stop` — 当前 range 边界

> 约定意义:很多函数(如 `toInt()`, `mean()` 不指定列时)**默认操作 `_value` 列**。修改下划线列名等于打破约定,后续函数会报错。

### 7.4 map / set 的区别

```flux
// map:逐行遍历,可读 r 中的字段
|> map(fn: (r) => ({r with _value: r._value * 100}))

// set:整列设为常量值,无需逐行
|> set(key: "hello", value: "world")
```

**只追加常量列时用 `set`**,性能远好于 `map`(见第 18 章实测)。

### 7.5 自定义管道函数

```flux
big100 = (tables=<-, x) => {
    return tables
      |> map(fn: (r) => ({r with _value: r._value * x}))
}

// 调用
from(bucket: "test_init")
  |> range(start: -1h)
  |> big100(x: 100)
```

> ⚠️ 关键语法:**第一个参数必须写 `tables=<-`**(参数名 `tables` 可改,但 `=<-` 不能变),`<-` 表示"从管道接收上游表流"。

### 7.6 window vs aggregateWindow

| | window | aggregateWindow |
| --- | --- | --- |
| 作用 | 仅按时间窗口**重新分组** | 按时间窗口分组**并聚合** |
| 分组结果 | 按 series + 窗口的组合分组 | **保留原 series 分组**(只在 series 内开窗) |
| 谓词下推 | ❌ 不下推 | ✅ 可下推(简单聚合时) |

```flux
// 推荐:aggregateWindow,等价于 window+aggregate+duplicate,且更优
|> aggregateWindow(every: 1m, fn: mean)
```

### 7.7 yield 与多结果

- FLUX 脚本顶层未被赋值的表流,**自动追加 `|> yield(name: "_result")`**
- 多个未赋值表流时,**必须显式 `yield(name: "唯一名")`**,否则名冲突
- 业界**不推荐一个脚本返回多个结果**

### 7.8 join 的争议

原讲义建议:**FLUX 内尽量不要做 join**。原因:

```
场景:每 15s 跑一次查询,join InfluxDB 时序数据 与 MySQL 维度表
   ▼
FLUX 无法缓存上次的维度表
   ▼
每 15s 全表扫一次 MySQL  →  IO 翻倍、压力大
```

**生产建议**:FLUX 只做时序数据查询,join 放到应用层(Java/Go 客户端)做,并对维度表做缓存。

---

## 八、第 18 章 FLUX 查询优化(实战要点)

### 8.1 谓词下推(Pushdown)

```
不下推:                        下推:
┌─────────┐                   ┌─────────┐
│  磁盘   │                   │  磁盘   │
└────┬────┘                   └────┬────┘
     │ 读全表                       │ 只读匹配
     ▼                              ▼
┌─────────┐                   ┌─────────┐
│ 内存    │                   │ 内存    │
│ 过滤    │                   │ 直接出  │
└─────────┘                   └─────────┘
```

**InfluxDB 2.4 支持下推的函数**(节选):
`count() / first() / last() / max() / mean() / min() / sum() / filter() / fill() / duplicate() / aggregateWindow()(简单 fn)` 等。

**`map()` 不会触发下推**,所以在 `map` 后面再做 `filter` 是反优化。

### 8.2 6 条优化清单

1. **善用谓词下推** — `range / filter / aggregateWindow` 尽量靠前,放在 `map / pivot / join` **之前**
2. **窗口不要太小** — 窗口越窄,每条数据"分到哪个窗"的计算开销越大
3. **慎用"沉重"函数** — `map / reduce / join / union / pivot` 都是内存密集型
4. **静态常量列用 `set`,不要用 `map`**
5. **时间范围 vs 数据精度的平衡** — 长时间范围用降采样后的数据,避免一次拉几亿点到内存
6. **降采样作为定时任务,把原始数据周期性聚合后回写到另一个 bucket**

### 8.3 性能分析:profiler 包

```flux
import "profiler"

option profiler.enabledProfilers = ["query", "operator"]

from(bucket: "test_init")
  |> range(start: -1h)
  |> filter(fn: (r) => r._measurement == "go_goroutines")
  |> aggregateWindow(every: 1m, fn: mean)
```

返回结果会**多出两张表**:

| 表 | 指标 |
| --- | --- |
| `profiler/query` | `TotalDuration` / `CompileDuration` / `MaxAllocated` / `flux/query-plan` ... |
| `profiler/operator` | 每个算子的 `Type` / `Count` / `MaxDuration` / `MeanDuration` |

### 8.4 如何识别下推

**算子被合并 = 下推成功**:

```
未下推:
   ReadRange ─▶ filter ─▶ map ─▶ aggregateWindow

下推后:
   merged_ReadRange_filter ─▶ map ─▶ aggregateWindow
                                              ↑
                                        合并意味着下推成功

更进一步(aggregateWindow 也下推):
   ReadWindowAggregateByTime ─▶ map
```

### 8.5 实测对比(原讲义示例 18.7)

| 查询版本 | MaxAllocated | TotalDuration |
| --- | --- | --- |
| `from → range → filter → map(+常量)` | ~52 KB | 17,365,972 ns |
| 加上 `aggregateWindow(every:1h)` 在 map 之后 | ~52 KB | 略快 |
| 把 `aggregateWindow` 移到 map **之前** | **0.8 KB** | **几倍提速** |
| 用 `set` 替代 `map(常量)` | ~52 KB(相近) | **4,612,562 ns(快 4 倍)** |

**结论**:
- `aggregateWindow` 在 `map` 之前 → 触发下推 → 内存骤降
- `set` 在内存上和 map 接近,**在时间上明显快**

---

## 九、本章勘误小结

| # | 原文位置 | 原文说法 | 实际情况 |
| --- | --- | --- | --- |
| 1 | 4.2 | "3 个示例用的函数都是一模一样的" | csv/sql 输出无 `_time` / `_measurement` 列,直接套用 `range/filter` 会报错。示例只是表达"统一接口"的设想 |
| 2 | 5.1.3.3 | "使用 `int( )` 或 **`unit()`**" | **`uint`**,中文/笔误 |
| 3 | 5.1.3.5 | "使用 **`date.add( )`** 函数从时间中减去持续时间" | 应为 **`date.sub()`** |
| 4 | 5.1.7.4 | 写 `NaN`(无引号) | FLUX 没有 `NaN` 字面量,必须 `float(v: "NaN")` |
| 5 | 5.1.10.1 | 把 "Null" 写两遍重复标题 | 5.1.10.1 应改名为"定义 null 的方法",5.1.10.2 是赘余 |
| 6 | 5.3.2.2 | `arr[2] // Returns two`(对 `["one","two","three"]`) | 应为 `Returns three`,下标 0 起算 |
| 7 | 5.3.1.4 | "正常的话应该返回 null" | 实际**直接编译期类型错误**,record 字段是静态的就是这个语义,文中描述容易引导初学者误判 |
| 8 | 5.3.3.1 | 例 `[1.0: {stable: 12, latest: 12}, ...]` | `dict` 的 key 类型必须同一类型,例子中是 float key,**OK**;但 value 类型也必须同一类型,例子中两个 value 都是 record,**OK**。要强调"统一" |
| 9 | 7.4 | 写 "filert"(笔误) | 应为 `filter` |
| 10 | 7.10 | 称 yield 的默认名为 `"_result"` | 截至 2.x,默认是 `"_results"`(带 s),容易拼错 |
| 11 | 18.7.3 example 中 filter `r["measurement"]` | 漏了下划线 | 正确是 `r["_measurement"]`,**否则永远 filter 不到任何数据**(标准列名带下划线) |
| 12 | 18.3 | 把 `pivot()` 列为沉重函数 | ✅ 对的;补充 `mapTransformation` 本身是 FLUX 中最贵的操作之一(每行调用函数) |
