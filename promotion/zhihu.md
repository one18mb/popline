# 一个 JSON 的"够用"不意味着"最优"

## 先看三个真实问题

**场景一：改一个配置项，PR 变成了 30 行 diff。**

你只是把端口从 8080 改成 9090，但在 JSON 里：

```diff
 {
   "server": {
-    "port": 8080,
     "host": "0.0.0.0",
-    "features": ["logging"],
+    "port": 9090,
+    "features": ["logging", "metrics"],
```

改了一行，diff 显示了六行。Reviewer 要花时间判断哪些是真正的改动，哪些只是被"携带"出来的。

**场景二：grep 日志找 error，但一行 800 个字符。**

```json
{"level":"error","message":"timeout","latency_ms":5030,"request_id":"req_12345","service":"gateway","path":"/api/users","method":"POST","user_id":"u_67890","ip":"10.0.0.1","user_agent":"Mozilla/5.0","timestamp":"2026-05-12T10:00:00Z"}
```

grep 是能搜到，但输出几乎不可读。想提取 `latency_ms` 和 `request_id`，要么用 jq，要么用 awk——你本来只想查个日志，却写了条管道命令。

**场景三：json.dumps 把一个简单对象变成了 17011 字节。**

一个 VS Code 扩展的 package.json。里面 3923 个字节是**语法符号**——引号、逗号、括号。也就是说 **23% 的字节不是数据，是噪声。**

这些问题不是 JSON"错了"。JSON 的设计目标一直是"机器可解析的轻量交换格式"，它做到了。但它只是**够用**，不是**最优**。

## 问题是：在真实使用中，JSON 被用来做什么？

| 用途 | 频率 | 是否 JSON 擅长 |
|------|------|---------------|
| 配置管理 | 每天 | ⚠️ 嵌套语法导致 diff 膨胀 |
| 日志记录 | 每天 | ⚠️ 行内结构不利 grep |
| API 调试 | 每天 | ⚠️ 括号配对干扰扫描 |
| 数据交换 | 每时每刻 | ✅ 作为 wire format 很成熟 |
| 深层嵌套数据 | 偶尔 | ✅ 括号配对在此场景帮了忙 |

JSON 在"数据交换"场景是合格的，但在"人每天和结构化数据交互"的高频场景中，它的语法反而成了阻碍。而且在机器端它也不是最优的。

## PopLine 的优化思路

> 所有语法设计，都为高频场景优化，而非为万能场景妥协。

JSON 的 `{}` 配对在深层嵌套时帮了你，但在 80% 的浅层场景中只是噪声。PopLine 的做法是：**用行替代配对，让每一行自包含。**

所以：

```json
{
  "server": {
    "port": 8080,
    "host": "0.0.0.0",
    "features": ["logging", "auth"],
    "database": {
      "url": "postgres://localhost:5432/db",
      "pool": 10
    }
  }
}
```

变成了：

```
{
server: {
port: 8080
host: "0.0.0.0"
features: [
"logging"
"auth"
1 database: {
url: "postgres://localhost:5432/db"
pool: 10
```

**改动只有两条：**

| JSON | PopLine |
|------|---------|
| `{}` `[]` 配对 | **单行** `{` 或 `[` 开头，`N ` 前缀弹出 |
| 逗号分隔 | **换行**分隔 |
| 键名要引号 | **键名裸写** |
| 字符串要转义 `\"` | **`""` = `"`**，唯一转义 |

就这两条，带来三个质的改变。

## 改变一：grep 直达字段

你要找 `port` 的值：

```bash
# JSON 做不到
cat config.json | grep port
# "port": 8080,  <— 输出很干净，但这是 JSON 本身语法的一部分

# PopLine
cat config.pln | grep port
port: 8080  <— 这就是你想要的，每一行都是业务数据，没有语法噪声
```

对于日志场景，差距更明显：

```bash
# 日志文件用 JSON
2026-05-12 10:00:00 {"level":"error","service":"api","message":"timeout","latency_ms":5030,"request_id":"req_12345","path":"/api/users","method":"POST","user_id":"u_67890"}

# grep level=error 和 latency 的关系？需要 jq：
cat log.jsonl | jq 'select(.level == "error" and .latency_ms > 1000)'

# PopLine 日志
2026-05-12 10:00:00
{
level: "error"
service: "api"
message: "timeout"
latency_ms: 5030
request_id: "req_12345"
path: "/api/users"
method: "POST"
user_id: "u_67890"

# grep level=error 命中那条日志，再 grep latency_ms 拿到值：
grep -A1 "level: \"error\"" log.pln | grep "latency_ms"
latency_ms: 5030
```

**`jq` 是很强大，但 `grep` 是每个人都会的命令行基本功。** PopLine 让这些基本功直接在结构化数据上生效。

## 改变二：改一行，diff 就是一行

**这是 PopLine 最惊艳的特性。**

