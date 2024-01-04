# Home

For full documentation visit [mkdocs.org](https://www.mkdocs.org).

# 命令行环境

* 同时执行多个不同的进程并追踪它们的状态
* 停止或暂停某个进程以及如何使进程在后台运行
* 改善 shell 及其他工具的工作流的方法

## 任务控制

某些情况下我们需要**中断**正在执行的任务，比如当一个命令需要执行很长时间才能完成时（假设我们在使用 find 搜索一个非常大的目录结构）。大多数情况下，可以使用 `Ctrl-C` 来停止命令的执行。

### 结束进程
shell 会使用 UNIX 提供的信号机制执行进程间通信。
>- 当一个进程接收到信号时，它会停止执行、处理该信号并基于信号传递的信息来改变其执行。因此信号可以看成是一种软件中断。
>- 当输入 `Ctrl-C` 时，shell 会发送一个SIGINT 信号到进程。
---
下面的程序捕获信号`SIGINT`，忽略它的基本操作，它并不会让程序停止。为了停止这个程序，需要使用`SIGQUIT`信号，通过输入`Ctrl-\`可以发送该信号。
```python
#!/usr/bin/env python
import signal, time

def handler(signum, time):
    print("\nI got a SIGINT, but I am not stopping")

signal.signal(signal.SIGINT, handler)
i = 0
while True:
    time.sleep(.1)
    print("\r{}".format(i), end="")
    i += 1
```
程序执行结果
```
$ python sigint.py
24^C
I got a SIGINT, but I am not stopping
26^C
I got a SIGINT, but I am not stopping
30^\[1]    39913 quit       python sigint.pyƒ
```
---
`SIGTERM` 是一个更加通用的、也更加优雅地退出信号。使用 `kill` 命令可以发出这个信号, 语法是： `kill -TERM PID`

### 暂停和后台执行进程

信号可以让进程做其他的事情，而不仅仅是终止它们。

例如，`SIGSTOP` 会让进程暂停。在终端中，键入 `Ctrl-Z` 会让 shell 发送 `SIGTSTP` 信号，`SIGTSTP`是 `Terminal Stop` 的缩写。

一些命令：

>`fg` 
> - `fg` 会将一个暂停（或后台）的工作恢复到前台执行。
> - 当执行`fg`，如果有多个作业，shell 通常会将最近的一个作业带回前台。
> - 也可以通过指定作业号（如 fg %1）来选择特定的作业恢复到前台

>`bg` 
> - `bg` 会将一个暂停的工作放到后台继续执行，但不会占据命令行界面。
> - 该作业会在后台异步运行，用户可以继续在当前 shell 进行其他操作。
> - 可以通过作业号（如 bg %1）来指明要恢复的特定后台作业。

>`jobs`
> - 这个命令会列出当前 shell 会话中的所有作业以及它们的状态。一个“作业”可以是一个单独的进程或一组进程。每个作业都有一个编号，前面通常带有一个 % 字符。
> - 可以通过作业编号来引用后台作业，在 jobs 列表输出中每个作业前都会有一个编号，你可以使用 % job_number 的形式与 fg 或 bg 命令结合使用，来控制这些作业。

一些参数

> - `&` 后缀可以让命令在直接在后台运行，这使得可以直接在 shell 中继续做其他操作
> - `$!` 如果要选择最近的一个任务，可以使用 `$!` 这一特殊参数。

Example

![1](../pics/1.png)

### 终端多路复用