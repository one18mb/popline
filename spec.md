# PopLine v1 语言规范

PopLine 是一种基于行的结构化数据序列化格式，作为 JSON 的替代方案设计。以行边界替代 JSON 的逗号和方括号/花括号配对，在同等工作量下有更好的压缩效果，同时保持同等表达力。

## 1. 词法结构

### 1.1 行

PopLine 文本被换行符（LF `\n` 或 CRLF `\r\n`）分割为行。每行承载一种语法结构：容器标记、键值对、数组元素、或容器关闭。

### 1.2 行首弹出前缀

行首可以有一个「弹出数字前缀」：一个十进制非负整数后跟一个空格（`N `），表示在解析该行内容之前关闭 N 层容器。弹出前缀必须与该行的实际内容合并，不允许纯弹出行（行中只有弹出数字而无其他内容）。

### 1.3 空行

长度为 0 的行（去掉 `\r` 后）是消息分隔符。连续多个空行不产生空消息。EOF 到达时自动关闭所有未闭合容器。

### 1.4 末尾空白

每行去掉末尾的 `\r` 后参与解析。行内不允许有行内注释语法。

## 2. 类型系统

PopLine 支持以下类型：

| 类型 | 表示 |
|------|------|
| 对象 (Object) | 键值对的无序集合 |
| 数组 (Array) | 值的有序列表 |
| 字符串 (String) | 双引号包裹的 Unicode 文本 |
| 整数 (Integer) | 64 位有符号整数 |
| 浮点数 (Float) | IEEE 754 双精度浮点数 |
| 布尔 (Boolean) | `true` 或 `false` |
| 空值 (Null) | `null` |

## 3. 语法

### 3.1 顶层

一个 PopLine 消息的顶层必须是对象或数组：

```
{
key: "value"
```

不允许标量作为根值。

### 3.2 对象

对象以 `{` 开始（独占一行），后续每行是一个键值对，格式为 `键: 值`。冒号后必须有一个空格。

```
{
name: "popline"
version: 2
active: true
```

键名采用**黑名单**校验——禁止包含以下字符：`:`, `"`, `{`, `}`, `[`, `]`, `#`, 空格, `\t`, `\n`, `\r`。支持 Unicode 字符（中文、数字、连字符、点号等）。

键名两侧无引号——PopLine 键名始终是裸字符串。

### 3.3 数组

数组以 `[` 开始（独占一行），后续每行是一个数组元素。元素不使用前缀符号（如 `-` 或 `*`）。

```
[
1
2
3
```

### 3.4 容器关闭

容器通过「弹出数字前缀」关闭。行首的 `N ` 表示在解析该行内容之前关闭 N 层容器。可一次关闭多层。

```
{
outer: {
inner: "value"
1 mid: "other"
```

以上示例中，`1 mid: "other"` 先弹出 1 层（关闭 `inner` 所在的内层对象），再在当前层（外层对象）添加键值对 `mid: "other"`。

EOF 到达时自动关闭所有未闭合容器，等同于在末尾追加必要的弹出前缀。

### 3.5 连缀容器（Inline Containers）

连续的容器开标识符 `{` 和 `[` 必须置于同一行，不得分占多行。即在数组上下文或顶层中，如果一个容器的首个元素是另一个容器，该内部容器必须与外部容器写在同一行。

```
[
[
1
2
1 [
3
```

以上等价于：

```
[[
1
2
1 [
3
```

连缀可递归，例如 `[ [ [` 表示三层嵌套数组。此项约束仅适用于容器开标识位于独立行的情况；对象中 `key: {` 和 `key: [` 本身已处于同一行，不受影响。

### 3.6 嵌套（容器值内联）

容器值行书写方式与普通值一致——对于对象，`{` 放在冒号空格后同一行；对于数组同理。

```
{
config: {
host: "localhost"
port: 8080
tags: [
"web"
"primary"
```

### 3.7 字符串

所有字符串必须用双引号 `"` 包裹。裸字符串（无引号）是语法错误。

#### 3.7.1 转义

字面双引号通过 `""`（两个连续的双引号）表示：

```
msg: "He said: ""Hello"""
```

解析后得到：`He said: "Hello"`

这是 PopLine 唯一的转义序列。反斜杠不具有转义意义，按字面处理。

#### 3.7.2 跨行字符串

如果一行的引号字符串未闭合（即 `"` 在行尾之前未找到匹配的闭合 `"`），字符串延续到后续行。换行符被保留为字符串内容的一部分。

```
description: "第一行
第二行
第三行"
```

解析后得到：`第一行\n第二行\n第三行`

### 3.8 数字

