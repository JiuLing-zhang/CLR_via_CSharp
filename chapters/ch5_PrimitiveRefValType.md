# 第 5 章 基元类型、引用类型和值类型

本章内容：
* <a href="#5_1">编程语言的基元类型</a>
* <a href="#5_2">引用类型和值类型</a>
* <a href="#5_3">值类型的装箱和拆箱</a>
* <a href="#5_4">对象哈希码</a>
* <a href="#5_5">dynamic 基元类型</a>

本章将讨论 Microsoft .NET Framework 开发人员经常要接触的各种类型。所以开发人员都应熟悉这些类型的不同行为。我首次接触 .NET Framework 时没有完全理解基元类型、引用类型和值类型的区别，造成在代码中不知不觉引入 bug 和性能问题。通过解释类型之间的区别，希望开发人员能避免我所经历的麻烦，同时提高编码效率。

## <a name="5_1">编程语言的基元类型</a>

某些数据类型如此常用，以至于许多编译器允许代码以简化语法来操纵它们。例如，可用以下语法分配一个整数：  

```C#
System.Int32 a = new System.Int32();
```

但你肯定不愿意用这种语法声明并初始化整数，它实在是太繁琐了。幸好，包括 C# 在内的许多编译器都允许换用如下所示的语法：

```C#
int a = 0;
```

这种语法不仅增强了代码可读性，生成的 IL 代码还与使用 `System.Int32` 生成的 IL 代码完全一致。编译器直接支持的数据类型称为 **基元类型**(primitive type)。基元类型直接映射到 Framework 类库(FCL)中存在的类型。例如，C# 的 `int` 直接映射到 `System.Int32` 类型。因此，以下 4 行代码都能正确编译，并生成完全相同的 IL：

```C#
  int            a = 0;                   // 最方便的语法
  System.Int32   a = 0;                   // 方便的语法
  int            a = new int();           // 不方便的语法
  System.Int32   a = new System.Int32();  // 最不方便的语法
```

表 5-1 列出的 FCL 类型在 C# 中都有对应的基元类型。只要是符合公共语言规范(CLS)的类型，其他语言都提供了类似的基元类型。但是，不符合 CLS 的类型语言就不一定要支持了。

表 5-1 C# 基元类型与对应的 FCL 类型
|C#基元类型|FCL类型|符合CLS|说明|
|:---:|:---:|:----:|:---:|
|`sbyte`|`System.SByte`|否|有符合 8 位值|
|`byte`|`System.Byte`|是|无符号 8 位值|
|`short`|`System.Int16`|是|有符号 16 位值|
|`ushort`|`System.UInt16`|否|无符号 16 位值|
|`int`|`System.Int32`|是|有符号 32 位值|
|`uint`|`System.UInt32`|否|无符号 32 位值|
|`long`|`System.Int64`|是|有符号 64 位值|
|`ulong`|`System.UInt64`|否|无符号 64 位值|
|`char`|`System.Char`|是|16位 Unicode 字符(`char` 不像在非托管 C++ 中那样代表一个 8 位置)|
|`float`|`System.Single`|是|IEEE 32 位浮点值|
|`double`|`System.Double`|是|IEEE 64 位浮点值|
|`bool`|`System.Boolean`|是|`true`/`false` 值|
|`decimal`|`System.Decimal`|是|128 位高精度浮点值，常用于不容许舍入误差的金融计算。 128 位中， 1 位是符号，96位是值本身(*N*)，8位是比例引子(*k*)。 `decimal` 实际值是 ±*N*×10<sup>k</sup>，其中 -28<= *k* <=0。其余位没有使用|
|`string`|`System.String`|是|字符数组|
|`object`|`System.Object`|是|所有类型的基类型|
|`dynamic`|`System.Object`|是|对于 CLR， `dynamic` 和 `object` 完全一致。但 C# 编译器允许使用简单的语法让 `dynamic` 变量参与动态调度。详情参见本章最后的 5.5 节“`dynamic` 基元类型”|

