# PopLine：JSON 在实际场景中到底够不够用？

## 先看几组数

JSON 的 `test-package.json`（17011 字节），去掉键名引号、逗号、闭合括号后是 13074 字节——**23% 的字节是语法噪声**。

这不是"为了可读性牺牲效率"——解析也更快：

```
C 序列化：     275 ms (JSON) → 186 ms (PopLine)   -32%
Python 序列化： 935 ms (JSON) → 213 ms (PopLine)   -77%
Go 序列化：    1486 ms (JSON) → 552 ms (PopLine)   -63%
Rust 序列化：  6783 ms (JSON) → 2323 ms (PopLine)  -66%
```

对人、对机器都有收益。

**PopLine 的思路很简单：把结构信息从配对符号挪到行上。**

## 对比

**JSON:**

```json
{"name":"popline","version":2,"tags":["serialization","line-based"]}
```

**PopLine:**

```
{
name: "popline"
version: 2
tags: [
"serialization"
"line-based"
```

改动就两行：

| | JSON | PopLine |
|---|------|---------|
| 容器 | `{}` `[]` 配对 | `{` `[` 单行，`N ` 前缀弹出 |
| 分隔 | `,` | 换行 |
| key | `"key"` | `key:` 裸写 |
| 转义 | `\"` `\n` `\t`... | `""` 唯一转义 |

## 效果

| 指标 | JSON | PopLine | 差距 |
|------|------|---------|------|
| 体积 | 17011 B | 13074 B | **-23%** |
| C 序列化 | 275 ms /5000次 | 186 ms | **-32%** |
| Go 序列化 | 1486 ms | 552 ms | **-63%** |
| Python 序列化 | 935 ms | 213 ms | **-77%** |
| Rust 序列化 | 6783 ms | 2323 ms | **-66%** |

## 最实际的好处

**1. grep 即查**

```bash
# 查配置
cat config.pln | grep port
port: 8080

# JSON 的话......
cat config.json | grep port
# "port": 8080,
# 还行，但这是 JSON 本身语法不是业务内容
```

**2. diff 精确到行**

```diff
-port: 8080
+port: 9090
```

而不是 JSON 的整块污染。

**3. 空行天然流式**

多条消息用空行隔开，逐条解析。比 JSON Lines 更直观。

## 多语言支持

实现全覆盖，一个语法规则，各语言接口都参考了标准 JSON 库的命名：

| 语言 | 解析 | 序列化 |
|------|------|--------|
| C | `pln_loads()` | `pln_dumps()` |
| Python | `pln.loads()` | `pln.dumps()` |
| Go | `pln.Unmarshal()` | `pln.Marshal()` |
| Rust | `pln::from_str()` | `pln::to_string()` |
| Java | `Pln.parse()` | `Pln.stringify()` |
| TS/JS | `Pln.parse()` | `Pln.stringify()` |

### CLI 工具

```bash
# 零依赖单二进制
pln convert test-package.json test-package.pln   # 23% smaller
pln validate schema.pln                  # 校验
```

下载：[github.com/one18mb/popline-cli/releases](https://github.com/one18mb/popline-cli/releases)

## 坦诚说一个弱点和它换来的东西

**PopLine 的弱点：** 嵌套结构没有 JSON 直观。`N ` 前缀要理解一下才知层级，不像 `{}` 一眼看出。

**PopLine 换来的：**

| 维度 | PopLine | 对比 JSON |
|------|---------|-----------|
| 体积 | 13074 B | **小 23%**，越深层嵌套省越多 |
| C 序列化 | 186 ms | **快 32%** |
| Python 序列化 | 213 ms | **快 77%** |
| Go 序列化 | 552 ms | **快 63%** |
| grep 检索 | `grep port:` | 直出结果，不需要 jq |
| diff | 改一行显一行 | PR review 不污染 |
| 流式 | 空行分隔 | 天然支持多消息 |

## 一句话

**PopLine 用层级直观性，换了存储/传输/检索/性能的全维度提升。这比它更好的交易。**

---

项目主页：[github.com/one18mb/popline](https://github.com/one18mb/popline)

VSCode 插件：[popline-vscode](https://github.com/one18mb/popline-vscode)
Vim 插件：[popline-vim](https://github.com/one18mb/popline-vim)

*欢迎 Star \* 欢迎 Issue \* 欢迎 PR*
