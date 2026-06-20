# Python 核心章节 知识点梳理 · 总览

> 本目录是对 `python/Article/` 下全部章节(codeSpecification + python0 ~ python28)逐章整理的知识点汇总。
>
> 整理范围:**全部 30 章**(30 份文档,覆盖原 ~85 个 markdown)。
>
> 编写原则:
> 1. 知识点按章节单文档化,便于速查。
> 2. 多文件/多组件调用场景 **画 ASCII 流程图**。
> 3. **明确标注原文档中过时或错误的描述**,给出 Python 3.10+(乃至 3.12 / 3.13)的正确写法。

---

## 目录

| 序号 | 文档 | 主题 | 是否含 ASCII 流程图 |
|------|------|------|---------------------|
| 00 | [00_why_python.md](./00_why_python.md) | 为什么学 Python(趋势、领域、版本现状) | — |
| 01 | [00_codeSpecification.md](./00_codeSpecification.md) | 代码规范(PEP 8、注释、命名) | — |
| 02 | [01_first_program.md](./01_first_program.md) | Python 简介、安装、第一个程序、IDE | ✅ Hello World 执行链 |
| 03 | [02_data_type_and_variable.md](./02_data_type_and_variable.md) | 基本数据类型、字符串编码、变量指向模型 | ✅ 变量绑定内存图 |
| 04 | [03_list_and_tuple.md](./03_list_and_tuple.md) | List、Tuple、可变 vs 不可变 | ✅ tuple 元素指向图 |
| 05 | [04_dict_and_set.md](./04_dict_and_set.md) | Dict、Set、frozenset、3.7+ 顺序保证 | — |
| 06 | [05_if_and_loop.md](./05_if_and_loop.md) | if/elif/else、match/case、for/while、for-else | — |
| 07 | [06_function.md](./06_function.md) | 函数、参数(位置/默认/可变/仅关键字)、传值机制、lambda、类型注解 | ✅ 传值机制对比图 |
| 08 | [07_iterator_and_generator.md](./07_iterator_and_generator.md) | Iterable/Iterator/Generator、yield 暂停恢复、推导式、zip | ✅ yield 时间线 |
| 09 | [08_oop.md](./08_oop.md) | 类、实例/类/静态方法、继承、super、多态、name mangling | ✅ 属性查找路径 |
| 10 | [09_module_and_package.md](./09_module_and_package.md) | 模块、包、命名空间包、import 流程、sys.modules 缓存 | ✅ **完整跨文件 import 流程** |
| 11 | [10_magic_method.md](./10_magic_method.md) | 魔术方法、`__new__/__init__/__del__`、属性钩子、描述器、容器、运算符重载 | ✅ 属性查找顺序 |
| 12 | [11_enum.md](./11_enum.md) | Enum、IntEnum、StrEnum(3.11+)、Flag、`@unique`、`match` 配合 | ✅ EnumMeta 驱动 |
| 13 | [12_metaclass.md](./12_metaclass.md) | 类即对象、`type()` 双重身份、自定义元类(3 写法)、`__init_subclass__` | ✅ 类创建三阶段 |
| 14 | [13_thread_and_process.md](./13_thread_and_process.md) | GIL、threading、multiprocessing、Pool、Queue、fork/spawn | ✅ **生产者消费者数据流 / 多进程通信** |
| 15 | [14_regex.md](./14_regex.md) | re 模块、字符集/数量词/分组、Match、sub、flags、贪婪/非贪婪/占有 | — |
| 16 | [15_closure.md](./15_closure.md) | 闭包、`nonlocal`、`__closure__`、循环陷阱 | ✅ 变量绑定时间线 |
| 17 | [16_decorator.md](./16_decorator.md) | 装饰器原理、`@` 语法糖、参数化、`wraps`、洋葱模型 | ✅ 装饰器执行流 |
| 18 | [17_type_annotations.md](./17_type_annotations.md) | 类型注解(3.10+)、Protocol、TypeAlias、mypy/pyright | ✅ 类型检查流水线 |
| 19 | [18_pathlib.md](./18_pathlib.md) | Path 对象、文件 I/O、目录遍历、os.path 迁移 | — |
| 20 | [19_exception.md](./19_exception.md) | try/except/else/finally、异常链、ExceptionGroup(3.11+)、add_note(3.11+) | ✅ 异常层级树 / ExceptionGroup 流 |
| 21 | [20_dataclass.md](./20_dataclass.md) | dataclass、Pydantic v2、NamedTuple、TypedDict | ✅ dataclass vs Pydantic 边界 |
| 22 | [21_context_manager.md](./21_context_manager.md) | `with`、`__enter__/__exit__`、`@contextmanager`、ExitStack、async with | ✅ with 语句执行时间线 |
| 23 | [22_async_await.md](./22_async_await.md) | async def/await、gather、Task、TaskGroup(3.11+)、Semaphore、to_thread | ✅ 事件循环调度流 |
| 24 | [23_pyproject_uv.md](./23_pyproject_uv.md) | pyproject.toml(PEP 621)、uv 包管理器、uv.lock | ✅ uv 工作流 |
| 25 | [24_ruff.md](./24_ruff.md) | ruff linter + formatter、配置、pre-commit、CI 集成 | — |
| 26 | [25_pytest.md](./25_pytest.md) | pytest、assert 重写、fixtures、参数化、conftest、coverage | ✅ 测试发现与 fixture 依赖图 |
| 27 | [26_logging.md](./26_logging.md) | logging 模块、Handler/Formatter、dictConfig、结构化日志 | ✅ Logger 层级与 Handler 管道 |
| 28 | [27_packaging_typer.md](./27_packaging_typer.md) | src layout、uv build/publish、OIDC、typer CLI | ✅ 构建→发布流水线 |
| 29 | [28_next_steps.md](./28_next_steps.md) | 三大职业方向、技术栈、入门项目、学习资源 | — |

