# PopLine

**基于行的序列化格式，JSON 的轻量替代**

PopLine 是一种自定义的、基于行的标记语言/序列化格式，专为替代 JSON 设计。通过行边界替代 JSON 的逗号和方括号/花括号配对，减少约 20% 体积，同时保持同等表达力。

## 快速示例

**JSON:**
```json
{
  "name": "popline",
  "version": 2,
  "tags": ["serialization", "line-based"],
  "active": true,
  "description": "A line-based\nserialization format."
}
```

**PopLine:**
```
{
name: "popline"
version: 2
tags: [
"serialization"
"primary"
]
active: true
description: "A line-based
serialization format."
```

## 语法要点

| 特性 | 规则 |
|------|------|
| **容器** | `{` 表示对象，`[` 表示数组，各占一行。顶层必须是 `{` 或 `[` |
| **弹出** | 行首 `N ` 前缀关闭 N 层容器。必须与内容合并（禁止纯弹出行）。EOF 自动关闭未闭合容器 |
| **对象** | `键: 值` 格式，冒号后一个空格。键名禁止含 `: " { } [ ] # 空格 制表符 换行符` |
| **数组** | 元素不带前缀，靠上下文区分 |
| **字符串** | 双引号包裹。`""` 表示字面双引号（唯一转义）。支持跨行 |
| **标量** | `true` / `false` / `null` / 数字（含 `.` `e` 为浮点，否则整数） |
| **消息流** | 空行分隔多条消息 |

## 项目结构

| 文件 | 说明 |
|------|------|
| `popline.h` | 公共头文件（DOM 类型、API 声明） |
| `popline.c` | 核心（DOM 类型/构造器、生成器 `pln_dumps`、转义） |
| `popline_parser.c` | 解析器 `pln_loads`（直接 DOM 构建，单遍扫描） |
| `popline_json.c` | JSON ↔ PopLine 双向转换（依赖 cJSON） |
| `popline_module.c` | Python C 扩展（直通 Python 对象，无中间 DOM） |
| `setup.py` | Python 模块构建脚本 |
| `spec.md` | 形式化语言规范（含 EBNF 语法） |

## 使用

### C API

```c
#include "popline.h"

// 解析
pln_value_t *v = pln_loads("{\nkey: \"value\"\n");
// 序列化
char *s = pln_dumps(v);
free(s);
pln_value_free(v);

// JSON 互转
pln_value_t *jv = pln_loads_json("{\"key\":\"value\"}");
char *js = pln_dumps_json(jv);
```

### Python

```python
import popline

obj = popline.loads('{\nkey: "value"\n')
text = popline.dumps({"key": "value"})
```

依赖：`libcjson-dev`（`apt install libcjson-dev`）

## 构建与测试

```bash
# C 完整测试（单元测试 + JSON 一致性 + 性能基准）
gcc -O2 -o test test.c popline.c popline_parser.c popline_json.c -lcjson -lm && ./test

# Python C 扩展测试
python test.py
```

完整命令列表见 [CLAUDE.md](./CLAUDE.md)。

## 性能

使用 package.json（17011 字节）基准测试：

| 操作 | 相比 cJSON/json |
|------|----------------|
| C 解析 | **0.75x**（快 25%） |
| C 序列化 | **0.65x**（快 1.5x） |
| Python 序列化 | **0.22x**（快 4.5x） |
| Python 解析 | **0.79x**（快 21%） |
| 体积比 | **76.9%**（小 23%） |

## 许可

MIT
