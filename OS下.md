# LI Operating Systems and Systems Programming：七份课件笔记（按考试重要性整理）

> 使用方式：这是一份**原生 Markdown 笔记**。我把七份课件按考试价值重新整理，不是逐页机械抄 PPT。  
> 重点等级：  
> - `⭐⭐⭐⭐⭐`：核心考点，必须会解释、会比较、会做小题。  
> - `⭐⭐⭐⭐`：高频考点，容易出简答或场景分析。  
> - `⭐⭐⭐`：中等重要，理解即可。  
> - `⭐`：图片展示、背景材料、远离考点，一句话带过。  

---

## 0. 总体考试地图

这七份课件主要覆盖以下 OS 大板块：

1. **Kernel Programming（内核编程）**
   - system call
   - user space / kernel space
   - copy_to_user / copy_from_user
   - process context / interrupt context
   - mutex / semaphore / spinlock

2. **Device Driver（设备驱动）**
   - `/dev` special file
   - open/read/write/ioctl/close
   - interrupt handler
   - top half / bottom half

3. **Synchronisation（同步）**
   - race condition
   - critical section
   - mutual exclusion / progress / bounded waiting
   - TestAndSet / spinlock
   - sleep/wakeup and missing wake-up
   - semaphore
   - producer-consumer / bounded buffer
   - readers-writers

4. **Processes（进程）**
   - process definition
   - process memory layout
   - process states
   - PCB
   - process creation / termination
   - context switch

5. **Scheduling（调度）**
   - ready queue / device queue
   - CPU-bound / I/O-bound
   - FCFS / Round Robin / SJF / Priority / Multilevel Queue
   - preemption
   - scheduling criteria
   - multiprocessor scheduling

6. **Memory Management（内存管理）**
   - logical address / physical address
   - swapping
   - fragmentation
   - paging
   - segmentation
   - virtual memory
   - demand paging
   - page replacement: FIFO / Optimal / LRU / Second Chance
   - thrashing

7. **File Systems（文件系统）**
   - logical file tree vs physical blocks
   - linked allocation vs indexed allocation
   - inode
   - FAT example
   - caching
   - journaling
   - disk scheduling

---

# 1. Linux Kernel Programming

对应课件：`os_03_kernelProgramming.pdf`

---

## 1.1 Kernel 的基本结构 ⭐⭐⭐

PPT 给出的简化内核结构类似：

```c
initialise data structures at boot time;
while (true) {
    while (timer not gone off) {
        assign CPU to suitable process;
        execute process;
    }
    select next suitable process;
}
```

核心意思：

- OS boot 时初始化核心数据结构。
- CPU 不断运行某个 suitable process。
- timer interrupt 到来后，OS 有机会重新选择下一个进程。
- 这就是后面 **preemption（抢占）** 和 **context switch（上下文切换）** 的基础。

考试常问方式：

- 为什么 timer interrupt 对现代 OS 很重要？
- 为什么 OS 可以强行拿回 CPU？
- 为什么没有 preemption 的系统响应性差？

---

## 1.2 Kernel Programming 的危险性 ⭐⭐⭐⭐

PPT 核心句：

> Kernel has access to all resources. Kernel programs are not subject to ordinary memory/hardware access constraints.

人话：

- 普通 user program 犯错，通常只是自己崩溃。
- kernel code 犯错，可能让整个系统崩溃。
- 因为 kernel 能访问所有内存、所有硬件、所有进程的数据。

关键英文：

- `kernel mode`：内核态，权限很高。
- `user mode`：用户态，权限受限制。
- `privileged resource`：特权资源，比如硬件、内核内存、I/O 设备。
- `system crash`：系统崩溃。

考试答题模板：

```text
Kernel code is dangerous because it runs with high privileges and can access hardware and kernel memory directly. A bug such as an invalid pointer, buffer overflow, or incorrect synchronisation may corrupt kernel data structures and crash the whole system, not just the current process.
```

---

## 1.3 System Calls ⭐⭐⭐⭐⭐

PPT 核心：

> Kernel provides its functions only via special functions, called system calls.

`system call` 是 user program 请求 kernel 服务的入口。

例子：

- `read()`
- `write()`
- `open()`
- `close()`
- `fork()`
- `exec()`
- `wait()`

重要理解：

- `printf()` 本身是 C library function。
- 但如果它真的要把内容输出到 terminal，底层通常会触发 `write()` system call。
- 所以不能说 console I/O 不需要 system call。

考试高频句：

```text
Library functions such as printf or scanf are not themselves necessarily system calls, but they normally use system calls such as write or read to communicate with the terminal or other I/O devices.
```

---

## 1.4 User Space 和 Kernel Space 的数据隔离 ⭐⭐⭐⭐⭐

PPT 核心：

> Have strict separation of kernel data and data for user programs. Need explicit copying between user program and kernel.

重点函数：

- `copy_to_user()`：kernel → user
- `copy_from_user()`：user → kernel

为什么需要这些函数？

因为 kernel 不能随便相信 user pointer。

如果 user program 传进来一个地址：

```c
char *buffer;
```

kernel 不能直接：

```c
memcpy(kernel_buffer, buffer, length);
```

原因：

