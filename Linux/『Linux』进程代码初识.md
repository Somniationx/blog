

# 进程代码初识

## `proc.c`

```c
#include <stdio.h>
#include <unistd.h>

int main() {
    while(1) {
        printf("I'm a process!\n");
        sleep(1);
    }
    return 0;
}
```

1.`#include <stdio.h>` 和 `#include <unistd.h>`：这两行是C语言的预处理指令，它们告诉编译器在编译程序之前将标准输入输出库（`stdio.h`）和**Unix系统调用库**。

2.`sleep(1);`：这是一个**系统调用**，它使进程休眠1秒钟。也就是说，每次循环执行后，程序会休眠1秒钟，然后再次执行循环。

> 这个程序是一个简单的无限循环，每秒打印一次"I'm a process!"，并在每次打印后休眠1秒钟。这可以用于演示一个运行中的进程，它以1秒的间隔输出一条消息。如果要终止程序，您可以手动中断它，例如按下`Ctrl+C`。

## `Makefile`

```bash
proc:proc.c 
	gcc -o proc proc.c
.PHONY:clean
clean:
	rm -f proc
```

1. `proc: proc.c`：这一行指示Make工具如何构建目标"proc"。它告诉Make工具，要生成目标"proc"，它依赖于"proc.c"文件。如果"proc.c"文件发生了变化，Make工具将重新编译"proc"目标。
2. `gcc -o proc proc.c`：这是实际的编译命令。它告诉`gcc`编译器将"proc.c"源文件编译成一个可执行文件，文件名为"proc"。"-o"选项用于指定输出文件的名称。
3. `.PHONY: clean`：这一行定义了一个伪目标，即"clean"。伪目标通常用于执行一些特殊的任务，而不是生成文件。在这种情况下，它用于清理生成的可执行文件。
4. `clean:`：这一行表示"clean"目标的开始。它告诉Make工具，下面的命令将执行"clean"操作。
5. `rm -f proc`：这是"clean"目标的命令部分。它使用`rm`命令来删除名为"proc"的文件。`-f`选项告诉`rm`命令在文件不存在时不报错，以避免出现错误消息。这个命令的效果是删除生成的可执行文件。

## 执行

上述的Makefile文件本身并不产生输出，它用于编译和清理程序。要执行该Makefile文件并生成可执行文件，请按照以下步骤进行操作：

1. 创建一个名为`proc.c`的C源代码文件，将其中的内容设置之前提供的程序代码。
2. 在同一目录下创建一个名为`Makefile`的文件，并将其内容设置为您之前提供的Makefile代码。
3. 打开终端，并导航到包含`proc.c`和`Makefile`的目录。
4. 执行以下命令以编译程序：

```bash
make
```

这将使用Makefile中的规则来编译`proc.c`并生成可执行文件`proc`。

1. 一旦编译完成，您可以运行生成的可执行文件`proc`：

```bash
./proc
```

这将启动程序，它会每秒输出一次"I'm a process!"，并**无限循环运行**。

如果您想要清理生成的可执行文件，可以运行以下命令：

```bash
make clean
```

这将删除名为`proc`的可执行文件。

请注意，**输出将在终端上显示**，以每秒一次的频率打印"I'm a process!"。要停止程序的运行，您可以在终端中按下`Ctrl+C`来中断它。

## 结果

