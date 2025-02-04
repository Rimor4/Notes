### 文件处理函数
1. char **fgets**(char * s, int n, FILE * stream);   <u>类似fread</u>
    - `s`: 是一个指向字符数组的指针，用于*存储*读取的字符串。该数组应具有足够的空间来容纳读取的字符串，通常是 `size` 大小。
	- `size`: 是一个整数，指定了字符数组 `s` 的大小（即最大能够存储的字符数，包括 null 字符 `\0`）。
	- `stream`: 是一个指向 `FILE` 结构的指针，表示文件流，从中*读取*字符串。
	返回一个<u>指向存储在缓冲区中的字符串的指针</u>
2. int **fputs**(const char * str, FILE * stream);   <u>类似fwrite</u>
  - `str`：要写入文件的字符串的地址。
  - `stream`：文件指针，指示要写入的文件。

 `fputs` 函数将字符串 `str` 中的字符写入到由 `stream` 指定的文件中，直到遇到字符串中的 null 字符 `\0`。如果成功，它<u>返回非负值</u>；如果发生错误，它<u>返回 `EOF`（通常为 -1）</u>。

	
3. int **fgetc**(char * str)
	读取一个字符
	返回的是读取到的<u>字符的 ASCII 值</u>作为无符号字符值

4. int **fputc**(int character, FILE * stream);
	`fputc` 的返回值是写入成功的字符，如果发生错误，则返回 `EOF`（宏定义为 -1）


5. int **fprintf**（FILE * stream, const char  format [, argument ]…）：
	只比printf函数多了 * file 文件参数
	// 同**sprintf**函数，多了 buffer 字符串参数
    输出格式化字符串到流或者是将格式化后的字符串写到输出流（文件）向文件输入
	返回值是<u>成功写入的字符数</u>，如果出现错误，则返回一个负数
	 // 通常用于按照特定格式输入字符串、数字
6. int **fscanf**(FILE  stream, char  format,[argument...])：
	fscanf(infile, "%s", name)
    输出文件中的内容到某个变量中 *从文件输出*
    返回值是<u>成功读取的数据项的数量</u>。如果出现错误或到达文件末尾，返回值可能小于提供的参数数量
    // 通常用于按照特定格式读取字符串、数字

7. int **feof**(FILE * stream);
	该函数在文件结束标志被设置时<u>返回非零值，否则返回0</u>
```c
	#include <stdio.h>

	int main() {
    FILE *file = fopen("example.txt", "r");
    
    if (file == NULL) {
        perror("打开文件时发生错误");
        return 1;
    }

    // 读取字符直到文件结束
    int ch;
    while (!feof(file)) {
        ch = fgetc(file);
        if (ch != EOF) {
            putchar(ch);
        }
    }

    // 关闭文件
    fclose(file);

    return 0;
 }
```

8. int **fread**(char  buf, int size, int count, FILE  rfp);
 - `char *buf`: 是一个字符数组的首元素指针，用于存放读入数据的开始地址。`fread` 会从文件中读取数据，并将数据存储到 `buf` 指向的内存位置。
 - `int size`: 指定每个数据块的字节数，即每次读取的数据块大小。
 - `int count`: 指定要读取的数据块的个数，即希望读取多少个 `size` 字节的数据块。
 - `FILE *rfp`: 是一个文件指针，指向要读取的文件。文件指针用于标识文件位置和状态。
 `fread` 函数的返回值是实际<u>成功读取的数据块的个数</u>。如果返回值小于 `count`，可能表示到达文件末尾或发生了错误
 // 通常用于读取二进制文件中的结构体、数组等数据块
