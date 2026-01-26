# 😡同事查日志太慢，我现场教他一套 grep 组合拳！

标题图：

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/29587979/1751537245903-b7bb5364-2693-49b4-bb69-f2c5af62a8f8.png)

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/29587979/1751538341364-d0fcd379-6f4f-4c14-8d14-60d70f2d4529.png)

描述：最近公司来了个新同事，年轻有活力，就是查日志的方式让我有点裂开。于是我当场掏出了一套我压箱底的“查日志组合拳”，一招一式手把手教他。他当场就“悟了”，连连称妙！

## 前言
最近公司来了个新同事，年轻有活力，就是查日志的方式让我有点裂开。

事情是这样的：他写的代码在测试环境报错了，报警信息也被钉钉机器人发到了我们群里。作为资深摸鱼战士，我寻思正好借机摸个鱼顺便指导一下新人，就凑过去看了眼。

结果越看我越急，差点当场喊出：“兄弟你是来写代码的，还是和日志谈恋爱的？”

**来看看他是怎么查日志的**

他先敲了一句：

```bash
tail -f a.log | grep "java.lang.NullPointerException"
```

想着等下次报错就能立刻看到。等了半天，终于蹦出来一行：

```bash
2025-07-03 11:38:48.339 [http-nio-8960-exec-1] [47gK4n32jEYvTYX8AYti48] [INFO] [GlobalExceptionHandler] java.lang.NullPointerException, ex: java.lang.NullPointerException
java.lang.NullPointerException: null
```

我提醒他：“这样看不到堆栈信息啊。”

他“哦”了一声，灵机一动，用 `vi`把整个文件打开，`/NullPointerException` 搜关键词，一个 `n` 一个 `n` 地翻……半分钟过去了，异常在哪都没找全，我都快给他跪下了。

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/jpeg/29587979/1751539698970-16ed20d2-1a19-410e-a593-e9466c78b571.jpeg)

于是我当场掏出了一套我压箱底的“查日志组合拳”，一招一式手把手教他。他当场就“悟了”，连连称妙，并表示想让我写成文章好让他发给他前同事看——因为他前同事也是这样查的……

现在，这套组合拳我也分享给你，希望你下次查日志的时候，能让你旁边的同事开开眼。

## 正式教学
核心的工具其实还是 `grep` 命令，下面我将分场景给你讲讲我的实战经验，保证你能直接套用！

### 场景一：查异常堆栈，不能只看一行！
Java 异常堆栈通常都是多行的，仅仅用 `grep "NullPointerException"` 只能看到最上面那一行，问题根源在哪你压根找不到。

**这时候使用 **`**grep**`** 的 **`**-A**`** (After) 参数来显示匹配行之后的N行。**

```bash
# 查找 NullPointerException，并显示后面 50 行
grep -A 50 "java.lang.NullPointerException" a.log
```

如果你发现异常太多，屏幕一闪而过，也可以用`less`加上分页查看：

```bash
grep -A 50 "java.lang.NullPointerException" a.log | less
```

在 `less` 视图中，你可以：

+ 使用 **箭头↑↓** 或 **Page Up/Down** 键来上下滚动
+ 输入 `G` 直接翻到末尾，方便快速查看最新的日志
+ 输入 `/Exception` 继续搜索
+ 按 `q` 键退出

这样你就能**第一时间拿到完整异常上下文信息**，告别反复 `vi` + `/` 的低效操作！

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/jpeg/29587979/1751534122387-52df2043-12f9-4b63-ba0a-91929fbf4aac.jpeg)

### 场景二：实时看新日志怎么打出来的
如果你的应用正在运行，并且你怀疑它会随时抛出异常，你可以实时监控日志文件的增长。

使用 `tail -f` 结合 `grep`：

```bash
# 实时监控 a.log 文件的新增内容，并只显示包含 "java.lang.NullPointerException" 的行及其后50行
tail -f a.log | grep -A 50 "java.lang.NullPointerException"
```

只要异常一出现，它就会自动打出来，堆栈信息也一并送到你面前！

+ 想停下？`Ctrl + C`
+ 想更准确？加 `-i` 忽略大小写，防止大小写拼错找不到

### 场景三：翻历史日志 or 查压缩日志
服务器上的日志一般都会按天或按大小分割并压缩，变成 `.log.2025-07-02.gz` 这种格式，查找这些文件的异常信息怎么办？

#### 🔍 查找当前目录所有 `.log` 文件：
```bash
# 在当前目录下查找所有以 .log 结尾的文件，-H 参数可以顺便打印出文件名
grep -H -A 50 "java.lang.NullPointerException" *.log
```

其中 `-H` 会帮你打印出**是哪个文件**中出现的问题，防止你找完还不知道是哪天的事。

#### 🔍 查找 `.gz` 文件（压缩日志）：
```bash
zgrep -H -A 50 "java.lang.NullPointerException" *.gz
```

`zgrep` 是专门处理 `.gz` 的 `grep`，它的功能和 `grep` 完全一样，**无需手动解压**，直接开整！

### 场景四：统计异常数量（快速判断异常是否频繁）
有时候你需要知道某个异常到底出现了多少次，是偶发还是成灾，使用 `grep -c`（count）：

```bash
grep -c "java.lang.NullPointerException" a.log
```

如果你要统计所有日志里的数量：

```bash
grep -c "java.lang.NullPointerException" *.log
```

### 其他常用的 grep 参数
| 参数 | 作用 |
| --- | --- |
| `-B N` | 匹配行之前的 N 行（Before） |
| `-A N` | 匹配行之后的 N 行（After） |
| `-C N` | 匹配行上下共 N 行（Context） |
| `-i` | 忽略大小写 |
| `-H` | 显示匹配的文件名 |
| `-r` | 递归搜索目录下所有文件 |


比如：

```bash
grep -C 25 "java.lang.NullPointerException" a.log
```

这个命令就能让你一眼看到异常前后的上下文，帮助定位代码逻辑是不是哪里先出问题了。



## 尾声
好了，这套组合拳我已经传授给你了，要是别人问你在哪学的，记得报我杆师傅的大名（doge）。

其实还有其他查日志的工具，比如`awk`、`wc` 等。

但是我留了一手，没有全部教给我这个同事，毕竟江湖规则，哪有一出手就把看家本领全都交出去的道理？

如果你也想学，先拜个师交个学费（点赞、收藏、关注），等学费凑够了，我下次再开新课，传授给大家~

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/29587979/1751537090452-dd68cff4-6bf6-415d-af2c-924c344a0ad1.png)

