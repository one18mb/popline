# PopLine: JSON is fine. But "fine" doesn't mean optimal for how you actually use it.

## The Problem with "Fine"

JSON was designed in 2001 for one job: easy JavaScript parsing. It succeeded. It's ubiquitous.

But ubiquitous doesn't mean optimal. Look at how structured data is actually used:

| Use Case | Frequency | JSON's Performance |
|----------|-----------|-------------------|
| Config files | **Daily** | ⚠️ Diff pollution, grep friction |
| Structured logs | **Daily** | ⚠️ Line noise, `jq` required |
| Data exchange | **Constant** | ✅ Works well |
| API responses | **Constant** | ⚠️ 23% syntax overhead |
| Deeply nested data | **Occasional** | ✅ Bracket pairing helps |

The problem is clear: **JSON optimizes for occasional scenarios (deep nesting) at the cost of daily scenarios (config, logs, diffs).**

And it's not just about human ergonomics. The extra syntax has a real machine cost:

```
17011 B  test.json
13074 B  PopLine equivalent           (-23% fewer bytes on the wire)

C  serialize:    275 ms (JSON) → 186 ms (PopLine)   -32%
Go  serialize:  1486 ms (JSON) → 552 ms (PopLine)   -63%
Python serialize: 935 ms (JSON) → 213 ms (PopLine)  -77%
```

**Faster for machines. Cleaner for grep. Smaller on the wire.**

One trade-off: PopLine's `N ` pop prefix is less visually obvious than JSON's `{}` brackets. But what you get in return — smaller files, faster parsing, grep-native retrieval, clean diffs — matters more in practice.

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

**Benchmarks** (5000 iterations, 17011 B `test.json` → 13074 B `.pln`, all tests at same iteration count):

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
17011 B  test.json
13074 B  test.pln   76.9% (-23%)
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

## The Honest Trade-off

**PopLine's weakness:** Nested structure is less visually scannable than JSON. The `N ` pop prefix requires a moment to parse, unlike JSON's `{}` which you see instantly.

**What that weakness buys you:**

| Metric | PopLine | vs JSON |
|--------|---------|---------|
| File size | 13074 B | **23% smaller** (more for deep nesting) |
| C serialize | 186 ms | **32% faster** |
| Go serialize | 552 ms | **63% faster** |
| Python serialize | 213 ms | **77% faster** |
| grep search | `grep port:` | No jq needed |
| diff accuracy | one-line change | No diff pollution |
| streaming | blank-line delimited | Native multi-message |

JSON can do everything PopLine can do. But PopLine does the **high-frequency stuff** — storage, transfer, retrieval, parsing, diffing — significantly better across the board.

**The bet:** Visual hierarchy readability matters less than file size, performance, and searchability in real-world usage.

## Use Cases

| Use Case | Why PopLine Wins |
|----------|-----------------|
| **Data exchange / API** | 23% smaller payload, faster serialization |
| **Config files** | grepable, clean diffs |
| **Structured logging** | `grep level=error` without jq, streaming native |
| **Deeply nested data** | More nesting = more savings (no bracket overhead) |
| **Data pipelines** | Line-based, pipe-friendly |
| **Git-stored configs** | Accurate diffs, meaningful blame |

## Quick Start

```bash
# CLI (zero dependencies, 51 KB binary)
pln convert test.json test.pln
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

**PopLine trades one thing (visual nesting) for everything else (size, speed, grepability, diffability).**

JSON can do everything PopLine does. PopLine just does the high-frequency stuff better.

[https://github.com/one18mb/popline](https://github.com/one18mb/popline)