- 包含 `.`、`e` 或 `E` 的数字被解析为浮点数
- 其余数字被解析为整数
- 支持负号前缀

```
pi: 3.14159
count: 42
avogadro: 6.022e23
negative: -1
```

### 3.9 关键字

`true`、`false`、`null` 为保留关键字，分别表示布尔真、布尔假、空值。

## 4. 流式多消息

空行作为消息分隔符。一个 PopLine 文本流可以包含多条消息：

```
{
type: "log"
level: "info"

{
type: "metric"
cpu: 42.5
```

以上文本包含两条消息。解析流时产出两个对象。

连续空行不产生空消息。EOF 自动闭合最后一条消息的容器。

## 5. 错误处理

符合规范的解析器在遇到以下情况时必须报错：

- 裸字符串（无引号包裹的值）
- 纯弹出行（弹出数字后无内容）
- 弹出层数超过当前嵌套深度
- 非法键名（包含黑名单字符）
- 字符串闭合引号后有非空白内容
- 顶层行不是 `{` 或 `[`
- 对象内行不是 `键: 值` 格式

## 6. 与 JSON 的对照

| 特性 | JSON | PopLine |
|------|------|---------|
| 对象分隔 | `,` 逗号 | 行边界 |
| 数组分隔 | `,` 逗号 | 行边界 |
| 对象括号 | `{` `}` 配对 | `{` + 弹出数字 |
| 数组括号 | `[` `]` 配对 | `[` + 弹出数字 |
| 键名 | `"` 引号包裹 | 裸键名（黑名单校验） |
| 字符串 | `"` 包裹，`\` 转义 | `"` 包裹，`""` 转义 |
| 注释 | 无 | 无 |
| 根值 | 任意类型 | 仅对象或数组 |
| 体积 | 基准 | 更小 |
| 嵌套闭合 | 需要递归匹配 | 弹出数字前缀（迭代） |

## 7. 形式化语法（EBNF）

```ebnf
document       = message *(empty-line message) [empty-line]
message        = top-container line-content *line [eof-close]
top-container  = "{" | "["
line           = [pop-prefix] line-content
line-content   = object-line | array-line
               | "{" { " " ( "{" | "[" ) }
               | "[" { " " ( "{" | "[" ) }
               | empty
object-line    = key ":" " " value
array-line     = value
pop-prefix     = digit+ " "
value          = string | number | "true" | "false" | "null"
               | "{" | "["
string         = '"' (char | escaped-quote)* ('"' | eol-continue)
escaped-quote  = '""'
eol-continue   = '\n' string-body '\n' ... closing-'"'
number         = "-"? digit+ ("." digit+)? (("e"|"E") "-"? digit+)?
key            = key-char+
key-char       = Unicode - forbidden
forbidden      = ':' | '"' | '{' | '}' | '[' | ']' | '#' | ' ' | '\t' | '\n' | '\r'
empty-line     = ""
```

## 8. C 实现 API

### 8.1 DOM 类型

```c
typedef enum { PL_NULL, PL_BOOL, PL_INT, PL_FLOAT, PL_STRING, PL_OBJECT, PL_ARRAY } pl_value_type_t;

typedef struct pl_value_t {
    pl_value_type_t type;
    union { int bool_val; long long int_val; double float_val; char *string_val; } data;
    struct pl_value_t *child, *next;
    char *key;
} pl_value_t;
```

### 8.2 核心函数

| 函数 | 说明 |
|------|------|
| `pl_loads(text)` | SAX 解析单条消息 → DOM 树 |
| `pl_loads_fast(text)` | 直接 DOM 解析单条消息（性能版） |
| `pl_dumps(v)` | DOM 树序列化 → PopLine 字符串 |
| `pl_value_free(v)` | 递归释放 DOM 树 |
| `pl_value_new_*()` | 创建各类型节点 |
| `pl_value_add_to_object(obj, key, val)` | 向对象添加键值对 |
| `pl_value_add_to_array(arr, val)` | 向数组追加元素 |

### 8.3 JSON 转换

```c
pl_value_t *pl_loads_json(const char *json);  /* JSON 文本 → PopLine DOM */
char       *pl_dumps_json(pl_value_t *v);     /* PopLine DOM → JSON 文本 */
```

## 9. Python 接口

```python
import popline

obj = popline.loads(text)          # 解析单条 PopLine 消息 → dict/list
text = popline.dumps(obj)          # 序列化 Python 对象 → PopLine 字符串
msgs = popline.loads_stream(text)  # 解析多条消息 → dict/list 列表
text = popline.dumps_stream(objs)  # 序列化多个对象 → PopLine 流文本
```

Python 模块使用 C 扩展实现，底层调用 `pl_loads_fast` / `pl_dumps`。