1. `buffer` 可能是非法地址。
2. `buffer` 可能指向 kernel 不该访问的位置。
3. 直接访问可能造成 kernel crash 或 security vulnerability。

正确思路：

```c
copy_from_user(kernel_buffer, user_buffer, length);
copy_to_user(user_buffer, kernel_buffer, length);
```

考试容易出：

- interrupt handler 能不能访问 user memory？
- kernel code 为什么不能直接 `memcpy` user buffer？
- process context 和 interrupt context 的区别。

---

## 1.5 Interrupts（中断） ⭐⭐⭐⭐⭐

PPT 核心：

- kernel asks hardware to perform action。
- hardware sends interrupt to kernel。
- interrupt must be processed quickly。
- code called from interrupts must not sleep。

人话：

中断就是硬件对 CPU 说：

> “我这边有事，快来处理一下。”

例子：

- 键盘输入
- 网卡收到 packet
- 磁盘 I/O 完成
- timer 到点

为什么 interrupt handler 不能慢？

因为中断处理时会打断正常执行。如果处理太久：

- 系统响应变差。
- 其他中断可能被延迟。
- 整体系统吞吐下降。

为什么 interrupt context 不能 sleep？

因为 interrupt handler 不是代表某个普通进程在运行。它没有普通 process context 可以安全地阻塞等待。如果它 sleep，kernel 调度逻辑会出问题。

考试答题模板：

```text
Interrupt handlers must be short because they run asynchronously and may block or delay other important kernel activities. Code in interrupt context must not sleep because there is no ordinary process context to block and later resume safely.
```

---

## 1.6 Process Context vs Interrupt Context ⭐⭐⭐⭐⭐

| Context | 中文 | 什么时候出现 | 能不能 sleep | 能不能访问 user data |
|---|---|---|---|---|
| `process context` | 进程上下文 | system call 代表某个 user process 执行 | 可以，在合适情况下 | 可以通过 copy_to_user/copy_from_user |
| `interrupt context` | 中断上下文 | hardware interrupt 触发 | 不可以 | 一般不可以 |

核心区别：

- process context 是 kernel 在“替某个进程办事”。
- interrupt context 是 kernel 在“响应硬件突发事件”。

考试例子：

> Can the interrupt handler which processes packets from a server store data directly in the compiler’s memory?

答案思路：

不能直接存到 user process memory。interrupt handler runs in interrupt context, not in the process context of that compiler process. It should store data in kernel buffers and wake up/schedule the relevant process later.

---

## 1.7 Kernel Modules ⭐⭐

PPT：

- kernel modules can add code to running kernel。
- useful for device drivers。
- `modprobe` inserts module。
- `rmmod` removes module。
- `lsmod` lists modules。

考点强度不高，通常不作为大题核心。记住它是动态加载 kernel code 的机制即可。

---

## 1.8 Kernel Concurrency ⭐⭐⭐⭐⭐

PPT 核心：

Kernel 里共享数据结构可能被不同 context 同时访问：

1. process context 和 interrupt context 共享。
2. interrupt context 和 interrupt context 共享。
3. multiprocessor 上多个 process context 共享。

因此需要 critical section。

重要：

- 如果 critical section 只在 process context 中使用，可以用 mutex/semaphore。
- 如果可能在 interrupt context 中使用，不能用会 sleep 的锁，要用 spinlock。

---

## 1.9 Mutex/Semaphore vs Spinlock ⭐⭐⭐⭐⭐

| Lock | 行为 | 优点 | 缺点 | 能否在 interrupt context 使用 |
|---|---|---|---|---|
| `mutex` / `semaphore` | 拿不到锁就 sleep | 不浪费 CPU | 不能在不能 sleep 的地方使用 | 否 |
| `spinlock` | 拿不到锁就 busy wait | 可用于 interrupt context，短临界区快 | 浪费 CPU | 可以 |

考试核心判断：

- 临界区可能 sleep？不要用 spinlock 包住会 sleep 的代码。
- 临界区在 interrupt handler 中？不能用 mutex/semaphore。
- 临界区很短、在 kernel/interrupt 中？spinlock 合理。

答题模板：

```text
If the critical section can be executed in interrupt context, a spinlock is appropriate because interrupt context must not sleep. If the code runs only in process context and the critical section may take longer, a mutex or semaphore is preferable because the waiting process can sleep instead of wasting CPU cycles.
```

---

# 2. Linux Device Drivers

对应课件：`os_04_deviceDriver.pdf`

---

## 2.1 User Space 视角：/dev 特殊文件 ⭐⭐⭐⭐

PPT 核心：

Device driver 在 user space 看起来像 `/dev` 下的 special file。

常见 system calls：

- `open`：make device available
- `read`：read from device
- `write`：write to device
- `ioctl`：perform special device operations
- `close`：make device unavailable

人话：

Linux 尽量把设备抽象成文件。你不是直接操作硬件，而是通过文件接口请求 kernel driver 帮你操作硬件。

考试可能问：

- 用户程序如何访问设备？
- device driver 为什么和 file operations 有关？

---

## 2.2 Kernel Side：file operations ⭐⭐⭐⭐

PPT 核心：

- each file may have functions associated with it。
- corresponding system calls trigger corresponding driver functions。
- driver usually implements open/read/write/close。

也就是说：

