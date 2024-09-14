# [介绍](https://www.nushell.sh/zh-CN/book/#介绍)

这个项目的目标是继承 Unix Shell 中用管道把简单的命令连接在一起的理念，并将其带到更具现代风格的开发中。因此，Nushell 不是一个纯粹的 shell 或编程语言，而是通过将一种丰富的编程语言和功能齐全的 shell 结合到一个软件包中，实现了二者的连接。
Nu 并不试图成为万金油，而是把精力集中在做好这几件事上：
- 作为一个具有现代感的灵活的跨平台 Shell；
- 作为一种现代的编程语言，解决与数据有关的问题；
- 给予清晰的错误信息和干净的 IDE 支持

一些例子

```sh
> ls | sort-by size | reverse
╭────┬───────────────────────┬──────┬───────────┬─────────────╮
│ #  │         name          │ type │   size    │  modified   │
├────┼───────────────────────┼──────┼───────────┼─────────────┤
│  0 │ Gemfile.lock          │ file │   6.9 KiB │ 3 days ago  │
│  1 │ SUMMARY.md            │ file │   3.7 KiB │ 3 days ago  │
│  2 │ Gemfile               │ file │   1.1 KiB │ 3 days ago  │
│  3 │ LICENSE               │ file │   1.1 KiB │ 3 days ago  │
│  4 │ CONTRIBUTING.md       │ file │     955 B │ 9 mins ago  │
│  5 │ books.md              │ file │     687 B │ 3 days ago  │
...
```

