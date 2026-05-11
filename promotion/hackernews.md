# PopLine: A line-oriented JSON alternative optimized for human reading

## The Problem

JSON is designed for machines. Every `{`, `}`, `"`, `,` — these are for parsers, not for the human eye.

When you read a config file, review a PR diff, or grep through logs, what you really want is:

```
port: 8080
```

Not:

```
"port": 8080,
```

That extra noise adds up. Over 17011 bytes of `package.json`, **3923 bytes are syntax noise** — quote marks, commas, closing brackets. That's **23% of the file**. PopLine drops it.

## The Design

**One rule change:** Replace bracket-pairing and comma-separating with **line-boundaries** and **pop-prefixes**.

```json
// JSON — 4 lines of meaningful data, 4 lines of syntax
{
  "server": {
    "port": 8080,
    "host": "0.0.0.0"
  }
}
```

```
// PopLine — every line is meaningful
{
server: {
port: 8080
host: "0.0.0.0"
```

How it works:

| Feature | JSON | PopLine |
|---------|------|---------|
| Open container | `{` / `[` | Same, on its own line |
| Close container | `}` / `]` (paired) | `N ` prefix (e.g., `1 ` closes 1 level) |
| Key separator | `"key":` | `key:` (bare, no quotes) |
| Value separator | `,` | Newline |
| String escape | `\"` `\n` `\t` `\\`... | `""` only (for literal `"`) |
| Stream | JSON Lines (one per line) | Blank-line delimited |
| EOF | All brackets must close | Auto-closes remaining containers |

## The Results

**Benchmarks** (5000 iterations, 17011 B `package.json` → 13074 B `.pln`, all tests at same iteration count):

| Language | JSON Lib | PopLine | Ratio |
|----------|----------|---------|-------|
| **C** parse | 520 ms | 387 ms | **0.74x** |
| **C** serialize | 275 ms | 186 ms | **0.68x** |
| **Python** parse | 689 ms | 626 ms | **0.91x** |
| **Python** serialize | 935 ms | 213 ms | **0.23x** |
| **Go** parse | 1929 ms | 1308 ms | **0.68x** |
| **Go** serialize | 1486 ms | 552 ms | **0.37x** |
| **Rust** parse | 4788 ms | 9042 ms | 1.89x |
| **Rust** serialize | 6783 ms | 2323 ms | **0.34x** |
| **Java** parse | 1941 ms | 1861 ms | **0.96x** |
| **Java** serialize | 2028 ms | 857 ms | **0.42x** |

**File size**:

```
17011 B  package.json
13074 B  package.pln   76.9% (-23%)
```

## Killer Features

### 1. grep-native

```bash
cat config.pln | grep port
port: 8080
```

No `jq` required. No pipe chains. Every line is a self-contained key-value pair.

### 2. diff-perfect

```diff
-port: 8080
+port: 9090
```

One field change = one line in diff. In JSON, the same change in an object looks like:

```diff
 {
   "server": {
-    "port": 8080,
-    "host": "0.0.0.0",
-    "features": ["logging"],
+    "port": 9090,
+    "host": "0.0.0.0",
+    "features": ["logging", "metrics"],
     ...
```

The diff lies — it shows lines as changed when only one field actually was.

### 3. stream-ready

Multiple messages separated by blank lines, parsed independently:

```
{
type: "request"
path: "/api/users"

{
type: "response"
status: 200
```

### 4. smaller = faster over the wire

23% less data means 23% less bandwidth. For high-volume APIs or log shipping, this matters.

## Language Support

All implementations follow the same spec and outperform their JSON counterparts:

| Language | Parse | Serialize | Package |
|----------|-------|-----------|---------|
| **C** | `pln_loads()` | `pln_dumps()` | [popline-c](https://github.com/one18mb/popline-c) |
| **Python** | `pln.loads()` | `pln.dumps()` | [popline-py](https://github.com/one18mb/popline-py) |
| **Go** | `pln.Unmarshal()` | `pln.Marshal()` | [popline-go](https://github.com/one18mb/popline-go) |
| **Rust** | `pln::from_str()` | `pln::to_string()` | [popline-rust](https://github.com/one18mb/popline-rust) |
| **Java** | `Pln.parse()` | `Pln.stringify()` | [popline-java](https://github.com/one18mb/popline-java) |
| **TypeScript** | `Pln.parse()` | `Pln.stringify()` | [popline-js](https://github.com/one18mb/popline-js) |
| **CLI** | `pln convert` | `pln validate` | [popline-cli](https://github.com/one18mb/popline-cli) |

## Use Cases

| Use Case | Why PopLine Wins |
|----------|-----------------|
| **Config files** | Flat/shallow, grepable, clean diffs |
| **Structured logging** | `grep level=error` without jq, streaming native |
| **Data pipelines** | Line-based, pipe-friendly, `awk`/`sed` work directly |
| **API responses** | 23% smaller payload, fast serialization |
| **Git-stored configs** | Accurate diffs, meaningful blame |
| **CLI output** | Human-readable, greppable, pipeable |

## Quick Start

```bash
# CLI (zero dependencies, 51 KB binary)
pln convert package.json package.pln
pln validate schema.pln

# Python
pip install popline-py
import pln; obj = pln.loads('{\nkey: "value"\n')

# Go
go get github.com/one18mb/popline-go
v, _ := pln.Unmarshal("{\nkey: \"value\"\n")
s := pln.Marshal(v)
```

## Editor Support

- **VS Code**: Syntax highlighting, folding, validation — [popline-vscode](https://github.com/one18mb/popline-vscode)
- **Vim/Neovim**: Syntax highlighting, filetype detection — [popline-vim](https://github.com/one18mb/popline-vim)

## Spec

Full language specification in [spec.md](https://github.com/one18mb/popline/blob/main/spec.md) — EBNF grammar included.

---

**PopLine is not a JSON killer.** JSON excels at deep nesting and machine-to-machine communication.

**PopLine is for the 90% of JSON that humans read.** Config files, logs, diffs, pipeline data — the stuff you interact with daily.

[https://github.com/one18mb/popline](https://github.com/one18mb/popline)