```text
user calls read(fd, ...)
        ↓
kernel checks fd refers to device file
        ↓
kernel calls driver's read function
```

关键英文：

- `file operations`
- `device driver`
- `kernel-side implementation`
- `linux/fs.h`

---

## 2.3 Device Categorisation ⭐⭐

PPT 提到：

- physical dependencies：比如 USB hub 下的设备。
- buses：PCI、USB 等。
- classes：keyboard、mouse 等。

这部分偏背景，通常一句话带过。

---

## 2.4 Interrupt Handling in Device Drivers ⭐⭐⭐⭐⭐

设备中断流程：

1. device sends interrupt。
2. CPU selects appropriate interrupt handler。
3. interrupt handler processes interrupt。
4. data is transferred to/from device。
5. waiting processes may be woken up。
6. interrupt handler clears interrupt bit。

为什么要 clear interrupt bit？

因为如果不清除，设备可能认为上一次中断还没处理完，后续中断无法正常到来。

---

## 2.5 Top Half / Bottom Half ⭐⭐⭐⭐⭐

PPT 核心：

> Interrupt processing time must be as short as possible. Separate interrupt processing in two halves.

| Part | 中文 | 做什么 | 特点 |
|---|---|---|---|
| `Top Half` | 上半部 | 立即处理最紧急的事，比如设备和 kernel buffer 间的数据搬运，安排 bottom half | 很短，直接由 interrupt handler 调用 |
| `Bottom Half` | 下半部 | 做剩余较慢工作，比如 protocol stack、wake up processes | 仍然和中断相关，但推迟处理 |

考试高频理解：

- top half = quick, urgent, minimal。
- bottom half = deferred, slower, non-urgent follow-up。

答题模板：

```text
The top half runs immediately in response to the hardware interrupt and should do only the minimum urgent work, such as acknowledging the interrupt and moving data to a kernel buffer. The bottom half performs the remaining processing later, such as protocol handling and waking waiting processes. This keeps interrupt latency low.
```

---

# 3. Synchronisation

对应课件：`os_06_sync.pdf`

这是七份课件里**最重要**的一份。考试中同步题几乎年年出现，只是外壳代码变来变去。

---

## 3.1 为什么需要 Synchronisation？ ⭐⭐⭐⭐⭐

PPT 背景：

> Concurrent access to shared data may result in data inconsistency.

多线程/多进程共享数据时，如果没有同步，可能出现数据不一致。

经典例子：bounded buffer / producer-consumer。

共享变量：

```c
int count;
```

Producer：

```c
count++;
```

Consumer：

```c
count--;
```

看起来一加一减应该不变，但实际 `count++` 不是一个不可分割操作。

---

## 3.2 Race Condition ⭐⭐⭐⭐⭐

PPT 展示：

```text
count++ could compile as:
register1 = count
register1 = register1 + 1
count = register1

count-- could compile as:
register2 = count
register2 = register2 - 1
count = register2
```

如果初始 `count = 5`：

```text
Producer: register1 = count        // 5
Producer: register1 = register1+1  // 6
Consumer: register2 = count        // 5
Consumer: register2 = register2-1  // 4
Producer: count = register1        // count = 6
Consumer: count = register2        // count = 4
```

结果变成 4，而不是 5。

定义：

```text
A race condition occurs when the correctness of a program depends on the timing or interleaving of concurrent operations on shared data.
```

中文：

Race condition 是指多个线程/进程并发访问共享数据，最终结果依赖于它们执行顺序，导致结果不可预测。

---

## 3.3 Critical Section ⭐⭐⭐⭐⭐

`critical section` 是访问共享资源、如果并发执行就可能出错的代码段。

例子：

```c
pthread_mutex_lock(&m);
count++;
pthread_mutex_unlock(&m);
```

中间 `count++` 就是 critical section。

考试答题模板：

```text
A critical section is a part of a program where shared data or shared resources are accessed or modified. It must not be executed by more than one thread/process at the same time if doing so could cause inconsistent state.
```

---

## 3.4 Critical Section Solution Criteria ⭐⭐⭐⭐⭐

PPT 三个标准：

### 1. Mutual Exclusion

如果一个 process 在 critical section 中，其他 process 不能同时进入。

中文：互斥。

### 2. Progress

如果没有进程在 critical section，而有进程想进入，那么选择谁进入不能被无关进程无限阻塞。

中文：系统要能继续推进，不能莫名卡住。

### 3. Bounded Waiting

一个进程请求进入 critical section 后，其他进程进入的次数必须有上界。

中文：不能让某个进程永远等，避免 starvation。

---

## 3.5 Peterson’s Solution ⭐⭐

Peterson’s Solution 是教学用的 two-process software solution。

共享变量：

```c
int turn;
Boolean wants_in[2];
```

逻辑：

```c
wants_in[i] = TRUE;
turn = j;
while (wants_in[j] && turn == j) ;

// critical section

wants_in[i] = FALSE;
```

考试价值：

- 了解即可。
- 重点不是背代码，而是理解它满足 mutual exclusion 和 fairness 的思想。
- 现代 CPU 上不完全现实，因为涉及 memory ordering 和 atomicity。

---

## 3.6 Hardware Synchronisation：TestAndSet ⭐⭐⭐⭐

`TestAndSet` 是 atomic instruction：

