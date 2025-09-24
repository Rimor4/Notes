#flashcards/C语言 
atoi函数`int atoi(const char *str);`
?
`atoi` 函数会从字符串 `str` 中读取字符，直到遇到第一个非数字字符或者字符串结束符（null字符 `\0`），然后将之前读取的数字字符解释为一个整数。如果字符串中包含无效字符，或者无法转换成整数，`atoi` 将返回0。