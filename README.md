malloc(strlen(newTitle) + 1) 加1是因为length的计算不带/0

static variable 在 data segment。

malloc(n * sizeof(T))

calloc(n,  sizeof(T))  分配并清零。

realloc 最好用临时指针

p = realloc(p, newSize); 把原来 p 指向的内存改变

如果失败，返回 NULL，原来的地址可能丢了，造成 memory leak。
```c
int *tmp = realloc(p, newSize);
if (tmp == NULL) {
    // p is still valid
} else {
    p = tmp;
}
```

`a->b` a 必须是“指向结构体/联合体的指针”，b 必须是这个结构体/联合体里面的成员名。
