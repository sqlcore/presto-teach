# Presto类型系统浅入

[返回首页](../README.md)

---

## 类型分类

这里我们可以先了解一下[Presto类型系统初探](https://zhuanlan.zhihu.com/p/55299409)中的类型分布。

![](vx_images/226360128042687.png)

### 基本类型

*  BooleanType：表示布尔值，通常用于条件和逻辑运算。
*  IntegerType：表示整数值，对应于 Java 的 `int`。
*  BigintType：表示更大范围的整数，对应于 Java 的 `long`。
*  SmallintType：表示较小范围的整数，对应于 Java 的 `short`。
*  TinyintType：表示最小范围的整数，通常用于节省空间的场景。
*  DoubleType：表示双精度浮点数，对应于 Java 的 `double`。
*  RealType：表示单精度浮点数，对应于 Java 的 `float`。

### 字符串和二进制类型

*  VarcharType：表示可变长度的字符串。
*  CharType：表示固定长度的字符串。
*  VarbinaryType：用于存储二进制数据，类似于 Java 的 `byte[]`。

### 日期和时间类型

*  DateType：表示日期。
*  TimeType：表示时间。
*  TimestampType：表示时间戳，即日期和时间的组合。
*  TimestampWithTimeZoneType：表示带有时区信息的时间戳。

### 复杂类型

*  ArrayType：表示数组类型，可以包含一系列同类型的元素。
*  MapType：表示映射类型，包含键值对。
*  RowType：表示结构体或行类型，可以包含多个不同类型的字段。

### 特殊类型

*  JsonType：用于处理 JSON 数据。
*  IPaddressType：专门用于存储 IP 地址。
*  UUIDType：用于处理 UUID。

## 其他关键概念 Slice Block Type

在开始学习这些类型前，我们需要知道 Presto 一些特殊的设计，以及他们之间的关系。

### 底层类型 Slice

Slice 是 Presto 自定义的一种数据结构，主要用于高效的处理二进制数据或原始字节序列。它是 Presto 中处理字符串、二进制数据和其他原始字节序列的关键组件。

片面的理解，可以认为在 Presto 中处理字符串类型时，一般会用到 Slice。

但是为什么要搞一个 Slice 类型呢？深入一点的理解，主要区别如下：

Slice 是一个更底层的数据结构，用于字节级操作，而 Type 类型是更高层次的抽象，代表 SQL 数据类型及其在 Presto 中的行为。在某些 Type 类型的内部实现中，如 VarcharType，Slice 被用作存储实际数据的底层机制。但是 Type 类型还包括了与该数据类型相关的其他逻辑和操作，如数据比较、哈希计算等。
 

### 数据的载体 Block

我们知道 Presto 是一个 OLAP 引擎，是可以对数据进行列式操作的，这里引用以下某本书里的一张图片。

Page 是 Presto 中处理的最小数据单元。一个 Page 对象包含多个 Block 对象，而每个 Block 对象是一个字节数组，存储一个字段的若干行。多个 Block 横切的一行是真实的一行数据。

![](vx_images/485282891372733.png)


Block 可以按照另一种更高一层的数据抽象来理解，里面同样有获取数据、数据大小和结构、数据状态，为了我们方便构建数据，还定义了一个 BlockBuilder。

注意，这里有一个接口叫 UncheckedBlock 是 Block 的一个变体，提供了一些无边界检查的数据访问方法。这意味着这些方法不会检查索引的有效性，因此它们比标准的 Block 方法更快，但使用时需要更小心。

这里有个疑问，为什么要设计 Block 呢？  

1. 第一点，这个我们都知道，为了方便列式操作
2. 第二点，是方便内存管理，通过 Block 和 BlockBuilder，Presto 能够有效地管理内存，特别是在处理大量数据时。
3. 第三点，是性能优化，Block 结构允许 Presto 对数据访问和处理进行优化，特别是通过 UncheckedBlock 提供了一种更快的数据访问方式

注意，这里介绍的 Block 与 UncheckedBlock，可以理解为基础接口，还有其他类型比如 Array、Map、Row 等等与类型对应的 Block，这块我们在介绍了类型之后再来介绍它们。

![](vx_images/437568052647907.png)

### Slice 与 Block 的关系

可能我这个词语不太对，我是这么理解的，Slice 是类型，Block 是载体。

比如 Block 中，可能有 int，bool，以及 slice 等类型的数据，再比如一个 Block 可能包含多个字符串数据，这些字符串数据是以 Slice 的形式存储的。

### Type 与 Block 的关系

在 Presto 中，Block 和各种 Type（如 IntegerType、VarcharType、JsonType 等）之间的关系是数据存储和表示的核心。这些 Type 和 Block 结构共同构成了 Presto 处理数据的基础。

简单的理解就是，Type 代表数据类型，Block 代表如何存储、访问以及处理数据。

## Presto 为什么要重新定义一套类型

第一点，也是比较好理解的一点就是，虽然我们可以直接延用 Java 中的数据类型，但是在面向 SQL 建模时，原始的 Java 的数据类型可能不够。但是单纯讲无法满足，我们不能很好的理解，这里举几个例子。

在 SQL 类型中通常有不同的范围和精度要求。例如 SQL 中需要的 DECIMAL 类型需要支持非常大的数字和精确的小数，这在 Java 的原生类型中无法直接表示，这里说的无法表示是说无法用直接原生的 Java 类型。还有就是时间类型，在 SQL 中一般会有多种时间类型的定义，如 `DATE`, `TIME`, `TIMESTAMP`，以及 SQL 中的 NULL 处理与 Java 中的 NULL 也有所不同，这些类型在 Java 中没有直接对应的类型，或者如果直接使用 Java 原生类型会与 SQL 标准不匹配，一般面对这些类型都会重新设计封装。

第二点，在面向 SQL 操作时，还有一个重要的概念 `元数据` 或者也可以叫 `schema`，因为 Presto 会借助元数据来对类型进行比如，`是否可以排序`、`是否可以比较`、`是否无边界`，等等操作。

第三点，因为 Presto 是一个 OLAP 引擎，是需要支持列存数据的查询。那么在列存中，会有 Block 的概念，重新设计一种类型也是方便 Block 的操作。

结合以上三点，所以 Presto 重新设计了一套类型系统。这个类型系统有个很重要的接口 `Type` 里面定义所有关键操作，接下来我们来看看这个接口的源码以及相关设计思想。

## Type 接口

接下来开始正经介绍：

Presto 的类型基本都基于这个接口，在这个接口中设计了很多重要的方法。

源码位置： com.facebook.presto.common.type.Type

### 类型识别

以下这 2 个方法主要实现一个是对内展示类型的识别，另一个则是对外比如写 SQL 时，显示的 Varchar，Boolean 等等。

- `getTypeSignature()` 方法：获取类型名称，不区分大小写，全局唯一，通常叫内部签名。

1. 类型的唯一标识：返回一个 `TypeSignature` 对象，这个对象代表了类型的唯一标识。在 Presto 中都用这个标识来进行操作。
2. 类型系统的内部工作：这个签名用于 Presto 的类型系统内部，如类型推断、类型匹配和函数重载解析等。
3. 杂类型的支持：Presto 支持复杂的数据类型，如结构体、地图和数组。`getTypeSignature()` 方法允许这些复杂类型以结构化的方式表示，包括其参数和嵌套类型。

- `getDisplayName()` 方法：最终显示的名称，也就是最终用户看到的。

1. 用户友好的类型名称：提供了一个更易于理解和阅读的类型名称，主要是方便用户理解。
2. SQL 标准和兼容性：在 SQL 查询和结果中，显示的类型名称通常遵循 SQL 标准或者是用户更熟悉的格式。例如，尽管内部可能使用不同的表示方法，但对于用户来说，整数类型可能仅显示为 int，以及我们常见的 Varchar Timestamp等等。
3. 区分相似类型：在某些情况下，不同的内部类型可能对用户来说是相同的。`getDisplayName()` 允许这些类型有区分性的展示，即使它们的内部表示可能不同。

### Java 类型映射

主要是提供了一种 Presto 数据类型映射到 Java 类型的机制。

- `getJavaType()` 

1. 内部表示：Presto 需要在内部处理各种数据类型，如整数、浮点数、字符串等。`getJavaType()` 方法指定了在 Java 中执行表达式时，这些 Presto 数据类型应该如何在内存中表示。例如，Presto 中的整数可能在 Java 中表示为 `long` 类型，而字符串可能表示为 `Slice` 类型。
2. 性能优化：Presto 是一个高性能的分布式 SQL 查询引擎，因此在内部处理数据时需要高效地管理内存和计算。通过将 Presto 类型映射到合适的 Java 类型，可以优化数据处理的性能，例如，使用原生类型（如 `int`, `long`, `double`）可以减少对象创建的开销，提高处理速度。
3. 类型安全：Java 是一种静态类型语言，这意味着在编译时就确定了变量的类型。`getJavaType()` 方法允许 Presto 在编译时期就确保类型的正确性，从而减少运行时的类型错误。
4. 与 Java 生态系统的集成：Presto 需要与 Java 生态系统中的其他库和工具集成。通过明确指定每个 Presto 类型在 Java 中的表示，可以更容易地与 Java 代码和库交互。
5. 通用性和扩展性：这种映射机制使得 Presto 能够支持多种数据类型，并且可以轻松地扩展以支持新的数据类型。开发者可以为新的数据类型定义 Java 表示，从而扩展 Presto 的功能。


### 类型参数

在 Java 和许多其他编程语言中，参数化类型是一种允许在定义类、接口或方法时指定一个或多个类型参数的机制。这种机制在 Java 中通常与泛型（Generics）相关联。参数化类型使得代码可以对不同的数据类型进行操作，同时保持类型安全和可重用性。

- `getTypeParameters()` 返回类型参数的列表。主要是用于处理一些复杂类型。

1. 支持复杂类型：Presto 支持多种复杂的数据类型，如数组、地图和结构体。这些类型通常是参数化的，意味着它们基于其他类型构建。例如，一个数组类型可能是基于整数类型的数组，或者是基于字符串类型的数组。
2. 类型的灵活性：`getTypeParameters()` 允许 Presto 灵活地处理这些复杂类型。通过知道一个类型的参数，Presto 可以正确地处理和操作这些类型的数据。
3. 代码的泛化和重用：在 Presto 的内部实现中，参数化类型的支持允许代码更加泛化和可重用。例如，相同的逻辑可以用来处理不同类型的数组或地图，而无需为每种数据类型编写特定的代码。

举例说明：

假设有一个数组类型 `ArrayType(IntegerType)`，这表示一个整数数组。在这里，IntegerType 是 ArrayType 的类型参数。

一般获取类型的参数列表主要用于以下几个场景

1.类型检查和转换：在执行查询时，Presto 需要确保数据类型的正确性。例如，当处理数组或地图类型时，Presto 需要知道其元素或键值的类型。
2.函数和操作符的重载解析：在 SQL 查询中使用函数或操作符时，Presto 需要根据参数类型来解析和选择正确的函数实现。
3.序列化和反序列化：在数据交换和存储过程中，了解复杂类型的具体参数有助于正确地序列化和反序列化数据。

### 块 Block 构建器

块 Block 是 Presto 处理和存储数据的基本机制之一。

简单介绍一下块 `Block` 的概念：`Page` 是 Presto 中处理的最小数据单元。一个 `Page` 对象包含多个 `Block` 对象，而每个 `Block` 对象是一个字节数组，存储一个字段的若干行。多个 `Block` 横切的一行则对应一行实际的数据。

- `createBlockBuilder(BlockBuilderStatus blockBuilderStatus, int expectedEntries, int expectedBytesPerEntry)` 方法用于创建此类型的首选块构建器，这是用于在查询中存储表达式投影后的值的构建器。

该方法有重载，区别在于是否含有 `expectedBytesPerEntry` 参数。重载的目的在于应对那些不需要精确控制每个条目字节大小的场景。

参数解释：

1. BlockBuilderStatus 提供当前 builder 的状态信息
2. expectedEntries 预期的条目数
3. expectedBytesPerEntry 预提条目的字节数，主要方便进行预先分配足够的内存空间


### 值获取和写入和无边界操作

从块 Block 中获取值，和写入值。

获取值：

- `getObjectValue(SqlFunctionProperties properties, Block block, int position)`，从 Block 中指定的位置 Position 获取值，这个方法主要是将原始数据转换为 Java 对象，就是说无论什么类型，都会转换成一个 Java 可以处理的对象。

同时这个方法还有个很关键的点，就是返回的对象必须是 JSON 可序列化的，同时也方便用于将数据转换为可以通过 REST API 等方式传输的格式。

- `boolean getBoolean(Block block, int position)`，有无边界获取
- `long getLong(Block block, int position)`，有无边界获取
- `double getDouble(Block block, int position)`，有无边界获取
- `Slice getSlice(Block block, int position)`，有无边界获取
- `Object getObject(Block block, int position)`，有无边界获取

写入值：

- `void writeBoolean(BlockBuilder blockBuilder, boolean value)`
- `void writeLong(BlockBuilder blockBuilder, long value)`
- `void writeDouble(BlockBuilder blockBuilder, double value)`
- `void writeSlice(BlockBuilder blockBuilder, Slice value)`
- `void writeObject(BlockBuilder blockBuilder, Object value)`，有重载，区别在于 slice 的 offset 与 length

这里有个关键的概念的 `无边界的操作`，比如 `getBoolean` 有个对应的 `getBooleanUnchecked`，主要目的是不进行边界检查的情况下，操作 Block 中的值。那么为什么要设计这个无边界的操作呢？根据相关资料了解，主要是出于性能优化的考虑，这种设计允许在特定情况下提高数据处理的效率。

因为有边界的检查方法中，会有各种边界检查，比如防止数组越界等错误等等，这些检查可以提高代码的健壮性和安全性，同时也会有性能衰减。所以设计了无边界的操作，这些是为了在一些特定场景中，性能敏感的地方使用。

这里举个简单的例子，比如已经确定数据在 Block 的有效范围内了。可以使用 getDoubleUnchecked 来加速获取值。

```java
// 假设已知 startIndex 和 endIndex 在 block 的有效范围内
for (int position = startIndex; position < endIndex; position++) {
    double value =*Type.getDoubleUnchecked(block, position);
    // 对 value 进行一些计算或操作
}
```

### 可比较性和可排序性 

类型的操作性，比如我们在写 SQL 时， WHERE 子句中字段比较等操作，ORDER BY 子句等等操作。

- `boolean isComparable()` 可比较
- `boolean isOrderable()` 可排序

注意这 2 个方法，的返回值是 boolean，所以这里只是代表类型是否具备可比较和可排序，具体的比较逻辑可以按照 Java 在设计一个实体类一样，Presto 中也是去实现 `equalTo` 和 `hash` 方法来实现比较的功能。

同时实现 `compareTo` 方法，来实现排序的功能。


### 值比较、哈希、追加

根据前面的可排序，可比较介绍，这里我们就可以按照设计一个 Java 实体来理解了。

- `equalTo()` 比较块中的值
- `hash()` 计算哈希
- `appendTo()` 将值追加到块构建器中

## 定长和变长接口

### FixedWidthType 

接口用于那些在内存中占用固定字节大小的数据类型。这意味着每个数据元素的大小是恒定的，不会根据实际值的不同而改变。

*  特点：
    *  内存效率：由于大小固定，这些类型的数据可以更高效地在内存中存储和访问。
    *  应用场景：典型的固定宽度类型包括整数（如 `INTEGER`, `BIGINT`）、浮点数（如 `DOUBLE`, `REAL`）和日期时间类型（如 `DATE`, `TIMESTAMP`）。

1. getFixedSize**
    
    *  功能：获取此类型的值的固定大小（以字节为单位）。对于实现 `FixedWidthType` 的所有类型，其值的大小是恒定的。
    *  返回值：返回一个整数，表示类型值的固定字节大小。
2. createFixedSizeBlockBuilder**
    
    *  功能：创建一个 `BlockBuilder`，用于构建此类型的数据块。这个 `BlockBuilder` 被初始化为能够容纳指定数量的位置（`positionCount`），每个位置的大小是固定的。
    *  参数：`positionCount` - 预期要存储的元素数量。
    *  返回值：返回一个 `BlockBuilder` 实例，用于构建和存储此类型的数据。


`getFixedSize` 方法允许 Presto 精确地知道每个元素占用多少内存，这对于内存管理和性能优化非常重要。而 `createFixedSizeBlockBuilder` 方法则提供了一种高效创建和管理固定大小数据的方式。这些特性使得 `FixedWidthType` 在处理如整数、浮点数等固定大小数据时非常高效。

### VariableWidthType

接口用于那些在内存中占用可变字节大小的数据类型。这些类型的数据元素根据实际值的不同，占用的字节大小也不同。

*  特点：
    *  灵活性：可以根据数据的实际内容动态调整所占用的空间。
    *  应用场景：典型的可变宽度类型包括字符串（如 `VARCHAR`）和二进制数据（如 `VARBINARY`）。

## 其他抽象类

### AbstractType

作为所有类型的基础抽象类，提供了一些通用的实现和工具方法。它是其他更具体类型类的基础。

设计此抽象类，presto的主要是为所有的类型设计一个统一的基准行为，比如我们可以看到里面的大部分方法都返回了统一的异常，这样就相当于给接口方法一个默认的实现，保证所有的子类在默认的情况下，具有一致的效果。

```
@Override
public boolean equalTo(Block leftBlock, int leftPosition, Block rightBlock, int rightPosition)
{
    throw new UnsupportedOperationException(getTypeSignature() + " type is not comparable");
}
@Override
public int compareTo(Block leftBlock, int leftPosition, Block rightBlock, int rightPosition)
{
    throw new UnsupportedOperationException(getTypeSignature() + " type is not orderable");
}
@Override
```

后续我们可以看到基本常用的子类都继承了它，所以该抽象类，也是起到了 强制子类提供具体实现，方便扩展，同时接口的通用实现放在下一层的抽象类中，也有助于代码的维护，和结构的清晰性和可读性。

![](vx_images/105628459685315.png)

### AbstractPrimitiveType AbstractVariableWidthType



### 为什么要再设计 Long Int Decimal DateTime 这些抽象类

* AbstractLongType 专门用于处理长整型数据（对应于 Java 中的 `long` 类型）。它提供了处理长整型数据的特定方法。
* AbstractIntType 专门用于处理整型数据（对应于 Java 中的 `int` 类型）。它提供了处理整型数据的特定方法。
* DecimalType 用于表示十进制数，支持固定和可变精度的十进制数。这个类提供了处理十进制数的特定方法。
* AbstractDateTimeType 用于表示日期和时间类型的数据，如 `DATE`, `TIME`, `TIMESTAMP` 等。

这里我们先思考一下，已经有一个 AbstractType 了，为什么还要设置具体类型的抽象呢？

在 Presto 中设计 AbstractLongType、AbstractIntType、AbstractDecimalType 和 AbstractDateTimeType 这样的具体类型抽象类，是为了提供针对特定数据类型的专属化实现和优化。这些抽象类继承自 AbstractType 并增加了一些新的功能和特性，以适应各自数据类型的特定需求。

这里我们可以分类的理解，分为长整型、大精度类型、和时间类型，那么这些3种类型里面，要设计哪些特殊处理呢？

#### 特殊类型长整型

AbstractLongType 抽象类的存储类型是 Long，AbstractIntType 是 Integer，首先在对这两种类型读取和写入时，都是使用的对应类型，减少不必要的类型转换。比如 AbstractLongType 的读写都是 long。

但是我们可以发现一些，这两个抽象类中，读取是 getLong，写入是 writeLong，为什么这里都用Long呢？是不是会有点误导呢？

这是统一数据处理的设计，它起到了在使用时简化的效果，对于处理整型类操作时，可以统一使用它们。而且如果对于不同的整型单独再去实现相关类型，会调用的复杂性和开销。使用统一的方法可以减少这种开销。

在某些情况下，使用 long 类型（即使对于实际上是 int 类型的数据）可以提高内存对齐和访问效率。同时在 presto 内部可能将所有数值类型统一表示为更大的类型（如 long），以便统一处理。这种表示方法在内部可以提高处理的灵活性和效率。

## Block 接口

#### 处理精度

TODO

#### 时间特殊处理

TODO

### AbstractFixedWidthType

用于表示固定宽度的数据类型，如整数和日期。这个类提供了处理固定大小数据的基础方法。

### AbstractVariableWidthType

用于表示可变宽度的数据类型，如字符串或二进制数据。这个类提供了处理可变大小数据的基础方法。

## 类型的赋值

TODO

## 类型的比较

### 常用类型

以时间类型来举例类型比较功能

### 类型隐式转换


TODO


## 参考

- https://zhuanlan.zhihu.com/p/55299409