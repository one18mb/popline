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

Test data: `package.json` (17011 bytes) → PopLine `package.pln` (13074 bytes, **76.9%**)

| Platform | Serialize | vs JSON | Parse | vs JSON |
|----------|-----------|---------|-------|---------|
| **C** | 1742 ms | **0.65x** 🟢 | 3718 ms | **0.75x** 🟢 |
| **Python** | 193 ms | **0.22x** 🟢 | 519 ms | **0.79x** 🟢 |
| **Go** | 503 ms | **0.33x** 🟢 | 1232 ms | **0.69x** 🟢 |
| **Java** | 703 ms | **0.41x** 🟢 | 1402 ms | **0.98x** 🟢 |
| **Rust** | — | TBD | — | TBD |
| **JS** | 2401 ms | 10.20x 🔴 | 3306 ms | 8.06x 🔴 |

> 🔴 = Pure TypeScript, no native optimization. Reference only.

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
