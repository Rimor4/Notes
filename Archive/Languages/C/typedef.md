#### 定义函数指针别名
```c
#include <stdio.h>

// 定义函数指针别名
typedef void (*FunctionPointer)(int);
// 定义符合函数指针要求的函数
void myFunction(int x) {
    printf("Value passed to myFunction: %d\n", x);
}

int main() {
    // 使用函数指针别名声明一个指针变量，并初始化为指向myFunction的地址
    FunctionPointer ptr = myFunction;

    // 通过函数指针调用函数
    ptr(42);  // 这会调用 myFunction，并输出 "Value passed to myFunction: 42"

    return 0;
}
```

#### 定义数组别名
```c
typedef int IntArray[5];  // 将int[5]定义为IntArray的别名

// 使用别名声明数组
IntArray numbers = {1, 2, 3, 4, 5};
```