```c
boolean TestAndSet(boolean *target) {
    boolean original = *target;
    *target = TRUE;
    return original;
}
```

作用：

- 一步完成“读取旧值 + 设置新值”。
- 因为 atomic，所以不会被中间打断。

用它实现 lock：

```c
while (TestAndSet(&lock)) ;
// critical section
lock = FALSE;
```

问题：

- 这种 while 空转叫 `busy waiting` / `spinning`。
- mutual exclusion 可以保证。
- bounded waiting 不一定保证。

---

## 3.7 Busy Waiting / Spinning ⭐⭐⭐⭐

Spinning：线程一直循环检查锁有没有释放。

优点：

- 临界区很短时速度快。
- 不需要 sleep/wakeup 的上下文切换成本。

缺点：

- 浪费 CPU。
- 临界区长时很差。

联系 kernel：

- spinlock 本质就是 busy waiting。
- interrupt context 不能 sleep，所以经常要用 spinlock。

---

## 3.8 Sleep/Wakeup 和 Missing Wake-up ⭐⭐⭐⭐⭐

PPT 讲：与其 spin，不如拿不到锁就 sleep，锁释放时 wake up。

但危险是 `missing wake-up`：

```text
Thread A checks condition and decides to sleep.
Before A actually sleeps, Thread B sends wakeup.
A misses the wakeup and then sleeps forever.
```

核心问题：

> check condition 和 go to sleep 必须以某种方式原子化。

这就是为什么 condition variable 要配合 mutex 使用。

---

## 3.9 Semaphore ⭐⭐⭐⭐⭐

Semaphore 是更高级的同步工具。

两个操作：

- `wait()` / `P()` / `down()`
- `signal()` / `V()` / `up()`

语义：

```text
wait(S):
    S--
    if S < 0, block the process

signal(S):
    S++
    if some process is waiting, wake one
```

用途：

1. mutual exclusion：binary semaphore。
2. counting resource：counting semaphore。
3. ordering：让一个线程等另一个线程完成某事。

---

## 3.10 Producer-Consumer / Bounded Buffer ⭐⭐⭐⭐⭐

这是同步最经典模型。

需要三个同步量：

```text
mutex      // 保护 buffer 本身
empty      // 空槽数量
full       // 已有 item 数量
```

Producer：

```c
wait(empty);
wait(mutex);

// add item to buffer

signal(mutex);
signal(full);
```

Consumer：

```c
wait(full);
wait(mutex);

// remove item from buffer

signal(mutex);
signal(empty);
```

对应 pthread condition variable 写法：

```c
pthread_mutex_lock(&m);
while (count == BUFFER_SIZE) {
    pthread_cond_wait(&not_full, &m);
}
// insert item
pthread_cond_signal(&not_empty);
pthread_mutex_unlock(&m);
```

为什么用 `while` 不用 `if`？

1. 可能有 spurious wakeup。
2. 被唤醒后条件可能又被其他线程改变。
3. wait 返回后必须重新检查共享条件。

---

## 3.11 Readers-Writers Problem ⭐⭐⭐⭐

问题：

- 多个 reader 可以同时读。
- writer 写的时候必须独占。

目标：

- 允许并发读，提高 parallelism。
- 写时互斥，防止数据不一致。

常见风险：

- reader preference 可能让 writer starvation。
- writer preference 可能降低 reader 并发。

考试中可能问：

- 为什么 read-only critical section 可以用 read-write lock？
- 如何最大化 parallelism？

---

## 3.12 Deadlock 和 Priority Inversion ⭐⭐⭐⭐

PPT 提到使用 semaphore/lock 要小心。

Deadlock 典型条件：

1. mutual exclusion
2. hold and wait
3. no preemption
4. circular wait

Priority inversion：

高优先级线程等低优先级线程释放锁，但低优先级线程又被中优先级线程抢占，导致高优先级线程间接受阻。

理解即可，不一定是计算题。

---

# 4. Process Concept and Context Switching

对应课件：`os_07_processes.pdf`

---

## 4.1 What is a Process? ⭐⭐⭐⭐⭐

PPT 定义：

> Process = a program in execution.

一个 process 包含：

- program text / code
- program counter, PC
- stack
- data section
- heap

中文：

程序是静态文件，进程是正在运行的程序实例。

例子：

- `chrome.exe` 文件是 program。
- 你打开两个 Chrome 窗口，可能有多个 process。

---

## 4.2 Process in Memory ⭐⭐⭐⭐⭐

典型内存布局：

```text
high address
+-------------+
| stack       |  grows downward
+-------------+
| free space  |
+-------------+
| heap        |  grows upward
+-------------+
| data        |
+-------------+
| text/code   |
+-------------+
low address
```

重点：

- `text`：程序代码。
- `data`：global/static variables。
- `heap`：malloc/new 动态分配。
- `stack`：函数调用、local variables、return address。

考试可能结合 C memory bug 问：

- local variable 在 stack 上，函数返回后不能继续引用。
- malloc 出来的东西在 heap 上，要 free。

但如果题目纯 C bug 不来自 PPT，你可以不用重点押。

---

## 4.3 Process States ⭐⭐⭐⭐⭐

状态：

