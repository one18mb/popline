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
| **Containers** | `{` = object, `[` = array, each on its own line. Root accepts any type. |
| **Pop suffix** | Trailing ` N` closes N containers. Must attach to value content. EOF auto-closes. |
| **Objects** | `key: value` (colon + space). Forbidden in keys: `:"{}[]#` space tab newline. |
| **Arrays** | Elements have no prefix. |
| **Strings** | Double-quoted. `""` escapes to literal `"`. Supports multi-line; closing quote can have ` N` pop suffix. |
| **Scalars** | `true`/`false`/`null`/numbers (`.` or `e` → float). Bare strings error. |
| **Streaming** | Empty lines separate multiple messages. |

Full spec: [spec.md](spec.md).

## Performance

Test data: `test.json` (17011 B) → `test.pln` (13076 B, **76.9%**), 5000 iterations.

| Platform | Serialize (vs JSON) | Parse (vs JSON) |
|----------|-------------------|----------------|
| **C** | **0.59x** 🟢 | **0.85x** 🟢 |
| **Python** | **0.25x** 🟢 | **0.89x** 🟢 |
| **Go** | **0.37x** 🟢 | **0.68x** 🟢 |
| **Java** | **0.42x** 🟢 | **0.96x** 🟢 |
| **Rust** | **0.34x** 🟢 | 1.89x |
| **JS** | 4.40x 🔴 | 5.60x 🔴 |

> Ratio < 1 means PopLine is faster. JS is pure TypeScript without native optimization.

## Ecosystem

| Project | Description |
|---------|-------------|
| [popline-c](https://github.com/one18mb/popline-c) | C reference (parser, generator, SAX interface, format converters) |
| [popline-py](https://github.com/one18mb/popline-py) | Python C extension (SAX parse + generator serialize, no DOM) |
| [popline-js](https://github.com/one18mb/popline-js) | JavaScript/TypeScript |
| [popline-go](https://github.com/one18mb/popline-go) | Go implementation |
| [popline-rust](https://github.com/one18mb/popline-rust) | Rust crate |
| [popline-java](https://github.com/one18mb/popline-java) | Java implementation |
| [popline-cli](https://github.com/one18mb/popline-cli) | CLI multi-format converter (SAX zero-DOM + PopLine DOM) |
| [popline-vscode](https://github.com/one18mb/popline-vscode) | VS Code extension |
| [popline-vim](https://github.com/one18mb/popline-vim) | Vim/Neovim plugin |

## License

MIT

## Acknowledgments
This project was developed with the assistance of:
- [Claude Code](https://claude.ai) (Anthropic)
- [DeepSeek](https://deepseek.com) (DeepSeek)
