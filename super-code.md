# OS C Code 考前突击版：只看代码题

> 目标：**考试看到 C code，快速扫错误**。  
> 不背文字题，只背两类：**Mutex / 并发 code** + **普通 C code**。  
> 原则：只保留五份卷里反复出现、下次也可能出现的“通用错误”。

---

## Part A：Mutex / 并发 code

### A1. 一眼判断：什么变量要加锁？

看到这些就是 **shared state**，多个 thread 会一起改，必须保护：

```c
int count;
int request_count;
int noOfMealsAvailable;
double values[STACK_SIZE];
int requests[8];
struct seat_t allSeats[SEATS_AVAILABLE];
struct messageList *msgList;
int msgSize;
```

**口诀：**

> 全局变量 + 多线程读写 = mutex  
> 数组 + index/count 一起改 = mutex  
> 链表 head + size 一起改 = mutex  
> check 再 update = 整段都要 lock

---

### A2. 最基础 mutex 模板

```c
pthread_mutex_t m = PTHREAD_MUTEX_INITIALIZER;

pthread_mutex_lock(&m);
/* critical section: 读/写共享变量 */
pthread_mutex_unlock(&m);
```

**考试写法：**

```c
lock before accessing shared variable;
unlock after shared variable is updated;
```

---

### A3. `push/pop/is_empty` stack 必考错误

错误版本常长这样：

```c
values[count++] = v;
return values[--count];
return count == 0;
```

#### 错在哪里？

| 代码 | 错误 |
|---|---|
| `count++` | 读 count + 改 count，不是 atomic |
| `--count` | 两个线程可能 pop 同一个位置 / count 变乱 |
| `is_empty()` | 读 count，也可能和 push/pop race |
| push 用 `m1`，pop 用 `m2` | 错，保护同一个 `count` 必须用同一把锁 |
| `return` 写在 unlock 前 | 错，unlock 永远执行不到 |

#### 正确模板

```c
pthread_mutex_t m = PTHREAD_MUTEX_INITIALIZER;

void push(double v) {
    pthread_mutex_lock(&m);
    if (count < STACK_SIZE) {
        values[count++] = v;
    }
    pthread_mutex_unlock(&m);
}

double pop() {
    pthread_mutex_lock(&m);

    double v = 0;
    if (count > 0) {
        v = values[--count];
    }

    pthread_mutex_unlock(&m);
    return v;
}

int is_empty() {
    pthread_mutex_lock(&m);
    int result = (count == 0);
    pthread_mutex_unlock(&m);
    return result;
}
```

**一句话解释：**

> `count` 和 `values` 是同一个数据结构，必须用同一把 mutex 保护；不能在 unlock 前直接 return。

---

### A4. Producer / Consumer：最大 parallelism 模板

错误版本：

```c
if (request_count < 8) {
    requests[request_count] = get();
    request_count++;
}

if (request_count > 0) {
    request_count--;
    handle(requests[request_count]);
}
```

#### 扫错点

| 位置 | 问题 |
|---|---|
| `request_count < 8` 和 `request_count++` | check-update race |
| `request_count > 0` 和 `request_count--` | check-update race |
| `get()` 在锁里面 | 不好，浪费并行；题目说 get thread-safe |
| `handle()` 在锁里面 | 不好，浪费并行；题目说 handle thread-safe |

#### 正确写法：只锁数组和 count，不锁 get/handle

```c
pthread_mutex_t m = PTHREAD_MUTEX_INITIALIZER;

void producer() {
    int r = get();              // outside lock

    pthread_mutex_lock(&m);
    if (request_count < 8) {
        requests[request_count] = r;
        request_count++;
    }
    pthread_mutex_unlock(&m);
}

void consumer() {
    int r;
    int has_request = 0;

    pthread_mutex_lock(&m);
    if (request_count > 0) {
        request_count--;
        r = requests[request_count];
        has_request = 1;
    }
    pthread_mutex_unlock(&m);

    if (has_request) {
        handle(r);              // outside lock
    }
}
```

**考试关键词：**

> To maximise parallelism, keep the critical section as short as possible.

---

### A5. 多资源：seat + meal 怎么加锁？

看到这种：

```c
allSeats[i].available = false;
allSeats[i].meal_booked = true;
noOfMealsAvailable--;
```

#### 错在哪里？

多个线程可能：

- 订到同一个 seat
- meal 数量被减错
- `listAllBookings()` 读到一半更新状态

#### 最简单安全答案

```c
pthread_mutex_t seatLocks[SEATS_AVAILABLE];
pthread_mutex_t mealLock;
```

思路：

