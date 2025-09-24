### phase1  
答：`The future will be better tomorrow.`

由`mov $0x4026f0,%esi`指令可知答案字符串存在0x4026f0位置

### phase2
答：`1 2 4 8 16 32`

1. `(%rsp) == 0x1` 
2. `(%rsp) * 2 == (%rsp + 4)`    
3. loop:前一个数的2倍等于后一个数

### phase3
答：`5 b 84`

由`mov $0x40273e,%esi`可知：输入的格式字符串为`"%d %c %d"` 
1. 第一个数控制switch跳转位置
2. 第二个字符等于跳转case对应的字符值
3. 第三个数字等于跳转case中需要比较的值

### phase4
答：`1 11 DrEvil`

翻译成c代码
```c
  int eax, ebx;
  int func4 (int edi, int esi, int edx)
  {
      // temporaries
      int tmp = ebx;

      eax = edx;
      eax = eax - esi;
      ebx = eax;
      ebx = (unsigned int)ebx >> 31;
      eax += ebx;
      eax >>= 1;
      ebx = eax + esi;
      if (ebx <= edi) goto L1;
      edx = ebx - 1;
      eax = func4 (edi, esi, edx);  # (1) edi <= 6 最后一次返回值为 4
      eax += ebx; # ebx = 7, eax = 4
      goto L2;    
  L1:
      eax = ebx;
      if (ebx >= edi) goto L2;    # (2) edi >= 8 
      esi = ebx + 1;  # esi = 8
      eax = func4 (edi, esi, edx);  # 最后一次返回值为4
      eax += ebx;
  L2:
      ebx = tmp;
      return eax;
  }
```
从**递归调用栈**最上层逐层分析
调用栈: 11 (4 + 7) <- 4 (1 + 3) <- 1

### phase5
答：`jdoefg`

- 由`cmp $0x6,%eax`可知要求输入6个数字
- `mov %dl,(%rsp,%rax,1)`6次分别将数组元素拷贝到从rsp开始的连续6个位置
- `mov $0x402747,%esi`可知需要比较的字符串为`oilers` 即 ` 6F(-- 0xa) | 69(-- 0x4) | 6C(-- 0xf) | 65(-- 0x5) | 72(-- 0x6) | 73(-- 0x7) [  j | d | o | e | f | g  ]` 
- 用输入每个数字的**最低字节**作为索引，找到数组中对应的字符


### phase6
答：`2 6 4 5 3 1`

代码中所用数据结构很明显为**链表**
1. 输入6个数字
2. 要求每个数字不大于6
3. 要求每个数字不相同
4. 每个数字对应一个旧链表（**内存**中）中节点的映射位置，存放到**栈**中
5. 之后代码将映射后的节点连接为一个**新链表**
6. 新链表每个节点的数据**降序排列**


### secret phase
答：`6`

数据结构为**平衡二叉树**
1. 前往右支（右边大于等于寻找的值）：`rax = 0`
2. 从左支返回，`rax *= 2`
3. 从右支返回，`rax = rax * 2 + 1`
4. 在二叉树中找到输入的值，就返回；如果到达二叉树最深处还没有找到，就返回-1

要使rax最后返回值为0，只需要让输入的值`rcx`等于最左侧任意节点即可，
这样在找到时将rax置为0，再从左支返回若干次，保证最后rax返回0