| State | 中文 | 含义 |
|---|---|---|
| `new` | 新建 | process 正在被创建 |
| `ready` | 就绪 | 在内存中，等 CPU |
| `running` | 运行 | 正在 CPU 上执行 |
| `waiting` | 等待/阻塞 | 等 I/O 或事件 |
| `terminated` | 结束 | 执行结束 |

转换：

```text
new -> ready             admitted
ready -> running         scheduler dispatch
running -> ready         preempted / time slice expired
running -> waiting       I/O or event wait
waiting -> ready         I/O or event completion
running -> terminated    exit
```

必会画/解释。

---

## 4.4 PCB：Process Control Block ⭐⭐⭐⭐⭐

PCB 是 kernel 用来记录 process 信息的数据结构。

包含：

- process state
- process number / pid
- program counter
- CPU registers
- CPU scheduling information
- memory-management information
- accounting information
- I/O status information
- list of open files

考试答题模板：

```text
A PCB stores all information the operating system needs to manage and later resume a process, such as its state, program counter, CPU registers, scheduling information, memory management information and open files.
```

---

## 4.5 Process Creation ⭐⭐⭐⭐

PPT 重点：

- parent process creates child processes。
- processes form a process tree。
- each process identified by pid。

Unix/Linux 典型：

- `fork()` creates child process。
- `exec()` replaces process image with new program。
- `wait()` parent waits for child。

资源共享可能有三种：

1. parent and child share all resources。
2. child shares subset。
3. parent and child share no resources。

---

## 4.6 Process Termination ⭐⭐⭐

进程终止原因：

- normal exit
- error exit
- killed by another process
- parent terminates child

了解即可。

---

## 4.7 Context Switch ⭐⭐⭐⭐⭐

定义：

`context switch` 是 OS 把 CPU 从一个 process/thread 切换到另一个 process/thread。

OS 需要做：

1. save current process context。
2. update PCB of current process。
3. choose next process from ready queue。
4. restore next process context。
5. switch memory/address-space information if needed。
6. resume execution of next process。

为什么不能太频繁？

因为 context switch 是 overhead：

- 不做用户有用工作。
- 保存/恢复寄存器需要时间。
- cache/TLB 可能失效。
- 太频繁会降低 throughput。

考试答题模板：

```text
Context switching is necessary for multitasking, but too many context switches waste CPU time on saving and restoring state rather than executing useful work. They may also reduce cache and TLB locality, so processes should usually run for a sufficient time slice.
```

---

# 5. Scheduling

对应课件：`os_08_scheduling.pdf`

这是考试非常核心的一份，尤其适合出场景题。

---

## 5.1 Scheduling Problem ⭐⭐⭐⭐⭐

问题：多个 processes 竞争 CPU、disk、I/O devices。

OS 要定义 schedule 来决定：

> next 让谁使用资源？

通常通过 queue 实现。

---

## 5.2 Scheduling Queues ⭐⭐⭐⭐

| Queue | 中文 | 含义 |
|---|---|---|
| `job queue` | 作业队列 | 系统中所有 processes |
| `ready queue` | 就绪队列 | 在内存中，准备运行，等 CPU |
| `device queue` | 设备队列 | 等待某个 I/O device 的 processes |

进程会在这些队列之间移动。

---

## 5.3 CPU-I/O Burst Cycle ⭐⭐⭐⭐⭐

程序执行通常在 CPU burst 和 I/O burst 之间交替：

```text
CPU burst -> I/O wait -> CPU burst -> I/O wait -> ...
```

PPT 提到：I/O 通常在一定时间后发生，因此这是 rescheduling 的好时机。

---

## 5.4 Preemptive Scheduling ⭐⭐⭐⭐⭐

`preemptive scheduling`：OS 可以强制 process relinquish CPU。

中文：抢占式调度。

为什么重要？

- 防止某个 CPU-bound process 长时间霸占 CPU。
- 保证 interactive/editor 这类任务响应快。
- Round Robin 和 preemptive priority 都依赖它。

---

## 5.5 Scheduling Criteria ⭐⭐⭐⭐⭐

| Criteria | 中文 | 目标 |
|---|---|---|
| `CPU utilisation` | CPU 利用率 | CPU 尽量别闲着 |
| `throughput` | 吞吐量 | 单位时间完成更多进程 |
| `turnaround time` | 周转时间 | 从提交到完成的总时间 |
| `waiting time` | 等待时间 | 在 ready queue 中等多久 |
| `response time` | 响应时间 | 请求提交到首次响应的时间 |

考试常问：场景中应该优化哪个？

- editor / UI：response time 最重要。
- batch CPU job：throughput / CPU utilisation 重要。
- interactive server：response time + fairness。

---

## 5.6 CPU-bound vs I/O-bound ⭐⭐⭐⭐⭐

| Type | 特点 | 适合策略 |
|---|---|---|
| `CPU-bound` | 长 CPU burst，少 I/O | 不应让它长期阻塞 interactive tasks |
| `I/O-bound` | 短 CPU burst，多 I/O | 给 CPU 后很快又去等 I/O，有利于响应性和 CPU 利用率 |

考试场景判断：

- image rendering / verification program：CPU-bound。
- editor / preview request：interactive / often I/O-bound。
- compiler with network library：CPU + I/O intensive。

---

