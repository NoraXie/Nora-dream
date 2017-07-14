### 计算机知识总框架

![](/assets/计算机知识体系结构.jpg)


### 深入理解计算机实验

#### DATA LAB

> bitCount:    
> bitCount(7) = 3, bitCount(5) = 2

![统计二进制数中1的数量](/assets/bitcount1.png "how many ones of a decimal integer")

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