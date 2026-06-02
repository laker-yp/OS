* `parallelism `
并行性

* malloc(strlen(newTitle) + 1) 加1是因为length的计算不带/0

* `static variable` 在 data segment。

* `malloc`(n * sizeof(T))

* `calloc`(n,  sizeof(T))  分配并清零。

* `realloc` 最好用临时指针

* p = realloc(p, newSize); 把原来 p 指向的内存改变，如果失败，返回 NULL，原来的地址可能丢了，造成 memory leak。
```c
int *tmp = realloc(p, newSize);
if (tmp == NULL) {
    // p is still valid
} else {
    p = tmp;
}
```

* `a->b` a 必须是“指向结构体/联合体的指针”，b 必须是这个结构体/联合体里面的成员名。

* `a` 通常表示 `&a[0]`

* 对于 `2D-array`, the `array name` represents the address of the `first row`, not the address of a single int element

* `fopen` 返回一个文件指针，如果文件是空的，*f == NULL

* `EOF` 是 int，通常是 -1

* `coherence 问题`如果 Core1 和 Core2 都缓存了同一个变量 counter，core1 改了，core2 的 cache 中的counter可能还是旧值。
* `pthread_join` ensures that one thread waits for another thread to finish
* 不同`thread`除了stack是自己独享之外，heap什么的都是和别的t共享的
* `arr` 在很多表达式中会退化为指向第一个元素的指针：arr == &arr[0], 所以`p++`就可以遍历
* `allocate`分配（分配地址）
* 如果 name 是字符数组`char name[20];`，不能这样赋值：`s1.name = "Tom"; `，能这样赋值的只有Int：`s1.age = 20;`
* 如果 name 是字符指针` char *name`   就可以这样赋值  `s1.name = "Tom";`   // ✅ 可以
* 如果想在函数里修改结构体，通常传结构体指针。比如`changeAge(&s1);`传入s1的地址就相当于指针了
* 

