## 5.7 FCFS：First-Come, First-Served ⭐⭐⭐⭐

特点：

- 按到达顺序执行。
- easy to implement。
- non-preemptive。

缺点：

- CPU-intensive process 可造成 long waiting time。
- convoy effect：短任务被长任务挡住。
- 对 interactive task 不友好。

场景题：如果 editor 需要 responsive，FCFS 通常不好。

---

## 5.8 Round Robin ⭐⭐⭐⭐⭐

PPT：

> FCFS with preemption is called Round-Robin.

特点：

- 每个 process 得到 time quantum。
- time slice expired 后被 preempt，放回 ready queue。
- time-sharing systems 标准方法。

time quantum 问题：

| Quantum | 结果 |
|---|---|
| too short | too many context switches |
| too long | process can monopolise CPU，退化像 FCFS |

优点：

- fairness 好。
- interactive response 通常比 FCFS 好。

缺点：

- 对不同重要性任务不够精准。
- quantum 选择很关键。

---

## 5.9 SJF：Shortest Job First ⭐⭐⭐⭐

特点：

- 选择 shortest CPU burst 的 job。
- 最小 average waiting time。

问题：

- 不可真正实现，因为未来 CPU burst 不知道。
- 只能预测。

预测公式：

```text
τ(n+1) = α t(n) + (1 - α) τ(n)
```

含义：

- `t(n)`：最近一次真实 burst time。
- `τ(n)`：之前预测。
- `α`：最近历史权重。

考试通常不要求复杂计算，理解公式即可。

---

## 5.10 Priority Scheduling ⭐⭐⭐⭐⭐

特点：

- 每个 process 有 priority。
- 最高优先级先运行。
- 同优先级可用 FCFS。

两种：

1. preemptive priority：新来的高优先级可以马上抢 CPU。
2. non-preemptive priority：新来的高优先级也要等当前 process 结束/让出 CPU。

优点：

- 可以保证重要任务响应快。

缺点：

- low-priority starvation。

解决：

- `ageing`：等待越久，priority 越高。

考试场景：

如果 editor must be responsive，通常给 editor 高优先级 + preemption。

---

## 5.11 Multilevel Queue Scheduling ⭐⭐⭐⭐

适用于 processes 可以分类：

- foreground / interactive
- background / batch
- system processes

每个队列可以有自己的 scheduling algorithm。

例子：

- interactive queue：Round Robin，小 quantum，高优先级。
- batch queue：FCFS 或 lower priority。

考试场景很好用：

> editor 要快，verification program 可后台慢慢跑。

可以答：使用 multilevel feedback/priority scheduling，把 editor 放高优先级 interactive queue。

---

## 5.12 Multiprocessor Scheduling ⭐⭐

PPT 提到：

- processor affinity：尽量让 process 回到同一 CPU，利用 cache locality。
- load balancing：避免某些 CPU 太忙，某些 CPU 空闲。

考点强度中低，除非题目明确问多核。

---

# 6. Memory Management

对应课件：`os7a.pdf`

---

## 6.1 Memory Management 的基本问题 ⭐⭐⭐⭐⭐

PPT 核心：

Memory 是 limited resource，但程序越来越 hungry。

程序看到的是：

- logical address：从 0x0 开始的地址空间。

硬件真实的是：

- physical address：RAM 里的真实地址。

OS 需要 mapping：

```text
logical address -> physical address
```

---

## 6.2 Address Binding / Mapping ⭐⭐⭐⭐

地址映射可以发生在：

1. compile time：编译时产生 absolute references。
2. load time：加载时决定地址。
3. execution time：运行时通过硬件支持动态映射。

现代系统主要依赖 execution time mapping。

---

## 6.3 Dynamic Linking ⭐⭐⭐

PPT：

> use only one copy of system library; OS has to help.

意思：多个 process 可以共享同一份 library code，节省内存。

这和 segmentation/paging 的 sharing/protection 有关系。

---

## 6.4 Swapping ⭐⭐⭐⭐⭐

Swapping：内存不够时，把某些 process 的 memory 转移到 disk。

特点：

- usually combined with scheduling。
- low priority processes may be swapped out。

问题：

1. big transfer time。
2. pending I/O 怎么办？

为什么 swapping 不是主要 memory management technique？

因为把整个 process memory 搬到 disk 太慢。

考试场景：

- memory full，需要新 process 运行，选谁 swap out？
- 目标是 maximize CPU utilisation and minimize response time。

答题思路：

- swap out low-priority / background / blocked process。
- 不要 swap out interactive editor。
- 不要 swap out process with pending I/O if unsafe。

---

## 6.5 Fragmentation ⭐⭐⭐⭐

两种 fragmentation：

### External Fragmentation

内存中有很多小洞，总空闲空间够，但没有连续大块。

### Internal Fragmentation

分配单位比实际需要大，块内部浪费。

选择 hole 的策略：

| Strategy | 中文 | 特点 |
|---|---|---|
| `First-fit` | 首次适配 | 从头找第一个够大的洞 |
| `Rotating first fit` | 循环首次适配 | 从上次位置后开始找 |
| `Best fit` | 最佳适配 | 找最小可用洞 |
| `Buddy system` | 伙伴系统 | 块大小为 2 的幂，可 split/recombine |

