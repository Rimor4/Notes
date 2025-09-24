### 实验思路
##### Part A
核心：<span style="background:#b1ffff">LRU</span>: 发生eviction时，我们要选择**最近最少访问的那一行**
步骤：
1. 用`getopt()`读取命令行参数
2. 配置 s(组索引位数),E(每组行数),b(块偏移位数),t(文件)
	地址的标记位   t = address >> (s + b)
	地址的组索引位 s = (address >> b) & (-1u >> (64 - s));
1. 构建cache数据结构:
	1> 结构体
	struct cache_line {
		(1) stamp (LRU conter) // 时刻。也可作为 valid_bits
		(2) Tag
	};
	2> 结构体数组
	struct cache_line cache[S][E];
4. 循环:
	(1) 用`fscanf()`读取文件每一行, 解析为 operation, address, size
	(2) 访问并更新cache, hits, misses, evictions
##### Part B
核心：
- 使用**分块技术**进行优化
- 避免**对角线**上元素原地转置引发的冲突未命中问题

1. 32x32: 用8 * 8 大小分块，再用`diag`局部变量存储解决对角线冲突不命中，在每行遍历完后再将`diag`赋值给B
2. 64x64: 发现trace中**每4行组索引正好重复**，采用8 * 8分块但是**4 * 4处理**。
	这个部分是我觉得本实验最难的部分，优化要求最高，按照上面的方法不能优化到满分
	最终参考了[知乎@林夕](https://zhuanlan.zhihu.com/p/456858668)的思路即下图：
	![[Pasted image 20240524085107.png]]

3. 61x67: 仍然使用32x32的方法，但是需要处理**余块**

### 实验中得到的一些教训
1. c语言分配cache模拟器机构, 需要一个层级一个层级地分配
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