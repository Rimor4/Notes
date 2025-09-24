
```c
#include<stdio.h>
void main()
{
	float x, y;
	printf("\t *****个人所得税计算器 *****\n\n");
	printf("请输入您的薪酬总金额(其中免税金额为 3500元:");
	scanf("%f", &x);
	/*低于3500免税*/
	if (x <= 3500)
	{
		printf("您输入的薪酬金额低于 3500元,不需要加个人所得税!\n");
		return;
	}
	x = x - 3500;
	/*x为应税金额*/
	switch ((int)(x / 1500))
	{
	case 0:y = 0.03 * x; break;/*应税金额小于 1500*/
	case 1:
	case 2:
	case 3:y = x * 0.10 - 105; break;/*应税金额在1500~4500*/
	case 4:
	case 5:
	case 6:y = x * 0.20 - 555; break; /*应税金额在4500~9000*/
	default:y = x * 0.25 - 1005;/*应税金额大于 9000*/
		printf("您的应税金额为:%.2f,应交个人所得税为:%.2f \n", x,y);
	}
}
```

## sprintf函数