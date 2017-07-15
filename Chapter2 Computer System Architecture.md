### 计算机知识总框架

![](/assets/计算机知识体系结构.jpg)


### 深入理解计算机实验

#### DATA LAB

> bitCount:    
> bitCount(7) = 3, bitCount(5) = 2

![统计二进制数中1的数量](/assets/bitcount1.png "how many ones of a de quitcimal integer")

```c
/*
 * bitCount - returns count of number of 1's in word
 *   Examples: bitCount(5) = 2, bitCount(7) = 3
 *   Legal ops: ! ~ & ^ | + << >>
 *   Max ops: 40
 *   Rating: 4
 */
int bitCount(int x) { 
  int every_2_bit = 0x55 | ( 0x55 << 8 ) | ( 0x55 << 16 ) | ( 0x55 << 24 );
  int every_4_bit = 0x33 | ( 0x33 << 8 ) | ( 0x33 << 16 ) | ( 0x33 << 24 );
  int every_8_bit = 0x0f | ( 0x0f << 8 ) | ( 0x0f << 16 ) | ( 0x0f << 24 );
  int every_16_bit = ( 0xff ) | ( 0xff << 16 );
  int every_32_bit = ( 0xff ) | ( 0xff << 8 );

  int result = 0;

  result = ( x & every_2_bit ) + ( ( x >> 1) & every_2_bit );
  result = ( result & every_4_bit ) + ( ( result >> 2 ) & every_4_bit );
  result = ( result + ( result >> 4 ) ) & every_8_bit;
  result = ( result + ( result >> 8 ) ) & every_16_bit;
  result = ( result + ( result >> 16 ) ) & every_32_bit;

  return result;
}
```

gcc编译后整个过程进行了39个操作,还有可以优化的空间.优化后的代码如下,这时的操作只有33个了.  

```c
/*
 * bitCount - returns count of number of 1's in word
 *   Examples: bitCount(5) = 2, bitCount(7) = 3
 *   Legal ops: ! ~ & ^ | + << >>
 *   Max ops: 40
 *   Rating: 4
 */
int bitCount(int x) {
  int tmp_every_2_bit = ( 0x55 ) | ( 0x55 << 8 );
  int every_2_bit = tmp_every_2_bit | ( tmp_every_2_bit << 16 );
  int tmp_every_4_bit = ( 0x33 )| ( 0x33 << 8 );
  int every_4_bit = tmp_every_4_bit | ( tmp_every_4_bit << 16);
  int tmp_every_8_bit = ( 0x0f ) | ( 0x0f << 8 );
  int every_8_bit = tmp_every_8_bit | ( tmp_every_8_bit << 16);
  int every_16_bit = ( 0xff ) | ( 0xff << 16 );
  int every_32_bit = ( 0xff ) | ( 0xff << 8 );

  int result = 0;

  result = ( x & every_2_bit ) + ( ( x >> 1) & every_2_bit );
  result = ( result & every_4_bit ) + ( ( result >> 2 ) & every_4_bit );
  result = ( result + ( result >> 4 ) ) & every_8_bit;
  result = ( result + ( result >> 8 ) ) & every_16_bit;
  result = ( result + ( result >> 16 ) ) & every_32_bit;

  return result;
}
```

> ilog2  
> ilog2(16) = 4

`分析思路如图,图1和图2分别分析了bit_16,bit_8,bit_4,bit_2,bit_1 是否为0的情况,最终的结果是二者交叉出现后的和`


![2为底的对数1](/assets/ilog1.png "how many bits of a positive integer")

![2为底的对数2](/assets/ilog2.png "how many bits of a positive integer")


```c
/*
 * ilog2 - return floor(log base 2 of x), where x > 0
 *   Example: ilog2(16) = 4: 得到多少位二进制表示即为答案
 *   Legal ops: ! ~ & ^ | + << >>
 *   Max ops: 90
 *   Rating: 4
 *   负数和0的结果应该为 0. 只能求正数的对数.
 */
int ilog2(int x) {
  // get a positive number; negative and zero will return 0, only get positive's bitcounts.
  // positive number's msb is 0, then !x will get 1, 1 << 31 get 0x10000000, 0x10000000 >> 31 get 0xffffffff.
  // negative and zero is 1, then !x will get 0, (0 << 31)>>31 always be 0x00000000.
  int isPositive = ( !((x >> 31) | (!x)) << 31 )>>31;
  // 二分法.见图中分析过程
  int bit_16, bit_8, bit_4, bit_2, bit_1;
  int result = 0;
  bit_16 = !(!(x >> 16)) << 4;
  x = x >> bit_16;
  bit_8 = !(!(x >> 8)) << 3;
  x = x >> bit_8;
  bit_4 = !(!(x >> 4)) << 2;
  x = x >> bit_4;
  bit_2 = !(!(x >> 2)) << 1;
  x = x >> bit_2;
  bit_1 = !(!(x >> 1));
  result = (isPositive) & (bit_16 + bit_8 + bit_4 + bit_2 + bit_1);
  return result;
}
```

### 浮点数在计算机中存储遵循IEEE754标准

总的来说,IEEE754标准将浮点数拆分成两个定点数后进行管理.这两个定点部分分别为阶码Exponent和尾数Significand.

For Example

1.234 * 2^10 

234 为 **尾数**, 计算机将其转换成二进制后存储为 尾数Significand(Single Presicise has 23 bits, Double Presicise 52 bits)


10 为 **指数(阶)**, 计算机将其转换成二进制后存储为 阶码E (SP 8 bits)


还有一位**符号位 S**

> 阶,阶码E,偏置常数bias(64位系统中)

bias for SP = 2^(8-1) - 1  
bias for DP = 2^(11-1) - 1

E = 阶 + bias

> 阶码

阶码在计算机中不是用补码的方式存储的,而是用移码,所以移码的编码规则即上面所述的E = 阶 + bias.偏置常数bias是根据喜好设置的,没有固定值(具体怎么来的有待查询)

> 尾数

Significand 使用原码存储,十转二后直接存储,SP只存储23位.

> 规格化的浮点数(SP为例)

- 整数部分必须为1
- 阶码的范围0000 0001(-126) ~ 1111 1110(127),即不能全为也不能为全1

> float = -12.75在计算机中实际存储形式

```zsh
-12.75 = -1100.11B
规格化为: -1.10011 * 2^3
S = 1
E = bias + 阶 = 127 + 3 = 130 = 128 + 2 = 10000010B
Significand = 10011

S E			 Significand
1 10000010 100 1100 0000 0000 0000 0000
```

> BEE00000 -> float

```zsh
S E			 Significand
1 01111101 110 0000 0000 0000 0000 0000

float a = (-1)^S * (1 + Significand) * 2^(E-bias)
        = -1 * (1 + 2^(-1) + 2^(-2)) * 2^(01111 101B - 127D)
        = -1 * 1.75 * 2 ^ (125 - 127)
        = -1 * 1.75 * 2^(-2)
        = -1.75*0.25
        = -0.4357
```
> 需要记住的一些内容

```zsh
N为机器位数
[X]补 = 2^N + X
[7]补 = 2^N + 7
[-7]补 = 2^N + (-7) = 2^N - 7
```