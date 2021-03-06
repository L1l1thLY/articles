# C语言中的位移操作

## 逻辑位移和算术位移

位移操作分为两种类型，分别是逻辑位移和算术位移，分别代表了对二进制数进行位移的不同规则。

首先对于，左移操作，逻辑左移和算术左移具有相同的表现：所有位依次向左移动若干位，最低位（右侧）补0，溢出最高位（左侧，对符号数来说是符号位）被丢弃。在下面的示例中，我们假设有一个8bit二进制数并分别进行两种位移操作：

```
// []代表符号位
[0]0001011;
// 逻辑左移4位
[1]0110000;
// 算术右移4位
[1]0110000;
```

但是，逻辑右移和算术右移在处理符号位时具有不同的行为：在逻辑右移中，将会直接在最高位（左侧）补0，最低位（右侧）被丢弃；而对于算术右移，最低位（右侧）依然被丢弃，但是最高位（左侧）会根据符号位的不同，补齐与符号位相同的数字。例如：

```
[1]0011011;
// 逻辑右移4位
[0]0001001;
// 算术右移4位
[1]1111001;
```

## 测试

我们使用64位`ubuntu16.04LTS`和`gcc 5.4`进行测试，测试程序如下：

```c
#include <stdio.h>

#define SRR(x, y) (((x) + (1U << ((y) - 1))) >> (y))

typedef unsigned int uint32;
typedef int int32;

int main(void) {
    uint32 x = 1;
    int32 y = 1;

    printf("initial value:\nx=1, y=1\n");

    x <<= 31;
    y <<= 31;
    printf("after 31 bit left shift:\nx=%u, y=%d\n", x, y);

    x >>= 31;
    y >>= 31;
    printf("then after 31 bit right shift:\nx=%u, y=%d\n", x, y);

    return 0;
}
```

首先分别定义32位无符号整数`x`和32位有符号整数`y`（在64位机器中，`int`长度为32bit）。

在C语言中，对于32位整数，有符号数的表示范围为`-2147483648~2147483647`（由于1位符号位的影响，实际上只有31位用于数字表示），无符号数的表示范围在`0~4294967295`；且对于有符号数的位移，执行算术位移，无符号数的位移，执行逻辑位移。

上述程序的执行结果为：

```
initial value:
x=1, y=1
after 31 bit left shift:
x=2147483648, y=-2147483648
then after 31 bit right shift:
x=1, y=-1
```

完全符合我们之前的描述。

如果对上述程序做一些小修改：

```c
#include <stdio.h>

#define SRR(x, y) (((x) + (1U << ((y) - 1))) >> (y))

typedef unsigned int uint32;
typedef int int32;

int main(void) {
    // 二进制数：
    // [1]011 1111 1111 1111 1111 1111 1111 1111
    int32 y = (int32)0xbfffffff;

    printf("initial value:y=%d\n", y);

    y <<= 1;
    // 输出
    // [0]111 1111 1111 1111 1111 1111 1111 1110
    printf("after 1 bit left shift:y=%d\n", y);

    return 0;
}
```

上述程序将会输出：

```
initial value:
y=-1073741825
after 31 bit left shift:
y=2147483646
```

这段程序证明了，算术左移会忽视符号位的存在，位移过程中存在改变符号位的风险。