```c
for each seat i:
    lock seatLocks[i]

    if seat available:
        if wantsMeal:
            lock mealLock
            if meal available:
                reserve seat
                decrement meal count
                unlock mealLock
                unlock seatLocks[i]
                return i
            unlock mealLock
        else:
            reserve seat without meal
            unlock seatLocks[i]
            return i

    unlock seatLocks[i]
```

`listAllBookings()`：

```c
for each seat i:
    lock seatLocks[i]
    print/check that seat
    unlock seatLocks[i]
```

**一句话：**

> seat 用 per-seat locks 提高 parallelism；meal count 是一个全局资源，用单独 mealLock。

---

### A6. `pthread_create(..., &i)` 必考

错误：

```c
for (i = 0; i < 4; i++) {
    pthread_create(&tid[i], NULL, foo, &i);
}
```

#### 错在哪里？

所有 threads 拿到的是 **同一个 i 的地址**。  
main thread 继续改 i，所以 thread 读到的值可能重复。

#### 正确写法 1：数组保存每个参数

```c
pthread_t tid[4];
int ids[4];

for (int i = 0; i < 4; i++) {
    ids[i] = i;
    pthread_create(&tid[i], NULL, foo, &ids[i]);
}

for (int i = 0; i < 4; i++) {
    pthread_join(tid[i], NULL);
}
```

#### 正确写法 2：malloc 每个参数

```c
for (int i = 0; i < 4; i++) {
    int *arg = malloc(sizeof(int));
    *arg = i;
    pthread_create(&tid[i], NULL, foo, arg);
}
```

thread 里面：

```c
void *foo(void *arg) {
    int id = *((int *)arg);
    free(arg);
    printf("%d\n", id);
    return NULL;
}
```

**一句话：**

> 不要传 loop variable 的地址；给每个 thread 一个独立地址。

---

### A7. lock 答题万能句

看到并发 code，不会写完整也要写这几句：

```text
The shared variables must be protected by the same mutex.
The lock must cover both the check and the update.
The critical section should be as short as possible.
Thread-safe functions such as get()/handle() should be called outside the lock.
Do not return before unlocking the mutex.
```

---

## Part B：普通 C code 找错误方法

### B1. 最先扫：有没有 malloc？

看到指针被写入，但之前没 malloc，就是错。

错误：

```c
char *arg;
sprintf(arg, "Argument %d is %s\n", i, argv[i]);
```

正确：

```c
char *arg = malloc(strlen(argv[i]) + 50);
sprintf(arg, "Argument %d is %s\n", i, argv[i]);
```

错误：

```c
struct List_t *head = NULL;
head->elem = arg;
```

正确：

```c
struct List_t *head = malloc(sizeof(struct List_t));
head->elem = arg;
```

**口诀：**

> `p->x = ...` 前，p 必须指向有效 struct。  
> `sprintf/strcpy` 写入 char* 前，char* 必须有空间。

---

### B2. malloc 大小模板

#### struct

```c
struct list_t *p = malloc(sizeof(struct list_t));
```

或更安全：

```c
struct list_t *p = malloc(sizeof(*p));
```

#### string

```c
char *s = malloc(strlen(src) + 1);
strcpy(s, src);
```

**为什么 +1？**

> 留给 `'\0'`。

---

### B3. `strcpy` / `sprintf` 高频坑

| 函数 | 本质 | 必须先做什么 |
|---|---|---|
| `strcpy(dest, src)` | 往 dest 复制字符串 | dest 有足够空间 |
| `sprintf(dest, ...)` | 往 dest 写格式化字符串 | dest 有足够空间 |
| `strlen(s)` | 不包括 `'\0'` | malloc 时要 `+1` |

错误：

```c
char title[80];
strcpy(title, newTitle);   // newTitle arbitrary length，可能 overflow
```

更稳：

```c
char *title = malloc(strlen(newTitle) + 1);
strcpy(title, newTitle);
```

---

### B4. 千万不要返回 / 保存 local variable 地址

错误：

```c
struct list_t newItem;
prevItem->next = &newItem;
return allItems;
```

#### 错在哪里？

`newItem` 是 local variable，函数结束就没了。  
链表里保存它的地址 = dangling pointer。

正确：

```c
struct list_t *newItem = malloc(sizeof(struct list_t));
```

**口诀：**

> 要放进链表 / 树里长期存在的 node，必须 heap malloc，不能是 local variable。

---

### B5. free 的三个必考坑

#### 1. 不能 free 非 heap 内存

错误：

```c
int value;
free(node->value);
```

因为 `value` 是 int，不是 malloc 出来的指针。

正确：

```c
// 不 free node->value
free(node);
```

#### 2. free 之后不能再访问

错误：

```c
free(items);
items = items->next;
```

正确：

```c
struct list_t *next = items->next;
free(items);
items = next;
```

#### 3. tree/list 要先递归 free children，再 free 自己