从另一个角度，可认为 C# 编译器自动假定所有源代码文件都添加了以下 `using` 指令(参考第 4 章)：

```C#
using sbyte  = System.SByte;
using byte   = System.Byte;
using short  = System.Int16;
using ushort = System.UInt16;
using int    = System.Int32;
using uint   = System.UInt32;
···
```

C# 语言规范称：“从风格上说，最好是使用关键字，而不是使用完整的系统类型名称。”我不同意语言规范：我情愿使用 FCL 类型名称，完全不用基元类型名称。事实上，我希望编译器根本不提供基元类型名称，而是强迫开发人员使用 FCL 类型名称。理由如下。

1. 许多开发人员纠结于是用 `string` 还是 `String` 。由于 C# 的 `using`(一个关键字)直接映射到 `System.String`(一个 FCL 类型)，所以两者没有区别，都可使用。类似地，一些开发人员说应用程序在 32 位操作系统上运行， `int` 代表 32 位整数；在 64 位操作系统上运行， `int` 代表 64 位整数。这个说法完全错误。C# 的 `int` 始终映射到 `System.Int32`，所以不管在什么操作系统上运行，代表的都是 32 位整数。如果程序员习惯在代码中使用 `Int32`，像这样的误解就没有了。

2. C#的 `long` 映射到 `System.Int64` ，但在其他编程语言中，`long` 可能映射到 `Int16` 或 `Int32`。例如， C++/CLI 就将 `long` 视为 `Int32`。习惯于用一种语言写程序的人在看用另一种语言写程序的人在看用另一种语言写的源代码时，很容易错误理解代码意图。事实上，大多数语言甚至不将 `long` 当作关键字，根本不编译使用了它的代码。

3. FCL 的许多方法都将类型名作为方法名的一部分。例如， `BinaryReader` 类型的方法包括 `ReadBoolean`，`ReadInt32`，`ReadSingle` 等；而 `System.Convert` 类型的方法包括 `ToBoolean`，`ToInt32`，`ToSingle` 等。以下代码虽然语法没问题，但包含 `float` 的那一行显得很别扭，无法一下子判断该行的正确性：  

    ```C#
    BinaryReader br = new BinaryReaser(...);
    float val    = br.ReadSingle();    // 正确，但感觉别扭
    Single val   = br.ReadSingle();    // 正确，感觉自然
    ```

4. 平时只用 C# 的许多程序员逐渐忘了还可以用其他语言写面向 CLR 的代码，“C#主义”逐渐入侵类库代码。例如， Microsoft 的 FCL 几乎是完全用 C# 写的，FCL 团队向库中引入了像 `Array` 的 `GetLongLength` 这样的方法。该方法返回 `Int64` 值。这种值在 C# 中确实是 `long`，但在其他语言(比如 C++/CLI)中不是。另一个例子是 `System.Linq.Enumerable` 的 `LongCount` 方法。

考虑到所有这些原因，本书坚持使用 FCL 类型名称。

在许多编程语言中，以下代码都能正确编译并运行：

```C#
Int32 i = 5;   // 32 位值
Int64 l = i;   // 隐式转型为 64 位值
```

但根据第 4 章对类型转换的讨论，你或许认为上述代码无法编译。毕竟，`System.Int32` 和 `System.Int64` 是不同的类型，相互不存在派生关系。但事实上，你会欣喜地发现 C# 编译器正确编译了上述代码，运行起来也没有问题。这是为什么呢？原因是 C# 编译器非常熟悉基元类型，会在编译代码时应用自己的特殊规则。也就是说，编译器能识别常见的编程模式，并生成必要的 IL，使写好的代码能像预期的那样工作。具体地说，C# 编译器支持与类型转换、*字面值*以及操作符有关的模式。接着的几个例子将对它们进行演示。
> 即 literal ，也称为直接量或文字常量。本书将采用“字面值”这一译法。 —— 译注

