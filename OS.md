# C 语言完整复习笔记

> 本笔记由原始乱序代码与注释整理而来，按照学习 C 语言的合理顺序重新排版。  
> 目标：适合复习、查漏补缺、写代码前快速翻阅。  
> 说明：代码块以 `c` 标注，方便在 VS Code / GitHub 中阅读。

---

## 目录跳转

- [1. C 程序基本结构](#1-c-程序基本结构)
- [2. `printf`、变量与基本数据类型](#2-printf变量与基本数据类型)
- [3. 运算符、类型转换与控制语句](#3-运算符类型转换与控制语句)
- [4. 数组与二维数组](#4-数组与二维数组)
- [5. 指针基础](#5-指针基础)
- [6. 字符串与字符数组](#6-字符串与字符数组)
- [7. 函数与函数指针](#7-函数与函数指针)
- [8. 结构体 `struct`](#8-结构体-struct)
- [9. 联合体 `union`](#9-联合体-union)
- [10. 动态内存：`malloc` / `calloc` / `realloc` / `free`](#10-动态内存malloc--calloc--realloc--free)
- [11. 文件操作：`fopen` / `fgetc` / `fgets` / `fread` / `fseek`](#11-文件操作fopen--fgetc--fgets--fread--fseek)
- [12. 链表 Linked List](#12-链表-linked-list)
- [13. 多线程基础：`pthread`](#13-多线程基础pthread)
- [14. 条件变量 Condition Variable](#14-条件变量-condition-variable)
- [15. 链表多线程同步：Mutex 与 Read-Write Lock](#15-链表多线程同步mutex-与-read-write-lock)
- [16. Socket 编程：Client / Server](#16-socket-编程client--server)
- [17. 常见易错点总结](#17-常见易错点总结)

---

# 1. C 程序基本结构

一个最基础的 C 程序通常包含：

```c
#include <stdio.h>

int main() {
    printf("Hello World!\n");
    return 0;
}
```

核心结构：

```c
#include <stdio.h>
```

表示引入标准输入输出库，里面包含 `printf`、`scanf` 等函数。

```c
int main()
```

是程序入口。C 程序运行时会从 `main` 函数开始执行。

```c
return 0;
```

表示程序正常结束。

---

## 注释

C 语言中有两种常见注释：

```c
// 单行注释

/*
多行注释
多行注释
*/
```

VS Code 常用快捷键：

```text
Ctrl + K + C  加注释
Ctrl + K + U  去注释
```

---

# 2. `printf`、变量与基本数据类型

## 2.1 `printf`

`printf` 用来把内容打印到终端。

```c
#include <stdio.h>

int main() {
    printf("Hello 也一样!\n");
    printf("Laker今年%d岁了，中文名：%s，身高%.1fcm\n", 22, "湖人", 177.5);
    return 0;
}
```

常见格式符：

| 格式符 | 含义 |
|---|---|
| `%d` | 打印 `int` |
| `%u` | 打印 `unsigned int` |
| `%f` | 打印 `float` / `double` |
| `%.2f` | 保留两位小数 |
| `%c` | 打印字符 |
| `%s` | 打印字符串 |
| `%zu` | 打印 `sizeof` 的结果 |

例子：

```c
int a = 12;
unsigned e = 40;
float g = 3.1415F;
double f = 3.1415;
char zf = 'a';

printf("%d\n", a);
printf("%u\n", e);
printf("%.2f\n", g);
printf("%.2lf\n", f);
printf("%c\n", zf);
printf("%zu\n", sizeof(a));
```

---

## 2.2 基本数据类型

```c
int a = 12;
int b = 13;
int c = 20, d = 30;

unsigned e = 40;

float g = 3.1415F;
double f = 3.1415;

char ch = 'a';
```

简单理解：

| 类型 | 含义 |
|---|---|
| `int` | 整数 |
| `unsigned` | 无符号整数，不存负数 |
| `float` | 单精度小数 |
| `double` | 双精度小数 |
| `char` | 单个字符 |

---

# 3. 运算符、类型转换与控制语句

## 3.1 除法与取余

```c
printf("%d\n", 5 / 2);  // 结果是 2
printf("%d\n", 9 % 2);  // 结果是 1
```

重点：

- `5 / 2` 如果两边都是整数，结果也是整数，所以是 `2`。
- `%` 是取余。
- `10 % -3` 的结果正负通常与第一个数一致，也就是 `10` 的正号。

---

## 3.2 自增与复合赋值

```c
a++;   // 先使用 a，再让 a 加 1
++a;   // 先让 a 加 1，再使用 a

a += b;  // 等价于 a = a + b
a -= b;  // 等价于 a = a - b
```

---

## 3.3 逻辑运算短路

```c
a && b
```

如果 `a` 不成立，`b` 不会继续计算。

```c
a || b
```

如果 `a` 成立，`b` 不会继续计算。

这叫 short-circuit evaluation，中文可以理解为“短路求值”。

---

## 3.4 三目运算符

```c
condition ? value_if_true : value_if_false;
```

例子：

```c
int max = a > b ? a : b;
```

人话：

如果 `a > b` 成立，就返回 `a`，否则返回 `b`。

---

## 3.5 `switch` 与 `break`

```c
switch (x) {
    case 1:
        printf("one\n");
        break;
    case 2:
        printf("two\n");
        break;
    default:
        printf("other\n");
}
```

重点：

如果 `case` 后面没有 `break`，程序会继续执行下一个 `case`，直到遇到 `break` 或结束。

---

## 3.6 `continue`

```c
for (int i = 0; i < 5; i++) {
    if (i == 2) {
        continue;
    }
    printf("%d\n", i);
}
```

`continue` 的意思是：

结束本轮循环，直接进入下一轮循环。

---

## 3.7 隐式类型转换与强制类型转换

隐式转换：

```c
int a = 10;
double b = 3.5;

double c = a + b;
```

这里 `a` 会自动转成 `double`。

常见规则：

- `int + double`：`int` 自动转成 `double`
- `short` 和 `char` 参与运算时通常会先转成 `int`

强制转换：

```c
int a = 300;
short b = (short)a;
```

---

# 4. 数组与二维数组

## 4.1 一维数组

```c
int arr[5] = {10, 20, 30, 40, 50};

printf("%d\n", arr[2]);      // 30
printf("%d\n", *(arr + 2));  // 30
printf("%d\n", 2[arr]);      // 30
```

为什么 `arr[2]` 和 `*(arr + 2)` 一样？

因为：

```c
arr[i]
```

本质上可以理解为：

```c
*(arr + i)
```

所以 `2[arr]` 也能工作，因为它等价于：

```c
*(2 + arr)
```

但实际写代码不要用 `2[arr]`，可读性很差。

---

## 4.2 数组名退化为指针

```c
int arr[] = {1, 2, 3, 4, 5};
int* p = arr;
```

这里 `arr` 在很多表达式中会退化为指向第一个元素的指针：

```c
arr == &arr[0]
```

遍历：

```c
for (int i = 0; i < 5; i++) {
    printf("%d ", *p);
    p++;
}
```

注意：

```c
int* p = arr;
```

`p++` 的步长是一个 `int`。

---

## 4.3 二维数组

二维数组可以理解成“行 + 列”，像电影院座位一样。

```c
int a2[4][3] = {
    {1, 2, 3},
    {4, 5, 6},
    {7, 8, 9},
    {10, 11, 12}
};
```

遍历：

```c
for (int i = 0; i < 4; i++) {
    for (int j = 0; j < 3; j++) {
        printf("%d ", a2[i][j]);
    }
    printf("\n");
}
```

---

## 4.4 二维数组指针

```c
int (*p2)[3] = a2;
```

意思是：

`p2` 是一个指针，它指向“包含 3 个 int 的数组”。

所以：

```c
p2++;
```

不是移动到下一个 `int`，而是移动到下一行。

遍历思路：

```c
int (*p2)[3] = a2;

for (int i = 0; i < 4; i++) {
    for (int j = 0; j < 3; j++) {
        printf("%d ", *(*(p2 + i) + j));
    }
    printf("\n");
}
```

---

# 5. 指针基础

## 5.1 地址与解引用

```c
int a = 10;
int* p = &a;
```

这里：

| 表达式 | 含义 |
|---|---|
| `a` | 变量本身 |
| `&a` | `a` 的地址 |
| `p` | 存放地址的变量 |
| `*p` | 通过地址找到里面的值 |

例子：

```c
printf("%d\n", a);
printf("%p\n", &a);
printf("%p\n", p);
printf("%d\n", *p);
```

---

## 5.2 二级指针

```c
int a = 10;
int* p = &a;
int** pp = &p;
```

关系：

```text
p   = &a
*p  = a

pp  = &p
*pp = p
**pp = a
```

人话理解：

- `p` 指向 `a`
- `pp` 指向 `p`
- `**pp` 才能最终拿到 `a`

---

## 5.3 `void*`

```c
int a = 10;
int* p1 = &a;

void* p2 = p1;
```

`void*` 可以接收任意类型的指针。

但是 `void*` 不能直接解引用，因为编译器不知道它指向的数据应该按几个字节读取。

常见用途：

```c
void swap(void* p1, void* p2) {
    // 通用指针参数
}
```

---

## 5.4 `scanf` 为什么要地址

```c
int x;
scanf("%d", &x);
```

因为 `scanf` 要修改变量本身，所以必须拿到变量的地址。

如果是字符数组：

```c
char str[100];
scanf("%s", str);
```

这里不用写 `&str`，因为 `str` 数组名会退化成指向第一个字符的指针。

注意：

```c
scanf("%s", str);
```

遇到空格就会停止读取，所以无法直接读取 `"hi lyp"` 这种带空格的输入。

---

# 6. 字符串与字符数组

## 6.1 字符数组字符串

```c
char str[4] = "abc";
```

虽然 `"abc"` 看起来只有 3 个字符，但 C 字符串最后需要一个隐藏的结束符：

```c
'\0'
```

所以实际存储是：

```text
'a' 'b' 'c' '\0'
```

因此 `char str[4]` 刚好够。

---

## 6.2 字符串字面量指针

```c
char* str1 = "abc";
```

这表示 `str1` 指向字符串字面量 `"abc"`。

重点：

- 字符串字面量通常放在只读常量区。
- 内容不应该修改。
- 如果创建同样内容的字符串，编译器可能复用同一块地址。

更安全的写法：

```c
const char* str1 = "abc";
```

---

## 6.3 字符串数组

### 方法一：二维字符数组

```c
char str2[3][50] = {
    "apple",
    "banana",
    "cat"
};
```

遍历：

```c
for (int i = 0; i < 3; i++) {
    printf("%s\n", str2[i]);
}
```

也可以逐字符打印：

```c
for (int i = 0; i < 3; i++) {
    for (int j = 0; str2[i][j] != '\0'; j++) {
        printf("%c", str2[i][j]);
    }
    printf("\n");
}
```

---

### 方法二：字符串指针数组

```c
char* strArr[3] = {
    "666",
    "777",
    "888"
};
```

这里 `strArr` 是一个数组，里面每个元素都是 `char*`，分别指向一个字符串。

---

## 6.4 用指针遍历字符串

```c
char str[4] = "abc";
char* p = str;

while (*p != '\0') {
    printf("%c", *p);
    p++;
}
printf("\n");
```

人话：

从第一个字符开始打印，每次移动到下一个字符，直到遇到 `'\0'` 停止。

---

## 6.5 `%s`

```c
char strs[12] = "string";
printf("%s\n", strs);
```

`%s` 的意思是：

从给定地址开始，一个字符一个字符往后打印，直到遇到 `'\0'`。

例子：

```c
char str2[3][50] = {
    "apple",
    "banana",
    "cat"
};

for (int i = 0; i < 3; i++) {
    char* p = str2[i];
    printf("%s\n", p);
}
```

---

## 6.6 `strcpy`

```c
#include <string.h>

char a2[12] = "abc";
strcpy(a2, "ABC");

printf("%s\n", a2);
```

因为数组名对应的内存可以修改，所以可以用 `strcpy` 把新字符串复制进去。

注意：

```c
char name[20];
name = "Tom";  // 错误
strcpy(name, "Tom");  // 正确
```

---

# 7. 函数与函数指针

## 7.1 普通函数

```c
int add(int a, int b) {
    return a + b;
}

int sub(int a, int b) {
    return a - b;
}
```

函数由三部分组成：

```text
返回值类型 函数名(参数列表)
```

---

## 7.2 函数指针

函数指针用来指向一个函数。

```c
int (*op)(int, int);
```

意思是：

`op` 是一个指针，指向“接收两个 int，返回 int”的函数。

例子：

```c
#include <stdio.h>

int add(int a, int b) {
    return a + b;
}

int sub(int a, int b) {
    return a - b;
}

void calculate(int a, int b, int (*op)(int, int)) {
    printf("%d\n", op(a, b));
}

int main() {
    calculate(5, 3, add);
    calculate(5, 3, sub);
    return 0;
}
```

重点：

只要函数类型匹配：

```c
int function_name(int, int)
```

就可以被：

```c
int (*op)(int, int)
```

指向。

---

## 7.3 函数指针数组

```c
int (*arr[2])(int, int) = {add, sub};
```

意思是：

`arr` 是一个数组，里面存的是函数指针。

使用：

```c
int result = arr[0](5, 3);  // 调用 add
```

---

# 8. 结构体 `struct`

## 8.1 基本结构体

```c
struct Student {
    char name[20];
    int age;
};
```

定义变量：

```c
struct Student s1;
```

赋值：

```c
strcpy(s1.name, "Tom");
s1.age = 20;
```

注意：

如果 `name` 是字符数组，不能这样赋值：

```c
s1.name = "Tom";  // 错误
```

要用：

```c
strcpy(s1.name, "Tom");
```
如果 `name` 是 char *name
```
struct Student {
    char *name;
};

struct Student s1;
s1.name = "Tom";   // ✅ 可以
```

---

## 8.2 初始化结构体

```c
struct Student s1 = {"Tom", 20};
```

也可以用 designated initializer：

```c
struct Student s2 = {
    .name = "Jack",
    .age = 19
};
```

注意：

初始化时可以直接写字符串；但初始化之后再修改字符数组，就要用 `strcpy`。

---

## 8.3 嵌套结构体

```c
#include <stdio.h>
#include <string.h>

struct Address {
    char city[20];
    int postcode;
};

struct Student {
    char name[20];
    int age;
    struct Address addr;
};

int main() {
    struct Student s1;

    strcpy(s1.name, "Tom");
    s1.age = 20;
    strcpy(s1.addr.city, "London");
    s1.addr.postcode = 12345;

    printf("Name: %s\n", s1.name);
    printf("Age: %d\n", s1.age);
    printf("City: %s\n", s1.addr.city);
    printf("Postcode: %d\n", s1.addr.postcode);

    return 0;
}
```

---

## 8.4 结构体指针与 `->`

```c
struct Student* p = &s1;
p->age = 30;
```

`p->age` 等价于：

```c
(*p).age
```

结构体指针访问成员时一般用 `->`，更方便。

---

## 8.5 结构体作为函数参数

如果想在函数里修改结构体，通常传结构体指针。

```c
#include <stdio.h>

struct Student {
    char name[50];
    int age;
};

void changeAge(struct Student* s) {
    s->age = 30;
}

int main() {
    struct Student s1 = {"Tom", 20};

    changeAge(&s1);

    printf("%d\n", s1.age);
    return 0;
}
```

重点：

```c
changeAge(&s1);
```

传的是地址。

---

## 8.6 结构体数组

```c
struct Student students[3] = {
    {"Tom", 20},
    {"Jack", 19},
    {"Lucy", 21}
};
```

访问：

```c
printf("%s\n", students[0].name);
printf("%d\n", students[1].age);
```

---

# 9. 联合体 `union`

## 9.1 `union` 基本概念
union（联合体） 很像 struct，但关键区别是：

* struct 里面每个成员都有自己的内存；union 里面所有成员共用同一块内存。

```c
union Data {
    int i;
    float f;
};
```

定义对象：

```c
union Data d;
```

赋值：

```c
d.i = 10;
printf("i = %d\n", d.i);

d.f = 3.14;
printf("f = %f\n", d.f);
```

重点：

`union` 里的所有成员共享同一块内存。

所以：

```c
d.i = 10;
d.f = 3.14;
```

第二次给 `d.f` 赋值会覆盖之前 `d.i` 的数据。

---

## 9.2 `union` 的大小

`union` 的内存大小通常由里面最大的成员决定。

例如：

```c
union Data {
    int i;
    char c;
};
```

如果 `int` 是 4 字节，`char` 是 1 字节，那么这个 `union` 至少需要 4 字节。

---

# 10. 动态内存：`malloc` / `calloc` / `realloc` / `free`

## 10.1 `malloc`
* `malloc` 返回的是地址值，所以用指针接收。
```c
#include <stdio.h>
#include <stdlib.h>

int main() {
    int* p;

    p = malloc(100 * sizeof(int));

    *p = 10;
    printf("%d\n", *p);

    free(p);

    return 0;
}
```

`malloc` 用于从 heap 上申请内存空间。

```c
malloc(100 * sizeof(int))
```

意思是申请 100 个 `int` 的空间。

---

## 10.2 `free`

```c
free(p);
```

释放申请的 heap 内存。

注意：

释放之后，`p` 会变成 dangling pointer，中文可以叫“悬空指针”。

不要：

```c
free(p);
free(p);  // double free，错误
```

也不要在 `free(p)` 后继续使用 `*p`。

---

## 10.3 `calloc`
* malloc 只申请内存，不初始化；里面的值是 shit code

* calloc 申请内存，并把内容初始化为 0。

```c
int* p1 = calloc(10, sizeof(int));
```
* malloc 要的是“总字节数”；calloc 要的是“几个元素 × 每个元素多大”。
`calloc` 申请空间并初始化为 0。

所以 `p1` 指向的 10 个 `int` 初始值都是 0。

---

## 10.4 `realloc`

```c
int* p2 = realloc(p, 200 * sizeof(int));
```

`realloc` 用于扩容或缩小已经申请的内存。

重点：

- 原本已有的数据会尽量保留。
- 新扩展出来的空间不会自动初始化，可能是垃圾值。
- 实际写代码时不要直接覆盖原指针，最好先用临时指针接收。

更安全的写法：

```c
int* temp = realloc(p, 200 * sizeof(int));
if (temp != NULL) {
    p = temp;
}
```

---

## 10.5 动态结构体

```c
struct Student {
    char name[50];
    int age;
};

struct Student* s;
s = malloc(sizeof(struct Student));

s->age = 20;

free(s);
```

---

## 10.6 不要返回局部变量地址

错误示范：

```c
int* f() {
    int a = 10;
    return &a;
}
```

原因：

`a` 是局部变量，存在 stack 上。函数结束后，`a` 的生命周期结束，地址不再可靠。

正确示范：

```c
int* f() {
    int* p = malloc(sizeof(int));
    *p = 10;
    return p;
}
```

这里 `p` 指向 heap，所以函数结束后这块内存仍然存在。用完后记得 `free`。

---

## 10.7 Memory Leak

Memory leak，内存泄漏：

程序申请了 heap 内存，但之后没有释放，导致这块内存一直占用，无法被再次使用。

典型情况：

```c
int* p = malloc(sizeof(int));
*p = 10;

// 忘记 free(p)
```
another case:
```c
p = realloc(p, 200 * sizeof(int));
```
如果 realloc 成功：p 会指向新的内存，没问题。

但是如果 realloc 失败：p = NULL;

这时候你原来那块内存其实没有被`free`，但你已经找不到它了。

---

## 10.8 Virtual Memory

Virtual memory，虚拟内存：

操作系统给每个进程一种“自己拥有一大片连续内存”的错觉。实际上，OS 会把这些虚拟地址映射到真实物理内存 RAM，必要时也可能映射到磁盘。

---

# 11. 文件操作：`fopen` / `fgetc` / `fgets` / `fread` / `fseek`

## 11.1 打开文件

```c
FILE* file = fopen("C:\\Users\\12058\\Desktop\\awe.txt", "r");
```

结构：

```c
fopen("路径", "模式");
```

常见模式：

| 模式 | 含义 |
|---|---|
| `"r"` | 只读 |
| `"w"` | 写入，会清空原文件 |
| `"a"` | 追加 |
| `"r+"` | 读写，文件必须存在 |
| `"w+"` | 读写，会清空原文件 |

打开后最好检查：

```c
if (file == NULL) {
    printf("文件打开失败\n");
    return 1;
}
```

---

## 11.2 `fgetc`：一次读一个字符

```c
int c;

while ((c = fgetc(file)) != EOF) {
    printf("%c", c);
}
```

重点：

- 读不到时返回 `EOF`, EOF 通常是 -1
- 返回类型用 `int`，不要用 `char`，因为要能表示 `EOF`(-1)

---

## 11.3 `fgets`：一次读一行

```c
char arr[1024];

if (fgets(arr, 1024, file) != NULL) {
    printf("%s", arr);
}
```

`fgets` 会读取一行，最多读取 `1024 - 1` 个字符，并自动补上 `'\0'`。

---

## 11.4 `fread`：读取多个字节

```c
char arr1[1024];

size_t n = fread(arr1, 1, 1024, file);
```

结构：

```c
fread(存放位置, 每个元素大小, 元素数量, 文件指针);
```

返回值是成功读到的元素数量。

---

## 11.5 `fseek`：随机访问

```c
fseek(file, 7, SEEK_SET);
```

结构：

```c
fseek(file指针, offset偏移量, 起点);
```

常见起点：

| 写法 | 含义 |
|---|---|
| `SEEK_SET` | 文件开头 |
| `SEEK_CUR` | 当前的位置 |
| `SEEK_END` | 文件末尾 |

例子：

```c
FILE* p = fopen("file.txt", "w+");

fprintf(p, "%s", "This is something");

fseek(p, 7, SEEK_SET);

fprintf(p, "%s", "C Language");

fclose(p);
```

---

## 11.6 文件操作完整例子

```c
#include <stdio.h>

int main() {
    FILE* file = fopen("example.txt", "r");

    if (file == NULL) {
        printf("文件打开失败\n");
        return 1;
    }

    int c;
    while ((c = fgetc(file)) != EOF) {
        printf("%c", c);
    }

    fclose(file);
    return 0;
}
```

---

# 12. 链表 Linked List

## 12.1 节点结构

```c
typedef struct Node {
    int data;
    struct Node* next;
} Node;
```

一个链表节点包含：

- `data`：当前节点的数据
- `next`：指向下一个节点的指针

---

## 12.2 创建新节点

```c
#include <stdlib.h>

Node* createNode(int value) {
    Node* newNode = malloc(sizeof(Node));

    newNode->data = value;
    newNode->next = NULL;

    return newNode;
}
```

---

## 12.3 升序插入

```c
void insert(Node** head, int value) {
    Node* newNode = createNode(value);

    if (*head == NULL || (*head)->data >= value) {
        newNode->next = *head;
        *head = newNode;
        return;
    }

    Node* curr = *head;

    while (curr->next != NULL && curr->next->data < value) {
        curr = curr->next;
    }

    newNode->next = curr->next;
    curr->next = newNode;
}
```

为什么参数是：

```c
Node** head
```

因为我们可能要修改“头指针本身”。

如果只传：

```c
Node* head
```

那么函数内部只是拿到了头指针的副本，不能真正改变外面的 `head` 指向。

---

## 12.4 查找 `member`

```c
int member(Node* head, int value) {
    Node* curr = head;

    while (curr != NULL && curr->data < value) {
        curr = curr->next;
    }

    if (curr != NULL && curr->data == value) {
        return 1;
    } else {
        return 0;
    }
}
```

因为链表是升序的，所以当 `curr->data >= value` 时就可以停下来。

---

## 12.5 打印链表

```c
void printList(Node* head) {
    Node* curr = head;

    while (curr != NULL) {
        printf("%d -> ", curr->data);
        curr = curr->next;
    }

    printf("NULL\n");
}
```

---

## 12.6 完整链表示例

```c
#include <stdio.h>
#include <stdlib.h>

typedef struct Node {
    int data;
    struct Node* next;
} Node;

Node* createNode(int value) {
    Node* newNode = malloc(sizeof(Node));

    newNode->data = value;
    newNode->next = NULL;

    return newNode;
}

void insert(Node** head, int value) {
    Node* newNode = createNode(value);

    if (*head == NULL || (*head)->data >= value) {
        newNode->next = *head;
        *head = newNode;
        return;
    }

    Node* curr = *head;

    while (curr->next != NULL && curr->next->data < value) {
        curr = curr->next;
    }

    newNode->next = curr->next;
    curr->next = newNode;
}

int member(Node* head, int value) {
    Node* curr = head;

    while (curr != NULL && curr->data < value) {
        curr = curr->next;
    }

    return curr != NULL && curr->data == value;
}

void printList(Node* head) {
    Node* curr = head;

    while (curr != NULL) {
        printf("%d -> ", curr->data);
        curr = curr->next;
    }

    printf("NULL\n");
}

int main() {
    Node* head = NULL;

    insert(&head, 7);
    insert(&head, 3);
    insert(&head, 10);
    insert(&head, 8);

    printf("当前链表：\n");
    printList(head);

    printf("是否存在 7: %d\n", member(head, 7));
    printf("是否存在 5: %d\n", member(head, 5));

    return 0;
}
```

---

# 13. 多线程基础：`pthread`

## 13.1 创建线程

```c
#include <stdio.h>
#include <pthread.h>
#include <unistd.h>

void* do_one_thing(void* arg) {
    for (int i = 0; i < 200; i++) {
        printf("doing one thing\n");
    }

    return NULL;
}

void* do_another_thing(void* arg) {
    for (int i = 0; i < 200; i++) {
        printf("doing another\n");
    }

    return NULL;
}

int main() {
    pthread_t thread1, thread2;

    pthread_create(&thread1, NULL, do_one_thing, NULL);
    pthread_create(&thread2, NULL, do_another_thing, NULL);

    pthread_join(thread1, NULL);
    pthread_join(thread2, NULL);

    return 0;
}
```

---

## 13.2 `pthread_create`

```c
pthread_create(&thread1, NULL, do_one_thing, NULL);
```

参数理解：

| 参数 | 含义 |
|---|---|
| `&thread1` | 保存线程 ID |
| `NULL` | 线程属性，默认即可 |
| `do_one_thing` | 线程要执行的函数 |
| `NULL` | 传给线程函数的参数 |

线程函数标准形式最好写成：

```c
void* function_name(void* arg)
```

---

## 13.3 `pthread_join`

```c
pthread_join(thread1, NULL);
```

意思是：

主线程等待 `thread1` 执行结束。

如果不用 `pthread_join`，主线程可能提前结束，导致子线程还没执行完程序就退出了。

---

# 14. 条件变量 Condition Variable

## 14.1 基本思想

条件变量用于线程之间等待某个条件发生。

典型场景：

- 消费者线程等待数据准备好。
- 生产者线程准备好数据后通知消费者。

需要三个东西：

```c
pthread_mutex_t mutex;
pthread_cond_t cond;
int data_ready = 0;
```

---

## 14.2 为什么条件变量要搭配 mutex

关键原因：

线程不只是等 signal，它还要检查某个共享条件，例如：

```c
data_ready == 1
```

这个共享条件必须被 mutex 保护，否则可能出现 lost wakeup。

也就是：

生产者 signal 发早了，但消费者还没进入等待状态，结果消费者永远等不到。

所以正确写法是：

```c
pthread_mutex_lock(&mutex);

while (data_ready == 0) {
    pthread_cond_wait(&cond, &mutex);
}

pthread_mutex_unlock(&mutex);
```

---

## 14.3 `pthread_cond_wait` 做了什么

```c
pthread_cond_wait(&cond, &mutex);
```

它会自动做三件事：

1. 释放 `mutex`
2. 当前线程进入睡眠，等待 signal
3. 被唤醒后，重新尝试获得 `mutex`

所以等待线程不是一直拿着锁睡觉，而是会释放锁，让其他线程有机会修改条件。

---

## 14.4 为什么用 `while` 而不是 `if`

推荐写法：

```c
while (data_ready == 0) {
    pthread_cond_wait(&cond, &mutex);
}
```

不用：

```c
if (data_ready == 0) {
    pthread_cond_wait(&cond, &mutex);
}
```

原因：

线程被唤醒后，不代表条件一定成立。可能有 spurious wakeup，也可能其他线程先抢走资源。因此醒来后要再次检查条件。

---

## 14.5 生产者消费者完整例子

```c
#include <stdio.h>
#include <pthread.h>
#include <unistd.h>

int data = 0;
int data_ready = 0;

pthread_mutex_t mutex;
pthread_cond_t cond;

void* producer(void* arg) {
    sleep(2);

    pthread_mutex_lock(&mutex);

    data = 100;
    data_ready = 1;

    printf("生产者的数据已经处理并准备好了\n");

    pthread_cond_signal(&cond);

    pthread_mutex_unlock(&mutex);

    return NULL;
}

void* consumer(void* arg) {
    pthread_mutex_lock(&mutex);

    while (data_ready == 0) {
        printf("消费者正在等待中...\n");
        pthread_cond_wait(&cond, &mutex);
    }

    printf("消费者：被生产者唤醒，得到数据 = %d\n", data);

    pthread_mutex_unlock(&mutex);

    return NULL;
}

int main() {
    pthread_t t1, t2;

    pthread_mutex_init(&mutex, NULL);
    pthread_cond_init(&cond, NULL);

    pthread_create(&t1, NULL, consumer, NULL);
    pthread_create(&t2, NULL, producer, NULL);

    pthread_join(t1, NULL);
    pthread_join(t2, NULL);

    pthread_mutex_destroy(&mutex);
    pthread_cond_destroy(&cond);

    return 0;
}
```

---

## 14.6 Signal 之后线程会立刻执行吗？

不一定。

假设生产者这样写：

```c
pthread_mutex_lock(&mutex);

data_ready = 1;
pthread_cond_signal(&cond);

pthread_mutex_unlock(&mutex);
```

当生产者 `signal` 后，消费者会被唤醒，但它还不能立刻继续执行，因为 mutex 还在生产者手里。

消费者必须等生产者：

```c
pthread_mutex_unlock(&mutex);
```

之后才能重新拿到锁并继续执行。

---

# 15. 链表多线程同步：Mutex 与 Read-Write Lock

## 15.1 链表并发访问问题

对于链表：

- read 操作，比如 `member`，多个线程同时读通常不会互相影响。
- write 操作，比如 `insert`、`delete`，会修改节点关系，不能随便同时执行。

---

## 15.2 方法一：整个链表一个 mutex

```c
pthread_mutex_lock(&mutex1);
member(value0);
pthread_mutex_unlock(&mutex1);

pthread_mutex_lock(&mutex1);
insert(value1);
pthread_mutex_unlock(&mutex1);
```

优点：

简单。

缺点：

如果大多数操作都是 read，这样很不划算，因为多个读操作本来可以同时进行，却被 mutex 强行变成一次只能一个线程访问。

---

## 15.3 方法二：每个 node 一个 mutex

```c
typedef struct Node {
    int data;
    struct Node* next;
    pthread_mutex_t mutex;
} Node_t;
```

基本思路：

1. lock 下一个 node
2. unlock 当前 node
3. 指针移动到下一个 node

缺点：

每一步都要加锁解锁，performance 可能更慢，代码也更复杂。

---

## 15.4 方法三：Read-Write Lock

Read-Write Lock 允许：

- 多个线程同时读
- 但同一时间只能有一个线程写
- 写的时候不能读

常用函数：

```c
pthread_rwlock_rdlock(&lock);  // 读锁，共享锁
pthread_rwlock_wrlock(&lock);  // 写锁，独占锁
pthread_rwlock_unlock(&lock);
```

读操作：

```c
pthread_rwlock_rdlock(&lock);
member(value1);
pthread_rwlock_unlock(&lock);
```

写操作：

```c
pthread_rwlock_wrlock(&lock);
insert(value2);
pthread_rwlock_unlock(&lock);
```

---

## 15.5 读写锁规则

`pthread_rwlock_rdlock`：

```text
如果没有线程持有锁：可以拿到读锁。
如果其他线程持有读锁：也可以拿到读锁。
如果有线程持有写锁：等待。
```

`pthread_rwlock_wrlock`：

```text
如果没有任何线程持有读锁或写锁：可以拿到写锁。
否则等待。
```

---

# 16. Socket 编程：Client / Server

## 16.1 Socket 基本流程

Server 端：

1. `socket` 创建 socket
2. `bind` 绑定 IP 和端口
3. `listen` 开始监听
4. `accept` 接受客户端连接
5. `read` 接收数据
6. `write` 发送数据
7. `close` 关闭连接

Client 端：

1. `socket` 创建 socket
2. 配置服务器地址
3. `connect` 连接服务器
4. `write` 发送数据
5. `read` 接收服务器回复
6. `close` 关闭连接

---

## 16.2 Server 端代码

```c
// server.c
#include <stdio.h>
#include <string.h>
#include <unistd.h>
#include <arpa/inet.h>

int main() {
    int server_fd, client_fd;

    struct sockaddr_in server_addr;
    struct sockaddr_in client_addr;

    socklen_t addr_len = sizeof(client_addr);

    char buffer[1024] = {0};

    // 1. 创建 socket
    server_fd = socket(AF_INET, SOCK_STREAM, 0);

    // 2. 配置地址
    server_addr.sin_family = AF_INET;
    server_addr.sin_addr.s_addr = INADDR_ANY;
    server_addr.sin_port = htons(8080);

    // 3. 绑定
    bind(server_fd, (struct sockaddr*)&server_addr, sizeof(server_addr));

    // 4. 监听
    listen(server_fd, 5);
    printf("Server is listening on port 8080...\n");

    // 5. 接受连接
    client_fd = accept(server_fd, (struct sockaddr*)&client_addr, &addr_len);
    printf("Client connected!\n");

    // 6. 接收数据
    read(client_fd, buffer, sizeof(buffer));
    printf("Client says: %s\n", buffer);

    // 7. 发送数据
    char* reply = "Hello from server!";
    write(client_fd, reply, strlen(reply));

    // 8. 关闭
    close(client_fd);
    close(server_fd);

    return 0;
}
```

---

## 16.3 Client 端代码

```c
// client.c
#include <stdio.h>
#include <string.h>
#include <unistd.h>
#include <arpa/inet.h>

int main() {
    int sock;
    struct sockaddr_in server_addr;

    char* message = "Hello from client!";
    char buffer[1024] = {0};

    // 1. 创建 socket
    sock = socket(AF_INET, SOCK_STREAM, 0);

    // 2. 配置服务器地址
    server_addr.sin_family = AF_INET;
    server_addr.sin_port = htons(8080);

    inet_pton(AF_INET, "127.0.0.1", &server_addr.sin_addr);

    // 3. 连接服务器
    connect(sock, (struct sockaddr*)&server_addr, sizeof(server_addr));

    // 4. 发送数据
    write(sock, message, strlen(message));

    // 5. 接收数据
    read(sock, buffer, sizeof(buffer));
    printf("Server says: %s\n", buffer);

    // 6. 关闭
    close(sock);

    return 0;
}
```

---

## 16.4 关键结构体：`sockaddr_in`

```c
struct sockaddr_in server_addr;
```

用于 IPv4 地址。

常见字段：

```c
server_addr.sin_family = AF_INET;
server_addr.sin_port = htons(8080);
server_addr.sin_addr.s_addr = INADDR_ANY;
```

解释：

| 字段 | 含义 |
|---|---|
| `AF_INET` | IPv4 |
| `SOCK_STREAM` | TCP |
| `htons(8080)` | 把端口转换成网络字节序 |
| `INADDR_ANY` | 接收所有本机 IP 的连接 |
| `127.0.0.1` | 本机 localhost |

---

## 16.5 为什么要强转成 `struct sockaddr*`

```c
bind(server_fd, (struct sockaddr*)&server_addr, sizeof(server_addr));
```

`server_addr` 是：

```c
struct sockaddr_in
```

但是 `bind` 需要更通用的：

```c
struct sockaddr*
```

所以要强制转换。

---

# 17. 常见易错点总结

## 17.1 字符串数组大小要包含 `'\0'`

```c
char str[3] = "abc";  // 错误，空间不够
char str[4] = "abc";  // 正确
```

---

## 17.2 字符数组不能直接赋值

```c
char name[20];

name = "Tom";          // 错误
strcpy(name, "Tom");   // 正确
```

---

## 17.3 `char* s = "abc"` 不要修改内容

```c
char* s = "abc";
s[0] = 'A';  // 不安全，可能崩溃
```

更安全：

```c
const char* s = "abc";
```

如果要修改：

```c
char s[] = "abc";
s[0] = 'A';
```

---

## 17.4 `scanf` 修改变量要传地址

```c
int x;
scanf("%d", &x);
```

---

## 17.5 `malloc` 后要 `free`

```c
int* p = malloc(sizeof(int));

if (p != NULL) {
    *p = 10;
    free(p);
}
```

---

## 17.6 不要返回局部变量地址

```c
int* f() {
    int a = 10;
    return &a;  // 错误
}
```

---

## 17.7 `pthread_cond_wait` 要放在 `while` 中

```c
while (data_ready == 0) {
    pthread_cond_wait(&cond, &mutex);
}
```

---

## 17.8 `pthread_cond_wait` 会自动释放并重新获取 mutex

它不是单纯睡觉，而是：

1. 释放 mutex
2. 睡眠等待 signal
3. 被唤醒后重新拿 mutex
4. 再检查条件

---

## 17.9 `pthread_join` 很重要

```c
pthread_join(thread1, NULL);
```

没有它，主线程可能提前结束，子线程还没运行完程序就退出。

---

## 17.10 Socket 运行顺序

先运行 server：

```bash
gcc server.c -o server
./server
```

再运行 client：

```bash
gcc client.c -o client
./client
```

如果反过来，client 可能连接失败，因为 server 还没开始监听。

---

# 附录：编译常用命令

## 普通 C 文件

```bash
gcc main.c -o main
./main
```

## pthread 程序

```bash
gcc main.c -o main -pthread
./main
```

## server / client

```bash
gcc server.c -o server
gcc client.c -o client
```

运行：

```bash
./server
./client
```

---

# 复习路线建议

建议按下面顺序复习：

1. 基本语法：变量、`printf`、运算符、控制语句
2. 数组和字符串
3. 指针：地址、解引用、数组退化
4. 结构体与动态内存
5. 链表
6. 文件操作
7. pthread：线程、mutex、condition variable
8. read-write lock
9. socket client/server

如果考试偏理论，重点放在：

- 指针与数组关系
- `malloc/free`
- 结构体指针 `->`
- condition variable 为什么要配 mutex
- `pthread_cond_wait` 的三步
- read-write lock 的读写规则
- socket server/client 的调用顺序
