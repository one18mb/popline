# PopLine

**基于行的序列化格式，JSON 的轻量替代**

PopLine 是一种自定义的、基于行的标记语言/序列化格式，专为替代 JSON 设计。通过行边界替代 JSON 的逗号和方括号/花括号配对，减少约 20% 体积，同时保持同等表达力。

## 示例

**JSON:**
```json
{"name": "popline", "version": 2, "active": true, "tags": ["serialization", "line-based"]}
```

**PopLine:**
```
{
name: "popline"
version: 2
active: true
tags: [
"serialization"
"line-based"
1 description: "A line-based
serialization format."
```

## 语法要点

| 特性 | 规则 |
|------|------|
| **容器** | `{` 对象，`[` 数组，各占一行。顶层必须是 `{` 或 `[` |
| **弹出** | 行首 `N ` 前缀关闭 N 层容器。必须与内容合并。EOF 自动闭合 |
| **对象** | `键: 值`，冒号后空格。键名禁止 `:"{}[]#` 空格制表符换行 |
| **数组** | 元素不带前缀 |
| **字符串** | 双引号 `""` 转义，支持跨行 |
| **标量** | `true`/`false`/`null`/数字 |
| **消息流** | 空行分隔多条消息 |

完整规范见 [spec.md](spec.md)。

## 生态项目

| 项目 | 说明 |
|------|------|
| [popline-c](https://github.com/one18mb/popline-c) | C 参考实现（解析器、生成器、JSON 互转） |
| [popline-py](https://github.com/one18mb/popline-py) | Python C 扩展 |
| [popline-js](https://github.com/one18mb/popline-js) | JavaScript/TypeScript 实现 |
| [popline-go](https://github.com/one18mb/popline-go) | Go 实现 |
| [popline-rust](https://github.com/one18mb/popline-rust) | Rust crate |
| [popline-java](https://github.com/one18mb/popline-java) | Java 实现 |
| [popline-vscode](https://github.com/one18mb/popline-vscode) | VS Code 扩展（语法高亮） |
| [popline-vim](https://github.com/one18mb/popline-vim) | Vim/Neovim 插件 |

## 规范

语言规范见 [spec.md](spec.md)。

## 许可

MIT
