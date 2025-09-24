第一个参数是要存储结果的字符数组;  
第二个参数是格式字符串
```c
#include <stdio.h>
int main() 
{
	//可以控制精度
    char str[20];
    double f=14.309948;
    sprintf(str,"%6.2f",f);
	char str[20];
	//可以将多个数值数据连接起来
	int a=20984,b=48090;
	sprintf(str,"%3d%6d",a,b);

	char str[20];
	char s1[5]={'A','B','C'};
	char s2[5]={'T','Y','X'};
	sprintf(str,"%.*s%.*s",2,s1,3,s2);
	sprintf(str, "%*.*f", 10, 2, 3.1415926);


    printf("%s",str);
    return 0;
}
```