首先，编译器能执行基元类型之间的隐式或显式转型，例如：

```C#
Int32 i = 5;          // 从 Int32 隐式转型为 Int32
Int64 l = i;          // 从 Int32 隐式转型为 Int64
Single s = i;         // 从 Int32 隐式转型为 Single
Byte b = (Byte) i;    // 从 Int32 显式转型为 Byte
Int16 v = (Int16) s;  // 从 Single 显式转型为 Int16
```

只有在转换“安全”的时候，C#才允许隐式转型。所谓“安全”，是指不会发生数据丢失的情况，比如从 `Int32` 转换为 `Int64`。但如果可能不安全，C# 就要求显示转型。对于数值类型，“不安全”意味着转换后可能丢失精度或数量级。例如， `Int32` 转换为 `Byte` 要求显式转型，因为大的 `Int32` 数字可能丢失精度；`Single` 转换为 `Int16` 也要求显式转型，因为 `Signle` 能表示比 `Int16` 更大数量级的数字。

注意，不同编译器可能生成不同代码来处理这些转型。例如，将值为 6.8 的 `Single` 转型为 `Int32`，有的编译器可能生成代码对其进行截断(向下取整)，最终将 6 放到一个 `Int32` 中；其他编译器则可能将结果向上取整为 7 。顺便说一句，C# 总是对结果进行截断，而不进行向上取整。要了解 C# 对基元类型进行转型时的具体规则，请参加 C# 语言规范的 “转换” 一节。

除了转型，基本类型还能写成字面值(literal)。字面值可被看成是类型本身的实例，所以可像下面为实例(123 和 456)调用实例方法：

```C#
Console.WriteLine(123.ToString() + 456.ToString());     // "123456"
```

另外，如果表达式由字面值构成，编译器在编译时就能完成表达式求值，从而增强应用程序性能:

```C#
Boolean found = false;   // 生成的代码将 found 设为 0
Int32 x = 100 + 20 + 3； // 生成的代码将 x 设为 123
String s ="a " + "bc";   // 生成的代码将 s 设为 "a bc"
```

最后，编译器知道如何和以什么顺序解析代码中的操作符(比如 `+`，`-`，`*`，`/`，`%`，`&`，`^`，`|`，`==`，`!=`，`>`，`<`，`>=`，`<=`，`<<`，`>>`，`~`，`!`，`++`，`--` 等):

```C#
Int32 x = 100;                     // 赋值操作符
Int32 y = x + 123;                 // 加和赋值操作符
Boolean lessThanFifty = (y < 50);  // 小于和赋值操作符
```

### **checked 和 unchecked 基元类型操作符**

对基元类型执行的许多算术运算都可能造成溢出：

```C#
Byte b = 100;
b = (Byte) (b + 200);  // b 现在包含 44 (或者十六进制 2C)
```

> 重要提示 执行上述算术运算时，第一步要求所有操作数都扩大为 32 位值(或者 64 位值，如果任何操作数需要超过 32 位来表示的话)。所以 b 和 200(两个都不超过 32 位)首先转换成 32 位值，然后加到一起。结果是一个 32 位值(十进制 300，或十六进制 12C)。该值在寄回变量 b 前必须转型为 `Byte` 。C# 不隐式执行这个转型操作，这正是第二行代码需要强制转型 `Byte` 的原因。

溢出大多数时候是我们不希望的。如果没有检测到这种溢出，会导致应用程序行为失常。但极少数时候(比如计算哈希值或者校验和)，这种溢出不仅可以接受，还是我们希望的。

不同语言处理溢出的方式不同。C 和 C++ 不将溢出视为错误，允许值*回滚(wrap)*;应用程序将“若无其事”地运行。相反， Microsoft Visual Basic 总是将溢出视为错误，并在检测到到溢出时抛出异常。
> 所谓"回滚"，是指一个值超过了它的类型所允许的最大值，从而”回滚“到一个非常小的、负的或者未定义的值。 wrap 是 wrap-around 的简称。——译注