---

## ASCII 流程图覆盖说明

按你的要求,所有 **真正发生多文件/多组件调用** 的场景都画了执行流图:

**多文件/多组件调用:**
- **跨文件 import 链**:`09_module_and_package.md` 完整画了 `main.py → mypkg/__init__.py → greet.py → utils/log.py` 的链式 import、`sys.modules` 缓存机制。
- **生产者-消费者数据流**:`13_thread_and_process.md` 同时画了 `threading.Queue` 与 `multiprocessing.Queue` 两个版本。
- **dataclass vs Pydantic 边界**:`20_dataclass.md` 画出内外部数据边界中两者的角色分工。
- **日志管道**:`26_logging.md` 画出 Logger → Handler → Formatter → 输出的完整管道。
- **构建→发布流水线**:`27_packaging_typer.md` 画出从开发到 PyPI 发布的完整 CI/CD 流程。

**关键执行流:**
- Hello World 从终端到 PVM 的完整链:`01`
- 变量赋值/重指向的内存模型:`02`
- tuple "指向不变,内容可变" 的结构:`03`
- 函数传值机制(对象引用 vs 重指向):`06`
- 生成器 yield 时间线:`07`
- 属性查找顺序、`__getattribute__` 与 `__getattr__` 关系:`08` `10`
- EnumMeta 创建枚举的步骤:`11`
- 类创建三阶段(`__prepare__/exec/__new__/__init__`):`12`
- 闭包变量绑定与生命周期:`15`
- 装饰器定义时 vs 调用时 + 多层洋葱模型:`16`
- 类型检查流水线 + Protocol 结构化子类型检查:`17`
- ExceptionGroup 树状结构 + try/except/else/finally 执行流:`19`
- `with` 语句完整时间线(`@contextmanager` yield 边界):`21`
- 事件循环 Task 调度 + create_task vs await 对比:`22`
- uv 项目初始化 → 依赖管理 → 环境同步 工作流:`23`
- pytest 测试发现/收集/执行 + Fixture 依赖图:`25`

---

## 原文档错误修正一览

每份知识点文档都包含一个"**与原文档差异速查**"小节,集中列出该章发现的 **过时/错误/不准确** 表述并给出正确版本。常见错误模式:

1. **Python 2 语法残留**:`print 'x'`、`u"..."`、`super(Cls, self)`、`__metaclass__`、`__cmp__`、`__div__`、`unichr`、`event.isSet()` 等。
2. **"必须的"过时建议**:`#-*-coding:utf-8-*-`、`~/.bash_profile`、强制 `__init__.py` 等。
3. **不准确的概念**:
   - "Python 解释成机器码"(应为字节码→PVM)
   - "Unicode 用两个字节表示"(应为码点表)
   - "dict 顺序与插入无关"(3.7+ 已保留)
   - "变量名必须英文"(3.x 允许 Unicode)
   - "重写实例方法不可能"(可用 `types.MethodType`)
   - "类销毁时调用 __del__"(应为实例被回收时)
   - "popitem 随机弹出"(3.7+ 是 LIFO)
4. **示例反模式**:`def f(b=[])` 默认参数、贪婪 `.*` 吞匹配、无 `with` 的 acquire/release、`__setattr__` 内部 `self.name=v` 死循环 等。

---

## 阅读建议

- 入门者 → 按 00(why) → 01 ~ 29 顺序读。
- 已有基础来 **查漏补缺** → 直接读"**与原文档差异速查**"小节。
- 写工程代码做参考 → 用本目录代替原 Article(因为原 Article 写于 2017 年前后,部分内容已过时)。

---

## 完成状态

**全部 30 章已整理完成**(codeSpecification + python0 ~ python28)，覆盖原 Article 目录下约 85 个 markdown 文件。
