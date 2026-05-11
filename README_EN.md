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

Test data: `package.json` (17011 B) → `package.pln` (13074 B, **76.9%**)

> Ratio < 1 means PopLine is faster. Iteration counts differ per language (C=50000, others=5000), absolute times are NOT comparable across languages. See each sub-project README for µs/op details.

| Platform | Serialize (vs JSON) | Parse (vs JSON) | Iterations |
|----------|-------------------|----------------|------------|
| **C** | **0.65x** 🟢 | **0.75x** 🟢 | 50000 |
| **Python** | **0.22x** 🟢 | **0.79x** 🟢 | 5000 |
| **Go** | **0.33x** 🟢 | **0.69x** 🟢 | 5000 |
| **Java** | **0.41x** 🟢 | **0.98x** 🟢 | 5000 |
| **Rust** | — | — | TBD |
| **JS** | 10.20x 🔴 | 8.06x 🔴 | 5000 |

> 🔴 = Pure TypeScript, reference only

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
