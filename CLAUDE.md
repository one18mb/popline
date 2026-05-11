# CLAUDE.md

本文件为 Claude Code (claude.ai/code) 在此仓库中工作时提供指导。

## 项目概述

PopLine 是一种自定义的、基于行的标记语言/序列化格式，作为 JSON 的替代方案设计。以行边界替代 JSON 的逗号和方括号/花括号配对，减少约 3-4% 体积。本项目包含 PopLine 语言的设计、规范定义与多语言实现（C 和 Python）。

## 运行测试

**C 完整测试（单元测试 + JSON 一致性 + 真实数据基准）：**
```bash
gcc -O2 -o test test.c popline.c popline_parser.c popline_json.c -lcjson -lm && ./test
```

**Python C 扩展测试（单元测试 + JSON 对比 + 性能基准）：**
```bash
python test.py
```

**Python C 扩展模块构建：**
```bash
python setup.py build_ext --inplace
python -c "import popline; ..."
```

依赖：`libcjson-dev`, `python3-dev`（apt install libcjson-dev python3-dev）

## PopLine 语法规则（v2 已确认）

- **容器**：`{` 表示对象（键值对集合），`[` 表示数组，各自占一行。顶层必须是 `{` 或 `[`，不允许标量作为根值。
- **嵌套**：通过层次隐含，无需缩进符。
- **弹出机制**（核心特性）：容器关闭统一使用「弹出数字前缀」——行首的 `N `（数字+空格）表示关闭 N 层容器。弹出必须与下一行内容合并（**禁止纯弹出行**）。可一次弹出多层。EOF 自动关闭所有未闭合容器。
- **对象**：格式 `键: 值`，冒号后一个空格。键名采用**黑名单**校验：禁止含 `:`、`"`、`{`、`}`、`[`、`]`、`#`、空格、制表符、换行符。支持中文、连字符、点号等。
- **数组**：元素不加前缀（`-`/`*`），靠上下文区分。
- **字符串**：统一使用双引号 `"` 包裹。内部 `""` 表示字面双引号（这是 PopLine 唯一的转义序列）。支持跨行字符串（`"` 打开后未闭合即延续到后续行，换行符保留为内容的一部分）。
- **消息分隔**：空行分隔流中的多条消息。连续空行不产生空消息。EOF 自动闭合容器。
- **标量**：数字（含 `.`/`e` 为浮点，否则整数）、布尔（`true`/`false`）、空值（`null`）。裸字符串抛出错误。

## 文件结构

| 文件 | 说明 |
|------|------|
| `spec.md` | PopLine v1 形式化语言规范（含 EBNF 语法） |
| `popline.h` | C 实现的公共头文件（事件类型、DOM 类型、API 声明） |
| `popline.c` | C 共享核心（DOM 类型/构造器/修改器、生成器、转义、print） |
| `popline_parser.c` | 单遍直接 DOM 构建解析器（`pln_loads`） |
| `popline_json.c` | JSON ↔ PopLine 双向转换（依赖 cJSON） |
| `popline_module.c` | Python C 扩展模块（CPython API 绑定） |
| `setup.py` | Python 模块构建脚本 |
| `test.c` | 完整 C 测试：单元测试 + JSON 一致性 + 真实数据基准 |
| `test.py` | Python C 扩展完整测试：单元测试 + JSON 对比 + 性能基准 |
| `package.json` | 测试数据（虚构的 VS Code 扩展，17011 字节） |
| `package.pln` | package.json 的 PopLine 等价文本（13074 字节） |

## C 实现架构

### 核心分离

**共享核心** (`popline.c`)：DOM 类型系统、构造/析构函数、生成器（`pln_dumps`）、字符串转义（`""` → `"`）、调试打印。不参与解析。

**单一解析器** (`popline_parser.c`)：单遍扫描，直接构建 DOM 树，零事件中转。使用 `fctx_t` 上下文维护帧栈（每帧包含容器的 `tail` 指针，达到 O(1) 追加）。
  - 使用 GCC 内置优化：`__builtin_expect`（分支预测）、`__attribute__((always_inline))`、`__attribute__((flatten))`
  - 错误信息存储在 `fctx_t.error[256]` 内（非全局，可重入）
  - `strchr` 定位换行符（而非逐字节扫描）
  - 字符串常见路径：无转义时直接 `pln_value_new_string_len` 零拷贝