考试中通常简答，不太会复杂算。

---

## 6.6 Paging ⭐⭐⭐⭐⭐

Paging：把 logical memory 和 physical memory 都分成固定大小单位。

- logical page
- physical frame

优点：

- avoids external fragmentation。
- allocation easier。
- protection info can be stored in page table。

地址结构：

```text
logical address = page number + offset
```

转换：

```text
page number -> page table -> frame number
physical address = frame number + offset
```

页表可能很大，所以需要 cache：

- TLB：Translation Lookaside Buffer。

PPT 原理：大 lookup table 用小而快的 cache 存最近使用项。

---

## 6.7 Segmentation ⭐⭐⭐⭐⭐

Segmentation：按程序逻辑用途划分内存。

常见 segment：

- code segment
- data segment
- stack segment
- symbol table

地址结构：

```text
logical address = segment number + offset
```

需要检查：

```text
offset < segment limit ?
```

如果超过 limit，addressing error。

---

## 6.8 Paging vs Segmentation ⭐⭐⭐⭐⭐

| 比较 | Paging | Segmentation |
|---|---|---|
| 划分依据 | 固定大小 page | 程序逻辑单位 |
| 主要目的 | 方便内存分配，避免 external fragmentation | 符合程序逻辑结构，方便 sharing/protection |
| 地址 | page number + offset | segment number + offset |
| 大小 | page 大小固定 | segment 大小可变 |
| 碎片 | 可能 internal fragmentation | 可能 external fragmentation |

考试答题模板：

```text
Paging divides memory into fixed-size pages mainly to simplify allocation and avoid external fragmentation. Segmentation divides memory according to the logical structure of a program, such as code, data and stack segments, and each segment may have a different size.
```

---

## 6.9 Virtual Memory ⭐⭐⭐⭐⭐

PPT 核心：

> complete separation of logical and physical memory.

虚拟内存让程序以为自己有很大的地址空间，但物理内存只加载当前常用部分。

为什么可行？

因为 locality：程序通常只频繁使用一小部分内存。

速度差距：

- memory access：约 ns 级。
- disk access：慢很多。

所以 page fault 很贵。

---

## 6.10 Demand Paging ⭐⭐⭐⭐⭐

Demand paging：需要某 page 时才加载。

每页有 valid/invalid bit：

- valid：page 在内存。
- invalid：page 不在内存或非法。

如果访问 invalid page：

- page fault。
- OS 从 disk 把 page 调入 memory。

两种角色：

- `swapper`：决定 swap out 哪个 process。
- `pager`：决定替换哪个 page。

---

## 6.11 Page Replacement Algorithms ⭐⭐⭐⭐⭐

### FIFO

First-In First-Out。

优点：easy。

缺点：does not take locality into account。

可能出现：

- Belady’s anomaly：frames 增加，page faults 反而增加。

### Optimal

替换未来最晚再用或不再用的 page。

优点：理论最优。

缺点：不可实现，因为不知道未来。

### LRU

Least Recently Used。

用过去预测未来，替换最长时间没用的 page。

缺点：需要硬件/复杂记录支持。

### Second Chance

LRU 近似。

- 每页有 reference bit。
- FIFO 扫描时，如果 bit=1，清零并跳过。
- bit=0 时替换。

---

## 6.12 Thrashing ⭐⭐⭐⭐⭐

Thrashing：process 缺少足够 frames，频繁 page fault，大量时间花在换页而不是执行。

表现：

- CPU utilisation 下降。
- disk activity 很高。
- throughput 下降。
- response time 变差。

如何区分 overloaded CPU / thrashing / overused disk？

| 情况 | CPU | Disk | Page Fault |
|---|---|---|---|
| overloaded CPU | CPU 高 | disk 不一定高 | page fault 不一定高 |
| overused disk | CPU 可能等待 I/O | disk 高 | page fault 不一定高 |
| thrashing | CPU 低或下降 | disk 高 | page fault 非常高 |

考试高频。

---

## 6.13 Linux Memory Notes ⭐⭐

PPT 后面提到 Linux kernel/user memory、page caches 等。

考试价值中低，记住：

- kernel memory 和 user memory 分开。
- page cache 用来缓存文件系统数据，提高 I/O performance。

---

# 7. File Systems

对应课件：`os_09_filesys.pdf`

---

## 7.1 File System 的功能 ⭐⭐⭐⭐⭐

PPT 核心：

> main permanent data storage.

文件系统提供：

- permanent storage。
- 文件和目录的逻辑树结构。
- read/write/create/delete。

两个视角：

| View | 中文 | 含义 |
|---|---|---|
| logical view | 程序员视角 | tree structure of files/directories |
| physical view | 磁盘视角 | sequence of blocks/sectors |

OS 的任务：

```text
map logical file tree to physical disk blocks
```

---

## 7.2 Linked Allocation ⭐⭐⭐⭐

每个 block 存指向下一个 block 的 pointer。

优点：

- 文件增长容易。
- 不需要连续空间。

缺点：

- random access / seek 很慢。
- 想访问第 n 个 block 必须从头走链表。

---

## 7.3 Indexed Allocation / inode ⭐⭐⭐⭐⭐

Indexed allocation：把 block pointers 存在一个 index block 中。

Unix 中叫 `inode`。

