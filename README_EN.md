# PopLine

**Line-oriented serialization format, a lightweight JSON alternative**

PopLine is a custom line-based markup language / serialization format designed as a JSON alternative. By using line boundaries instead of JSON's commas and bracket pairs, it reduces file size by ~20% while maintaining equivalent expressiveness.

## Example

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

## Syntax

| Feature | Rule |
|---------|------|
| **Containers** | `{` = object, `[` = array, each on its own line. Root must be `{` or `[`. |
| **Pop** | `N ` prefix closes N containers. Must merge with content. EOF auto-closes. |
| **Objects** | `key: value` (colon + space). Forbidden in keys: `:"{}[]#` space tab newline. |
| **Arrays** | Elements have no prefix. |
| **Strings** | Double-quoted. `""` escapes to literal `"`. Supports multi-line. |
| **Scalars** | `true`/`false`/`null`/numbers (`.` or `e` → float). Bare strings error. |
| **Streaming** | Empty lines separate multiple messages. |

Full spec: [spec.md](spec.md).

## Performance

Test data: `package.json` (17011 B) → `package.pln` (13074 B, **76.9%**), 5000 iterations for all tests.

| Platform | Serialize (vs JSON) | Parse (vs JSON) |
|----------|-------------------|----------------|
| **C** | **0.68x** 🟢 | **0.74x** 🟢 |
| **Python** | **0.23x** 🟢 | **0.91x** 🟢 |
| **Go** | **0.37x** 🟢 | **0.68x** 🟢 |
| **Java** | **0.42x** 🟢 | **0.96x** 🟢 |
| **Rust** | **0.34x** 🟢 | 1.89x |
| **JS** | 10.20x 🔴 | 8.06x 🔴 |

> Ratio < 1 means PopLine is faster. JS is pure TypeScript without native optimization.

## Ecosystem

| Project | Description |
|---------|-------------|
| [popline-c](https://github.com/one18mb/popline-c) | C reference implementation |
| [popline-py](https://github.com/one18mb/popline-py) | Python C extension |
| [popline-js](https://github.com/one18mb/popline-js) | JavaScript/TypeScript |
| [popline-go](https://github.com/one18mb/popline-go) | Go implementation |
| [popline-rust](https://github.com/one18mb/popline-rust) | Rust crate |
| [popline-java](https://github.com/one18mb/popline-java) | Java implementation |
| [popline-vscode](https://github.com/one18mb/popline-vscode) | VS Code extension |
| [popline-vim](https://github.com/one18mb/popline-vim) | Vim/Neovim plugin |

## License

MIT

## Acknowledgments
This project was developed with the assistance of:
- [Claude Code](https://claude.ai) (Anthropic)
- [DeepSeek](https://deepseek.com) (DeepSeek)
