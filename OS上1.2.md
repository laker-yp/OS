# OS 上半部分 C 补充笔记

> 定位：这是**补充版**，不是重写已经有的两份笔记 PPT 里容易漏、但考试可能借题发挥的点。  
> 重点等级：`⭐⭐⭐⭐⭐` 必会；`⭐⭐⭐⭐` 高概率；`⭐⭐⭐` 中等；`⭐` 背景/图片/远离考点。

---

## 目录跳转

- [0. 这份补充笔记怎么用](#sec0)
- [1. Week 1：计算机体系结构 + C 基础里容易漏的点](#sec1)
  - [1.1 Stored-program computer / von Neumann architecture](#sec1-1)
  - [1.2 CPU、ALU、Control Unit、Register](#sec1-2)
  - [1.3 Latency numbers：为什么 register/cache/memory/disk 差距重要](#sec1-3)
  - [1.4 Undefined Behaviour 与数组越界](#sec1-4)
  - [1.5 `scanf` 格式和 `&` 的隐藏高频坑](#sec1-5)
- [2. Week 2：String / Pointer / Memory Layout / Dynamic Memory](#sec2)
  - [2.1 String 的本质：char array + `\0`](#sec2-1)
  - [2.2 `strcpy` / `strncpy` 的安全边界](#sec2-2)
  - [2.3 Pointer type、scale factor、endianness](#sec2-3)
  - [2.4 Stack / Heap / Data / Text：和考试 bug 的关系](#sec2-4)
  - [2.5 `malloc/calloc/realloc/free` 补充易错点](#sec2-5)
- [3. Week 3：2D array / void pointer / function pointer / files / Makefile](#sec3)
  - [3.1 2D array 的 row-major 与 `a`、`&a[0][0]` 区别](#sec3-1)
  - [3.2 Pass-by-value vs pass-by-reference](#sec3-2)
  - [3.3 `void*`：能存任何地址，但不能直接做 pointer arithmetic](#sec3-3)
  - [3.4 Function pointer：考试不大，但 callback 概念要懂](#sec3-4)
  - [3.5 File handling：EOF、`fopen` mode、`getline`](#sec3-5)
  - [3.6 Makefile / object file / preprocessor](#sec3-6)
- [4. Week 4：Sockets / Multicore / Threads / Mutex](#sec4)
  - [4.1 Socket：像 file 一样 read/write，但 server/client 初始化不同](#sec4-1)
  - [4.2 Sequential server vs concurrent server](#sec4-2)
  - [4.3 Multicore：为什么 clock frequency 停滞后转向多核](#sec4-3)
  - [4.4 Cache coherence：共享数据为什么麻烦](#sec4-4)
  - [4.5 Thread、process、main thread](#sec4-5)
  - [4.6 `pthread_create` 参数传递高频坑](#sec4-6)
  - [4.7 `pthread_join`：不是互斥，是等待线程结束](#sec4-7)
  - [4.8 Mutex pitfalls：deadlock、trylock、锁粒度](#sec4-8)
- [5. Week 5：Condition Variable + Read-Write Lock](#sec5)
  - [5.1 Condition variable 的本质](#sec5-1)
  - [5.2 为什么 condition variable 必须配 mutex](#sec5-2)
  - [5.3 `while` 检查条件，不是 `if`](#sec5-3)
  - [5.4 Shared linked list：三种同步方案](#sec5-4)
  - [5.5 Read-write lock：读多写少场景高价值](#sec5-5)
- [6. PPT 中一句话带过的内容](#sec6)
- [7. 最后补充速背模板](#sec7)

---

<a id="sec0"></a>
## 0. 这份补充笔记怎么用

你的原笔记已经有：

- C 基础语法
- pointer / array / struct / union
- malloc/free
- linked list
- pthread / mutex / condition variable
- socket
- 历年考试重点

所以这份文件不重复铺开，而是补那些**容易被你觉得“无所谓”，但考试可能突然问一句**的点。

最值得看的顺序：

```text
Week 5 condition variable / read-write lock
→ Week 4 pthread_create / join / mutex pitfalls
→ Week 2 memory layout + string overflow + free 后访问
→ Week 3 2D array / void* / file handling
→ Week 1 architecture 背景
```

---

<a id="sec1"></a>
# 1. Week 1：计算机体系结构 + C 基础里容易漏的点

<a id="sec1-1"></a>
## 1.1 Stored-program computer / von Neumann architecture ⭐⭐⭐

PPT 前面很多 Enigma、British bombe、历史图片，考试价值很低，一句话：

> 这些图只是为了说明早期计算机偏 application-specific，后来 stored-program computer 把 program instructions 和 data 都放在 memory 里，所以程序可以通过改变 memory 中的指令来改变行为。

真正要懂的是 **von Neumann architecture**：

```text
Memory + CPU + I/O interfaces
```

其中：

- `Memory`：存放 instructions 和 data。
- `CPU`：执行指令。
- `I/O interfaces`：和外部设备交互。

考试可能问：为什么 memory 是程序员视角的核心？

答：

```text
Because both instructions and data are stored in memory, and program execution consists of repeatedly fetching instructions and data from memory, executing them in the CPU, and writing results back to memory or I/O devices.
```

---

<a id="sec1-2"></a>
## 1.2 CPU、ALU、Control Unit、Register ⭐⭐⭐

CPU 里两个核心部件：

| Component | 中文 | 作用 |
|---|---|---|
| `ALU` | Arithmetic and Logic Unit | 做数学和逻辑运算，比如 `+ - * && ||` |
| `Control Unit` | 控制单元 | 决定下一条执行什么指令、从哪里取 operands |

Register 是 CPU 内部非常小、非常快的存储位置。

容易考的理解：

```text
Registers are faster than main memory because they are inside or very close to the CPU. They are used to hold values that are currently being operated on, reducing repeated slow memory access.
```

中文：寄存器不是 RAM，它在 CPU 内部/附近，用来暂存当前要计算的数据。

---

<a id="sec1-3"></a>
## 1.3 Latency numbers：为什么 register/cache/memory/disk 差距重要 ⭐⭐⭐⭐

PPT 有一页 approximate latency numbers：

```text
Register / L1 cache reference: very fast
Main memory reference: much slower
Disk access: extremely slower
```

你不需要背具体 ns 数字，但要知道数量级差距。

考试联系：

- 为什么 CPU 要 cache？因为 main memory 慢。
- 为什么 OS 要 page cache？因为 disk 慢。
- 为什么 context switch 太多会伤性能？cache/TLB locality 被破坏。
- 为什么 thrashing 很严重？因为 page fault 要去 disk，慢到爆炸。

一句话模板：

```text
The memory hierarchy exists because storage closer to the CPU is much faster but smaller and more expensive, while larger storage such as main memory and disk is slower. Good systems exploit locality to keep recently used data close to the CPU.
```

---

<a id="sec1-4"></a>
## 1.4 Undefined Behaviour 与数组越界 ⭐⭐⭐⭐⭐

Week 1 里提到：C compiler/runtime does not check array limits。

这点和历年题非常贴。

标准答案：

```text
Accessing an array outside its bounds in C causes undefined behaviour. The program may crash, silently corrupt memory, appear to work sometimes, or create a security vulnerability.
```

不要只答：

```text
It gives an error.
```

因为 C 很多时候不会马上给 error，而是偷偷错。

---

<a id="sec1-5"></a>
## 1.5 `scanf` 格式和 `&` 的隐藏高频坑 ⭐⭐⭐⭐

Week 1 后面讲 `scanf`。

常见正确写法：

```c
int x;
scanf("%d", &x);
```

因为 `scanf` 需要知道把读入值写到哪里，所以要传 address。

错误：

```c
int x;
scanf("%d", x);   // wrong
```

因为这里传的是未初始化的 `x` 的值，不是地址，`scanf` 会把它当地址写，undefined behaviour。

但字符串数组例外：

```c
char str[100];
scanf("%s", str);  // OK, because str decays to &str[0]
```

不过 `%s` 仍然有 overflow 风险，最好限制宽度：

```c
scanf("%99s", str);
```

---

<a id="sec2"></a>
# 2. Week 2：String / Pointer / Memory Layout / Dynamic Memory

<a id="sec2-1"></a>
## 2.1 String 的本质：char array + `\0` ⭐⭐⭐⭐⭐

C string 不是 Java String object。

C string 是：

```text
连续 char + 末尾的 '\0'
```

例子：

```c
char name[] = "Comp Sc";
```

内存里是：

```text
C o m p   S c \0
```

重点：

- `strlen(name)` 不数 `\0`。
- 存储空间必须包含 `\0`。
- 长度为 7 的字符串，需要至少 8 个 char 空间。

考试看到：

```c
char title[80];
strcpy(title, newTitle);
```

要马上想到：如果 `newTitle` arbitrary length，就可能 buffer overflow。

---

<a id="sec2-2"></a>
## 2.2 `strcpy` / `strncpy` 的安全边界 ⭐⭐⭐⭐⭐

`strcpy(dest, src)`：

- 会复制整个 `src`，包括最后 `\0`。
- 不检查 `dest` 够不够大。
- 不会自动 malloc。

所以如果题目说 source string arbitrary length，最稳修法是动态分配：

```c
char *title = malloc(strlen(newTitle) + 1);
if (title == NULL) {
    // handle error
}
strcpy(title, newTitle);
```

`strncpy` 也不是万能安全：

```c
strncpy(dest, src, n);
```

坑：如果 `src` 长度 >= n，可能不会自动补 `\0`。

安全模板：

```c
strncpy(dest, src, DEST_SIZE - 1);
dest[DEST_SIZE - 1] = '\0';
```

考试答题句：

```text
strncpy reduces the risk of overflowing the destination buffer, but the programmer must ensure that the destination is explicitly null-terminated when the source is too long.
```

---

<a id="sec2-3"></a>
## 2.3 Pointer type、scale factor、endianness ⭐⭐⭐⭐

Pointer 不只是地址，还带有“它指向什么类型”的解释方式。

例子：

```c
int *p;
char *q;
```

如果 `p + 1`，地址增加 `sizeof(int)`。

如果 `q + 1`，地址增加 `sizeof(char)`。

这叫 pointer arithmetic 的 scale factor。

PPT 还讲了 big-endian / little-endian，用来说明一个多字节整数在内存中的字节顺序。

考试不太可能让你细算 endian，但可能借它问：

> 为什么不同类型 pointer 读同一段 memory，结果可能不同？

答：

```text
Because the pointer type determines how many bytes are read and how those bytes are interpreted. An int pointer reads sizeof(int) bytes, while a char pointer reads one byte.
```

---

<a id="sec2-4"></a>
## 2.4 Stack / Heap / Data / Text：和考试 bug 的关系 ⭐⭐⭐⭐⭐

典型 C 程序内存布局：

```text
high address
+---------+
| stack   | local variables, function frames
+---------+
| heap    | malloc/calloc/realloc
+---------+
| data    | global/static variables
+---------+
| text    | program code
+---------+
low address
```

考试联系非常强：

### 返回 local variable 地址：错

```c
int* foo() {
    int x = 10;
    return &x;    // wrong
}
```

因为 `x` 在 stack frame 里，函数返回后 stack frame 失效。

### `malloc` 后返回地址：可以

```c
int* foo() {
    int *p = malloc(sizeof(int));
    *p = 10;
    return p;     // OK, caller must free
}
```

因为 heap memory 不会因为函数 return 自动消失。

### static local variable：可以长期存在

```c
int* foo() {
    static int x = 10;
    return &x;    // technically OK, but shared/global-like state
}
```

因为 static variable 在 data segment。

---

<a id="sec2-5"></a>
## 2.5 `malloc/calloc/realloc/free` 补充易错点 ⭐⭐⭐⭐⭐

你的笔记已经覆盖很多，这里只补考试句式。

### `malloc` vs `calloc`

```c
malloc(n * sizeof(T))
```

只分配，不初始化。

```c
calloc(n, sizeof(T))
```

分配并清零。

### `realloc` 最好用临时指针

危险写法：

```c
p = realloc(p, newSize);
```

如果失败，返回 `NULL`，原来的地址可能丢了，造成 memory leak。

安全写法：

```c
int *tmp = realloc(p, newSize);
if (tmp == NULL) {
    // p is still valid
} else {
    p = tmp;
}
```

### `free` 只能 free heap memory

错误：

```c
int x;
free(&x);       // wrong, stack memory
```

错误：

```c
struct Node n;
free(n.value);  // wrong if value is int, not a heap pointer
```

标准句：

```text
free must only be called on a pointer previously returned by malloc, calloc or realloc, and not already freed.
```

---

<a id="sec3"></a>
# 3. Week 3：2D array / void pointer / function pointer / files / Makefile

<a id="sec3-1"></a>
## 3.1 2D array 的 row-major 与 `a`、`&a[0][0]` 区别 ⭐⭐⭐⭐⭐

C 的二维数组按 row-major order 存：

```c
int a[3][4];
```

内存顺序：

```text
a[0][0], a[0][1], a[0][2], a[0][3],
a[1][0], a[1][1], ...
```

高频坑：

```c
int *p = a;          // warning: incompatible pointer type
int *p = &a[0][0];   // correct if you want int pointer
```

为什么？

- 对 1D array：`a` 通常表示 `&a[0]`。
- 对 2D array：`a` 表示第 0 行的地址，类型类似 `int (*)[4]`，不是 `int*`。

考试答题模板：

```text
For a two-dimensional array, the array name represents the address of the first row, not the address of a single int element. To traverse all elements using an int pointer, use &a[0][0].
```

---

<a id="sec3-2"></a>
## 3.2 Pass-by-value vs pass-by-reference ⭐⭐⭐⭐⭐

C 默认 pass-by-value。

错误理解：

```c
void changeAge(struct Student s) {
    s.age = 30;
}
```

这里只改 copy，不改原 struct。

正确：

```c
void changeAge(struct Student *s) {
    s->age = 30;
}
```

这不是要求 `age` 本身是 pointer，而是要求函数拿到整个 struct 的地址。

考试句式：

```text
C passes arguments by value, so modifying a parameter only modifies the local copy. To modify the caller's object, pass a pointer to it and dereference the pointer inside the function.
```

---

<a id="sec3-3"></a>
## 3.3 `void*`：能存任何地址，但不能直接做 pointer arithmetic ⭐⭐⭐⭐

`void*` 表示 generic pointer。

用途：

- callback
- generic data structure
- pthread function argument / return value

例子：

```c
void* myfunc(void *arg) {
    int x = *((int*)arg);
    return NULL;
}
```

重点：

```text
void* does not know the size of the object it points to, so pointer arithmetic on void* is illegal in standard C. Cast it to the correct pointer type first.
```

---

<a id="sec3-4"></a>
## 3.4 Function pointer：考试不大，但 callback 概念要懂 ⭐⭐⭐

Function pointer 例子：

```c
int (*func)(float, int);
```

意思：`func` 是一个指针，指向一个函数；这个函数接收 `(float, int)`，返回 `int`。

常见用途：callback。

比如库函数让你传一个比较函数、处理函数，库在合适时机调用它。

这部分不太像大题核心，但可以作为理解 `pthread_create` 的函数参数背景。

---

<a id="sec3-5"></a>
## 3.5 File handling：EOF、`fopen` mode、`getline` ⭐⭐⭐⭐

常见文件模式：

| Mode | 含义 |
|---|---|
| `r` | 读已有文件，不存在则失败 |
| `w` | 写文件，存在则清空，不存在则创建 |
| `a` | append，写到末尾 |
| `r+` | 读写已有文件，不清空 |
| `w+` | 读写，先清空/创建 |
| `a+` | 读和 append |

错误处理：

```c
FILE *fp = fopen("data.txt", "r");
if (fp == NULL) {
    // handle error
}
```

EOF 相关：

- `feof(fp)` 只有读过 EOF 后才会变真。
- 不要在未检查读取结果的情况下盲目使用 buffer。

`getline` 常见写法：

```c
char *line = NULL;
size_t len = 0;
ssize_t nread;

while ((nread = getline(&line, &len, fp)) != -1) {
    // use line
}
free(line);
```

重点：`getline` 可能帮你分配/扩容 line，所以最后要 `free(line)`。

---

<a id="sec3-6"></a>
## 3.6 Makefile / object file / preprocessor ⭐⭐⭐

这部分考试概率低于 memory/pthread，但可能出简答。

### Makefile

```makefile
target: file0 file1
	command
```

含义：如果 target 不存在，或者依赖文件更新了，就执行 command。

### Object file

`.o` 文件包含 compiled machine code，但还没 link 成最终 executable。

### Preprocessor

Preprocessor 在 compilation 前做 text substitution。

例子：

```c
#define SIZE 100
#define SQUARE(x) ((x) * (x))
```

Macro 坑：一定加括号。

```c
#define BAD(x) x * x
BAD(1+2)   // becomes 1+2*1+2, wrong
```

正确：

```c
#define GOOD(x) ((x) * (x))
```

---

<a id="sec4"></a>
# 4. Week 4：Sockets / Multicore / Threads / Mutex

<a id="sec4-1"></a>
## 4.1 Socket：像 file 一样 read/write，但 server/client 初始化不同 ⭐⭐⭐⭐

Socket 一旦连接好，就很像 file descriptor：可以 `read()` / `write()` / `close()`。

Server 流程：

```c
sockfd = socket(...);
bind(sockfd, ...);
listen(sockfd, backlog);
newsockfd = accept(sockfd, ...);
read(newsockfd, ...);
write(newsockfd, ...);
close(newsockfd);
```

Client 流程：

```c
sockfd = socket(...);
connect(sockfd, ...);
write(sockfd, ...);
read(sockfd, ...);
close(sockfd);
```

考试常问区别：

```text
The server binds to a local port, listens for incoming connections, and accepts each client connection. The client connects to a server IP/port and then communicates using read/write on the connected socket.
```

---

<a id="sec4-2"></a>
## 4.2 Sequential server vs concurrent server ⭐⭐⭐⭐⭐

Sequential server：一次只服务一个 client。

问题：

- Client1 很慢/buggy/不 close，server 会卡在 Client1。
- Client2 即使已经 connect/write，也要等 Client1 完成。

Concurrent server：每个 client 交给单独 thread/process 处理。

优点：

- 多个 clients 可以同时被服务。
- 避免一个慢 client 阻塞其他所有 client。

考试句式：

```text
A sequential server processes one client at a time, so a slow or buggy client can block all later clients. A concurrent server creates a separate thread or process for each client, allowing multiple clients to be served concurrently and improving responsiveness.
```

---

<a id="sec4-3"></a>
## 4.3 Multicore：为什么 clock frequency 停滞后转向多核 ⭐⭐⭐

PPT 展示 Pentium 到 Core i7：早期频率增长明显，后来频率增长变慢，于是处理器靠增加 cores 提升性能。

一句话：

```text
Because clock frequency scaling became limited by power and heat, processors improved performance by placing multiple cores on one chip so programs can run tasks in parallel.
```

这部分偏背景，不用背图片型号。

---

<a id="sec4-4"></a>
## 4.4 Cache coherence：共享数据为什么麻烦 ⭐⭐⭐⭐

多核里每个 core 可能有自己的 cache。

如果 Core1 和 Core2 都缓存了同一个变量 `counter`，一个 core 改了，另一个 core 的 cache 可能还是旧值。

这就是 coherence 问题。

考试通常不会让你讲 MESI 协议，但你要能说：

```text
In multicore systems, shared data must remain coherent across cores and caches. Without synchronization, concurrent reads and writes can observe stale or inconsistent values.
```

和 race condition 直接相关。

---

<a id="sec4-5"></a>
## 4.5 Thread、process、main thread ⭐⭐⭐⭐⭐

Thread 定义：

```text
A thread of execution is the smallest sequence of programmed instructions that can be managed independently by a scheduler.
```

进程和线程关系：

- process 是 resource container：address space、heap、files。
- thread 是 execution flow：registers、stack、program counter。
- 一个 process 至少有一个 main thread。
- 同一 process 内多个 threads 共享 global data、heap、files，但每个 thread 有自己的 stack。

考试高频：为什么 threads 容易有 race condition？

答：

```text
Threads in the same process share global variables, heap objects and open files, so concurrent access to these shared objects must be synchronized.
```

---

<a id="sec4-6"></a>
## 4.6 `pthread_create` 参数传递高频坑 ⭐⭐⭐⭐⭐

经典错法：

```c
for (int i = 1; i <= 10; i++) {
    pthread_create(&tid, NULL, myfunc, &i);
}
```

问题：所有 threads 拿到的是同一个 `i` 的地址。

等 thread 真运行时，`i` 可能已经变了，甚至 loop 已经结束。

修法 1：每个 thread 一个独立参数数组：

```c
pthread_t tids[10];
int args[10];

for (int i = 0; i < 10; i++) {
    args[i] = i + 1;
    pthread_create(&tids[i], NULL, myfunc, &args[i]);
}

for (int i = 0; i < 10; i++) {
    pthread_join(tids[i], NULL);
}
```

修法 2：malloc 每个参数，thread 内 free：

```c
int *arg = malloc(sizeof(int));
*arg = i;
pthread_create(&tid, NULL, myfunc, arg);
```

thread：

```c
void* myfunc(void *arg) {
    int i = *(int*)arg;
    free(arg);
    return NULL;
}
```

---

<a id="sec4-7"></a>
## 4.7 `pthread_join`：不是互斥，是等待线程结束 ⭐⭐⭐⭐

`pthread_join(tid, NULL)` 的作用：

```text
block the calling thread until the target thread terminates
```

它解决的是 ordering / waiting，不是 shared data mutual exclusion。

比如 main thread 要等 worker threads 都算完再打印结果，可以 join。

但如果两个 worker 同时修改 `counter`，join 不能防止 race condition，仍然要 mutex。

考试句式：

```text
pthread_join ensures that one thread waits for another thread to finish, but it does not protect shared data from concurrent access. A mutex is still needed for mutual exclusion.
```

---

<a id="sec4-8"></a>
## 4.8 Mutex pitfalls：deadlock、trylock、锁粒度 ⭐⭐⭐⭐⭐

### Pitfall 1：return 前忘记 unlock

```c
pthread_mutex_lock(&m);
if (error) {
    return -1;  // wrong, lock not released
}
pthread_mutex_unlock(&m);
```

修：

```c
pthread_mutex_lock(&m);
if (error) {
    pthread_mutex_unlock(&m);
    return -1;
}
pthread_mutex_unlock(&m);
```

### Pitfall 2：两个锁顺序不一致

Thread1：

```c
lock(A);
lock(B);
```

Thread2：

```c
lock(B);
lock(A);
```

可能 deadlock。

解决：所有线程按同一顺序拿锁，或用 `pthread_mutex_trylock` 失败则释放已拿锁再重试。

### Pitfall 3：锁太大影响 parallelism

如果题目说 maximise parallelism，不要把整个函数都锁住。

原则：

```text
Only protect the shared data and keep the critical section as short as possible.
```

---

<a id="sec5"></a>
# 5. Week 5：Condition Variable + Read-Write Lock

<a id="sec5-1"></a>
## 5.1 Condition variable 的本质 ⭐⭐⭐⭐⭐

Condition variable 用于：

```text
one thread waits until a condition becomes true; another thread signals when the condition may have changed
```

基本 API：

```c
pthread_cond_t cond = PTHREAD_COND_INITIALIZER;
pthread_cond_wait(&cond, &mutex);
pthread_cond_signal(&cond);
```

它不是锁。它只是“睡觉/叫醒”的机制。

真正保护 shared condition 的是 mutex。

---

<a id="sec5-2"></a>
## 5.2 为什么 condition variable 必须配 mutex ⭐⭐⭐⭐⭐

原因：要把以下动作连起来：

1. 检查 shared condition。
2. 如果不满足，就进入 sleep。
3. 被唤醒后重新拿到锁，重新检查 condition。

如果没有 mutex，会有 missing wake-up：

```text
Thread A checks condition: false.
Thread B makes condition true and signals.
Thread A has not yet started waiting, so signal is lost.
Thread A then waits forever.
```

`pthread_cond_wait(&cond, &mutex)` 会原子地：

```text
release mutex + sleep
```

醒来后：

```text
reacquire mutex
```

这就是它必须接收 mutex 参数的原因。

---

<a id="sec5-3"></a>
## 5.3 `while` 检查条件，不是 `if` ⭐⭐⭐⭐⭐

正确模板：

```c
pthread_mutex_lock(&m);
while (!condition) {
    pthread_cond_wait(&cond, &m);
}
// use shared data
pthread_mutex_unlock(&m);
```

为什么不用 `if`？

1. 可能 spurious wakeup。
2. 多个线程被唤醒后，条件可能被别人先消耗掉。
3. signal 只表示“条件可能变了”，不是保证条件现在一定成立。

考试一句话：

```text
The condition must be checked in a while loop because a thread may wake up even though the condition is still false or has become false again before it reacquires the mutex.
```

---

<a id="sec5-4"></a>
## 5.4 Shared linked list：三种同步方案 ⭐⭐⭐⭐⭐

Week 5 重点应用：多线程操作共享 sorted linked list。

操作：

- `Member`：只读。
- `Insert`：写。
- `Delete`：写。

### Solution 1：one mutex for whole list

优点：简单、安全。

缺点：所有操作串行化，多个 readers 也不能同时读，parallelism 差。

### Solution 2：one mutex per node / granular locking

优点：理论上 parallelism 高。

缺点：代码复杂、容易错、locking overhead 高。

### Solution 3：read-write lock

适合读多写少。

- 多个 `Member` 可以同时运行。
- `Insert/Delete` 必须拿 write lock，独占。

考试建议：如果题目强调 linked list + mostly reads + maximise parallelism，优先考虑 read-write lock。

---

<a id="sec5-5"></a>
## 5.5 Read-write lock：读多写少场景高价值 ⭐⭐⭐⭐⭐

声明：

```c
pthread_rwlock_t lock = PTHREAD_RWLOCK_INITIALIZER;
```

读：

```c
pthread_rwlock_rdlock(&lock);
// read-only operation, e.g. Member
pthread_rwlock_unlock(&lock);
```

写：

```c
pthread_rwlock_wrlock(&lock);
// modifying operation, e.g. Insert/Delete
pthread_rwlock_unlock(&lock);
```

规则：

| 当前状态 | 新 reader 能进吗 | 新 writer 能进吗 |
|---|---|---|
| no lock | 可以 | 可以 |
| readers active | 可以，通常可以 | 不可以，等 readers 完 |
| writer active | 不可以 | 不可以 |

一句话：

```text
A read-write lock improves parallelism when most operations are reads, because multiple readers can access the data structure concurrently while writers still get exclusive access.
```

---

<a id="sec6"></a>
# 6. PPT 中一句话带过的内容

这些内容在 5 份 PPT 里出现，但考试性价比不高：

- Enigma / British bombe / 电影图片：历史背景，知道 application-specific vs stored-program 即可。
- Intel Pentium II/III/4/Core i7 具体参数：图片背景，不背型号和频率。
- 手机芯片图片、die photo：证明多核普及，了解即可。
- Valgrind 输出截图细节：会用它找 leak 即可，不用背每行输出。
- Makefile 大量细节：知道 target/dependency/command 结构即可。
- `strlcpy`：知道它比 `strncpy` 更常被认为安全，但不是标准 C，考试主线仍是 buffer size + null terminator。

---

<a id="sec7"></a>
# 7. 最后补充速背模板

## 7.1 Pointer / array

```text
A pointer stores the address of another object. The pointer type determines how dereferencing and pointer arithmetic are interpreted.
```

```text
For a two-dimensional array, the array name is the address of the first row, not an int pointer. Use &a[0][0] to get a pointer to the first int element.
```

## 7.2 String overflow

```text
C strings are null-terminated character arrays. Functions such as strcpy do not allocate memory or check destination size, so the programmer must ensure enough space including the terminating null byte.
```

## 7.3 Heap vs stack

```text
Local variables are stored in stack frames and become invalid after the function returns. Dynamically allocated memory is stored on the heap and remains valid until it is freed.
```

## 7.4 Threads share memory

```text
Threads in the same process share global variables, heap objects and open files, but each thread has its own stack and execution state.
```

## 7.5 Condition variable

```text
A condition variable lets a thread sleep until a condition may have become true. It must be used with a mutex, and the condition must be checked in a while loop.
```

## 7.6 Read-write lock

```text
A read-write lock allows multiple concurrent readers but only one writer. It is useful when a shared data structure has many read-only operations and fewer modifying operations.
```
