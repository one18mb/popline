# PopLine：一个让人不再数括号的 JSON 替代品

## 背景

JSON 的问题是：**明明是人读，语法却是为机器设计的。**

看配置、查日志、审 diff——你其实只想找字段的值，却要先跨过 `{}` `""` `,` 这些机器符号。

**PopLine 把结构信息从"配对符号"转成了"行"。**

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
pln convert package.json package.pln   # 23% smaller
pln validate schema.pln                  # 校验
```

下载：[github.com/one18mb/popline-cli/releases](https://github.com/one18mb/popline-cli/releases)

## 适合什么场景

| 场景 | 适合 | 原因 |
|------|------|------|
| 配置文件 | ✅ | 扁平，grep 直达，diff 精确 |
| 结构化日志 | ✅ | 空行分隔，逐行解析，grep 友好 |
| pipeline 中间数据 | ✅ | 一行一条，管道原语畅通 |
| 代码仓库配置 | ✅ | diff 好看，review 轻松 |
| 复杂嵌套数据 | 需要插件 | 编辑器折叠辅助（VS Code / Vim 插件已就绪） |
| 机器间通信 | JSON 继续用 | 生态在那，PopLine 做补充 |

## 一句话

**JSON 给机器，PopLine 给人。**

---

项目主页：[github.com/one18mb/popline](https://github.com/one18mb/popline)

VSCode 插件：[popline-vscode](https://github.com/one18mb/popline-vscode)
Vim 插件：[popline-vim](https://github.com/one18mb/popline-vim)

*欢迎 Star \* 欢迎 Issue \* 欢迎 PR*
