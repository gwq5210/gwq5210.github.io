title: protobuf的编码
date: 2016-07-30 19:35:53
tags: [protobuf, 编码方式]
categories: 编程
---

这篇文章介绍protobuf消息的二进制格式。你在你的应用中使用protobuf不必了解这些，但是它可以更好的让你知道protobuf的编码格式是怎样影响你编码后消息的大小的。

## 一个简单的消息
比方说，你有一个非常简单的消息定义：
```
message Test1 {
	required int32 a = 1;
}
```
在一个应用中，如果你创建一个Test1的消息，将a设置成150，然后序列化到流中，如果你能看到序列化的消息，会看到三个字节：
```
08 96 01
```
这是什么意思呢？继续阅读。。。

## Base 128 Varints
为了理解这个简单的protobuf编码，你首先需要了解varints，varints是一个使用一个字节或多个字节编码整数的方法。较小的数字使用较少的字节。

在varint中，除了最后一个字节，都会设置最高位（most significant bit，msb），这个最高位指示后边仍然有字节。低7bits用来存储数字补码表示，低的7bits组会优先存储。

因此，这里是数字1的表示，它是单个字节的因此，msb不会被设置：
```
0000 0001
```
下边是比较复杂的300：
```
1010 1100 0000 0010
```
怎样才能得出这个是300呢？首先你丢弃每个字节的msb，它仅仅是用来告诉我们数字字节表示的结尾。
```
1010 1100 0000 0010
→ 010 1100  000 0010
```
然后交换这两个7bits的组，因为varint先存储低7bits组，组合之后就能得到最终的数字：
```
000 0010  010 1100
→  000 0010 ++ 010 1100
→  100101100
→  256 + 32 + 8 + 4 = 300
```

## 消息结构
正如你了解到的，protobuf消息是一系列key-value的组合，二进制格式的消息，仅仅使用字段数字作为key，这个key的名称和声明的类型可以在解码时引用消息类型定义得到，如proto文件。

当编码消息时，键和值会组合成字节流。解析的时候，解析器需要能够跳过不能解析的字段。这样的话，新的字段加入的时候，较老的程序就不必修改，它可以不使用这些字段。这样，每个键值对编码成二进制的时候就会有两个值，一个是proto文件中的字段数字，另一个是能够提供值长度的二进制类型。

可用的二进制类型如下表：
| 类型   | 意义               | 用于                                       |
| ---- | ---------------- | ---------------------------------------- |
| 0    | Varint           | int32, int64, uint32, uint64, sint32, sint64, bool, enum |
| 1    | 64-bit           | fixed64, sfixed64, double                |
| 2    | Length-delimited | string, bytes, embedded messages, packed repeated fields |
| 3    | Start group      | groups (deprecated)                      |
| 4    | End group        | groups (deprecated)                      |
| 5    | 32-bit           | fixed32, sfixed32, float                 |

在序列化后的消息中，每一个key是类似这样的varint：(field_number << 3) | wire_type，换句话说，低3位用来存储二进制类型。

现在让我们来看看开始提到的那个例子，现在知道序列化后的消息中，第一个数字永远是varint，这里是08，丢弃msb之后：
```
000 1000
```
取出低3位得到二进制类型0，右移3位后得到字段数字1。因此，我们知道了标签为1的值是varint，也就是说接下来的序列是varint，使用varint编码规则我们可以得到接下来的两个字节96 01存储的是150：
```
96 01 = 1001 0110  0000 0001
       → 000 0001  ++  001 0110 (drop the msb and reverse the groups of 7 bits)
       → 10010110
       → 2 + 4 + 16 + 128 = 150
```

## 更多值得类型

### 有符号整数
正如前面提到的，和wire类型0关联的值都被编码成varint。然而，对于有符号整数（sint32和sint64）与一般整数（int32和int64）的一个很重要的不同点是它们对负数编码的处理。如果你使用int32或int64表示负数，那么这个varint总是10个字节长——它被当做一个非常大的无符号数。如果你使用任意一个有符号数，那么varint将会使用ZigZag编码，它对负数编码更有效率。

ZigZag编码将有符号整数映射到无符号整数，这样它的绝对值就很小，相应的varint编码长度也比较小。zig-zags在正数和负数之间来回交叉进行编码，-1编码是当做1,1在编码是当做2，-2编码成3，等等：
| Signed Original | Encoded As |
| --------------- | ---------- |
| 0               | 0          |
| -1              | 1          |
| 1               | 2          |
| -2              | 3          |
| 2147483647      | 4294967294 |
| -2147483648     | 4294967295 |

换句话说，n在sint32的时候被编码为(n << 1) ^ (n >> 31)，在sint64的时候被编码为(n << 1) ^ (n >> 63)。

