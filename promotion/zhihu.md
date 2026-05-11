# 你每天都在读 JSON，但 JSON 不是为了人读而设计的

## 一个关于"读配置"的故事

下午三点，你打开生产环境的配置文件。

其实你只是想知道一件事：数据库连接的 `timeout` 是多少。

但你的眼睛却被迫做这件事：

```
往下翻... 跳过 array... 跳过另一个 object...
找到 database 那层... 数一下左括号在哪...
好的，timeout 在这个 block 里面... 看到了。
```

**你花 80% 的时间在"定位结构"，只有 20% 的时间在"读取信息"。**

这不是你的问题。这是 JSON 的问题。

## JSON 的原始设计目标

JSON 诞生于 2001 年，目标极其明确：**让 JavaScript 轻松解析**。它成功了。它成为了互联网的事实标准。

但 JSON 的语法——`{}` 配对、逗号分隔、引号包裹 keys——每一项都是为**机器解析**优化的，不是为**人眼扫描**优化的。

当你在终端里 `curl` 一个 API，或者在 GitHub 上看一个 PR diff，或者在日志文件里找一个字段时——JSON 的括号配对没有帮到你，反而阻碍了你。

## PopLine 的设计哲学

PopLine 把问题换了个方向思考：

> 如果语法是为"人读"优化，而非"机器解析"优化的，它会是什么样子？

答案是：**用行来表达结构，让每一行都是一条完整的信息。**

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

## 这不是替代，而是补充

PopLine 不是要取代 JSON——JSON 在机器间通信、深层嵌套数据、现有生态方面依然不可替代。

PopLine 做的是另一件事：**在"人读结构化数据"这个 JSON 从来没优化过的场景里，给出一个更好的选择。**

| 场景 | 推荐格式 |
|------|---------|
| 机器 ↔ 机器 API | JSON |
| 深层嵌套数据 | JSON |
| 配置文件 | **PopLine** |
| 结构化日志 | **PopLine** |
| PR diff / Code review | **PopLine** |
| 数据管道 / ETL | **PopLine** |
| 终端输出 / 调试 | **PopLine** |

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