![](https://image-somn.oss-cn-beijing.aliyuncs.com/proc.png)

```bash
ps axj | head -1 && ps axj | grep proc | grep -v grep
```

1. `ps axj | head -1`：这部分命令首先运行`ps axj`，它用于显示所有进程的详细信息。然后，通过管道`|`将其输出传递给`head -1`，`head`命令用于显示输出的前几行，`-1`表示只显示第一行，即进程表的标题行。
2. `&&`：这是逻辑操作符，用于在前一个命令成功执行后才执行下一个命令。
3. `ps axj | grep proc | grep -v grep`：这部分命令运行`ps axj`以获取进程列表，然后通过两次`grep`过滤出包含关键字"proc"的进程。具体来说：
   - 第一个`grep`命令`grep proc`用于筛选包含"proc"关键字的行。
   - 第二个`grep`命令`grep -v grep`用于排除包含"grep"关键字的行，以避免匹配到自身的`grep`进程。

> 最终的输出显示了进程列表中包含关键字"proc"的进程信息，包括它们的父进程ID（PPID）、进程ID（PID）、进程组ID（PGID）、会话ID（SID）、终端（TTY）、终端进程组ID（TPGID）、状态（STAT）、用户ID（UID）、累计CPU时间（TIME）和命令（COMMAND）。

根据输出，可以看到一个进程具有PID 24055，它是通过执行`./proc`命令启动的。

## 终止进程

```bash
kill -9 24055 //24055为上述查询到的进程id
```

执行`kill -9 24055`命令会发送一个**强制终止信号（SIGKILL）**给进程PID为24055的进程。这会立即终止该进程，而不会给予它进行清理或保存数据的机会。

请注意，使用`kill -9`是一种强制终止进程的方法，应该小心使用，因为它不允许进程进行善后工作。只有在有必要时才应该使用这种信号，例如当一个进程不响应其他终止信号并且必须立即停止时。

在执行`kill -9 24055`后，进程PID 24055将被终止，不再运行。**如果需要终止其他进程，请将其PID替换为所需的PID**。

# 系统调用初识

```c
#include <stdio.h>
#include <unistd.h>

int main() {
    while (1) {
        printf("I'm a process! My pid:%d\n", getpid());
        sleep(1);
    }
    return 0;
}
```

1. `#include <unistd.h>`：这是包含Unix标准库的预处理指令，其中包含了sleep函数，用于在程序中添加延迟。
2. `#include <unistd.h>`：这是包含Unix标准库的预处理指令，其中包含了 `getpid()` 函数和 `sleep()` 函数的声明。
3. `printf("I'm a process! My pid:%d\n", getpid());`：这是一个输出语句，用于在终端上打印一条消息。`getpid()`函数用于获取当前进程的进程ID（PID），并将其插入到消息中。

## 结果

![](https://image-somn.oss-cn-beijing.aliyuncs.com/%E5%88%9D%E8%AF%86%E7%B3%BB%E7%BB%9F%E8%B0%83%E7%94%A8.png)

## `getpid()`

> `getpid()` 是一个C标准库函数，通常用于获取当前进程的进程ID（PID）。PID是一个唯一标识运行中进程的正整数值。

以下是对`getpid()`函数和示例程序的详细解释：

1. 包含头文件：为了使用`getpid()`函数，首先需要包含合适的头文件。通常，我们会包含 `<unistd.h>` 头文件，因为该头文件包含了`getpid()`函数的声明。

```c
#include <unistd.h>
```

1. 声明变量：在示例程序中，我们声明了一个名为`pid`的变量，其类型是 `pid_t`。`pid_t` 是一个数据类型，通常用于表示进程ID。

```c
pid_t pid;
```

1. 使用`getpid()`函数：接下来，我们使用`getpid()`函数来获取当前进程的PID，并将其存储在`pid`变量中。

```c
pid = getpid();
```

1. 打印PID：最后，我们使用`printf`函数将获取到的PID打印到终端上，以便查看。

```c
printf("My PID is: %d\n", pid);
```

示例程序中的完整代码：

```c
#include <stdio.h>
#include <unistd.h>

int main() {
    pid_t pid;

    pid = getpid();  // 获取当前进程的PID

    printf("My PID is: %d\n", pid);

    return 0;
}
```

当您运行这个程序时，它会执行以下操作：

1. 调用`getpid()`函数，该函数返回当前进程的PID，并将其存储在`pid`变量中。
2. 使用`printf`函数将获取到的PID打印到终端上，形成一条类似 "My PID is: 12345" 的输出，其中12345是当前进程的实际PID。
3. 然后，程序返回0作为退出状态码，表明程序正常退出。

通过这个示例程序，可以获得当前运行进程的PID，这在许多进程管理和通信场景中非常有用。每个运行的进程都有一个唯一的PID，允许操作系统和其他程序识别和管理它们。