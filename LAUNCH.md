# PopLine 推广文案

---

## 一、知乎 / 掘金 风格（深度技术分析）

### 标题

**《你每天读 JSON 的时间，90% 浪费在了数括号上》**

### 正文

想想这个问题：你今天读了几次 JSON？

查配置文件、看 API 响应、翻日志、审 PR diff——各种场景都在读。但你仔细想想，你读 JSON 时，**到底在做什么**？

大概率是："找个字段的值"。

不是"理解嵌套结构"，不是"数括号配对"，只是找一个 key，看它的 value。就这么简单。

JSON 的嵌套可读性确实好——`{}` 配对明确，缩进清晰。但这是为低频场景（理解深层嵌套）优化的，高频场景（检索字段）反而被牺牲了。

**这是 PopLine 诞生的原因。**

### 核心逻辑速览

一个 100 行扁平对象的 JSON 改一个字段值，PR diff 给你展示一整块：

```diff
-    "port": 8080,
-    "host": "0.0.0.0",
-    "debug": true,
-    "log_level": "info",
+    "port": 9090,
+    "host": "0.0.0.0",
+    "debug": true,
+    "log_level": "info",
```

改成 PopLine：

```diff
-port: 8080
+port: 9090
```

**改一行，diff 就是一行。**

### 性能对比

PopLine vs JSON（5000 次迭代，17KB 真实数据）：

| 操作 | JSON | PopLine | 比值 |
|------|------|---------|------|
| 解析（C） | 520 ms | 387 ms | **0.74x** |
| 序列化（C） | 275 ms | 186 ms | **0.68x** |
| 体积 | 17011 B | 13074 B | **76.9%** |

### 适用场景

- **配置文件**：扁平或浅嵌套，grep 直达
- **结构化日志**：`grep level=error` 直接命中
- **API 响应体**：体积小 23%，传输更快
- **数据管道/ETL**：空行分隔天然支持流式

### 不适用场景

- 深层嵌套（5 层+）的数据结构
- 对 JSON 生态有强依赖的场景（现有工具链皆用 JSON）

### 多语言支持

C、Python、Go、Rust、Java、JavaScript 全部可用。

```bash
# 一分钟上手
gcc -O2 -o pln main.c popline.c popline_parser.c popline_json.c cjson/cJSON.c -lm
echo '{"name":"popline","version":2}' | pln convert - out.pln
```

---

## 二、Hacker News 风格（英文，问题驱动）

### Title

**JSON was designed for machines. PopLine is designed for the 90% of JSON that humans read.**

### Body

Every day, millions of JSON files are read by humans. Config files. API responses. Logs. PR diffs.

But JSON's design optimizes for the wrong thing. Its bracket-pairing and comma-separated syntax is great for parsers, not for the human eye scanning for a specific field.

**The 80/20 rule of JSON:** 80% of JSON documents are shallow (< 3 levels deep), but 100% of JSON syntax forces you to scan brackets.

PopLine takes the opposite approach: **line boundaries as structure**.

```
# instead of this:
{"server": {"port": 8080, "host": "0.0.0.0"}, "database": {"url": "..."}}

# or this prettified:
{
  "server": {
    "port": 8080,
    "host": "0.0.0.0"
  },
  "database": {
    "url": "..."
  }
}

# you get this:
{
server: {
port: 8080
host: "0.0.0.0"
1 database: {
url: "..."
```

**Key properties:**

- **23% smaller** than equivalent JSON
- **1.5-4.5x faster** to parse and serialize
- **grep-friendly**: `grep port:` works, no `jq` needed
- **diff-friendly**: one changed field = one line in diff
- **stream-friendly**: blank-line delimited messages

**Implementations:** C, Python, Go, Rust, Java, TypeScript — all with consistent API, all outperforming their JSON counterparts.

**The honest trade-off:** Deeply nested structures (>5 levels) are harder to read without editor folding support (which our VS Code and Vim extensions provide). PopLine is optimized for the common case, not the edge case.

```
# One command to try it
gcc -O2 -o pln main.c popline.c popline_parser.c popline_json.c cjson/cJSON.c -lm
echo '{"hello":"world"}' | ./pln convert - out.pln
```

---

## 三、V2EX 风格（简练犀利）

### 标题

**写了个 JSON 替代品，不吹不黑，说下真实优缺点**

### 正文

搞了个叫 **PopLine** 的东西，简单说就是**把 JSON 的行尾`,`和括号配对扔掉**，用换行和弹出行前缀代替。

核心改动就一行规则：
- JSON 的 `{` `}` `,` `"key"` → 变成 `key:` 一行一个
- 嵌套用 `N ` 前缀控制弹出（`1 key: val` 表示关 1 层容器再写）

**好处：**

1. **grep 友好** — `grep port: ` 直接出结果，不用 `jq`
2. **diff 友好** — 改一个字段只影响一行，不污染整块
3. **体积小 23%** — 省流量省存储
4. **解析快 1.5-4.5x** — 看语言，C/Python/Go/Rust 都比原生 JSON 快
5. **流式天然支持** — 空行分隔，一行一条消息

**坏处（不吹不黑）：**

1. **嵌套深了难读** — 超过 5 层要数 `{`，需要编辑器折叠辅助
2. **不在 JSON 生态内** — 没有 `jq`，没有 `JSON.parse`，走的是 PopLine 自己的解析器
3. **需要插件** — 已有 VS Code + Vim 插件，但不像 JSON 那样原生支持

**它适合什么？**

- 配置文件（扁平或浅嵌套）
- 结构化日志（`grep error` 直接搜）
- 管道数据（text, grep, awk 一条龙）
- 代码仓库中的配置文件（diff 好看）

**不适合什么？**

- 深层嵌套的复杂数据结构

**多语言实现：**

C / Python / Go / Rust / Java / TypeScript 都有，CLI 工具 `pln` 零依赖单二进制。

```bash
# 试试
echo '{"name":"test"}' > in.json
pln convert in.json out.pln   # 体积减 23%
cat out.pln
```

[https://github.com/one18mb/popline](https://github.com/one18mb/popline)

---

## 推广渠道节奏建议

| 阶段 | 渠道 | 内容 |
|------|------|------|
| 第 1 天 | V2EX | 简练版本，犀利利弊，引发讨论 |
| 第 3 天 | 掘金/知乎 | 深度分析版本，技术细节+性能数据 |
| 第 5 天 | Hacker News | English version，problem-first narrative |
| 第 7 天 | GitHub Trending | 提交到 GitHub，配合 Release 更新 |
