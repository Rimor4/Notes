### datalab
1. 用掩码mask来获取特定位数：
```c
int allOddBits(int x) {
    // 不能用bigInt 101010...1010但是可以构造
    int mask = 0xAA + (0xAA << 8);
    mask = mask + (mask << 16);
    return !((mask & x) ^ mask);
}
```
2. 用 >> 31 & 1来获取**符号位**
3. 用 & 来表示两个**必要条件**同时成立才行； 用 | 来表示两个**充分条件**有一个成立即可
(& 为且， | 为或)
4. （int）float时，可能发生的溢出行为（overflow / underflow）
	上溢可能是未定义的行为，一般用`return 0x80000000;`来返回这个值作为警告

### bomblab
1. 先看**大局**，看每个关键**寄存器**里面存的是什么， 汇编函数是要干什么，不要上来就一步一步调试
2. 每次*函数调用*时注意看<font color="#4f81bd">传入的参数</font>（寄存器中或栈中）
3. **链接的库**是怎么运行的不用看根据名称推断功能即可（sscanf）
4. 遇到“奇怪”的数字赋给寄存器等， 用`x/s` 命令**打印出该地址处的值**。

### attacklab
1. 想要在利用栈来ret到另一个地址，别忘用`pushq ...(跳转的地址)` ，这样可以直接在push指令后面加上ret
2. 用`x/100gx ...`查看数据时![[Pasted image 20240329123332.png]]
     每一个gw中**低地址在右**（0x5561dc13： 0x3539......）
3. 不插入代码的攻击方法（有栈随机化保护）：可以利用程序代码中的原有编码，“断章取义”地从程序中**任何位置** (只能用farm里的】（自由组合连续的机器码字节！）来构成攻击代码
4. 接上3. 使用这种攻击方法往往利用栈相对地址不变，需要： **获得rsp的值，并将其加上偏移量得到string address 然后传给rdi**， 本题由于`加上偏移量`这一操作只有`lea (%rdi, %rsi, 1),%rax`这一可用命令，故涉及到了将string的存放位置 -0x10(%rsp) 用<font color="#ffff00">不同寄存器多次传递</font>
5. <font color="#ff0000"> 注释要写全</font>。。。。。。。。。。

### archlab
1. **链表**节点跳转指令 `mrmovq 8(%rsi),%rsi`  把指针域（地址）传送给寄存器即可
2. **戳气泡**技术：
当
- `andq` Memory 访存阶段
- `rmovq` Execute 执行阶段
- `jle` Decode 解码阶段

此时，`jle` 可以立即使用正确的 M_Cnd，避免控制冒险，即在 Decode 解码阶段就可以知道是否需要跳转，避免了预测失败时的 2 个气泡周期的惩罚。
3. **循环展开**循环条件**边界**：```
```asm
	iaddq $-8, %rdx     # 开头先减8
	jl handle_remainder
loop:
	...
	...
	...
	iaddq $-8,%rdx      # 循环体中再减8
	jge loop
```
4. **平衡二叉树**判断余数，将%rdx(len)值-8~-1**映射**为0~7次余数操作
![[8ce30297a2e6e135eb9e9c27bc37d3c.jpg]]

### cachelab
1. c语言分配cache模拟器机构, 一个层级一个层级地分配
```c
void init_cache() {
    cache _cache_ = (cache)malloc(sizeof(cache_set) * S);   // 分配 S 个 高速缓存组
    for (int i = 0; i < S; i++) {
        _cache_[i] = (cache_set)malloc(sizeof(cache_line * E));  // 一行一行分配
        for (int j = 0; j < E; j++) {
            _cache_[i][j].valid_bits = 0;
            _cache_[i][j].tag = -1;
            _cache_[i][j].stamp = -1;
        }
    }
}
```
2. cache 的 组数 S 和地址的组索引 s 对应, 不会超出范围。
3. 命令行参数处理方法 `while ((opt = getopt(argc, argv, "hvs:E:b:t:")) != -1)` 
4. PartA: 驱逐eviction前也要算miss
5. PartB：优化黄金法则：**空间换时间**
6. <span style="background:#b1ffff">64x64</span>
![[Pasted image 20240610214952.png]]

### malloclab
1. `bp`：C语言**修改指针本身的值**需要向函数传递二级指针。
2. 在if条件中使用**赋值语句**时注意**加括号**
`if (nsize = oldsize + GET_SIZE(HDRP(NEXT_BLKP(ptr))) > size) { /* ... */ }`
3. 下面这种寻找空闲链表索引的方法有一个巨大的bug，没有处理**size大于2^LISTMAX**的情况
```
while (listnumber < LISTMAX) {
    /* 寻找对应链 */
    if (((searchsize <= 1) && (segregated_free_lists[listnumber] != NULL))) {
        ptr = segregated_free_lists[listnumber];
        /* 在该链寻找大小合适的free块 */
        while ((ptr != NULL) && ((size > GET_SIZE(HDRP(ptr))))) {
            ptr = PRED(ptr);
        }
        /* 找到对应的free块 */
        if (ptr != NULL)
            break;
    }

    searchsize >>= 1;
    listnumber++;
}
```


### proxylab
1. 定义<span style="background:#b1ffff">结构体中的字符串</span>成员记得**分配空间**（要么用数组形式不用char*，要么之后要malloc动态分配内存），否则用<font color="#ff0000">字符串指针赋值</font>的方法来使用该结构体成员会出现不可预估的危险。
```c
typedef struct Key {
    char* host;
    char* port;
    char* path;
} Key;
// 错误
// Construct the Key of request
key.host = hostname;
key.path = filename;
key.port = port;

// 正确

```