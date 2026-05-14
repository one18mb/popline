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
| **容器** | `{` 对象，`[` 数组，各占一行。根值允许所有类型 |
| **弹出** | 行末 ` N` 后缀关闭 N 层容器。EOF 自动闭合 |
| **对象** | `键: 值`，冒号后空格。键名禁止 `:"{}[]` 空格制表符换行 |
| **数组** | 元素不带前缀 |
| **字符串** | 双引号 `""` 转义，支持跨行，闭合后可跟 ` N` 弹出 |
| **标量** | `true`/`false`/`null`/数字（含 `.`/`e` 为浮点）。裸字符串错误 |
| **消息流** | 空行分隔多条消息 |

完整规范见 [spec.md](spec.md)。

## 性能对比

测试数据：`test.json`（17011 B）→ `test.pln`（13076 B，**76.9%**），统一 5000 次迭代。

| 平台 | 序列化 (vs JSON) | 解析 (vs JSON) |
|------|-----------------|---------------|
| **C** | **0.59x** 🟢 | **0.85x** 🟢 |
| **Python** | **0.25x** 🟢 | **0.89x** 🟢 |
| **Go** | **0.37x** 🟢 | **0.68x** 🟢 |
| **Java** | **0.42x** 🟢 | **0.96x** 🟢 |
| **Rust** | **0.34x** 🟢 | 1.89x |
| **JS** | 4.40x 🔴 | 5.60x 🔴 |

> 比值 < 1 表示 PopLine 更快。仅 JS 为纯 TypeScript 无原生优化。

## 生态项目

| 项目 | 说明 |
|------|------|
| [popline-c](https://github.com/one18mb/popline-c) | C 参考实现（解析器、生成器、SAX 接口、格式转换器） |
| [popline-py](https://github.com/one18mb/popline-py) | Python C 扩展（SAX 解析 + 生成器序列化，无中间 DOM） |
| [popline-js](https://github.com/one18mb/popline-js) | JavaScript/TypeScript 实现 |
| [popline-go](https://github.com/one18mb/popline-go) | Go 实现 |
| [popline-rust](https://github.com/one18mb/popline-rust) | Rust crate |
| [popline-java](https://github.com/one18mb/popline-java) | Java 实现 |
| [popline-cli](https://github.com/one18mb/popline-cli) | CLI 工具：多格式互转（SAX 零 DOM + PopLine DOM） |
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
