# 说明

Lua BitOp是Lua 5.1 / 5.2的C扩展模块，它对数字增加了按位操作。

http://bitop.luajit.org/index.html 是项目的官方网站

本项目对此扩展的api及相关说明文档进行翻译



# 术语

- BitOp：Bit Operations Module，位运算模块



# API Functions -API函数

This list of API functions is not intended to replace a tutorial. If you are not familiar with the terms used, you may want to study the [Wikipedia article on bitwise operations](http://en.wikipedia.org/wiki/Bitwise_operation) first.

本API函数列表不能作为一个教程。如果您不熟悉使用的术语，您可能需要首先学习关于按位操作的维基百科文章。



## Loading the BitOp Module-加载BitOp模块

The suggested way to use the BitOp module is to add the following to the start of *every* Lua file that needs one of its functions:

推荐使用BitOp模块的方法是，在每一个需要调用其中任意函数的每个lua文件开头添加下面内容：

```
local bit = require("bit")
```

This makes the dependency explicit, limits the scope to the current file and provides faster access to the `bit.*` functions, too. It's good programming practice *not* to rely on the global variable `bit` being set (assuming some other part of your application has already loaded the module). The `require` function ensures the module is only loaded once, in any case.

这样能够让依赖性明显，作用域限制在当前文件并且对bit.*函数的访问更快速。这是良好的编程实践，不要依赖于已经设置的全局变量（假设应用的其余部分已经加载过模块）。require 函数确保在任何情况下模块只会加载一次。



## Defining Shortcuts-定义快捷方式

It's a common (but not a required) practice to cache often used module functions in locals. This serves as a shortcut to save some typing and also speeds up resolving them (only relevant if called hundreds of thousands of times).

这是个将常用的模块函数缓存到本地的一个常见（但不是必须）的方法。

这是一个捷径来少一些输入并且对解析他们也有一些提速（只有在调用数十万次时才能体现）。

```
local bnot = bit.bnot
local band, bor, bxor = bit.band, bit.bor, bit.bxor
local lshift, rshift, rol = bit.lshift, bit.rshift, bit.rol
-- etc...

-- Example use of the shortcuts:
local function tr_i(a, b, c, d, x, s)
  return rol(bxor(c, bor(b, bnot(d))) + a + x, s) + b
end
```

Remember that **and**, **or** and **not** are reserved keywords in Lua. They cannot be used for variable names or literal field names. That's why the corresponding bitwise functions have been named `band`, `bor`, and `bnot` (and `bxor` for consistency).

请记住**and**,**or**和**not**是Lua的保留关键字，他们不能被用作变量名或字面字段名。这也是为什么相应的按位运算函数被命名为band, bor，和bnot（bxor是为了保持一致性）

While we are at it: a common pitfall is to use `bit` as the name of a local temporary variable — well, don't! :-)

在这里我们使用`bit`作为一个临时的局部变量名，这是一个常见的陷阱，尽量别这样做。

## About the Examples-关于示例

The examples below show small Lua one-liners. Their expected output is shown after `-->`. This is interpreted as a comment marker by Lua so you can cut & paste the whole line to a Lua prompt and experiment with it.

下面演示了一些简短的单行lua示例，预期输出在`-->`符号后面展示。这个符号被Lua视为注释标记，所以你可以直接整行复制粘贴到lua提示符进行实验。

Note that all bit operations return *signed* 32 bit numbers ([rationale](http://bitop.luajit.org/semantics.html#range)). And these print as signed decimal numbers by default.

注意，所有的位运算都返回32位带符号数字（操作语义和原理）。并且默认的作为带符号数字打印。

For clarity the examples assume the definition of a helper function `printx()`. This prints its argument as an *unsigned* 32 bit hexadecimal number on all platforms:

为了清晰起见，示例假设定义了一个辅助函数`printx()`。这个函数在所有平台以32位无符号16进制整数打印它的参数。

```
function printx(x)
  print("0x"..bit.tohex(x))
end
```

## Bit Operations-位运算

### `y = bit.tobit(x)`

Normalizes a number to the numeric range for bit operations and returns it. This function is usually not needed since all bit operations already normalize all of their input arguments. Check the [operational semantics](http://bitop.luajit.org/semantics.html) for details.

将一个数字规格化到位运算范围的数值范围内。这个函数通常不需要，因为所有的位运算已经对他们的输入参数进行了规格化处理。更详细的信息查看操作语义和原理

```
print(0xffffffff)                --> 4294967295 (*)
print(bit.tobit(0xffffffff))     --> -1
printx(bit.tobit(0xffffffff))    --> 0xffffffff
print(bit.tobit(0xffffffff + 1)) --> 0
print(bit.tobit(2^40 + 1234))    --> 1234

```

(*) See the treatment of [hex literals](http://bitop.luajit.org/semantics.html#hexlit) for an explanation why the printed numbers in the first two lines differ (if your Lua installation uses a `double` number type).

如果你的Lua使用double数字类型安装，（*）部分请参阅十六进制文字以解释为什么前两行输出的数字不同

### `y = bit.tohex(x [,n])`

Converts its first argument to a hex string. The number of hex digits is given by the absolute value of the optional second argument. Positive numbers between 1 and 8 generate lowercase hex digits. Negative numbers generate uppercase hex digits. Only the least-significant 4*|n| bits are used. The default is to generate 8 lowercase hex digits.

将第一个参数转换位一个十六进制字符串，十六进制的位数由可选的第二个参数的绝对值决定，1和8之间的正数生成小写十六进制数。负数则生成大写的十六进制数。

```
print(bit.tohex(1))              --> 00000001
print(bit.tohex(-1))             --> ffffffff
print(bit.tohex(0xffffffff))     --> ffffffff
print(bit.tohex(-1, -8))         --> FFFFFFFF
print(bit.tohex(0x21, 4))        --> 0021
print(bit.tohex(0x87654321, 4))  --> 4321

```

### `y = bit.bnot(x)`

Returns the bitwise **not** of its argument.

返回参数值按位取反的结果

```
print(bit.bnot(0))            --> -1
printx(bit.bnot(0))           --> 0xffffffff
print(bit.bnot(-1))           --> 0
print(bit.bnot(0xffffffff))   --> 0
printx(bit.bnot(0x12345678))  --> 0xedcba987

```

### `y = bit.bor(x1 [,x2...])`

### `y = bit.band(x1 [,x2...])`

### `y = bit.bxor(x1 [,x2...])`

Returns either the bitwise **or**, bitwise **and**, or bitwise **xor** of all of its arguments. Note that more than two arguments are allowed.

分别返回参数的**或、与、非**运算结果。注意，参数个数可以大于两个。

```
print(bit.bor(1, 2, 4, 8))                --> 15
printx(bit.band(0x12345678, 0xff))        --> 0x00000078
printx(bit.bxor(0xa5a5f0f0, 0xaa55ff00))  --> 0x0ff00ff0
```

### `y = bit.lshift(x, n)`

### `y = bit.rshift(x, n)`

### `y = bit.arshift(x, n)`

Returns either the bitwise **logical left-shift**, bitwise **logical right-shift**, or bitwise **arithmetic right-shift** of its first argument by the number of bits given by the second argument.

返回第一个参数按第二参数指定的位数进行逻辑左移、逻辑右移、算术右移的结果。

Logical shifts treat the first argument as an unsigned number and shift in 0-bits. Arithmetic right-shift treats the most-significant bit as a sign bit and replicates it.

逻辑左移将第一个参数视为无符号数字并以0进行移位。算术右移将最大的位视为符号位并以它进行移位。

Only the lower 5 bits of the shift count are used (reduces to the range [0..31]).

移位计数器只有低五位被使用（范围限制在[0..31]）

```
print(bit.lshift(1, 0))              --> 1
print(bit.lshift(1, 8))              --> 256
print(bit.lshift(1, 40))             --> 256
print(bit.rshift(256, 8))            --> 1
print(bit.rshift(-256, 8))           --> 16777215
print(bit.arshift(256, 8))           --> 1
print(bit.arshift(-256, 8))          --> -1
printx(bit.lshift(0x87654321, 12))   --> 0x54321000
printx(bit.rshift(0x87654321, 12))   --> 0x00087654
printx(bit.arshift(0x87654321, 12))  --> 0xfff87654
```

### `y = bit.rol(x, n)`

### `y = bit.ror(x, n)`

Returns either the bitwise **left rotation**, or bitwise **right rotation** of its first argument by the number of bits given by the second argument. Bits shifted out on one side are shifted back in on the other side.
Only the lower 5 bits of the rotate count are used (reduces to the range [0..31]).

返回将第一个参数按第二个参数的给定值进行按位**左旋转**、按位**右旋转**后的结果。比特位从一端移除后从另一端移入。

```
printx(bit.rol(0x12345678, 12))   --> 0x45678123
printx(bit.ror(0x12345678, 12))   --> 0x67812345
```

### `y = bit.bswap(x)`

Swaps the bytes of its argument and returns it. This can be used to convert little-endian 32 bit numbers to big-endian 32 bit numbers or vice versa.

交换参数的字节并返回。这可以用于将32位的小端序数字转化为32位大端序数字，反之亦然。

```
printx(bit.bswap(0x12345678)) --> 0x78563412
printx(bit.bswap(0x78563412)) --> 0x12345678

```

## Example Program-示例程序

This is an implementation of the (naïve) *Sieve of Eratosthenes* algorithm. It counts the number of primes up to some maximum number.

这是一个找质数算法的实现。它统计质数的个数直到某个最大值为止。

A Lua table is used to hold a bit-vector. Every array index has 32 bits of the vector. Bitwise operations are used to access and modify them. Note that the shift counts don't need to be masked since this is already done by the BitOp shift and rotate functions.

Lua表用于保存位向量。每个数组索引都有32位的向量。位运算用于访问和修改它们。请注意，移位计数不需要被屏蔽，因为这已经由BitOp shift和rotate函数完成了。

```
local bit = require("bit")
local band, bxor = bit.band, bit.bxor
local rshift, rol = bit.rshift, bit.rol

local m = tonumber(arg and arg[1]) or 100000
if m < 2 then m = 2 end
local count = 0
local p = {}

for i=0,(m+31)/32 do p[i] = -1 end

for i=2,m do
  if band(rshift(p[rshift(i, 5)], i), 1) ~= 0 then
    count = count + 1
    for j=i+i,m,i do
      local jx = rshift(j, 5)
      p[jx] = band(p[jx], rol(-2, j))
    end
  end
end

io.write(string.format("Found %d primes up to %d\n", count, m))
```

Lua BitOp is quite fast. This program runs in less than 90 milliseconds on a 3 GHz CPU with a standard Lua installation, but performs more than a million calls to bitwise functions. If you're looking for even more speed, check out [LuaJIT](http://luajit.org/).

Lua BitOp很快。这个程序在一个3GHz CPU以标准方式安装的Lua上运行不足90毫秒，但执行了一百万次以上的位运算操作。如果你在需要更快的速度，查看LuaJIT。

## Caveats-注意事项

### Signed Results-有符号结果

Returning signed numbers from bitwise operations may be surprising to programmers coming from other programming languages which have both signed and unsigned types. But as long as you treat the results of bitwise operations uniformly everywhere, this shouldn't cause any problems.

在其他具有有符号和无符号类型的语言中，从位运算中返回有符号数可能对程序造成一些意想不到的结果。但是，只要将位操作的结果统一处理，这不应该引起任何问题。

Preferably format results with `bit.tohex` if you want a reliable unsigned string representation. Avoid the `"%x"` or `"%u"` formats for `string.format`. They fail on some architectures for negative numbers and can return more than 8 hex digits on others.

如果想要一个可靠的无符号的字符串表示最好使用`bit.tohex`对结果进行格式化。避免对`string.format`使用`"%x"`和`"%u"`。他们在一些结构上会导致负数操纵失败，在其他结构上可能返回朝贡8位的十六进制数。

You may also want to avoid the default number to string coercion, since this is a signed conversion. The coercion is used for string concatenation and all standard library functions which accept string arguments (such as `print()` or `io.write()`).

您也可以避免使用默认的字符串强制号码，因为这是一个有符号的转换。这种强制被用于连接和所有接受字符串参数的标准库函数（例如`print()`和`io.write()`）

### Conditionals-条件

If you're transcribing some code from C/C++, watch out for bit operations in conditionals. In C/C++ any non-zero value is implicitly considered as "true". E.g. this C code:

如果你正在从C/C++中改写一些代码，小心位运算的条件。在C/C++中任何非0的值都隐式的被视作“true”。例如这段C代码：

`  if (x & 3) ...`
must not be turned into this Lua code:

一定不要改写成这样的Lua代码：

`  if band(x, 3) then ... -- *wrong!*`

In Lua all objects except `nil` and `false` are considered "true". This includes all numbers. An explicit comparison against zero is required in this case:

在Lua的所有对象中，只有`nil`和`false`被视为“true”。这包括了所有的数字。这种情况下需要显示的和0进行比较。

`  if band(x, 3) ~= 0 then ... -- *correct!*`

### Comparing Against Hex Literals-比较16进制字面量

Comparing the results of bitwise operations (*signed* numbers) against hex literals (*unsigned* numbers) needs some additional care. The following conditional expression may or may not work right, depending on the platform you run it on:

对位运算的结果（有符号数）和十六进制字面量（无符号数）进行比较时需要一些额外的注意。下列的条件表达式可能不能正常运行，取决于运行的平台：

`  bit.bor(x, 1) == 0xffffffff`
E.g. it's never true on a Lua installation with the default number type. Some simple solutions:

例如，在一个以默认数字类型安装的Lua上，它永远不会为“true”。一些简单的解决方法：

- Either never use hex literals larger than 0x7fffffff in comparisons:

  在比较的时候永远不要使用比0x7fffffff大的十六进制字面量：

  `  bit.bor(x, 1) == -1`

- Or convert them with `bit.tobit()` before comparing:

  在比较前使用`bit.tobit()`将他们进行转换：

  `  bit.bor(x, 1) == bit.tobit(0xffffffff)`

- Or use a generic workaround with `bit.bxor()`:

  或者借由`bit.bxor()`使用一个通用的解决方案：

  `  bit.bxor(bit.bor(x, 1), 0xffffffff) == 0`

- Or use a case-specific workaround:

  或者使用一个特定情况的解决方案：

  `  bit.rshift(x, 1) == 0x7fffffff`


# Operational Semantics and Rationale-操作语义和原理

Lua uses only a single number type which can be redefined at compile-time. By default this is a `double`, i.e. a floating-point number with 53 bits of precision. Operations in the range of 32 bit numbers (and beyond) are exact. There is no loss of precision, so there is no need to add an extra integer number type. Modern desktop and server CPUs have fast floating-point hardware — FP arithmetic is nearly the same speed as integer arithmetic. Any differences vanish under the overhead of the Lua interpreter itself.

Lua只使用一种数字类型，它可以在编译的时候重新定义。默认的是`double`，即一个53位精度的浮点数。32位数（或更大）范围内的操作是准确的。并且没有精度丢失，所以没有必要增加一个额外的整数类型。现在涿县和服务器CPU拥有快速浮点运算硬件使得浮点算术的运算几乎和整数算术的速度一样快。任何的差异在Lua解释器本身的开销下都将消失。

Even today, many embedded systems lack support for fast FP operations. These systems benefit from compiling Lua with an integer number type (with 32 bits or more).

即使在今天，很多嵌入式系统都缺乏对快速浮点运算的支持，这些系统使用整数类型（32位或更多）编译Lua会更好。

The different possible number types and the use of FP numbers cause some problems when defining bitwise operations on Lua numbers. The following sections define the operational semantics and try to explain the rationale behind them.

不同的数字类型和浮点数的使用在为Lua数字定义位运算的时候会造成一些问题，下面的内容定义运算的语义并尝试解释背后的原理。

## Input and Output Ranges-输入输出范围

- Bitwise operations cannot sensibly be applied to FP numbers (or their underlying bit patterns). They must be converted to integers before operating on them and then back to FP numbers.

  位运算不能直接应用到浮点数上（或者他们的底层位模式）。他们必须在运算前被转换位整数，然后再转换回浮点数。

- It's desirable to define semantics that work the same across all platforms. This dictates that **all operations are based on** the common denominator of **32 bit integers**.

  最好定义所有平台上工作相同的语义。这规定**所有操作均基于****32位整数**的相同标准。

- The `float` type provides only 24 bits of precision. This makes it unsuitable for use in bitwise operations. Lua BitOp refuses to compile against a Lua installation with this number type.

  `float`类型只提供了24位精度，这使得它不适合应用再位运算中。Lua BitOP拒绝使用这个数字类型进行编译安装。

- Bit operations only deal with the underlying bit patterns and generally ignore signedness (except for arithmetic right-shift). They are commonly displayed and treated like unsigned numbers, though.

  位运算只处理底层位模式，并且通常忽略符号位（除了算术右移）。虽然他们被惯常的表示并且和无符号数一样对待。

- But the Lua number type must be signed and may be limited to 32 bits. Defining the result type as an unsigned number would not be cross-platform safe. All bit operations are thus defined to **return results in the range of signed 32 bit numbers** (converted to the Lua number type).

  但是Lua的数字类型必须是有符号的，并且被限制到32位。将结果类型定义为一个无符号数对于跨平台来说是不安全的。所有的位运算也因此被定义为**返回结果在32位的有符号数字的范围内**（被转换为Lua数字类型）。

- **Hexadecimal literals are** treated as **unsigned numbers** by the Lua parser before converting them to the Lua number type. This means they can be out of the range of signed 32 bit integers if the Lua number type has a greater range. E.g. 0xffffffff has a value of 4294967295 in the default installation, but may be -1 on embedded systems.

  Lua解释器在将**十六进制字面量**转换为Lua数字类型前将他们视为**无符号数**。也意味着，如果Lua的数字类型又一个更大的范围，他们可能会超出32位有符号数的范围。例如，0xffffffff在默认安装的方式下值为4294969295，但是在嵌入式系统中可能是1。

- It's highly desirable that hex literals are treated uniformly across systems when used in bitwise operations. **All bit operations accept arguments in the signed or the unsigned 32 bit range** (and more, see below). Numbers with the same underlying bit pattern are treated the same by all operations.

  非常希望十六进制字面量在跨平台使用位运算的时候能被相同的处理。所有的位运算都接受32位的有符号或无符号数字（或者更多，往下看）。相同位模式的数字在所有的运算中都被同样的对待。

## Modular Arithmetic-模运算

Arithmetic operations on n-bit integers are usually based on the rules of [modular arithmetic](http://en.wikipedia.org/wiki/Modular_arithmetic) modulo 2^n. Numbers wrap around when the mathematical result of operations is outside their defined range. This simplifies hardware implementations and some algorithms actually require this behavior (like many cryptographic functions).

n位整数的模运算通常基于模数为2^n的的原则。当运算的数学计算结果超出了定义的范围时数字将被折叠。这简化了硬件实现并且一些算法通常也要求这个行为（比如很多加密函数）

E.g. for 32 bit integers the following holds: `0xffffffff + 1 = 0`

例如，对于32位整数： `0xffffffff + 1 = 0`

**Arithmetic modulo 232** is trivially available if the Lua number type is a 32 bit integer. Otherwise normalization steps must be inserted. Modular arithmetic should work the same across all platforms as far as possible:

如果Lua数字类型是32位整形那么**算术模数232**是可用的。否则必须进行标准化步骤。模运算应该在所有平台上尽可能的保持一致。

- For the default number type of `double`, **arguments can be in the range of ±251** and still be safely normalized across all platforms by taking their least-significant 32 bits. The limit is derived from the way doubles are converted to integers.

  对于默认数字类型`double`的，**参数可以在±251范围内**，并且通过取低32位的方式安全的进行跨平台标准化。这个限制是由doubles转换到integers的方式导出的

- The function `bit.tobit` **can be used to explicitly normalize numbers** to implement **modular addition or subtraction**. E.g. `bit.tobit(0xffffffff + 1)` returns 0 on all platforms.

  函数`bit.tobit`可以显示的标准化数字以实现模加减。例如：`bit.tobit(0xffffffff + 1)`在所有平台返回0

- The limit on the argument range implies that modular multiplication is usually restricted to multiplying already normalized numbers with small constants. FP numbers are limited to 53 bits of precision, anyway. E.g. (230+1)2 does not return an odd number when computed with doubles.

  参数范围的限制意味着模乘通常被限制为将已经归一化的数乘以小常数。浮点数无论如何都被限制为53位精度。例如(2^30+1)^2在使用doubles计算的时候不会返回一个基数。

BTW: The `tr_i` function shown [here](http://bitop.luajit.org/api.html#shortcuts) is one of the non-linear functions of the (flawed) MD5 cryptographic hash and relies on modular arithmetic for correct operation. The result is fed back to other bitwise operations (not shown) and does not need to be normalized until the last step.

顺便说一下：定义快捷方式一节中出现的`tr_i`函数是（有缺陷）MD5加密哈希中的一个非线性函数，并且依赖模运算的正确性。结果被返回到其他按位操作（未示出），并且在最后一步之前不需要被归一化。

## Restricted and Undefined Behavior - 限制和未定义的行为

The following rules are intended to give a precise and useful definition (for the programmer), yet give the implementation (interpreter and compiler) the maximum flexibility and the freedom to apply advanced optimizations. It's strongly advised *not* to rely on undefined or implementation-defined behavior.

下面的规则旨在给出一个精确和游泳的定义（对程序员），但给予实现（解释器和编译器）最大的灵活性和自由度以应用到高级的优化。强烈建议不要依赖与未定义的活实现定义的行为。

- All kinds of floating-point numbers are acceptable to the bitwise operations. None of them cause an error, but some may invoke undefined behavior:

  所有的浮点数都对与位运算都是可接受的。他们不会出现错误，但可能出现未定义的行为。

  - -0 is treated the same as +0 on input and is never returned as a result.

    -0和+0在输入中被视为相同的，并且也绝不会作为结果返回。

  - Passing **±Inf, NaN or numbers outside the range of ±251** as input yields an **undefined** result.

    传递 **±Inf, NaN 或者超出±251范围的数字**作为输入将产生未定义的结果。

  - **Non-integral numbers** may be rounded or truncated in an **implementation-defined** way. This means the result could differ between different BitOp versions, different Lua VMs, on different platforms or even between interpreted vs. compiled code (as in [LuaJIT](http://luajit.org/)).
    Avoid passing fractional numbers to bitwise functions. Use `math.floor()` or `math.ceil()` to get defined behavior.

    非整数可能以实现定义的方式四舍五入或者截断。这意味着不同版本的BitOp，不同的LuaVm和不同的平台，甚至在解释或编译代码（如在LuaJIT中）之间都是不同的。

- Lua provides **auto-coercion of string arguments** to numbers by default. This behavior is **deprecated** for bitwise operations.

  Lua默认提供提供了从字符串参数到数字的自动类型。这个行为在位运算中是无效的。