JSON 中改了 `port` 的值：

```diff
 {
   "server": {
-    "port": 8080,
     "host": "0.0.0.0",
-    "features": ["logging", "auth"],
+    "port": 9090,
+    "features": ["logging", "auth", "metrics"],
     "database": {
       "url": "...",
       "pool": 10
     }
   }
 }
```

看到问题了吗？`port` 的改动和 `features` 的改动**在 diff 里纠缠在一起**。

PopLine 中改同样的东西：

```diff
 {
 server: {
 port: 8080
 host: "0.0.0.0"
 features: [
 "logging"
 "auth"
+ "metrics"
1 database: {
url: "..."
pool: 10
```

**改的是什么行，diff 就只显示什么行。**

在代码评审中，这意味着 reviewer 不用在几十行改动里找真正有意义的变化。在 Git blame 里，每行数据都能准确追溯到真正的提交。

## 改变三：体积小 23%，全语言快 1.5-4.5x

这不是理论分析，是真实数据测试结果。测试数据：VS Code 扩展的 `package.json`（17011 字节）。

| 指标 | JSON | PopLine | 提升 |
|------|------|---------|------|
| **文件体积** | 17011 B | 13074 B | **-23%** |
| **C 解析** | 520 ms | 387 ms | **-26%** |
| **C 序列化** | 275 ms | 186 ms | **-32%** |
| **Python 序列化** | 935 ms | 213 ms | **-77%** |
| **Go 解析** | 1929 ms | 1308 ms | **-32%** |
| **Go 序列化** | 1486 ms | 552 ms | **-63%** |

体积优势来自语法层面：键名无引号、无逗号、无闭合括号。

性能优势来自解析复杂度降低：行结构天然可分块处理，无需递归配对。

## 优势总结：无论对人还是对机器都有收益

PopLine 的优势不是"对人好 vs 对机器好"的取舍，而是同一个设计选择（行结构替代配对结构）在两端同时产生收益：

| 维度 | 对人 | 对机器 |
|------|------|--------|
| **体积** | 更少的屏幕噪声 | 小 23%，传输/存储都省 |
| **性能** | — | 快 1.5-4.5x（依赖具体语言） |
| **语法** | grep 直达，diff 精确到行 | 无需递归配对，解析路径更短 |
| **流式** | 空行分隔，肉眼即可区分 | 无需 delimitor 转义，块边界天然 |
| **嵌套** | 浅层无需数括号 | 不支持深层\n | 可以深层，需编辑器辅助 |

**JSON 适合的场景：** 深层嵌套的数据交换、对 JSON 生态有强依赖的现有系统

**PopLine 适合的场景：**

| 场景 | 原因 |
|------|------|
| 配置文件 | grep 直达，diff 精确，体积小 |
| 结构化日志 | 空行分隔流式，grep 直接定位 |
| 数据管道 / ETL | 行结构天然适配 pipe 哲学 |
| API 响应体 | 体积小 23%，序列化快 |
| Git 仓库中的结构化数据 | 行级 diff + blame 准确 |

## 多语言支持

| 语言 | 项目 | 使用方式 |
|------|------|---------|
| **C** | [popline-c](https://github.com/one18mb/popline-c) | `pln_loads()` / `pln_dumps()` |
| **Python** | [popline-py](https://github.com/one18mb/popline-py) | `import pln; pln.loads()` |
| **Go** | [popline-go](https://github.com/one18mb/popline-go) | `pln.Unmarshal()` / `pln.Marshal()` |
| **Rust** | [popline-rust](https://github.com/one18mb/popline-rust) | `pln::from_str()` / `pln::to_string()` |
| **Java** | [popline-java](https://github.com/one18mb/popline-java) | `Pln.parse()` / `Pln.stringify()` |
| **TypeScript** | [popline-js](https://github.com/one18mb/popline-js) | `Pln.parse()` / `Pln.stringify()` |
| **CLI** | [popline-cli](https://github.com/one18mb/popline-cli) | `pln convert` / `pln validate` |

## 一分钟体验

```bash
# 1. 安装 CLI
gcc -O2 -o pln main.c popline.c popline_parser.c popline_json.c cjson/cJSON.c -lm

# 2. 准备一份 JSON
echo '{"server":"web-01","port":8080,"active":true,"tags":["production","primary"]}' > server.json

# 3. 转成 PopLine
./pln convert server.json server.pln

# 4. 看看效果
cat server.pln
```

输出：

```
{
server: "web-01"
port: 8080
active: true
tags: [
"production"
"primary"
```

体积对比：

```bash
$ wc -c *.json *.pln
  83 server.json
  65 server.pln    # 小了 22%
```

---

**PopLine 是开源的** — [github.com/one18mb/popline](https://github.com/one18mb/popline)

*不是要取代 JSON，而是让人在每天读 JSON 的那些场景里，不用再数括号了。*