运算的第二部分，(n>>31)是算术右移，也就是说，如果n是正数，右移补的位为0，否则是1。

当sint32或sint64解析的时候，会解析成原来得数字。

### 非varint数字
这个数字类型很简单——double和fixed64的wire类型是1，这告诉解析器接下来是固定的64bits数据，相应的float和fixed32的wire类型时5，说明拥有32bits数据。这两种情况下的值都以小端字节序存储。

### strings
wire类型为2意味着值是一个varint编码的长度，接下来是指定字节长度的数据。
```
message Test2 {
	required string b = 2;
}
```
将b设置成testing的编码为：
```
12 07 74 65 73 74 69 6e 67
```
后七个字节是UTF-8编码的testing，key是0x12，wire类型是2，tag是2，0x07表示值的长度是7，然后是7个字节的字符串。

## 嵌套消息
现在定义一个消息，其字段是Test1类型的消息：
```
message Test3 {
	required Test1 c = 3;
}
```
我们将a设置成150，下边是编码后的消息：
```
1a 03 08 96 01
```
正如你看到的，最后三个字节和第一个例子是一样的，它前边有一个值为3的varint标识长度——嵌套消息被当做和字符串一样处理。

## 可选和可重复元素
在proto2中定义的repeated字段（没有[packed=true]选项），编码后的消息是零个或多个拥有相同标签的键值对。这些重复的值不一定连续出现，可能会和其他字段交错出现。解析的时候，各个元素之间的顺序被保留，尽管其他字段的顺序是不确定的。在proto3中，repeated字段使用[packed encoding][1]，下边将会介绍。

For any non-repeated fields in proto3, or optional fields in proto2, the encoded message may or may not have a key-value pair with that tag number.

Normally, an encoded message would never have more than one instance of a non-repeated field. However, parsers are expected to handle the case in which they do. For numeric types and strings, if the same field appears multiple times, the parser accepts the last value it sees. For embedded message fields, the parser merges multiple instances of the same field, as if with the Message::MergeFrom method – that is, all singular scalar fields in the latter instance replace those in the former, singular embedded messages are merged, and repeated fields are concatenated. The effect of these rules is that parsing the concatenation of two encoded messages produces exactly the same result as if you had parsed the two messages separately and merged the resulting objects. That is, this:

```
MyMessage message;
message.ParseFromString(str1 + str2);
```

is equivalent to this:
```
MyMessage message, message2;
message.ParseFromString(str1);
message2.ParseFromString(str2);
message.MergeFrom(message2);
```
This property is occasionally useful, as it allows you to merge two messages even if you do not know their types.

### Packed Repeated Fields

Version 2.1.0 introduced packed repeated fields, which in proto2 are declared like repeated fields but with the special [packed=true] option. In proto3, repeated fields are packed by default. These function like repeated fields, but are encoded differently. A packed repeated field containing zero elements does not appear in the encoded message. Otherwise, all of the elements of the field are packed into a single key-value pair with wire type 2 (length-delimited). Each element is encoded the same way it would be normally, except without a tag preceding it.

For example, imagine you have the message type:
```
message Test4 {
  repeated int32 d = 4 [packed=true];
}
```
Now let's say you construct a Test4, providing the values 3, 270, and 86942 for the repeated field d. Then, the encoded form would be:
```
22        // tag (field number 4, wire type 2)
06        // payload size (6 bytes)
03        // first element (varint 3)
8E 02     // second element (varint 270)
9E A7 05  // third element (varint 86942)
```
Only repeated fields of primitive numeric types (types which use the varint, 32-bit, or 64-bit wire types) can be declared "packed".

Note that although there's usually no reason to encode more than one key-value pair for a packed repeated field, encoders must be prepared to accept multiple key-value pairs. In this case, the payloads should be concatenated. Each pair must contain a whole number of elements.

Protocol buffer parsers must be able to parse repeated fields that were compiled as packed as if they were not packed, and vice versa. This permits adding [packed=true] to existing fields in a forward- and backward-compatible way.

## Field Order

While you can use field numbers in any order in a .proto, when a message is serialized its known fields should be written sequentially by field number, as in the provided C++, Java, and Python serialization code. This allows parsing code to use optimizations that rely on field numbers being in sequence. However, protocol buffer parsers must be able to parse fields in any order, as not all messages are created by simply serializing an object – for instance, it's sometimes useful to merge two messages by simply concatenating them.

If a message has unknown fields, the current Java and C++ implementations write them in arbitrary order after the sequentially-ordered known fields. The current Python implementation does not track unknown fields.



[1]: https://developers.google.com/protocol-buffers/docs/encoding#packed