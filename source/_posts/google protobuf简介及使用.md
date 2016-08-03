title: google protobuf简介及使用
date: 2016-07-30 14:17:10
tags: [protobuf, python, c++]
categories: 编程
---

## 什么是Google ProtoBuf？

ProtoBuf是一种灵活高效的结构化数据存储机制，像XML一样，但是其更加轻巧、快速、简单。
使用ProtoBuf十分简单，只需要在.proto文件中定义数据就好了，然后你可以利用ProtoBuf提供的工具生成不同语言的序列化和反序列化的代码，目前提供C++、Java、Python、C#、GO等语言版本。

## ProtoBuf的优点
同XML数据相比，ProboBuf有这许多优点：
1：简单
2：小3-10倍
3：快20-100倍
4：更少的二义性
5：生成的数据访问类更加容易编程

<!-- more -->

举个例子，为了描述一个人的name和email，在XML中你需要这样做：
```
<person>
	<name>John Doe</name>
	<email>jdoe@example.com</email>
</person>
```

相应的在ProtoBuf中，类似这样（文本格式，编码后的二进制格式不是这样）：
```
# Textual representation of a protocol buffer.
# This is *not* the binary format used on the wire.
person {
	name: "John Doe"
	email: "jdoe@example.com"
}
```
编码为二进制后，其仅仅占用28个字节，解析也只需要100-200纳秒；但是XML版本就需要至少69个字节来表示（去掉空格以后），解析需要的时间为5000-10000纳秒。

同样在访问上，ProtoBuf也更加简便：
```
	cout << "Name: " << person.name() << endl;
	cout << "E-mail: " << person.email() << endl;

	//Whereas with XML you would have to do something like:
	cout << "Name: "
		<< person.getElementsByTagName("name")->item(0)->innerText()
		<< endl;
	cout << "E-mail: "
		<< person.getElementsByTagName("email")->item(0)->innerText()
		<< endl;
```

## 定义一个简单的Message
ProtoBuf文件以proto为后缀，如定义一个person类型的消息person.proto，如下：
```
// person.proro
message person {
	required string name = 1;
	required string email = 2;
	required int32 age = 3;
	optional string address = 4;
    optional string phone = 5;
}
```
上述文件可以使用以下命令生成C++的源代码：
```
// 需要安装protoc
protoc person.proto -cpp_out=.
```

我们注意到，消息中的每一个字段由四个部分构成，如name字段，其结构为：
```
// 规则 类型 名字 = 唯一标签
required string name = 1;
```

### 字段的规则
ProtoBuf消息中的字段有如下三种规则：
1、required：这个字段是必须的。
2、optional：这个字段是可选的，在消息中有或者没有均可。
3、repeated：这个字段是可以重复任意次（包括0），可以理解为数组。

出于历史原因，repeated字段对于数字类型来说，不一定以最高效的方式编码，可以使用[packed=true]选项来获得更高的效率。
```
repeated int32 samples = 4 [packed=true];
```
对于required字段，一定要谨慎的使用，required如果更改为optional字段，新生成的消息在老版本的程序中就会拒绝解析，认为这是一个不完整的消息；但是对于optional和repeated字段就没有这个问题。

### 字段的类型
在上边的例子中，我们使用了两种类型：string和int32，除此之外你还可以声明其他的一些基本类型，如bool等，同时还可以是一些自定义的类型，如枚举和一个pb消息。

### 分配标签
每一个字段后，都分配了一个独一无二的数字标号，这个标号是用来在二进制格式中区分某一个字段用的。因此，一个标号只能使用一次，也就是说即使你将旧的字段删除，也不能再重用为之分配的标签了，需要重新指定一个没使用过得标签号。

标签号的值在1-15之前的，编码后仅仅使用一个字节（包括标识号和字段类型）。标签号范围在16-2047占用2个字节，因此你应该将1-15分配给频率使用很高的字段，可以给使用频繁的字段预留1-15的标签号。

标签号最小为1，最大为2^29-1（536870911），数字19000到19999 (FieldDescriptor::kFirstReservedNumber到FieldDescriptor::kLastReservedNumber)是预留给ProtoBuf实现的，因此你也不能使用这些标签号。

### 添加更多的消息
可以再一个proto文件中定义多个message。当你需要定义多个有关联的消息时，这就十分有用。例如你想定义一个请求和回复的消息，在一个proto文件中定义就十分方便。
```
message SearchRequest {
	required string query = 1;
	optional int32 page_number = 2;
	optional int32 result_per_page = 3;
}

message SearchResponse {
	...
}
```

