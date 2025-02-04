```c
// I
/*
 * bitXor - x^y using only ~ and &
 *   Example: bitXor(4, 5) = 1
 *   Legal ops: ~ &
 *   Max ops: 14
 *   Rating: 1
 */
int bitXor(int x, int y) {
    // 将异或的否的两种情况合并，再取反
    return ~(x & y) & ~(~x & ~y);
}
/*
 * tmin - return minimum two's complement integer
 *   Legal ops: ! ~ & ^ | + << >>
 *   Max ops: 4
 *   Rating: 1
 */
int tmin(void) {
    return 1 << 31;
}

// II
/*
 * isTmax - returns 1 if x is the maximum, two's complement number,
 *     and 0 otherwise
 *   Legal ops: ! ~ & ^ | +
 *   Max ops: 10
 *   Rating: 1
 */
int isTmax(int x) {
    return !((x + 1) + (x + 1) + !(x + 1));
}
/*
 * allOddBits - return 1 if all odd-numbered bits in word set to 1
 *   where bits are numbered from 0 (least significant) to 31 (most significant)
 *   Examples allOddBits(0xFFFFFFFD) = 0, allOddBits(0xAAAAAAAA) = 1
 *   Legal ops: ! ~ & ^ | + << >>
 *   Max ops: 12
 *   Rating: 2
 */
int allOddBits(int x) {
    // 不能用bigInt 101010...1010但是可以构造
    int mask = 0xAA + (0xAA << 8);
    mask = mask + (mask << 16);
    return !((mask & x) ^ mask);
}
/*
 * negate - return -x
 *   Example: negate(1) = -1.
 *   Legal ops: ! ~ & ^ | + << >>
 *   Max ops: 5
 *   Rating: 2
 */
int negate(int x) {
    return ~x + 1;
}

// III
/*
 * isAsciiDigit - return 1 if 0x30 <= x <= 0x39 (ASCII codes for characters '0' to '9')
 *   Example: isAsciiDigit(0x35) = 1.
 *            isAsciiDigit(0x3a) = 0.
 *            isAsciiDigit(0x05) = 0.
 *   Legal ops: ! ~ & ^ | + << >>
 *   Max ops: 15
 *   Rating: 3
 */
int isAsciiDigit(int x) {
    // x + 6 不进位(检查0x2i)， 且此时右移4位后 末尾为3
    return !((((x + 0x6) >> 4) ^ 0x3) + (x >> 4 ^ 0x3));
}
/*
 * conditional - same as x ? y : z
 *   Example: conditional(2,4,5) = 4
 *   Legal ops: ! ~ & ^ | + << >>
 *   Max ops: 16
 *   Rating: 3
 */
int conditional(int x, int y, int z) {
    // x != 0:  final x = 1111....1111
    // x == 0:  mask = 0000....0000
    x = !!x;
    x = ~x + 1;
    return (x & y) | (~x & z);
}
/*
 * isLessOrEqual - if x <= y  then return 1, else return 0
 *   Example: isLessOrEqual(4,5) = 1.
 *   Legal ops: ! ~ & ^ | + << >>
 *   Max ops: 24
 *   Rating: 3
 */
int isLessOrEqual(int x, int y) {
    // 先判断x,y是否异号, 异号时可能会溢出(在计算x - y时)
    int sign_x = (x >> 31) & 1;     // 负数时为1
    int sign_y = (y >> 31) & 1;

    // case1: 异号时
    int isOpp = sign_x & !sign_y;   // x 为负， y 为正
    // case2: 同号时
    int notOpp = (!(sign_x ^ sign_y)) & ((x + ~y) >> 31 & 1); // 计算 x <= y 即 x - y < 1

    return isOpp | notOpp;
}

// IV
/*
 * logicalNeg - implement the ! operator, using all of
 *              the legal operators except !
 *   Examples: logicalNeg(3) = 0, logicalNeg(0) = 1
 *   Legal ops: ~ & ^ | + << >>
 *   Max ops: 12
 *   Rating: 4
 */
int logicalNeg(int x) {
    // int isNeg = (~x >> 31) & 1;    // 负数取反后符号位为0
    // int isPos = (~(~x + 1) >> 31) & 1;    // 正数取逆元再取反后符号位为0
    // return isNeg & isPos

    // optimized: 
    // 考虑 x | (~x + 1) 与自己的相反数作位或
    // 利用 1.当x为除0以外(甚至是Tmin)的数时,上述结果的符号位都为1
    //     2. 符号位"向右扩展"后 1111....1111 + 1 = 0; 0000....0000 + 1 = 1
    return ((x | (~x + 1)) >> 31) + 1;
}
/* howManyBits - return the minimum number of bits required to represent x in
 *             two's complement
 *  Examples: howManyBits(12) = 5
 *            howManyBits(298) = 10
 *            howManyBits(-5) = 4
 *            howManyBits(0)  = 1
 *            howManyBits(-1) = 1
 *            howManyBits(0x80000000) = 32
 *  Legal ops: ! ~ & ^ | + << >>
 *  Max ops: 90
 *  Rating: 4
 */
 // 将整数用2^4 + 2^3 + 2^2...拆开的思想（用2的幂减）
int howManyBits(int x) {
    int b16, b8, b4, b2, b1, b0;
    // 把负数转换为正数，只计算一次offset
    x = (x >> 31) ^ x;
    // 先看x高16位有没有1
    b16 = !!(x >> 16) << 4;    // << 4，得到让x是否发生实际移位的结果b16
    x = x >> b16;
    // 再看x剩余位数（后16位）的高8位
    b8 = !!(x >> 8) << 3;
    x = x >> b8;
    // 同理
    b4 = !!(x >> 4) << 2;
    x = x >> b4;
    b2 = !!(x >> 2) << 1;
    x = x >> b2;
    b1 = !!(x >> 1);
    x = x >> b1;
    b0 = x;
    return b16 + b8 + b4 + b2 + b1 + b0 + 1;    // + 1 (再加上符号位)
}

// V
/*
 * floatScale2 - Return bit-level equivalent of expression 2*f for
 *   floating point argument f.
 *   Both the argument and result are passed as unsigned int's, but
 *   they are to be interpreted as the bit-level representation of
 *   single-precision floating point values.
 *   When argument is NaN, return argument
 *   Legal ops: Any integer/unsigned operations incl. ||, &&. also if, while
 *   Max ops: 30
 *   Rating: 4
 */
unsigned floatScale2(unsigned uf) {
    // int exp_Sign = (uf << 1) >> 24;    这里写到一步中会进行无符号整数的逻辑右移
    int s = uf >> 31;
    int exp = uf >> 23 & 0xFF;
    int frac = uf & 0x7fffff;

    // uf is NaN
    if (exp == 0xFF) {
        return uf;
    }
    // denormalized
    else if (!exp) {
        frac = frac << 1;
    }
    // normalized
    else {
        exp = exp + 1;
    }
    return s << 31 | exp << 23 | frac;
}
/*
 * floatFloat2Int - Return bit-level equivalent of expression (int) f
 *   for floating point argument f.
 *   Argument is passed as unsigned int, but
 *   it is to be interpreted as the bit-level representation of a
 *   single-precision floating point value.
 *   Anything out of range (including NaN and infinity) should return
 *   0x80000000u.
 *   Legal ops: Any integer/unsigned operations incl. ||, &&. also if, while
 *   Max ops: 30
 *   Rating: 4
 */
int floatFloat2Int(unsigned uf) {
    int sig = uf >> 31;
    int exp = uf >> 23 & 0xFF;
    int bias = 0x7F;
    int frac = uf & 0x7fffff;

    int result;
    unsigned E, M;

    // overflow
    if (exp >= bias + 31) {
        // 一律返回最大值,无论溢出为多少
        return 0x80000000u;
    }
    // the float number is less than 1
    else if (exp < bias) {
        return 0;
    }
    // normal
    else {
        E = exp - bias;
        M = (1 << 23) | frac;

        // 浮点数与所对应整数对齐
        if (E > 23) {
            result = M << (E - 23);
        }
        else {
            // 向零舍入
            result = M >> (23 - E);
        }

        if (sig) {
            result = ~result + 1;
        }
        return result;
    }
}
/*
 * float-i2f.c
 */
#include <stdio.h>
#include <assert.h>
#include <limits.h>

typedef unsigned float_bits;

// Get bit length for integer i.
int bits_length(int i) {
    if ((i & INT_MIN) != 0) {
        return 32;
    }

    int length = 0;
    unsigned u = (unsigned)i;
    while (u >= (1 << length)) {
        length++;
    }
    return length;
}

// Generage mask for length len.
unsigned generate_mask(int len) {
    return (unsigned) -1 >> (32 - len);
}

float_bits float_i2f(int i) {
    unsigned sig, exp, frac, exp_frac;
    unsigned bias = 0x7F;

    // i = 0
    if (i == 0) {
        sig = 0;
        exp = 0;
        frac = 0;
        return sig << 31 | exp << 23 | frac;
    }

    // i = INT_MIN
    if (i == INT_MIN) {
        sig = 1;
        exp = 31 + bias;
        frac = 0;
        return sig << 31 | exp << 23 | frac;
    }

    sig = 0;
    // i is negative
    if (i < 0) {
        sig = -1;
        i = -i;
    }

    int length = bits_length(i);
    int flength = length - 1;
    unsigned mask = generate_mask(flength);
    unsigned f = i & mask;
    exp = bias + flength;   // the key of i2f:  bits of i ==> exp of f

    if (flength <= 23) {
        // 精确
        frac = f << (23 - flength);     // 移到IEEE frac位置
        exp_frac = exp << 23 | frac;    // 获得exp + frac位模式
    } else {
        // "overflow" 需要舍入
        int offset = flength - 23;
        frac = f >> offset;             // 减去多出来的 2的幂, 剩下小数
        exp_frac = exp << 23 | frac;
        int round_mid = 1 << (offset - 1);          // e.g. ...Y1000
        int round_part = f & generate_mask(offset); // e.g. ...Y2F59(round_part > round_mid)     / ...Y0042(round_part < round_mid)

        // round to exen
        if (round_part < round_mid) {

        } else if (round_part > round_mid) {
            exp_frac += 1;            // 向上舍入
        } else {
            if  ((frac & 0x1) == 1) {
                exp_frac += 1;        // 如果在可能的舍入结果中间, 保证向偶数舍入
            }
        }
    }
    return sig << 31 | exp_frac;
}

/*
 * floatPower2 - Return bit-level equivalent of the expression 2.0^x
 *   (2.0 raised to the power x) for any 32-bit integer x.
 *
 *   The unsigned value that is returned should have the identical bit
 *   representation as the single-precision floating-point number 2.0^x.
 *   If the result is too small to be represented as a denorm, return
 *   0. If too large, return +INF.
 *
 *   Legal ops: Any integer/unsigned operations incl. ||, &&. Also if, while
 *   Max ops: 30
 *   Rating: 4
 */
unsigned floatPower2(int x) {
    int exp, frac, fPos;
    int bias = 0x7F;

    // normal
    // x = e - bias
    if (x > -bias) {
        // Overflow: e > 0xFE, namely x > too large
        if (x > bias) {
            return 0x7F800000;
        }
        exp = (bias + x) << 23;
        return exp;
    }
    // denorm
    // x = 1 - bias
    else {
        // Underflow
        if (x < (-bias - 22)) {
            return 0;
        }
        fPos = bias - x + 1;
        frac = 1 << (23 - fPos);
        return frac;
    }
}
```