你可以看到，为了达到这个目的，我们没有向 [`ls`](https://www.nushell.sh/commands/docs/ls.html) 传递命令行参数。取而代之的是，我们使用了 Nu 提供的 [`sort-by`](https://www.nushell.sh/commands/docs/sort-by.html) 命令来对 [`ls`](https://www.nushell.sh/commands/docs/ls.html) 命令的输出进行排序。为了在顶部看到最大的文件，我们还使用了 [`reverse`](https://www.nushell.sh/commands/docs/reverse.html) 命令。

过滤 [`ls`](https://www.nushell.sh/commands/docs/ls.html) 表的内容，使其只显示超过 1 千字节的文件。

```sh
> ls | where size > 1kb
╭───┬───────────────────┬──────┬─────────┬────────────╮
│ # │       name        │ type │  size   │  modified  │
├───┼───────────────────┼──────┼─────────┼────────────┤
│ 0 │ Gemfile           │ file │ 1.1 KiB │ 3 days ago │
│ 1 │ Gemfile.lock      │ file │ 6.9 KiB │ 3 days ago │
│ 2 │ LICENSE           │ file │ 1.1 KiB │ 3 days ago │
│ 3 │ SUMMARY.md        │ file │ 3.7 KiB │ 3 days ago │
╰───┴───────────────────┴──────┴─────────┴────────────╯
```

如果我们想显示那些正在活跃使用 CPU 的进程呢？就像我们之前对 [`ls`](https://www.nushell.sh/commands/docs/ls.html) 命令所做的那样，我们也可以利用 [`ps`](https://www.nushell.sh/commands/docs/ps.html) 命令返回给我们的表格来做到：

```sh
> ps | where cpu > 5
```

```sh
> date now
2022-03-07 14:14:51.684619600 -08:00
```

为了将获得的日期以表格形式展示，我们可以把它输入到 [`date to-table`](https://www.nushell.sh/commands/docs/date_to-table.html) 中：

```
> date now | date to-table
╭───┬──────┬───────┬─────┬──────┬────────┬────────┬──────────╮
│ # │ year │ month │ day │ hour │ minute │ second │ timezone │
├───┼──────┼───────┼─────┼──────┼────────┼────────┼──────────┤
│ 0 │ 2022 │     3 │   7 │   14 │     45 │      3 │ -08:00   │
╰───┴──────┴───────┴─────┴──────┴────────┴────────┴──────────╯
```

运行 [`sys`](https://www.nushell.sh/commands/docs/sys.html) 可以得到 Nu 所运行的系统的信息：

```
> sys
╭───────┬───────────────────╮
│ host  │ {record 6 fields} │
│ cpu   │ [table 4 rows]    │
│ disks │ [table 3 rows]    │
│ mem   │ {record 4 fields} │
│ temp  │ [table 1 row]     │
│ net   │ [table 4 rows]    │
╰───────┴───────────────────╯
```

这与我们之前看到的表格有些不同。[`sys`](https://www.nushell.sh/commands/docs/sys.html) 命令输出了一个在单元格中包含结构化表格而非简单值的表格。要查看这些数据，我们需要**获取**（[`get`](https://www.nushell.sh/commands/docs/get.html)）待查看的列：

```
> sys | get host
╭────────────────┬────────────────────────╮
│ name           │ Debian GNU/Linux       │
│ os version     │ 11                     │
│ kernel version │ 5.10.92-v8+            │
│ hostname       │ lifeless               │
│ uptime         │ 19day 21hr 34min 45sec │
│ sessions       │ [table 1 row]          │
╰────────────────┴────────────────────────╯
```

[`get`](https://www.nushell.sh/commands/docs/get.html) 命令让我们深入表的某一列内容中。在这里，我们要查看的是 `host` 列，它包含了 Nu 正在运行的主机的信息：操作系统名称、主机名、CPU，以及更多。让我们来获取系统上的用户名：

```
> sys | get host.sessions.name
╭───┬────╮
│ 0 │ st │
╰───┴────╯
```

现在，系统中只有一个名为 sophiajt 的用户。你会注意到，我们可以传递一个列路径（`host.sessions.name` 部分），而不仅仅是简单的列名称。Nu 会接受列路径并输出表中相应的数据。

你可能已经注意到其他一些不同之处：我们没有一个数据表，而只有一个元素：即字符串 `"sophiajt"`。

Nu 既能处理数据表，也能处理字符串。字符串是使用 Nu 外部命令的一个重要部分。

让我们看看字符串在 Nu 外部命令里面是如何工作的。我们以之前的例子为例，运行外部的 `echo` 命令（`^` 告诉 Nu 不要使用内置的 [`echo`](https://www.nushell.sh/commands/docs/echo.html) 命令）：

```
> sys | get host.sessions.name | each { |it| ^echo $it }
sophiajt
```

### [获取帮助](https://www.nushell.sh/zh-CN/book/#获取帮助)

任何 Nu 的内置命令的帮助文本都可以通过 [`help`](https://www.nushell.sh/commands/docs/help.html) 命令来找到。要查看所有命令，请运行 `help commands` 。你也可以通过执行 `help -f <topic>` 来搜索一个主题

# nushell 与 bash

Nushell 既是一种编程语言，也是一种 Shell。正因为如此，它有自己的方式来处理文件、目录、网站等等。我们对其进行了建模，以使其与你可能熟悉的其他 Shell 的工作方式接近。其中管道用于将两个命令连接在一起：

```
> ls | length
```

Nushell 也支持其他常见的功能，例如从之前运行的命令中获取退出代码（Exit Code）。

虽然它确实有这些功能，但 Nushell 并不是 Bash。Bash 的工作方式以及一般的 POSIX 风格，并不是 Nushell 所支持的。例如，在 Bash 中你可以使用：

```
> echo "hello" > output.txt
```

在 Nushell 中，我们使用 `>` 作为大于运算符，这与 Nushell 的语言特质比较吻合。取而代之的是，你需要用管道将其连接到一个可以保存内容的命令：

```
> "hello" | save output.txt
```

**以 Nushell 的方式思考：** Nushell 看待数据的方式是，数据在管道中流动，直到它到达用户或由最后的命令处理。你可以简单地输入数据，从字符串到类似 JSON 的列表和记录，然后使用 `|` 将其通过管道发送。Nushell 使用命令来执行工作并生成更多数据。学习这些命令以及何时使用它们有助于你组合使用多种管道。

## [把 Nushell 想象成一种编译型语言](https://www.nushell.sh/zh-CN/book/thinking_in_nu.html#把-nushell-想象成一种编译型语言)

Nushell 设计的一个重要部分，特别是它与许多动态语言不同的地方是，Nushell 将你提供给它的源代码转换成某种可执行产物，然后再去运行它。Nushell 没有 `eval` 功能，因此也不允许你在运行时继续拉入新的源代码。这意味着对于诸如引入文件使其成为你项目的一部分这样的任务，需要知道文件的具体路径，就如同 C++ 或 Rust 等编译语言中的文件引入一样。
`source` 命令将引入被编译的源码，但前面那行 `save` 命令还没有机会运行。Nushell 运行整个程序块就像运行一个文件一样，而不是一次运行一行。在这个例子中，由于 `output.nu` 文件是在“编译”步骤之后才创建的，因此 `source` 命令在解析时无法从其中读取定义。

另一个常见的问题是试图动态地创建文件名并 `source`，如下：

```
> source $"($my_path)/common.nu"
```

这就需要求值器（Evaluator）运行并对字符串进行求值（Evaluate），但不幸的是，Nushell 在编译时就需要这些信息。

**以 Nushell 的方式思考：** Nushell 被设计为对你输入的所有源代码使用一个单一的“编译”步骤，这与求值是分开的。这将允许强大的 IDE 支持，准确的错误提示，并成为第三方工具更容易使用的语言，以及在未来甚至可以有更高级的输出，比如能够直接将 Nushell 编译为二进制文件等。

## [变量是不可变的](https://www.nushell.sh/zh-CN/book/thinking_in_nu.html#变量是不可变的)

对于来自其他语言的人来说（Rustaceans 除外），另一个常见的令人惊愕之处是 Nushell 的变量是不可变的（事实上，有些人已经开始称它们为“**常量**”来反映这一点）。接触 Nushell，你需要花一些时间来熟悉更多的函数式风格，因为这往往有助于写出与**不可变的变量**最相容的代码。

你可能想知道为什么 Nushell 使用不可变的变量，在 Nushell 开发的早期，我们决定看看我们能在语言中使用多长时间的以数据为中心的函数式风格。最近，我们在 Nushell 中加入了一个关键的功能，使这些早期的实验显示出其价值：并行性。通过在任何 Nushell 脚本中将 [`each`](https://www.nushell.sh/commands/docs/each.html) 切换到 [`par-each`](https://www.nushell.sh/commands/docs/par-each.html) ，你就能够在“输入”上并行地运行相应的代码块。这是可能的，因为 Nushell 的设计在很大程度上依赖于不可变性、组合和流水线。

Nushell 的变量是不可变的，但这并不意味着无法表达变化。Nushell 大量使用了 "Shadowing" 技术（变量隐藏）。变量隐藏是指创建一个与之前声明的变量同名的新变量。例如，假设你有一个 `$x` 在当前作用域内，而你想要一个新的 `$x` 并将其加 1：

```
let x = $x + 1
```

这个新的 `x` 对任何跟在这一行后面的代码都是可见的。谨慎地使用变量隐藏可以使变量的使用变得更容易，尽管这不是必须的。

循环计数器是可变变量的另一种常见模式，它被内置于大多数迭代命令中，例如，你可以使用 [`each`](https://www.nushell.sh/commands/docs/each.html) 上的 `-n` 标志同时获得每个元素的值和索引：

```
> ls | enumerate | each { |it| $"Number ($it.index) is size ($it.item.size)" }
```

你也可以使用 [`reduce`](https://www.nushell.sh/commands/docs/reduce.html) 命令来达到上述目的，其方式与你在循环中修改一个变量相同。例如，如果你想在一个字符串列表中找到最长的字符串，你可以这样做：

```
> [one, two, three, four, five, six] | reduce {|curr, max|
    if ($curr | str length) > ($max | str length) {
        $curr
    } else {
        $max
    }
}
```

**以 Nushell 的方式思考：** 如果你习惯于使用可变的变量来完成不同的任务，那么你将需要一些时间来学习如何以更加函数式的方式来完成每个任务。Nushell 有一套内置的能力来帮助处理这样的模式，学习它们将帮助你以更加 Nushell 的风格来写代码。由此带来的额外的好处是你可以通过并行运行你的部分代码来加速脚本执行。

## [Nushell 的环境是有作用域的](https://www.nushell.sh/zh-CN/book/thinking_in_nu.html#nushell-的环境是有作用域的)

Nushell 从编译型语言中获得了很多设计灵感，其中一个是语言应该避免全局可变状态。Shell 经常通过修改全局变量来更新环境，但 Nushell 避开了这种方法。

在 Nushell 中，代码块可以控制自己的环境，因此对环境的改变是发生在代码块范围内的。

在实践中，这可以让你用更简洁的代码来处理子目录，例如，如果你想在当前目录下构建每个子项目，你可以运行：

```
> ls | each { |it|
    cd $it.name
    make
}
```

`cd` 命令改变了 `PWD` 环境变量，这个变量的改变只在当前代码块有效，如此即可允许每次迭代都从当前目录开始，进入下一个子目录。

环境变量具有作用域使命令更可预测，更容易阅读，必要时也更容易调试。Nushell 还提供了一些辅助命令，如 [`def --env`](https://www.nushell.sh/commands/docs/def.html)、[`load-env`](https://www.nushell.sh/commands/docs/load-env.html)，作为对环境变量进行批量更新的辅助方法。

>
>TIP
>
>这里有一个例外，[`def --env`](https://www.nushell.sh/commands/docs/def.html) 允许你创建一个可以修改并保留调用者环境的命令
>


**以 Nushell 的方式思考：** 在 Nushell 中，没有全局可修改变量的编码最佳实践延伸到了环境变量。使用内置的辅助命令可以让你更容易地处理 Nushell 中的环境变量问题。利用环境变量对代码块具有作用范围这一特性，也可以帮助你写出更简洁的脚本，并与外部命令互动，而不需要在全局环境中添加你不需要的东西。
# [在系统中四处移动](https://www.nushell.sh/zh-CN/book/moving_around.html#在系统中四处移动)

## [通配符](https://www.nushell.sh/zh-CN/book/moving_around.html#通配符)

上述可选参数 `*.md` 中的星号（*）有时被称为通配符（wildcards）或 Glob，它让我们可以匹配任何东西。你可以把 glob `*.md` 理解为“匹配以 `.md` 结尾的任何文件名”。

最通用的通配符是 `*`，能够匹配所有路径。它经常和其他模式（pattern）组合使用，比如 `*.bak` 和 `temp*`。

Nu 也使用现代 Globs，它允许你访问更深的目录。比如，`ls **/*.md` 将递归地罗列当前目录下、所有后缀为 `.md` 的非隐藏文件:

```
 ls **/*.md
───┬───────────────────────────────────────────┬──────┬─────────┬───────────
 # │ name                                      │ type │ size    │ modified
───┼───────────────────────────────────────────┼──────┼─────────┼───────────
 0 │ CODE_OF_CONDUCT.md                        │ File │  3.4 KB │ 5 days ago
 1 │ CONTRIBUTING.md                           │ File │   886 B │ 5 days ago
 2 │ README.md                                 │ File │ 15.0 KB │ 5 days ago
 3 │ TODO.md                                   │ File │  1.6 KB │ 5 days ago
 4 │ crates/nu-source/README.md                │ File │  1.7 KB │ 5 days ago
 5 │ docker/packaging/README.md                │ File │  1.5 KB │ 5 days ago
 6 │ docs/commands/README.md                   │ File │   929 B │ 5 days ago
 7 │ docs/commands/alias.md                    │ File │  1.7 KB │ 5 days ago
 8 │ docs/commands/append.md                   │ File │  1.4 KB │ 5 days ago
```

- `**` 表示“从这里开始的任何目录中”；
- `*.md` 表示“任意后缀为 `.md` 的文件名”（不包括隐藏文件，要额外添加 `--all, -a` 选项）；
- 除了 `*`，还有 `?` 用来匹配单个字符。比如，可以使用 `p???` 模式匹配 `port` 字符串。

结合 [字符串的处理](https://www.nushell.sh/zh-CN/book/working_with_strings.html) 能够写出更强大的模式。但是，请牢记 Nu 类似一种 [编译型语言](https://www.nushell.sh/zh-CN/book/thinking_in_nu.html#%E6%8A%8A-nushell-%E6%83%B3%E8%B1%A1%E6%88%90%E4%B8%80%E7%A7%8D%E7%BC%96%E8%AF%91%E5%9E%8B%E8%AF%AD%E8%A8%80)。

## [改变当前目录](https://www.nushell.sh/zh-CN/book/moving_around.html#改变当前目录)

```
> cd new_directory
```

要从当前目录换到一个新目录，我们使用 [`cd`](https://www.nushell.sh/commands/docs/cd.html) 命令。就像在其他 Shells 中一样，我们可以使用目录的名称，或者如果我们想进入父目录，我们可以使用 `..` 的快捷方式。

如果 [`cd`](https://www.nushell.sh/commands/docs/cd.html) 被省略，只给出一个路径本身，也可以改变当前工作目录：

```
> ./new_directory
```

>WARNING
>
>用 [`cd`](https://www.nushell.sh/commands/docs/cd.html) 改变目录会改变 `PWD` 环境变量。这意味着目录的改变会保留到当前代码块中，一旦你退出这个代码块，你就会返回到以前的目录。你可以在 [环境篇](https://www.nushell.sh/zh-CN/book/environment.html) 中了解更多关于这方面的信息。

## [文件系统命令](https://www.nushell.sh/zh-CN/book/moving_around.html#文件系统命令)

Nu 还提供了一些基本的文件系统命令，并且可以跨平台工作。

我们可以使用 [`mv`](https://www.nushell.sh/commands/docs/mv.html) 命令将一个目录或文件从一个地方移动到另一个地方：

```
> mv item location
```

我们可以通过 [`cp`](https://www.nushell.sh/commands/docs/cp.html) 命令把一个目录或文件从一个地方复制到另一个地方：

```
> cp item location
```

>DANGER
>
>`cp` 在 Windows 系统，即使目标文件存在，命令也没有附加 `--force, -f`，目标文件仍会被覆盖！

我们也可以通过 [`rm`](https://www.nushell.sh/commands/docs/rm.html) 命令删除一个目录或文件：

```
> rm item
```

这三个命令也可以使用我们先前看到的 [`ls`](https://www.nushell.sh/commands/docs/ls.html) 的 Glob 功能。

最后，我们可以使用 [`mkdir`](https://www.nushell.sh/commands/docs/mkdir.html) 命令创建一个新目录：

```
> mkdir new_directory
```