inode 还存：

- file size
- permissions
- ownership
- timestamps
- block pointers

优点：

- random access 更快。
- 可通过 direct/indirect pointers 支持不同大小文件。

考试答题模板：

```text
Linked allocation stores a pointer to the next block in each block, which makes sequential access simple but random access expensive. Indexed allocation stores block pointers in an index block, called an inode in Unix-like systems, allowing more efficient random access and storing metadata such as size and permissions.
```

---

## 7.4 FAT Example ⭐⭐

PPT 用 FAT16 解释文件系统概念：

- Boot sector
- FAT
- Root Directory
- Data area
- Cluster chain

这部分有很多图片和十六进制例子，主要用于理解，不是最可能的大题核心。

一句话：

> FAT 用 File Allocation Table 维护 cluster chain，现代文件系统更复杂，但 FAT 适合教学。

---

## 7.5 FAT Limits ⭐

FAT12/FAT16/FAT32 的容量限制、bootsector 细节等偏图片演示和背景，除非老师特别强调，否则一句话带过。

---

## 7.6 Caching ⭐⭐⭐⭐

文件系统慢，因为 disk access 慢。

Caching：把最近使用或可能再用的数据放在 memory 中。

优点：

- 减少 disk I/O。
- 提高 performance。

风险：

- crash 前缓存没写回，可能丢数据。

---

## 7.7 Journaling File Systems ⭐⭐⭐⭐

Journaling：修改真正文件系统结构前，先记录 log/journal。

目的：

- crash recovery。
- 防止文件系统 metadata 不一致。

人话：

就像写账本：先记“我要做什么”，再真的做。崩溃后可以看 journal 恢复。

---

## 7.8 Disk Access and Disk Scheduling ⭐⭐⭐⭐

Disk access 涉及移动磁头，顺序很重要。

常见 disk scheduling：

- FCFS：按请求到达顺序。
- SSTF：shortest seek time first。
- SCAN：电梯算法，一个方向扫过去。
- LOOK：类似 SCAN，但不扫到尽头，只扫到最后一个请求。

PPT 强调 LOOK scheduling improvement。

考试可能问：

- 为什么 disk scheduling 能提高性能？
- 不同 workload 为什么需要不同 disk scheduling？

---

# 8. 最容易被忽略但考试爱问的连接点

---

## 8.1 printf/scanf 与 system call

`printf()` 和 `scanf()` 是 C library functions，但读写 terminal 通常要通过 `read/write` system call。

答题不要说：

```text
printf is a system call.
```

更准确：

```text
printf is a library function that may invoke the write system call to output data.
```

---

## 8.2 为什么 producer 也要 wait？

bounded buffer 中：

- consumer 在 buffer empty 时 wait。
- producer 在 buffer full 时 wait。

因为题目常说：

> whenever get() is called, the resulting request must be inserted.

所以 producer 不能在满时直接丢数据。它必须等待 not_full。

---

## 8.3 为什么 condition variable 总是配 mutex？

因为要避免：

- lost wakeup
- race on shared condition

正确模式：

```c
pthread_mutex_lock(&m);
while (!condition) {
    pthread_cond_wait(&cond, &m);
}
// use shared resource
pthread_mutex_unlock(&m);
```

`pthread_cond_wait` 会原子地：

1. release mutex。
2. sleep。
3. 被唤醒后 reacquire mutex。

---

## 8.4 最大化 parallelism 的答题思想

如果题目说：

> maximise the degree of possible parallelism

不要粗暴把所有代码都锁住。

原则：

1. 只锁 shared data。
2. 不锁 thread-safe 且耗时的函数。
3. 不要在锁内做 I/O 或长计算，除非必须。
4. read-only 操作可用 read-write lock。

例子：

```c
unsigned int r = get();      // get() thread-safe, 放锁外
pthread_mutex_lock(&m);
// 修改 shared queue
pthread_mutex_unlock(&m);
handle(r);                  // handle() thread-safe, 放锁外
```

---

# 9. 图片/背景材料一句话带过清单

以下内容主要是 PPT 图示或背景，不是最值得押的大题的地方：

- Windows EPROCESS / PEB 结构图：知道是 Windows 的 PCB 类似结构即可。
- FAT16 bootsector 十六进制字段：用于演示，不建议背具体 offset。
- FAT root directory 十六进制例子：理解 filename、length、sector 信息即可。
- Linux kernel tour：背景阅读，考试价值低。
- device buses/classes 细分：知道 USB/PCI、keyboard/mouse class 即可。
- Android specifics 标题：这份课件主体其实讲 memory management，不要被标题干扰。

---

# 10. 最后复习优先级

如果时间紧，按这个顺序复习：

1. Synchronisation：race condition、critical section、mutex/semaphore、condition variable、producer-consumer。
2. Scheduling：FCFS/RR/Priority/SJF，场景分析，preemption，response time。
3. Memory Management：paging vs segmentation、virtual memory、page replacement、thrashing。
4. Processes：process states、PCB、context switch。
5. Kernel Programming：system calls、kernel/user space、interrupt context、spinlock vs mutex。
6. Device Drivers：/dev、file operations、interrupt top/bottom half。
7. File Systems：inode、linked vs indexed allocation、journaling、disk scheduling。

