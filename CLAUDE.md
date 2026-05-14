# CLAUDE.md

PopLine 生态系统的中心仓库。包含语言规范和各平台实现的索引。详细实现指南（构建命令、架构模式、语法变更清单）见仓库根目录的 `CLAUDE.md`。

## 项目结构

| 文件 | 说明 |
|------|------|
| `spec.md` | PopLine v0.4.0 形式化语言规范（含 EBNF 语法） |
| `README.md` | 生态总览 |
| `promotion/` | 推广文案 |

## PopLine 语法规则（v0.3.0）

- **容器**：`{` 表示对象，`[` 表示数组，各自占一行。根值允许所有类型（标量或容器）。
- **嵌套**：通过层次隐含，无需缩进符。
- **弹出后缀**：行末 ` N` 关闭 N 层容器。必须附着在值内容之后（禁止纯弹出行）。EOF 自动闭合。
- **对象**：`键: 值`，冒号后必须有一个空格。键名黑名单：`:"{}[]` 空格制表符换行。
- **数组**：元素不加前缀。
- **字符串**：双引号 `""` 转义，支持跨行，跨行字符串闭合后可跟 ` N` 弹出后缀。
- **空行**：分隔多条消息。
- **标量**：数字（含 `.`/`e` 为浮点）、`true`/`false`/`null`。裸字符串错误。

## 生态项目

| 项目 | 仓库 |
|------|------|
| C | `github.com/one18mb/popline-c` |
| Python | `github.com/one18mb/popline-py` |
| JavaScript/TypeScript | `github.com/one18mb/popline-js` |
| Go | `github.com/one18mb/popline-go` |
| Rust | `github.com/one18mb/popline-rust` |
| Java | `github.com/one18mb/popline-java` |
| VS Code | `github.com/one18mb/popline-vscode` |
| Vim | `github.com/one18mb/popline-vim` |
