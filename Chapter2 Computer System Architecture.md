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