9. int **fwrite**(char  buf, int size, intcount, FILE  wfp);`
 - `char *buf`: 是一个字符数组的首元素指针，用于指定要输出数据的开始地址。`fwrite` 将从 `buf` 指向的内存位置写入数据到文件中。
 - `int size`: 指定每个数据块的字节数，即每次写入的数据块大小。
 - `int count`: 指定要写入的数据块的个数，即希望写入多少个 `size` 字节的数据块。
 - `FILE *wfp`: 是一个文件指针，指向要写入的文件。文件指针用于标识文件位置和状态。
 - // 通常用于写入结构体、数组等

10. void **rewind**(FILE * fp)
	
	函数rewind()的作用是使文件当前位置<u>重新回到文件之首</u>。该函数<u>没有返回值</u> 
11. int **fseek**(FILE * fp, long offset, int whence);
 - `stream`：文件指针，指向要进行定位的文件。
 - `offset`：偏移量，表示从 `whence` 参数指定位置的偏移量。 
 - `whence`：相对位置，可以取以下值：
    - `SEEK_SET`：从文件开头开始。
    - `SEEK_CUR`：从当前文件指针位置开始。
    - `SEEK_END`：从文件末尾开始。

 `fseek` 函数将文件指针 `stream` 移动到由 `offset` 和 `whence` 指定的新位置。如果成 功，函数<u>返回0</u>；如果发生错误，返回<u>非零值</u>。
 fseek(fp, 40L, SEEK_SET);当前位置定于离文件头40个字节处 
 fseek(fp, 20L, SEEK_CUR);文件当前位置定于离当前位置20个字节处 
 fseek(fp, -30L, SEEK_END);将文件的当前位置定于文件尾后退30个字节处     
12. long **ftell**(FILE * fp)
  返回的是 `long` 类型，表示文件指针当前位置<u>相对于文件开头的位移</u>
	利用函数 ftell() 也能方便地知道一个文件的长：  
	fseek(fp, 0L, SEEK_END);           当前位置移到文件的末尾  
	len = ftell(fp)；                         获得当前位置相对于文件首的位移


### 字符串函数
1. char * **strchr**(const char * str, int c) 
    用于在字符串中查找指定字符的第一个匹配。                                                函数返回一个指向*第一次出现*<u>指定字符 `c` 的指针</u>。如果在字符串中未找到该字符，则返回 `NULL`。

2. size_t **strcspn**(const char * str1, const char * str2);
 - `str1`: 要被搜索的字符串。
 - `str2`: 包含要搜索的字符集合的字符串。

 `strcspn` 函数返回的是<u> `str1` 字符串的前缀中不包含 `str2` 字符集合中的任何字符的长度</u>。
 ```c
 #include <stdio.h>
 #include <string.h>

 int main() 
 {
    const char *str1 = "Hello, World!";
    const char *str2 = "aeiou";

    size_t length = strcspn(str1, str2);

    printf("Length of prefix without vowels: %zu\n", length);

    return 0;
 }
```

3. char * **strstr**(const char * haystack, const char * needle);
 在一个字符串中查找**指定子串**的第一次出现位置
 返回一个指向第一次出现子串的指针

### 内存管理函数
1. void * **realloc**(void * ptr, size_t size);  返回一个新的内存块的指针
 ```c
	#include <stdlib.h>
	int main() {
	    int *ptr;
	
	    // 分配一个初始大小的内存块
	    ptr = (int *)malloc(10 * sizeof(int));
	
	    // 使用内存块
	
	    // 重新分配内存块的大小
	    ptr = (int *)realloc(ptr, 20 * sizeof(int));
	
	    // 使用重新分配后的内存块
	
	    // 释放内存块
	    free(ptr);
	
	    return 0;
	}
```

2. void * **memmove**(void * <u>dest</u>, const void * <u>src</u>, size_t <u>n</u>);
 - `dest`：目标内存的指针，即数据将要被复制到的位置。
 - `src`：源内存的指针，即要被复制的数据的起始位置。
 - `n`：要复制的字节数。
	`memmove` 函数将从源内存 `src` 复制 `n` 个字节到目标内存 `dest`。与 `memcpy` 不同的是，`memmove` 能够处理 `src` 和 `dest` 指向的内存区域有重叠的情况，保证在移动数据时不会产生错误。
	```c
	memmove(position + newLen, position + oldLen, strlen(position + oldLen) + 1);
	// 将原子串从position + oldLen开始的后面长strlen(position + oldLen) + 1)的部分复制到新子串position + newLen位置后面
	```
