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

#### BOMB LAB

##### PHASE_4

```c
 421 0000000000400fce <func4>:
 422   400fce:   48 83 ec 08             sub    $0x8,%rsp
 423   400fd2:   89 d0                   mov    %edx,%eax
 424   400fd4:   29 f0                   sub    %esi,%eax
 425   400fd6:   89 c1                   mov    %eax,%ecx
 426   400fd8:   c1 e9 1f                shr    $0x1f,%ecx
 427   400fdb:   01 c8                   add    %ecx,%eax
 428   400fdd:   d1 f8                   sar    %eax
 429   400fdf:   8d 0c 30                lea    (%rax,%rsi,1),%ecx
 430   400fe2:   39 f9                   cmp    %edi,%ecx
 431   400fe4:   7e 0c                   jle    400ff2 <func4+0x24>
 432   400fe6:   8d 51 ff                lea    -0x1(%rcx),%edx
 433   400fe9:   e8 e0 ff ff ff          callq  400fce <func4>
 434   400fee:   01 c0                   add    %eax,%eax
 435   400ff0:   eb 15                   jmp    401007 <func4+0x39>
 436   400ff2:   b8 00 00 00 00          mov    $0x0,%eax
 437   400ff7:   39 f9                   cmp    %edi,%ecx
 438   400ff9:   7d 0c                   jge    401007 <func4+0x39>
 439   400ffb:   8d 71 01                lea    0x1(%rcx),%esi
 440   400ffe:   e8 cb ff ff ff          callq  400fce <func4>
 441   401003:   8d 44 00 01             lea    0x1(%rax,%rax,1),%eax
 442   401007:   48 83 c4 08             add    $0x8,%rsp
 443   40100b:   c3                      retq
 444
 445 000000000040100c <phase_4>:
 446   40100c:   48 83 ec 18             sub    $0x18,%rsp
 447   401010:   48 8d 4c 24 0c          lea    0xc(%rsp),%rcx
 448   401015:   48 8d 54 24 08          lea    0x8(%rsp),%rdx
 449   40101a:   be cf 25 40 00          mov    $0x4025cf,%esi
 450   40101f:   b8 00 00 00 00          mov    $0x0,%eax
 451   401024:   e8 c7 fb ff ff          callq  400bf0 <__isoc99_sscanf@plt>
 452   401029:   83 f8 02                cmp    $0x2,%eax
 453   40102c:   75 07                   jne    401035 <phase_4+0x29>
 454   40102e:   83 7c 24 08 0e          cmpl   $0xe,0x8(%rsp)
 455   401033:   76 05                   jbe    40103a <phase_4+0x2e>
 456   401035:   e8 00 04 00 00          callq  40143a <explode_bomb>
 457   40103a:   ba 0e 00 00 00          mov    $0xe,%edx
 458   40103f:   be 00 00 00 00          mov    $0x0,%esi
 459   401044:   8b 7c 24 08             mov    0x8(%rsp),%edi
 460   401048:   e8 81 ff ff ff          callq  400fce <func4>
 461   40104d:   85 c0                   test   %eax,%eax
 462   40104f:   75 07                   jne    401058 <phase_4+0x4c>
 463   401051:   83 7c 24 0c 00          cmpl   $0x0,0xc(%rsp)
 464   401056:   74 05                   je     40105d <phase_4+0x51>
 465   401058:   e8 dd 03 00 00          callq  40143a <explode_bomb>
 466   40105d:   48 83 c4 18             add    $0x18,%rsp
 467   401061:   c3                      retq

```

> phase_4

phase_4接收两个参数, 根据`40101a:   be cf 25 40 00          mov    $0x4025cf,%esi` 用gdb查看phase_4接收`%d %d`格式的参数.

```c
463   401051:   83 7c 24 0c 00          cmpl   $0x0,0xc(%rsp)
464   401056:   74 05                   je     40105d <phase_4+0x51>
465   401058:   e8 dd 03 00 00          callq  40143a <explode_bomb>
```
 
 

通过463-465行代码可以判断出第二个参数必定为0,难点在于如何确定phase_4的每一个参数.根据phase_4中参数寄存器`rdi,rsi,rdx`可知func4的三个参数分别为: `rdi(未知),rsi=0,rdx=14(0xe)`.其中`rdi`必须保证func4的返回值寄存器`eax=0`且`0<= rdi <=14`.

> func4

根据func4的反汇编的代码可以看出这是一个递归调用过程.来看一下第一次调用func4时,发生了什么.

```c
400fd2:   89 d0                   mov    %edx,%eax  // eax = edx
400fd4:   29 f0                   sub    %esi,%eax  // eax = edx - esi
400fd6:   89 c1                   mov    %eax,%ecx  // ecx = eax
400fd8:   c1 e9 1f                shr    $0x1f,%ecx // ecx = ecx >> 31 = 0, 32位表示的正数经过逻辑右移31位后是0 
400fdb:   01 c8                   add    %ecx,%eax  // eax = ecx + eax = 0+eax
400fdd:   d1 f8                   sar    %eax // eax = eax/2;
400fdf:   8d 0c 30                lea    (%rax,%rsi,1),%ecx //ecx = rax + rsi

进行等量代换后的关系即为 ecx = (rdi + rsi) / 2;
```

经过分析后写出伪代码如下

```c
func4(rdi,rsi,rdx) {
      int result = eax;
      int ecx = (rsi + rdx)/2;
1     if(ecx > rdi) {
         rdx = ecx - 1;
         func4(rdi,rsi,rdx);
         result += result;
      }else{
         eax = 0;
         result = eax;
2         if(ecx >= rdi){
             return result;
         }else{
             rsi = tmp + 1;
             func4(rdi, rsi, rdx);
         }
     }
}
```
通过伪代码再来看一下,这个递归过程结束的条件语句是`2`的条件成立,而进入`2`所在的代码区域前提是`ecx <= rdi`, 这与`ecx >= rdi`交集很容易得出,即`ecx = rdi`,而进入第一次调用func4后马上有`ecx = (rsi + rdx)/2 = 7`, 于是有`rdi = 7`, 此时返回值正好为0.

> 结果

phase_4的两个输入为:` 7 0 `


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