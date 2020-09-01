> Rule 04. Integers (INT)
>
> INT32-C. Ensure that operations on signed integers do not result in overflow

Signed integer overflow is [undefined behavior 36](https://wiki.sei.cmu.edu/confluence/display/c/BB.+Definitions#BB.Definitions-undefinedbehavior). Consequently, [implementations](https://wiki.sei.cmu.edu/confluence/display/c/BB.+Definitions#BB.Definitions-implementation) have considerable latitude in how they deal with signed integer overflow. (See [MSC15-C. Do not depend on undefined behavior](https://wiki.sei.cmu.edu/confluence/display/c/MSC15-C.+Do+not+depend+on+undefined+behavior).) An implementation that defines signed integer types as being modulo, for example, need not detect integer overflow. Implementations may also trap on signed arithmetic overflows, or simply assume that overflows will never happen and generate object code accordingly.  It is also possible for the same conforming implementation to emit code that exhibits different behavior in different contexts. For example, an implementation may determine that a signed integer loop control variable declared in a local scope cannot overflow and may emit efficient code on the basis of that determination, while the same implementation may determine that a global variable used in a similar context will wrap.

有符号整数溢出是 [undefined 行为 36](https://wiki.sei.cmu.edu/confluence/display/c/BB.+Definitions#BB.Definitions-undefinedbehavior) ，因此，处理有符号整数溢出的 [实现方式(implementations)](https://wiki.sei.cmu.edu/confluence/display/c/BB.+Definitions#BB.Definitions-implementation) 多种多样。 (See [MSC15-C. Do not depend on undefined behavior](https://wiki.sei.cmu.edu/confluence/display/c/MSC15-C.+Do+not+depend+on+undefined+behavior).) ???例如，把有符号整数类型定义为模，就不需要检测是否溢出。也可以通过捕获整数算数运算溢出异常来实现，或者直接假设数据不会溢出。即便是相同的符合标准的代码，在不同的代码环境中也可能产生不同的行为。???比如，假设在局部作用域内的有符号整数循环控制变量不会溢出，就可以写出简单高效的代码，但是，这个假设可能同时默认附近的全局变量也不会溢出，这样就带来了风险。

For these reasons, it is important to ensure that operations on signed integers do not result in overflow. Of particular importance are operations on signed integer values that originate from a [tainted source](https://wiki.sei.cmu.edu/confluence/display/c/BB.+Definitions#BB.Definitions-taintedsource) and are used as

鉴于以上原因，必须确保有符号整数相关的操作不会溢出。尤其要命的是，如果有符号整数的数值来自 [外部未知的数据(tainted source)](https://wiki.sei.cmu.edu/confluence/display/c/BB.+Definitions#BB.Definitions-taintedsource) ，而且要计算下面这些数据的时候：

- Integer operands of any pointer arithmetic, including array indexing
- The assignment expression for the declaration of a variable length array
- The postfix expression preceding square brackets `[]` or the expression in square brackets `[]` of a subscripted designation of an element of an array object
- Function arguments of type `size_t` or `rsize_t` (for example, an argument to a memory allocation function)
- 一切指针运算，包括数组下标
- 定义变长度数组的表达式
- 方括号 [] 前面的后缀表达式或者方括号 [] 里面取数组元素的表达式
- `size_t` 或 `rsize_t` 类型的函数参数 (例如内存申请函数的参数)

Integer operations will overflow if the resulting value cannot be represented by the underlying representation of the integer. The following table indicates which operations can result in overflow.

如果整数的运算结果无法用底层二进制数表示(underlying representation)，整数就会溢出。下表列出了可能造成溢出的操作：

| Operator | Overflow | Operator | Overflow | Operator  | Overflow | Operator | Overflow |
| :------- | :------- | :------- | :------- | :-------- | :------- | :------- | :------- |
| `+`      | Yes      | `-=`     | Yes      | `<<`      | Yes      | `<`      | No       |
| `-`      | Yes      | `*=`     | Yes      | `>>`      | No       | `>`      | No       |
| `*`      | Yes      | `/=`     | Yes      | `&`       | No       | `>=`     | No       |
| `/`      | Yes      | `%=`     | Yes      | `|`       | No       | `<=`     | No       |
| `%`      | Yes      | `<<=`    | Yes      | `^`       | No       | `==`     | No       |
| `++`     | Yes      | `>>=`    | No       | `~`       | No       | `!=`     | No       |
| `--`     | Yes      | `&=`     | No       | `!`       | No       | `&&`     | No       |
| `=`      | No       | `|=`     | No       | `unary +` | No       | `||`     | No       |
| `+=`     | Yes      | `^=`     | No       | `unary -` | Yes      | `?:`     | No       |

The following sections examine specific operations that are susceptible to integer overflow. When operating on integer types with less precision than `int`, integer promotions are applied. The usual arithmetic conversions may also be applied to (implicitly) convert operands to equivalent types before arithmetic operations are performed. Programmers should understand integer conversion rules before trying to implement secure arithmetic operations. (See [INT02-C. Understand integer conversion rules](https://wiki.sei.cmu.edu/confluence/display/c/INT02-C.+Understand+integer+conversion+rules).)

下面具体分析一下容易造成整数溢出的操作。如果参与整型运算的数据类型精度比`int` 类型低，它的类型就会被提升为`int`。如果运算符两边的操作数类型不同，在算数运算之前，通常会隐式转换为相同类型。程序员在拉出安全的运算代码前，有必要了解一下整数类型转换规则。(See [INT02-C. Understand integer conversion rules](https://wiki.sei.cmu.edu/confluence/display/c/INT02-C.+Understand+integer+conversion+rules).)



## Implementation Details

## 实施细节(Implementation Details)

GNU GCC invoked with the `-fwrapv` command-line option defines the same modulo arithmetic for both unsigned and signed integers.

GNU GCC 的`-fwrapv`参数：把无符号整数和有符号整数的取模运算定义为相同的运算。

GNU GCC invoked with the `-ftrapv` command-line option causes a trap to be generated when a signed integer overflows, which will most likely abnormally exit. On a UNIX system, the result of such an event may be a signal sent to the process.

GNU GCC 的`-ftrapv`参数：有符号整数溢出时，产生一个异常信息，可能使系统异常退出。在 UNIX 环境中，可能产生一个信号并发送给进程。

GNU GCC invoked without either the `-fwrapv` or the `-ftrapv` option may simply assume that signed integers never overflow and may generate object code accordingly.

GNU GCC 如果不设置`-fwrapv`或`-ftrapv`参数，就会假设无符号整数永远不会溢出。

## Atomic Integers

## ???原子整数(Atomic Integers)

The C Standard defines the behavior of arithmetic on atomic signed integer types to use two's complement representation with silent wraparound on overflow; there are no undefined results. Although defined, these results may be unexpected and therefore carry similar risks to [unsigned integer wrapping](https://wiki.sei.cmu.edu/confluence/display/c/BB.+Definitions#BB.Definitions-unsignedintegerwrapping). (See [INT30-C. Ensure that unsigned integer operations do not wrap](https://wiki.sei.cmu.edu/confluence/display/c/INT30-C.+Ensure+that+unsigned+integer+operations+do+not+wrap).) Consequently, signed integer overflow of atomic integer types should also be prevented or detected.

C 标准定义原子有符号整数使用二者补码运算，???溢出时没有提示(silent wraparound)；因此结果不会产生 undefined 行为。然而就算这样定义了，这些结果仍然可能不是期望值，因此与 [无符号整数溢出(unsigned integer wrapping)](https://wiki.sei.cmu.edu/confluence/display/c/BB.+Definitions#BB.Definitions-unsignedintegerwrapping) 有类似的风险。(See [INT30-C. Ensure that unsigned integer operations do not wrap](https://wiki.sei.cmu.edu/confluence/display/c/INT30-C.+Ensure+that+unsigned+integer+operations+do+not+wrap).) 所以仍然有必要防止或检测原子整数的溢出。



## Addition

## 加法

Addition is between two operands of arithmetic type or between a pointer to an object type and an integer type. This rule applies only to addition between two operands of arithmetic type. (See [ARR37-C. Do not add or subtract an integer to a pointer to a non-array object](https://wiki.sei.cmu.edu/confluence/display/c/ARR37-C.+Do+not+add+or+subtract+an+integer+to+a+pointer+to+a+non-array+object) and [ARR30-C. Do not form or use out-of-bounds pointers or array subscripts](https://wiki.sei.cmu.edu/confluence/display/c/ARR30-C.+Do+not+form+or+use+out-of-bounds+pointers+or+array+subscripts).)

Incrementing is equivalent to adding 1.

加法操作应用于两个算数类型的操作数之间，或者指针和对象、整数之间。以下的规则仅适用于两个算数类型的操作数的加法操作。 (See [ARR37-C. Do not add or subtract an integer to a pointer to a non-array object](https://wiki.sei.cmu.edu/confluence/display/c/ARR37-C.+Do+not+add+or+subtract+an+integer+to+a+pointer+to+a+non-array+object) and [ARR30-C. Do not form or use out-of-bounds pointers or array subscripts](https://wiki.sei.cmu.edu/confluence/display/c/ARR30-C.+Do+not+form+or+use+out-of-bounds+pointers+or+array+subscripts).)

自增运算相当于 +1

### Noncompliant Code Example

### 新手行为(Noncompliant Code Example)

This noncompliant code example can result in a signed integer overflow during the addition of the signed operands `si_a` and `si_b`:

这种新手行为的代码，在计算`si_a`和`si_b`的和的时候可能造成整数溢出。

```c
void func(signed int si_a, signed int si_b) {
	signed int sum = si_a + si_b;
    /* ... */
}
```

### Compliant Solution

### 规范代码

This compliant solution ensures that the addition operation cannot overflow, regardless of representation:

优秀的代码确保加法操作不会溢出，虽然复杂了些(regardless of representation)：

```c
#include <limits.h>

void f(signed int si_a, signed int si_b) {
    signed int sum;
    if (((si_b > 0) && (si_a > (INT_MAX - si_b))) ||
        ((si_b < 0) && (si_a < (INT_MIN - si_b)))) {
        /* Handle error */
    } else {
        sum = si_a + si_b;
    }
    /* ... */
}
```



## Subtraction

## 减法

Subtraction is between two operands of arithmetic type, two pointers to qualified or unqualified versions of compatible object types, or a pointer to an object type and an integer type. This rule applies only to subtraction between two operands of arithmetic type. (See [ARR36-C. Do not subtract or compare two pointers that do not refer to the same array](https://wiki.sei.cmu.edu/confluence/display/c/ARR36-C.+Do+not+subtract+or+compare+two+pointers+that+do+not+refer+to+the+same+array), [ARR37-C. Do not add or subtract an integer to a pointer to a non-array object](https://wiki.sei.cmu.edu/confluence/display/c/ARR37-C.+Do+not+add+or+subtract+an+integer+to+a+pointer+to+a+non-array+object), and [ARR30-C. Do not form or use out-of-bounds pointers or array subscripts](https://wiki.sei.cmu.edu/confluence/display/c/ARR30-C.+Do+not+form+or+use+out-of-bounds+pointers+or+array+subscripts) for information about pointer subtraction.)

Decrementing is equivalent to subtracting 1.

减法操作应用于两个算数类型的操作数之间，???或者指向同类型对象的两个指针之间(two pointers to qualified or unqualified versions of compatible object types)。以下的规则仅适用于两个算数类型的操作数的减法操作。(See [ARR36-C. Do not subtract or compare two pointers that do not refer to the same array](https://wiki.sei.cmu.edu/confluence/display/c/ARR36-C.+Do+not+subtract+or+compare+two+pointers+that+do+not+refer+to+the+same+array), [ARR37-C. Do not add or subtract an integer to a pointer to a non-array object](https://wiki.sei.cmu.edu/confluence/display/c/ARR37-C.+Do+not+add+or+subtract+an+integer+to+a+pointer+to+a+non-array+object), and [ARR30-C. Do not form or use out-of-bounds pointers or array subscripts](https://wiki.sei.cmu.edu/confluence/display/c/ARR30-C.+Do+not+form+or+use+out-of-bounds+pointers+or+array+subscripts) for information about pointer subtraction.)

自减运算相当于 -1

### Noncompliant Code Example

### 新手行为

This noncompliant code example can result in a signed integer overflow during the subtraction of the signed operands `si_a` and `si_b`:

这种新手行为的代码，在计算`si_a`和`si_b`的差的时候可能造成整数溢出。

```c
void func(signed int si_a, signed int si_b) {
    signed int diff = si_a - si_b;
    /* ... */
}
```

### Compliant Solution

### 规范代码

This compliant solution tests the operands of the subtraction to guarantee there is no possibility of signed overflow, regardless of representation:

优秀的代码确保减法操作不会溢出，虽然复杂了些：

```c
#include <limits.h>

void func(signed int si_a, signed int si_b) {
    signed int diff;
    if ((si_b > 0 && si_a < INT_MIN + si_b) ||
        (si_b < 0 && si_a > INT_MAX + si_b)) {
        /* Handle error */
    } else {
        diff = si_a - si_b;
    }
    /* ... */
}
```



## Multiplication

## 乘法

Multiplication is between two operands of arithmetic type.

乘法操作应用于两个算数类型的操作数之间。

### Noncompliant Code Example

### 新手行为

This noncompliant code example can result in a signed integer overflow during the multiplication of the signed operands `si_a` and `si_b`:

这种新手行为的代码，在计算`si_a`和`si_b`的积的时候可能造成整数溢出。

```c
void func(signed int si_a, signed int si_b) {
    signed int result = si_a * si_b;
    /* ... */
}
```

### Compliant Solution

### 规范代码

The product of two operands can always be represented using twice the number of bits than exist in the precision of the larger of the two operands. This compliant solution eliminates signed overflow on systems where `long long` is at least twice the precision of `int`:

两数的积可以用较大操作数有效二进制位数的两倍表示。优秀的代码限制整数溢出，前提条件是系统的`long long`类型长度至少是`int`长度的 2 倍。

```c
#include <assert.h>
#include <inttypes.h>
#include <limits.h>
#include <stddef.h>

extern size_t popcount(uintmax_t);
#define PRECISION(umax_value) popcount(umax_value)

void func(signed int si_a, signed int si_b) {
    signed int result;
    signed long long tmp;
    assert(PRECISION(ULLONG_MAX) >= 2 * PRECISION(UINT_MAX));
    tmp = (signed long long)si_a * (signed long long)si_b;

    /*
   * If the product cannot be represented as a 32-bit integer,
   * handle as an error condition.
   */
    if ((tmp > INT_MAX) || (tmp < INT_MIN)) {
        /* Handle error */
    } else {
        result = (int)tmp;
    }
    /* ... */
}
```

The assertion fails if `long long` has less than twice the precision of `int`. The `PRECISION()` macro and `popcount()` function provide the correct precision for any integer type. (See [INT35-C. Use correct integer precisions](https://wiki.sei.cmu.edu/confluence/display/c/INT35-C.+Use+correct+integer+precisions).)

如果`long long`类型的长度小于`int`类型长度的 2 倍，`assert`断言语句抛出异常。`PRECISION()`宏和`popcount()`函数获取任意整数二进制位长度。(See [INT35-C. Use correct integer precisions](https://wiki.sei.cmu.edu/confluence/display/c/INT35-C.+Use+correct+integer+precisions).)

### Compliant Solution

### 规范代码

The following portable compliant solution can be used with any conforming implementation, including those that do not have an integer type that is at least twice the precision of `int`:

以下代码为兼容方案，兼容任意标准，适用于没有 2 倍`int`长度数据类型的系统。

```c
#include <limits.h>

void func(signed int si_a, signed int si_b) {
    signed int result;
    if (si_a > 0) {     /* si_a is positive */
        if (si_b > 0) { /* si_a and si_b are positive */
            if (si_a > (INT_MAX / si_b)) {
                /* Handle error */
            }
        } else { /* si_a positive, si_b nonpositive */
            if (si_b < (INT_MIN / si_a)) {
                /* Handle error */
            }
        }               /* si_a positive, si_b nonpositive */
    } else {            /* si_a is nonpositive */
        if (si_b > 0) { /* si_a is nonpositive, si_b is positive */
            if (si_a < (INT_MIN / si_b)) {
                /* Handle error */
            }
        } else { /* si_a and si_b are nonpositive */
            if ((si_a != 0) && (si_b < (INT_MAX / si_a))) {
                /* Handle error */
            }
        } /* End if si_a and si_b are nonpositive */
    }     /* End if si_a is nonpositive */

    result = si_a * si_b;
}
```



## Division

## 除法

Division is between two operands of arithmetic type. Overflow can occur during two's complement signed integer division when the dividend is equal to the minimum (negative) value for the signed integer type and the divisor is equal to `−1`. Division operations are also susceptible to divide-by-zero errors. (See [INT33-C. Ensure that division and remainder operations do not result in divide-by-zero errors](https://wiki.sei.cmu.edu/confluence/display/c/INT33-C.+Ensure+that+division+and+remainder+operations+do+not+result+in+divide-by-zero+errors).)

除法操作应用于两个算数类型的操作数之间。当被除数等于有符号整数(负的)最小值，除数等于 -1 时，就会溢出。当除数为 0 时除法也会报错。(See [INT33-C. Ensure that division and remainder operations do not result in divide-by-zero errors](https://wiki.sei.cmu.edu/confluence/display/c/INT33-C.+Ensure+that+division+and+remainder+operations+do+not+result+in+divide-by-zero+errors).)

### Noncompliant Code Example

### 新手行为

This noncompliant code example prevents divide-by-zero errors in compliance with [INT33-C. Ensure that division and remainder operations do not result in divide-by-zero errors](https://wiki.sei.cmu.edu/confluence/display/c/INT33-C.+Ensure+that+division+and+remainder+operations+do+not+result+in+divide-by-zero+errors) but does not prevent a signed integer overflow error in two's-complement.

这段代码检查了除数为 0 的情况  [INT33-C. Ensure that division and remainder operations do not result in divide-by-zero errors](https://wiki.sei.cmu.edu/confluence/display/c/INT33-C.+Ensure+that+division+and+remainder+operations+do+not+result+in+divide-by-zero+errors) ， 但是没有检查有符号整数除法溢出的情况。

```c
void func(signed long s_a, signed long s_b) {
    signed long result;
    if (s_b == 0) {
        /* Handle error */  
    } else {
        result = s_a / s_b;  
    }
    /* ... */
}
```

### Implementation Details

### 实现细节

On the x86-32 architecture, overflow results in a fault, which can be exploited as a [denial-of-service attack](https://wiki.sei.cmu.edu/confluence/display/c/BB.+Definitions#BB.Definitions-denial-of-service).

在 x86-32 环境下，溢出会导致错误，可能会被 DOS 攻击([denial-of-service attack](https://wiki.sei.cmu.edu/confluence/display/c/BB.+Definitions#BB.Definitions-denial-of-service))钻空子。

### Compliant Solution

### 规范代码

This compliant solution eliminates the possibility of divide-by-zero errors or signed overflow:

下面的代码是规范的实现，避免了除数为 0 的错误和整数溢出。

```c
#include <limits.h>

void func(signed long s_a, signed long s_b) {
  signed long result;
  if ((s_b == 0) || ((s_a == LONG_MIN) && (s_b == -1))) {
    /* Handle error */
  } else {
    result = s_a / s_b;
  }
  /* ... */
}
```

## Remainder

## 取余

The remainder operator provides the remainder when two operands of integer type are divided. Because many platforms implement remainder and division in the same instruction, the remainder operator is also susceptible to arithmetic overflow and division by zero. (See [INT33-C. Ensure that division and remainder operations do not result in divide-by-zero errors](https://wiki.sei.cmu.edu/confluence/display/c/INT33-C.+Ensure+that+division+and+remainder+operations+do+not+result+in+divide-by-zero+errors).)

取余运算符获取 2 个操作数相除的余数。很多平台用同一套指令实现取余和除法，所以取余运算符也要注意除数为 0 和整数溢出的问题。(See [INT33-C. Ensure that division and remainder operations do not result in divide-by-zero errors](https://wiki.sei.cmu.edu/confluence/display/c/INT33-C.+Ensure+that+division+and+remainder+operations+do+not+result+in+divide-by-zero+errors).)

### Noncompliant Code Example

### 新手行为

Many hardware architectures implement remainder as part of the division operator, which can overflow. Overflow can occur during a remainder operation when the dividend is equal to the minimum (negative) value for the signed integer type and the divisor is equal to −1. It occurs even though the result of such a remainder operation is mathematically 0. This noncompliant code example prevents divide-by-zero errors in compliance with [INT33-C. Ensure that division and remainder operations do not result in divide-by-zero errors](https://wiki.sei.cmu.edu/confluence/display/c/INT33-C.+Ensure+that+division+and+remainder+operations+do+not+result+in+divide-by-zero+errors) but does not prevent integer overflow:

很多硬件平台把取余操作当成除法运算的一部分，所以取余也会溢出。当被除数等于有符号整数(负的)最小值，除数等于 -1 时，就会溢出。而且就算余数为 0 ，也可能溢出。下面的代码检查了除数为 0 的情况  [INT33-C. Ensure that division and remainder operations do not result in divide-by-zero errors](https://wiki.sei.cmu.edu/confluence/display/c/INT33-C.+Ensure+that+division+and+remainder+operations+do+not+result+in+divide-by-zero+errors) ， 但是没有检查有符号整数除法溢出的情况。

```c
void func(signed long s_a, signed long s_b) {
  signed long result;
  if (s_b == 0) {
    /* Handle error */
  } else {
    result = s_a % s_b;
  }
  /* ... */
}
```

### Implementation Details

### 实现细节

On x86-32 platforms, the remainder operator for signed integers is implemented by the `idiv` instruction code, along with the divide operator. Because `LONG_MIN / −1` overflows, it results in a software exception with `LONG_MIN % −1` as well.

在 x86-32 平台上，有符号整数的取余运算符和除法运算符都用`idiv`指令实现。因为 `LONG_MIN / −1` 会溢出，所以 `LONG_MIN % −1` 也会抛出异常。

### Compliant Solution

### 规范代码

This compliant solution also tests the remainder operands to guarantee there is no possibility of an overflow:

下面的代码是规范的实现，避免了除数为 0 的错误和整数溢出。

```c
#include <limits.h>

void func(signed long s_a, signed long s_b) {
  signed long result;
  if ((s_b == 0 ) || ((s_a == LONG_MIN) && (s_b == -1))) {
    /* Handle error */
  } else {
    result = s_a % s_b;
  }
  /* ... */
}
```


## Left-Shift Operator

## 左移运算符

The left-shift operator takes two integer operands. The result of `E1 << E2` is `E1` left-shifted `E2` bit positions; vacated bits are filled with zeros.

左移运算符需要 2 个操作数。`E1 << E2`代表`E1`向左移动`E2`个二进制位；空位补零。

The C Standard, 6.5.7, paragraph 4 [[ISO/IEC 9899:2011](https://wiki.sei.cmu.edu/confluence/display/c/AA.+Bibliography#AA.Bibliography-ISO-IEC9899-2011)], states

C 标准 6.5.7，第 4 段 [[ISO/IEC 9899:2011](https://wiki.sei.cmu.edu/confluence/display/c/AA.+Bibliography#AA.Bibliography-ISO-IEC9899-2011)]：

> If `E1` has a signed type and nonnegative value, and `E1 × 2E2` is representable in the result type, then that is the resulting value; otherwise, the behavior is undefined.
>
> 如果`E1`是有符号非负整数，且结果可以用`E1 × 2E2`表示，则其为运算结果；否则该操作为 undefined 行为。

In almost every case, an attempt to shift by a negative number of bits or by more bits than exist in the operand indicates a logic error. These issues are covered by [INT34-C. Do not shift an expression by a negative number of bits or by greater than or equal to the number of bits that exist in the operand](https://wiki.sei.cmu.edu/confluence/display/c/INT34-C.+Do+not+shift+an+expression+by+a+negative+number+of+bits+or+by+greater+than+or+equal+to+the+number+of+bits+that+exist+in+the+operand).

几乎任何情况下，移动负数个二进制位或者移动的位数超过操作数上限，都会产生逻辑错误。详细内容参考 [INT34-C. Do not shift an expression by a negative number of bits or by greater than or equal to the number of bits that exist in the operand](https://wiki.sei.cmu.edu/confluence/display/c/INT34-C.+Do+not+shift+an+expression+by+a+negative+number+of+bits+or+by+greater+than+or+equal+to+the+number+of+bits+that+exist+in+the+operand)。

### Noncompliant Code Example

### 新手行为

This noncompliant code example performs a left shift, after verifying that the number being shifted is not negative, and the number of bits to shift is valid.  The `PRECISION()` macro and `popcount()` function provide the correct precision for any integer type. (See [INT35-C. Use correct integer precisions](https://wiki.sei.cmu.edu/confluence/display/c/INT35-C.+Use+correct+integer+precisions).) However, because this code does no overflow check, it can result in an unrepresentable value.

下面是一个左移操作的例子，确定了被移动数非负，移动数有效。`PRECISION()` 宏和`popcount()`函数获得所有整数的位数。 (See [INT35-C. Use correct integer precisions](https://wiki.sei.cmu.edu/confluence/display/c/INT35-C.+Use+correct+integer+precisions).) 然而，这段代码没有做溢出检查，所以还是可能得到预期以外的结果。

```c
#include <limits.h>
#include <stddef.h>
#include <inttypes.h>

extern size_t popcount(uintmax_t);
#define PRECISION(umax_value) popcount(umax_value)

void func(signed long si_a, signed long si_b) {
  signed long result;
  if ((si_a < 0) || (si_b < 0) ||
      (si_b >= PRECISION(ULONG_MAX)) {
    /* Handle error */
  } else {
    result = si_a << si_b;
  }
  /* ... */
}
```

### Compliant Solution

### 规范代码

This compliant solution eliminates the possibility of overflow resulting from a left-shift operation:

下面的代码是规范的实现，确保了左移操作不会溢出。

```c
#include <limits.h>
#include <stddef.h>
#include <inttypes.h>

extern size_t popcount(uintmax_t);
#define PRECISION(umax_value) popcount(umax_value)

void func(signed long si_a, signed long si_b) {
  signed long result;
  if ((si_a < 0) || (si_b < 0) ||
      (si_b >= PRECISION(ULONG_MAX)) ||
      (si_a > (LONG_MAX >> si_b))) {
    /* Handle error */
  } else {
    result = si_a << si_b;
  }
  /* ... */
}
```

## Unary Negation

## 一元运算符

The unary negation operator takes an operand of arithmetic type. Overflow can occur during two's complement unary negation when the operand is equal to the minimum (negative) value for the signed integer type.

一元运算符需要 1 个算数类型的操作数。当操作数是有符号整数(负的)最小值的时候，一元运算就会溢出。

### Noncompliant Code Example

### 新手行为

This noncompliant code example can result in a signed integer overflow during the unary negation of the signed operand `s_a`:

下面的代码取`s_a`的负值，可能导致整数溢出。

```c
void func(signed long s_a) {
  signed long result = -s_a;
  /* ... */
}
```

### Compliant Solution

### 规范代码

This compliant solution tests the negation operation to guarantee there is no possibility of signed overflow:

下面的代码是规范的实现，保证不会溢出：

```c
#include <limits.h>

void func(signed long s_a) {
  signed long result;
  if (s_a == LONG_MIN) {
    /* Handle error */
  } else {
    result = -s_a;
  }
  /* ... */
}
```

Risk Assessment

风险评估

Integer overflow can lead to buffer overflows and the execution of arbitrary code by an attacker.

整数溢出可能导致缓冲区溢出，执行黑客的任意代码。

| Rule    | Severity | Likelihood | Remediation Cost | Priority | Level  |
| :------ | :------- | :--------- | :--------------- | :------- | :----- |
| INT32-C | High     | Likely     | High             | **P9**   | **L2** |

### Automated Detection

| Tool                                                         | Version                                                      | Checker                                                      | Description                                                  |
| :----------------------------------------------------------- | :----------------------------------------------------------- | :----------------------------------------------------------- | :----------------------------------------------------------- |
| [Astrée](https://wiki.sei.cmu.edu/confluence/pages/viewpage.action?pageId=87152428) | ![img](https://wiki.sei.cmu.edu/confluence/plugins/servlet/confluence/placeholder/macro?definition=e2luY2x1ZGU6QXN0csOpZV9WfQ&locale=en_US&version=2) | **integer-overflow **                                        | Fully checked                                                |
| [CodeSonar](https://wiki.sei.cmu.edu/confluence/display/c/CodeSonar) | ![img](https://wiki.sei.cmu.edu/confluence/plugins/servlet/confluence/placeholder/macro?definition=e2luY2x1ZGU6Q29kZVNvbmFyX1Z9&locale=en_US&version=2) | **ALLOC.SIZE.ADDOFLOW ****ALLOC.SIZE.IOFLOW ****ALLOC.SIZE.MULOFLOW ****ALLOC.SIZE.SUBUFLOW ****MISC.MEM.SIZE.ADDOFLOW ****MISC.MEM.SIZE.BAD ****MISC.MEM.SIZE.MULOFLOW ****MISC.MEM.SIZE.SUBUFLOW** | Addition overflow of allocation size Integer overflow of allocation size Multiplication overflow of allocation size Subtraction underflow of allocation size Addition overflow of size Unreasonable size argument Multiplication overflow of size Subtraction underflow of size |
| [Coverity](https://wiki.sei.cmu.edu/confluence/display/c/Coverity) | ![img](https://wiki.sei.cmu.edu/confluence/plugins/servlet/confluence/placeholder/macro?definition=e2luY2x1ZGU6Q292ZXJpdHlfVn0&locale=en_US&version=2) | **TAINTED_SCALAR****BAD_SHIFT**                              | Implemented                                                  |
| [LDRA tool suite](https://wiki.sei.cmu.edu/confluence/display/c/LDRA) | ![img](https://wiki.sei.cmu.edu/confluence/plugins/servlet/confluence/placeholder/macro?definition=e2luY2x1ZGU6TERSQV9WfQ&locale=en_US&version=2) | **493 S,** **494 S**                                         | Partially implemented                                        |
| [Parasoft C/C++test](https://wiki.sei.cmu.edu/confluence/display/c/Parasoft) | ![img](https://wiki.sei.cmu.edu/confluence/plugins/servlet/confluence/placeholder/macro?definition=e2luY2x1ZGU6UGFyYXNvZnRfVn0&locale=en_US&version=2) | **CERT_C-INT32-a** **CERT_C-INT32-b** **CERT_C-INT32-c**     | Avoid integer overflows Integer overflow or underflow in constant expression in '+', '-', '*' operator Integer overflow or underflow in constant expression in '<<' operator |
| [Parasoft Insure++](https://wiki.sei.cmu.edu/confluence/display/c/Parasoft) |                                                              |                                                              | Runtime analysis                                             |
| [Polyspace Bug Finder](https://wiki.sei.cmu.edu/confluence/display/c/Polyspace+Bug+Finder) | ![img](https://wiki.sei.cmu.edu/confluence/plugins/servlet/confluence/placeholder/macro?definition=e2luY2x1ZGU6UG9seXNwYWNlIEJ1ZyBGaW5kZXJfVn0&locale=en_US&version=2) | [CERT C: Rule INT32-C](https://www.mathworks.com/help/bugfinder/ref/certcruleint32c.html) | Checks for:Integer overflowTainted division operandTainted modulo operandRule partially covered. |
| [PRQA QA-C](https://wiki.sei.cmu.edu/confluence/display/c/PRQA+QA-C) | ![img](https://wiki.sei.cmu.edu/confluence/plugins/servlet/confluence/placeholder/macro?definition=e2luY2x1ZGU6UFJRQSBRQS1DX3Z9&locale=en_US&version=2) | **2800, 2801, 2802, 2803,****2860, 2861, 2862, 2863**        | Fully implemented                                            |
| [PRQA QA-C++](https://wiki.sei.cmu.edu/confluence/pages/viewpage.action?pageId=88046345) | ![img](https://wiki.sei.cmu.edu/confluence/plugins/servlet/confluence/placeholder/macro?definition=e2luY2x1ZGU6Y3BsdXNwbHVzOlBSUUEgUUEtQysrX1Z9&locale=en_US&version=2) | **2800, 2801, 2802, 2803,****2860, 2861, 2862, 2863**        |                                                              |
| [PVS-Studio](https://wiki.sei.cmu.edu/confluence/display/cplusplus/PVS-Studio) | ![img](https://wiki.sei.cmu.edu/confluence/plugins/servlet/confluence/placeholder/macro?definition=e2luY2x1ZGU6UFZTLVN0dWRpb19WfQ&locale=en_US&version=2) | **[V1026](https://www.viva64.com/en/w/v1026/)**              |                                                              |
| [TrustInSoft Analyzer](https://wiki.sei.cmu.edu/confluence/display/c/TrustInSoft+Analyzer) | ![img](https://wiki.sei.cmu.edu/confluence/plugins/servlet/confluence/placeholder/macro?definition=e2luY2x1ZGU6VHJ1c3RJblNvZnQgQW5hbHl6ZXJfVn0&locale=en_US&version=2) | **signed_overflow **                                         | Exhaustively verified (see [one compliant and one non-compliant example](https://taas.trust-in-soft.com/tsnippet/t/06486475)). |

### Related Vulnerabilities

Search for [vulnerabilities](https://wiki.sei.cmu.edu/confluence/display/c/BB.+Definitions#BB.Definitions-vulnerability) resulting from the violation of this rule on the [CERT website](https://www.kb.cert.org/vulnotes/bymetric?searchview&query=FIELD+KEYWORDS+contains+INT32-C).

## Related Guidelines

[Key here](https://wiki.sei.cmu.edu/confluence/display/c/How+this+Coding+Standard+is+Organized#HowthisCodingStandardisOrganized-RelatedGuidelines) (explains table format and definitions)

| Taxonomy                                                     | Taxonomy item                                                | Relationship                                        |
| :----------------------------------------------------------- | :----------------------------------------------------------- | :-------------------------------------------------- |
| [CERT C](https://wiki.sei.cmu.edu/confluence/display/c/SEI+CERT+C+Coding+Standard) | [INT02-C. Understand integer conversion rules](https://wiki.sei.cmu.edu/confluence/display/c/INT02-C.+Understand+integer+conversion+rules) | Prior to 2018-01-12: CERT: Unspecified Relationship |
| [CERT C](https://wiki.sei.cmu.edu/confluence/display/c/SEI+CERT+C+Coding+Standard) | [INT35-C. Use correct integer precisions](https://wiki.sei.cmu.edu/confluence/display/c/INT35-C.+Use+correct+integer+precisions) | Prior to 2018-01-12: CERT: Unspecified Relationship |
| [CERT C](https://wiki.sei.cmu.edu/confluence/display/c/SEI+CERT+C+Coding+Standard) | [INT33-C. Ensure that division and remainder operations do not result in divide-by-zero errors](https://wiki.sei.cmu.edu/confluence/display/c/INT33-C.+Ensure+that+division+and+remainder+operations+do+not+result+in+divide-by-zero+errors) | Prior to 2018-01-12: CERT: Unspecified Relationship |
| [CERT C](https://wiki.sei.cmu.edu/confluence/display/c/SEI+CERT+C+Coding+Standard) | [INT34-C. Do not shift an expression by a negative number of bits or by greater than or equal to the number of bits that exist in the operand](https://wiki.sei.cmu.edu/confluence/display/c/INT34-C.+Do+not+shift+an+expression+by+a+negative+number+of+bits+or+by+greater+than+or+equal+to+the+number+of+bits+that+exist+in+the+operand) | Prior to 2018-01-12: CERT: Unspecified Relationship |
| [CERT C](https://wiki.sei.cmu.edu/confluence/display/c/SEI+CERT+C+Coding+Standard) | [ARR30-C. Do not form or use out-of-bounds pointers or array subscripts](https://wiki.sei.cmu.edu/confluence/display/c/ARR30-C.+Do+not+form+or+use+out-of-bounds+pointers+or+array+subscripts) | Prior to 2018-01-12: CERT: Unspecified Relationship |
| [CERT C](https://wiki.sei.cmu.edu/confluence/display/c/SEI+CERT+C+Coding+Standard) | [ARR36-C. Do not subtract or compare two pointers that do not refer to the same array](https://wiki.sei.cmu.edu/confluence/display/c/ARR36-C.+Do+not+subtract+or+compare+two+pointers+that+do+not+refer+to+the+same+array) | Prior to 2018-01-12: CERT: Unspecified Relationship |
| [CERT C](https://wiki.sei.cmu.edu/confluence/display/c/SEI+CERT+C+Coding+Standard) | [ARR37-C. Do not add or subtract an integer to a pointer to a non-array object](https://wiki.sei.cmu.edu/confluence/display/c/ARR37-C.+Do+not+add+or+subtract+an+integer+to+a+pointer+to+a+non-array+object) | Prior to 2018-01-12: CERT: Unspecified Relationship |
| [CERT C](https://wiki.sei.cmu.edu/confluence/display/c/SEI+CERT+C+Coding+Standard) | [MSC15-C. Do not depend on undefined behavior](https://wiki.sei.cmu.edu/confluence/display/c/MSC15-C.+Do+not+depend+on+undefined+behavior) | Prior to 2018-01-12: CERT: Unspecified Relationship |
| [CERT C](https://wiki.sei.cmu.edu/confluence/display/c/SEI+CERT+C+Coding+Standard) | [CON08-C. Do not assume that a group of calls to independently atomic methods is atomic](https://wiki.sei.cmu.edu/confluence/display/c/CON08-C.+Do+not+assume+that+a+group+of+calls+to+independently+atomic+methods+is+atomic) | Prior to 2018-01-12: CERT: Unspecified Relationship |
| [CERT Oracle Secure Coding Standard for Java](https://wiki.sei.cmu.edu/confluence/display/java/SEI+CERT+Oracle+Coding+Standard+for+Java) | [INT00-J. Perform explicit range checking to avoid integer overflow](https://wiki.sei.cmu.edu/confluence/display/java/NUM00-J.+Detect+or+prevent+integer+overflow) | Prior to 2018-01-12: CERT: Unspecified Relationship |
| [ISO/IEC TR 24772:2013](https://wiki.sei.cmu.edu/confluence/display/c/AA.+Bibliography#AA.Bibliography-ISO-IECTR24772-2013) | Arithmetic Wrap-Around Error [FIF]                           | Prior to 2018-01-12: CERT: Unspecified Relationship |
| [ISO/IEC TS 17961](https://wiki.sei.cmu.edu/confluence/display/c/AA.+Bibliography#AA.Bibliography-ISO-IECTS17961) | Overflowing signed integers [intoflow]                       | Prior to 2018-01-12: CERT: Unspecified Relationship |
| [CWE 2.11](https://cwe.mitre.org/data/index.html)            | [CWE-190](http://cwe.mitre.org/data/definitions/190.html), Integer Overflow or Wraparound | 2017-05-18: CERT: Partial overlap                   |
| [CWE 2.11](http://cwe.mitre.org/)                            | [CWE-191](https://cwe.mitre.org/data/index.html191.html)     | 2017-05-18: CERT: Partial overlap                   |
| [CWE 2.11](http://cwe.mitre.org/)                            | [CWE-680](https://cwe.mitre.org/data/index.html680.html)     | 2017-05-18: CERT: Partial overlap                   |

## CERT-CWE Mapping Notes

[Key here](https://wiki.sei.cmu.edu/confluence/pages/viewpage.action?pageId=87152408#HowthisCodingStandardisOrganized-CERT-CWEMappingNotes) for mapping notes

### CWE-20 and INT32-C

See CWE-20 and ERR34-C

### CWE-680 and INT32-C

Intersection( INT32-C, MEM35-C) = Ø

Intersection( CWE-680, INT32-C) =

- Signed integer overflows that lead to buffer overflows

CWE-680 - INT32-C =

- Unsigned integer overflows that lead to buffer overflows

INT32-C – CWE-680 =

- Signed integer overflows that do not lead to buffer overflows

### CWE-191 and INT32-C

Union( CWE-190, CWE-191) = Union( INT30-C, INT32-C)

Intersection( INT30-C, INT32-C) == Ø

Intersection(CWE-191, INT32-C) =

- Underflow of signed integer operation

CWE-191 – INT32-C =

- Underflow of unsigned integer operation

INT32-C – CWE-191 =

- Overflow of signed integer operation

### CWE-190 and INT32-C

Union( CWE-190, CWE-191) = Union( INT30-C, INT32-C)

Intersection( INT30-C, INT32-C) == Ø

Intersection(CWE-190, INT32-C) =

- Overflow (wraparound) of signed integer operation

CWE-190 – INT32-C =

- Overflow of unsigned integer operation

INT32-C – CWE-190 =

- Underflow of signed integer operation

## Bibliography

| [[Dowd 2006](https://wiki.sei.cmu.edu/confluence/display/c/AA.+Bibliography#AA.Bibliography-Dowd06)] | Chapter 6, "C Language Issues" ("Arithmetic Boundary Conditions," pp. 211–223) |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| [[ISO/IEC 9899:2011](https://wiki.sei.cmu.edu/confluence/display/c/AA.+Bibliography#AA.Bibliography-ISO-IEC9899-2011)] | Subclause 6.5.5, "Multiplicative Operators"                  |
| [[Seacord 2013b](https://wiki.sei.cmu.edu/confluence/display/c/AA.+Bibliography#AA.Bibliography-Seacord2013)] | Chapter 5, "Integer Security"                                |
| [[Viega 2005](https://wiki.sei.cmu.edu/confluence/display/c/AA.+Bibliography#AA.Bibliography-Viega05)] | Section 5.2.7, "Integer Overflow"                            |
| [[Warren 2002](https://wiki.sei.cmu.edu/confluence/display/c/AA.+Bibliography#AA.Bibliography-Warren02)] | Chapter 2, "Basics"                                          |



------

[![SEI CERT C Coding Standard > SEI CERT C Coding Standard > button_arrow_left.png](https://wiki.sei.cmu.edu/confluence/download/attachments/87152044/button_arrow_left.png?version=1&modificationDate=1201021124000&api=v2)](https://wiki.sei.cmu.edu/confluence/pages/viewpage.action?pageId=87152211) [![SEI CERT C Coding Standard > SEI CERT C Coding Standard > button_arrow_up.png](https://wiki.sei.cmu.edu/confluence/download/attachments/87152044/button_arrow_up.png?version=1&modificationDate=1201021146000&api=v2)](https://wiki.sei.cmu.edu/confluence/pages/viewpage.action?pageId=87152052) [![SEI CERT C Coding Standard > SEI CERT C Coding Standard > button_arrow_right.png](https://wiki.sei.cmu.edu/confluence/download/attachments/87152044/button_arrow_right.png?version=1&modificationDate=1201021137000&api=v2)](https://wiki.sei.cmu.edu/confluence/pages/viewpage.action?pageId=87152254)
