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

## 性能对比

测试数据：`package.json`（17011 B）→ `package.pln`（13074 B，**76.9%**），统一 5000 次迭代。

| 平台 | 序列化 (vs JSON) | 解析 (vs JSON) |
|------|-----------------|---------------|
| **C** | **0.68x** 🟢 | **0.74x** 🟢 |
| **Python** | **0.23x** 🟢 | **0.91x** 🟢 |
| **Go** | **0.37x** 🟢 | **0.68x** 🟢 |
| **Java** | **0.42x** 🟢 | **0.96x** 🟢 |
| **Rust** | **0.34x** 🟢 | 1.89x |
| **JS** | 10.20x 🔴 | 8.06x 🔴 |

> 比值 < 1 表示 PopLine 更快。仅 JS 为纯 TypeScript 无原生优化。

## 生态项目

| 项目 | 说明 |
|------|------|
| [popline-c](https://github.com/one18mb/popline-c) | C 参考实现（解析器、生成器、JSON 互转） |
| [popline-py](https://github.com/one18mb/popline-py) | Python C 扩展 |
| [popline-js](https://github.com/one18mb/popline-js) | JavaScript/TypeScript 实现 |
| [popline-go](https://github.com/one18mb/popline-go) | Go 实现 |
| [popline-rust](https://github.com/one18mb/popline-rust) | Rust crate |
| [popline-java](https://github.com/one18mb/popline-java) | Java 实现 |
| [popline-cli](https://github.com/one18mb/popline-cli) | CLI 工具：`pln` 命令，JSON ↔ PopLine 互转 |
| [popline-vscode](https://github.com/one18mb/popline-vscode) | VS Code 扩展（语法高亮） |
| [popline-vim](https://github.com/one18mb/popline-vim) | Vim/Neovim 插件 |

## 规范

语言规范见 [spec.md](spec.md)。

## 许可

MIT

## 致谢
本项目的开发得到了以下 AI 工具的大力协助：
- [Claude Code](https://claude.ai)（Anthropic）
- [DeepSeek](https://deepseek.com)（深度求索）
