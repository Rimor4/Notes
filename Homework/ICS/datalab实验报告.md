### Puzzle实现思路
1. bitXor：将异或的**否**的两种情况&合并，再取反
2. tmin：1左移31位即为补码最小数
3. isTmax：
	1. Tmax + 1 = Tmin（x + 1）
	2. Tmin + Tmin = 0 (`(x + 1) + (x + 1) = 0`)
	3. 满足上一条条的只有-1和Tmax，再加`!(x+1)`, <font color="#ffff00">排除 -1</font>的情况
4. allOddBits：构造10101010...1010再用其作为掩码
5. negate：公式 `~x + 1 = -x` 
6. isAsciiDigit：
	1. 0x30~0x39的数满足`x + 6`不进位（<font color="#ffff00">排除0x2i</font>的情况）
	2. 且右移四位后末尾为3（用^0x3检测）
7. conditional：
	1. x != 0时，让x最终为1111....1111，和y位与
	2. x == 0时，让x最终为0000....0000，取反再和z位与
8. isLessOrEqual:
	1. 先判断x，y是否异号，异号时计算`x - y`可能**溢出**
	2. 
		- 异号：看谁为正数（谁的**符号位为0**）
		- 同号：`x <= y`  <==> `x - y < 1` <==> `x + ~y < 1`
	3. 最后用 `|` 合并两种情况
9. logicalNeg：
	考虑`x | (~x + 1)`自己和自己的逆元作位或
	1. x除等于0 以外，上式符号位为1
	2. 作符号扩展再加1
10. howManyBits：
	1. 将整数用**2^4 + 2^3 + 2^2...拆开**的思想（用2的幂减）
	2. 先将正负数都统一为正数 `x = (x >> 31) ^ x` 正负数结果一样
	3. 分别用b16,b8,b4,b2,b1.b0六个临时变量记录x高16位、一位后高8位、移位后高4位....是否有为1的位
	4. 直到找到最高的非0位
	5. 最后再加上符号位
11. floatScale2：
	1. 获取浮点数表示的**s**（符号位）、**exp**（指数部分）、**frac**（小数部分）
	2. 讨论
		1. uf is NaN (exp == 0xFF)：直接返回uf
		2. uf is denormalized (exp == 0)：frac乘2即可
		3. uf is normalize：exp加一即可
12. floatFloat2Int：
	1. 同样先获取三个部分
	2. 讨论
		1. 向上溢出 (exp >= bias + 31) ：返回NaN
		2. 浮点数小于1 (denormalized)：返回0
		3. 浮点数大于1 (normalized)：利用**浮点数和对应整数二进制表示对齐**的特点：
			1. E（exp - bias） > 23时，M 左移 E - 23
			2. E <= 23 时，需要向零舍入，M右移 23 - E
13. floatPower2：
	讨论x：
		1. 如果x > bias, x 过大，上溢
		2. x > -bias： exp即为x + bias后左移23位(**定义**)
		3. x > -bias - 23：用23位的小数部分表示
		4. 否则，**浮点下溢**
			