错误：

```c
free(node);
deleteNode(node->left);
deleteNode(node->right);
```

正确：

```c
void deleteNode(treeNode_t *node) {
    if (node == NULL) return;

    deleteNode(node->left);
    deleteNode(node->right);
    free(node);
}
```

**口诀：**

> list：先存 next，再 free 当前。  
> tree：先 free children，再 free parent。  
> free 之后不要碰它。

---

### B6. NULL check 必扫

看到：

```c
currentItem->next
currentItem->entry
msgList->message
node->left
```

先问：

> 这个指针有没有可能是 NULL？

错误：

```c
while (currentItem->entry.entry_id < newID) {
    currentItem = currentItem->next;
}
```

正确：

```c
while (currentItem != NULL && currentItem->entry.entry_id < newID) {
    ...
}
```

错误：

```c
memcpy(buffer, msgList->message, length);
```

正确思路：

```c
if (msgList == NULL) return error;
```

---

### B7. linked list 插入通用模板

#### 头插

```c
newNode->next = head;
head = newNode;
```

#### 如果要修改 head，函数参数用 `**`

```c
void insert(Node **head, Node *newNode) {
    newNode->next = *head;
    *head = newNode;
}
```

#### 有序插入模板

```c
int enqueue(NODE **queue, NODE *p) {
    NODE *prev = NULL;
    NODE *cur = *queue;

    while (cur != NULL && cur->priority >= p->priority) {
        prev = cur;
        cur = cur->next;
    }

    if (prev == NULL) {
        p->next = *queue;
        *queue = p;
    } else {
        p->next = cur;
        prev->next = p;
    }

    return 0;
}
```

如果同 priority 要 FIFO：

```c
while (cur != NULL && cur->priority >= p->priority)
```

因为相同 priority 会继续往后走，新来的放在旧的后面。

---

### B8. array / buffer 越界

看到这些就检查边界：

```c
arr[i]
values[count++]
values[--count]
requests[request_count]
char title[80]
```

必须有：

```c
if (i >= 0 && i < SIZE)
if (count < STACK_SIZE)
if (count > 0)
```

**一句话：**

> C 不检查 array bounds，越界可能 crash，也可能 silently corrupt memory。

---

### B9. user space buffer / kernel code 特殊但常见

看到 kernel / driver code：

```c
memcpy(buffer, msgList->message, length);  // buffer is userspace
memcpy(msgList->message, buffer, length);
```

考试答案：

```text
Do not use memcpy directly with user-space pointers in kernel code.
Use copy_to_user() when copying data to user space.
Use copy_from_user() when copying data from user space.
Check return values.
```

同时还要检查：

```text
msgList 是否 NULL
message 是否 malloc
length 是否超过实际 message size
msgSize 更新是否加锁
```

---

## 最后 2 分钟扫 code 顺序

### 1. 先扫所有 `->`

```text
p->x
```

问：

```text
p 有没有 malloc？
p 会不会 NULL？
p 有没有已经 free？
```

### 2. 再扫所有 string 函数

```text
strcpy
sprintf
memcpy
```

问：

```text
dest 有没有空间？
长度够不够？
要不要 +1 给 '\0'？
```

### 3. 再扫所有 free

```text
free(x)
```

问：

```text
x 是不是 heap malloc 出来的？
free 后有没有继续 x->next？
树是不是先 free children？
```

### 4. 再扫所有 shared global variable

```text
count
request_count
msgList
msgSize
allSeats
noOfMealsAvailable
```

问：

```text
有没有同一把 mutex？
check 和 update 有没有一起 lock？
有没有 return before unlock？
```

### 5. 再扫 pthread_create

```c
pthread_create(..., &i);
```

问：

```text
是不是传了 loop variable 的地址？
每个 thread 有没有独立参数？
有没有 pthread_join？
```

---

## 超短答案模板

### 普通 code 错误题

```text
The code stores/uses an invalid pointer.
Memory must be allocated before writing through the pointer.
The code stores the address of a local variable, which becomes invalid after the function returns.
The code may dereference NULL.
The code frees memory and then accesses it again.
The code may overflow the fixed-size buffer.
```

### mutex code 错误题

```text
There is a race condition on the shared variables.
The check and the update are not atomic.
The same shared data structure must be protected by the same mutex.
The lock should be held only while accessing shared state.
Unlock must happen before every return.
```

---

## 本卷反复出现的高频 code 题来源

- `pthread_create(..., &i)`：多份卷出现，核心是 loop variable 地址共享。
- Stack / request queue / seat booking：核心是 shared state + mutex。
- linked list / tree / string：核心是 malloc、NULL、free、local variable address、buffer overflow。
- kernel message list：核心是 user buffer 不能直接 memcpy、共享链表要保护。