### 使用注释
在proto文件中你可以使用C/C++风格的注释：//
```
message SearchRequest {
	required string query = 1;
	optional int32 page_number = 2;// Which page number do we want?
	optional int32 result_per_page = 3;// Number of results to return per page.
}
```

### 保留字段
在你更新你的消息时，你可能删除掉一些字段，当其他人更新时，可能会重新使用之前用到的标签号，这个使用你可以使用reversed关键字，当其他人使用到reversed中的标号时，ProtoBuf会给出警告。你不能将名字和标号在一个reserved语句中混用。
```
message Foo {
	reserved 2, 15, 9 to 11;
	reserved "foo", "bar";
}
```

### ProtoBuf从proto文件中生成了什么？
ProtoBuf编译器会将proto文件编译成你选择语言的相应代码，这些代码中包括了获取和设置字段的方法，序列化到输出流和从输入流解析的方法。
对于C++，编译器对每一个proto文件生成相应的h和cc文件，其中包括了每个消息对应的类。
对于Java，编译器为每一个消息生成一个对应的java文件。
对于生成代码的API的详细介绍，可以在[API reference][3]找到。

## 基本数据类型
一个只使用基本类型的消息，可以选择的类型在如下的表格中，包括proto中可以选择的类型和生成类的相应类型：

| .proto类型 | 备注                                      | C++类型  | Java类型     | Python类型    |
| -------- | --------------------------------------- | ------ | ---------- | ----------- |
| double   |                                         | double | double     | float       |
| float    |                                         | float  | float      | float       |
| int32    | 使用变长编码，编码负数效率不高，如果字段大部分是负数，应该使用sint32代替 | int32  | int        | int         |
| int64    | 使用变长编码，编码负数效率不高，如果字段大部分是负数，应该使用sint64代替 | int64  | long       | int/long    |
| uint32   | 使用变长编码                                  | uint32 | int        | int/long    |
| uint64   | 使用变长编码                                  | uint64 | long       | int/long    |
| sint32   | 使用变长编码，编码负数效率比int32高                    | int32  | int        | int         |
| sint64   | 使用变长编码，编码负数效率比int64高                    | int64  | long       | int/long    |
| fixed32  | 总是4个字节，当值大于2^28时比uint32效率高              | uint32 | int        | int         |
| fixed64  | 总是8个字节，当值大于2^28时比uint32效率高              | uint64 | long       | int/long    |
| sfixed32 | 总是4个字节                                  | int32  | int        | int         |
| sfixed64 | 总是8个字节                                  | int64  | long       | int/long    |
| bool     |                                         | bool   | boolean    | bool        |
| string   | 必须由UTF-8编码的或者7-bit的ASCII组成的文本           | string | String     | str/unicode |
| bytes    | 可以包括任意序列的字节                             | string | ByteString | str         |

## 可选字段和默认值
一个消息中的字段可以定义为optional，即可选的，一个合法的消息可以包括也可以不包括这个可选的字段。当解析消息时，如果不包括可选字段，那么可选字段就会被设置为默认值，这个默认值可以指定。例如：
```
optional int32 result_per_page = 3 [default = 10];
```
如果没有显示指定默认值，ProtoBuf会为每个类型指定默认值：string类型指定为空串，bool指定为false，对于数值类型，指定为0，对于枚举类型，默认为列表中的第一个值。因此如果修改枚举定义，就需要注意兼容性问题了。

## 枚举

## 使用其他类型消息

### 导入定义

### 嵌套类型

### 组

## 更新一个消息类型

## 扩展

### 嵌套扩展

### 选择扩展数字

## Oneof

### 使用Oneof

### Oneof特性

### 向后兼容问题

### 标签重用问题

## Maps

### Map特性

### 向后兼容问题

## 包

### 包和名称解析

## 定义服务

## 选项

### 自定义选项

## 生成你的消息类

### 如何使用？

#### C++

#### Python


## 参考
1) [Google Protocol Buffer 的在线帮助][1]
2) [Google Protocol Buffer 的使用和原理 - IBM][2]

[1]: https://developers.google.com/protocol-buffers
[2]: https://www.ibm.com/developerworks/cn/linux/l-cn-gpb
[3]: https://developers.google.com/protocol-buffers/docs/reference/overview