**JSON 转换** (`popline_json.c`)：`pln_loads_json` / `pln_dumps_json`，依赖 cJSON 库。

### 生成器

`pln_gen_t` 在 `popline.c` 中实现：通过事件驱动 API（`pln_gen_begin_object`, `pln_gen_key`, `pln_gen_value_string`, `pln_gen_end_object` 等）将 DOM 树或手动调用序列化为 PopLine 文本。`pln_dumps` 是便捷封装。

### DOM API

- `pln_value_t` — 树形数据结构，单向链表组织兄弟节点
- 内置 `itoa`（非 `snprintf`）提升性能
- 字符串常见路径：无 `"` 需要转义时一次 `memcpy` 写入

### 关键设计决策

- 类型在解析时通过文本形状推断（含 `.`/`e` 为浮点 → `pln_value_new_float`，否则整数 → `pln_value_new_int`）
- 键名在解析器中通过 `fctx.key` 直接挂载到 `pln_value_t.key`
- `pln_value_add_to_object_nocopy` 接管键名指针所有权，避免拷贝

## Python C 扩展 (`popline_module.c`)

Python C 扩展通过 `py_loads_direct`/`py_dumps_direct` 直接操作 Python 对象（`PyDict`/`PyList`/`PyUnicode` 等），跳过中间 `pln_value_t` DOM 树。同时提供 JSON 转换函数（`loads_json`/`dumps_json`，通过 cJSON 桥接）。

## 错误处理模式

- **解析器**：`pln_loads` 返回 NULL 表示错误，错误细节在内部 `fctx.error` 中
- **错误类型**：裸字符串、纯弹出行、弹出层数超限、非法键名、引号后有多余内容、顶层非容器

## 性能基准（18KB 真实数据，package.json / package.pln）

以下所有测试使用 `package.json`（17011 字节）及其 PopLine 等价文本 `package.pln`（13074 字节）。

### C 实现（gcc -O2，50000 次迭代）

| 路径 | 操作 | 耗时 | vs cJSON |
|------|------|------|----------|
| cJSON | 解析（含 DOM） | ~5170 ms | — |
| PopLine | 解析（含 DOM） | ~3825 ms | **~0.74x** (快 26%) |
| cJSON | 序列化 | ~2760 ms | — |
| PopLine | 序列化 | ~1640 ms | **~0.59x** (快 1.7x) |

### Python C 扩展（5000 次迭代）

| 路径 | 操作 | 耗时 | vs json |
|------|------|------|---------|
| json.dumps | 序列化 | ~880 ms | — |
| **popline.dumps** | 序列化 | **~210 ms** | **~0.24x** (快 4.2x) |
| json.loads | 反序列化 | ~620 ms | — |
| **popline.loads** | 反序列化 | **~520 ms** | **~0.83x** (快 17%) |

### 分析

**序列化（0.24x / 0.58x）** 差距最大，源于语法设计本身：
- 键名裸写（无需引号包裹 + 转义扫描）
- 行边界天然分隔（无需 `,` 跟踪 + 递归 `{}` 配对）
- EOF 自动闭合容器（无需写闭合括号）
- 字符串只转义 `"`（无需查 `\` `\n` `\t` 等）

**Python 序列化（0.24x）比 C 序列化（0.58x）优势更大**，因为 `json.dumps` 在 Python 端额外做了 Python 对象序列化（类型检查、嵌套递归），而 PopLine 的格式更简单放大了这个差距。

**反序列化/解析** 的加速比在 Python 端（0.83x）和 C 端（0.74x）基本一致，说明 C 扩展的 `popline.loads` 实现了与 `json.loads` 同级别的"一步到位"直接建 PyObject。

### 体积

| 格式 | 大小 | 比值 |
|------|------|------|
| JSON | 17011 B | — |
| PopLine | 13074 B | **76.9%**（小 23%） |

## 项目状态

PopLine v2 已确认：C 实现（解析器、生成器、JSON 双向转换）、Python C 扩展模块。所有测试通过。