CLR 提供了一些特殊的 IL 指令，允许编译器选择它认为最恰当的行为。 CLR 有一个 `add` 指令，作用是将两个值相加，但不执行溢出检查。还有一个 `add.ovf` 指令，作用也是将两个相加，但会在发生溢出时抛出 `System.OverflowException` 异常。除了用于加法运算的 IL 指令，CLR 还为减、乘和数据转换提供了类似的 IL 指令，分别是 `sub/sub.ovf,mul/mul.ovf` 和 `conv/conv.ovf`。

C# 允许程序员自己决定如何处理溢出。溢出检查默认关闭。也就是说，编译器生成 IL 代码时，将自动使用加、减、乘以及转换指令的无溢出检查版本。结果是代码能更快地运行 —— 但开发人员必须保证不发生溢出，或者代码能预见到溢出。

让 C# 编译器控制溢出的一个办法是使用 `/checked+` 编译器开关。该开关指示编译器在生成代码时，使用加、减、乘和转换指令的溢出检查版本。这样生成的代码在执行时会稍慢一些，因为 CLR 会检查这些运算，判断是否发生溢出。如果发生溢出， CLR 抛出 `OverflowException` 异常。

除了全局性地打开或关闭溢出检查，程序员还可在代码的特定区域控制溢出检查。C# 通过 `checked` 和 `unchecked` 操作符来提供这种灵活性。下面是使用了 `unchecked` 操作符的例子：

```C#
UInt32 invalid = unchecked( (UInt32) (-1));   // OK
```

下例则使用了 `checked` 操作符：

```C#
Byte b = 100;
b = checked((Byte) (b + 200));    // 抛出 OverflowException 异常
```

在这个例子中， `b` 和 `200` 首先转换成 32 位值，然后加到一起，结果是 300。然后，因为显式转型的存在， `300` 被转换成一个 `Byte`，这造成 `OverflowException` 异常。`Byte` 在 `checked` 操作符外部转型则不会发生异常：  

```C#
b = (Byte) checked(b + 200);    // b 包含 44； 不会抛出 OverflowException 异常
```

除了 `checked` 和 `unchecked` 操作符，C# 还支持 `checked` 和 `unchecked` 语句，它们造成一个块中的所有表达式都进行或不进行溢出检查：

```C#
checked {                   // 开始 checked 块
  Byte b = 100;
  b = (Byte) (b + 200);     // 该表达式会进行溢出检查
}                           // 结束 checked 块
```

事实上，如果使用了 `checked` 语句块，就可将 += 操作符用于 `Byte`，稍微简化一下代码：  

```C#
checked {             // 开始 checked 块
  Byte b = 100;
  b += 200;           // 该表达式会进行溢出检查
}                     // 结束 checked 块
```

> 重要提示 由于 `checked` 操作符和 `checked` 语句唯一的作用就是决定生成哪个版本的加、减、乘和数据转换 IL 指令，所以在 `checked` 操作符或语句中调用方法，不会对该方法造成任何影响，如下例所示：

```C#
checked {
  // 假定 SomeMethod 试图把 400 加载到一个 Byte 中
  SomeMethod(400);
  // SomeMethod 可能会、也可能不会抛出 OverflowException 异常
  // 如果 SomeMethod 使用 checked 指令编程，就可能会抛出异常
  // 但这和当前的 checked 语句无关
}
```

根据我的经验，许多计算都会产生令人吃惊的结果。这一般是由于无效的用户输入，但也可能是由于系统的某个部分返回了程序员没有预料到的值。所以我对程序员有以下建议。

1. 尽量使用有符号数值类型(比如 `Int32` 和 `Int64`)而不是无符号数值类型(比如 `UInt32` 和 `UInt64`)。这允许编译器检测更多的上溢/下溢错误。除此之外，类库多个部分(比如 `Array` 和 `String` 的 `Length` 属性)被硬编码为返回有符号的值。这样在代码中四处移动这些值时，需要进行的强制类型转换就少了。较少的强制类型转换使代码更整洁，更容易维护。除此之外，无符号数值类型不符合 CLS。

2. 写代码时，如果代码可能发生你不希望的溢出(可能是因为无效的输入，比如要求使用最终用户或客户机提供的数据)，就把这些代码放到 `checked` 块中。同时捕捉 `OverflowException`，得体地从错误中恢复。

3. 写代码时，将允许发生溢出的代码显式放到 `unchecked` 块中，比如在计算校验和时。

4. 对于没有使用 `checked` 或 `unchecked` 的任何代码，都假定你希望在发生溢出时抛出一个异常，比如在输入已知的前提下计算一些东西(比如质数)，此时的溢出应被计为 bug。

开发应用程序时，打开编译器的 `/checked+` 开关进行调试性生成。这样系统会对没有显式标记 `checked` 或 `unchecked` 的代码进行溢出检查，所以应用程序运行起来会慢一些。此时一旦发生异常，就可以轻松检测到，而且能及时修正代码中的 bug。但是，为了正式发布而生成应用程序时，应使用编译器的 `/checked-` 开关，确保代码能更快运行，不会产生溢出异常。要在 Microsoft Visual Studio 中更改 Checked 设置，请打开项目的属性页，点击”生成“标签，单击”高级“，再勾选”检查运算上溢/下溢“，如图 5-1 所示。  
![5_1](../resources/images/5_1.png)  
图 5-1 在 Visual Studio 的”高级生成设置“对话框中指定编译器是否检查溢出  

如果应用程序能容忍总是执行 `checked` 运算而带来的轻微性能损失，建议即使是为了发布而生成应用程序，也用 `/checked` 命令行开关进行编译，这样可防止应用程序在包含已损坏的数据(甚至可能是安全漏洞)的前提下继续运行。例如，通过乘法运算来计算数组索引时，相较于因为数学运算的 *”回滚“* 而访问到不正确的数组元素，抛出 `OverflowException` 异常才是更好的做法。
> 乘法运算可能产生一个较大的值，超出数组的索引范围。参见上一条关于 ”wrap“ 的注释。 —— 译注

>> 重要提示 `System.Decimal` 是非常特殊的类型。虽然许多编程语言(包括 C# 和 Visual Basic)将 `Decimal` 视为基元类型，但 CLR 不然。这意味着 CLR 没有知道如何处理 `Decimal` 值的 IL 指令。在文档中查看 `Decimal` 类型，可以看到它提供了一系列 `pulbic static` 方法，包括 `Add`，`Subtract`，`Multiply`，`Divide` 等。此外， `Decimal` 类型还为 `+`，`-`，`\*`，`/`等提供了操作重载方法。

>> 编译使用了 `Decimal` 值的程序时，编译器会生成代码来调用 `Decimal` 的成员，并通过这些成员来执行实际运算。这意味着 `Decimal` 值的处理速度慢于 CLR 基元类型的值。另外，由于没有相应的 IL 指令来处理 `Decimal` 值，所以 `checked` 和 `unchecked` 操作符、语句以及编译器开关都失去了作用。如果对 `Decimal` 值执行的运算是不安全的，肯定会抛出 `OverflowException` 异常。

>> 类似地，`System.Numerics.BigInteger` 类型也在内部使用 `UInt32` 数组来表示任意大的整数，它的值没有上限和下限。因此，对 `BigInteger` 执行的运算永远不会造成 `OverflowException` 异常。但如果值太大，没有足够多的内存来改变数组大小，对 `BigInteger` 的运算可能抛出 `OutOfMemoryException` 异常。

## <a name="5_2">5.2 引用类型和值类型</a>

CLR 支持两种类型：**引用类型**和**值类型**。虽然 FCL 的大多数类型都是引用类型，但程序员用得最多的还是值类型。引用类型总是从托管堆分配， C# 的 `new` 操作符返回对象内存地址 —— 即指向对象数据的内存地址。