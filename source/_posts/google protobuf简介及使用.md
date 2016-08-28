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
protobuf同样提供了枚举类型，这种类型只能使用预定义列表中的值。例如你想为SearchResult添加corpus字段，它可以是：UNIVERSAL, WEB, IMAGES, LOCAL, NEWS, PRODUCTS or VIDEO。这利用枚举就可以很方便的实现。下边是一个例子：
```
message SearchRequest {
	required string query = 1;
	optional int32 page_number = 2;
	optional int32 result_per_page = 3 [default = 10];
	enum Corpus {
		UNIVERSAL = 0;
		WEB = 1;
		IMAGES = 2;
		LOCAL = 3;
		NEWS = 4;
		PRODUCTS = 5;
		VIDEO = 6;
	}
	optional Corpus corpus = 4 [default = UNIVERSAL];
}
```
枚举类型中，可以把两个不同的标识赋为相同的值，进而实现别名的功能。为了能够支持这个特性，你需要设置allow_alias为true，否则，将会编译错误。
```
enum EnumAllowingAlias {
	option allow_alias = true;
	UNKNOWN = 0;
	STARTED = 1;
 	RUNNING = 1;
}
enum EnumNotAllowingAlias {
	UNKNOWN = 0;
	STARTED = 1;
	// RUNNING = 1;  // Uncommenting this line will cause a compile error inside Google and a warning message outside.
}
```
枚举常量必须是32-bit的整数。枚举使用[varint encoding][4]编码，负数效率不高，因此不推荐使用负数。你可以将枚举定义在一个消息内或者整个proto文件中，此外，还可以通过MessageType.EnumType语法在一个消息中使用另一个消息中定义的枚举类型。

当protobuf编译器生成enum类型时，生成的代码将会包含一个相应的枚举类型（Java和C++）或一个包含符号常量的特定的EnumDescriptor类（python）。可以参考[generated code guide][5]获取更多信息。

## 使用其他类型消息
你可以将一个消息中的字段设置为其他消息类型。例如，你想在SearchResponse中包含一个Result类型：
```
message SearchResponse {
	repeated Result result = 1;
}

message Result {
	required string url = 1;
	optional string title = 2;
	repeated string snippets = 3;
}
```

### 导入定义
上边的例子中，Result和SearchResponse定一在一个proto文件中，同样，也可以使用其他proto文件中定义的类型。这就需要在你的proto文件的开头加入类似的语句：
```
import "myproject/other_protos.proto";
```

一般来说，你直接使用import就可以满足要求了。但有时，你可能想将一个proto文件移动到新的位置。你可以不直接移动proto文件，在老的位置放一个dummy .proto文件使用import public来将所有的imports跳转到新的位置。import public的依赖可以通过导入任何包含了import public语句的proto文件来传递。
```
// new.proto
// All definitions are moved here
// old.proto
// This is the proto that all clients are importing.
```
```
import public "new.proto";
import "other.proto";
```
```
// client.proto
import "old.proto";
// You use definitions from old.proto and new.proto, but not other.proto
```

protobuf编译器通过命令行参数-I或--proto_path来指定导入文件的搜索目录，如果没有指定，则使用编译器被调用的目录，一般来说，应该在命令行指定这个参数，确保全部合法的名字是可以被真长导入的。

### 使用proto3消息类型
可以在proto2中导入[proto3][6]的消息类型，反之亦然。但是，proto2的枚举不能在proto3语法中使用。

## 嵌套类型
你可以将一个消息定义在另一个消息内。例如：
```
message SearchResponse {
	message Result {
		required string url = 1;
		optional string title = 2;
		repeated string snippets = 3;
	}
	repeated Result result = 1;
}
```
可以在其他消息中用Parent.Type使用嵌套的消息类型：
```
message SomeOtherMessage {
	optional SearchResponse.Result result = 1;
}
```
同样的，可以支持多层嵌套：
```
message Outer {                  // Level 0
	message MiddleAA {  // Level 1
		message Inner {   // Level 2
			required int64 ival = 1;
			optional bool  booly = 2;
		}
	}
	message MiddleBB {  // Level 1
		message Inner {   // Level 2
			required int32 ival = 1;
			optional bool  booly = 2;
		}
    }
}
```

### 组
这个特性已经被弃用，不应该使用其来创建新的类型，可以用嵌套消息类型代替。

## 更新一个消息类型
如果一个已存在的消息类型已经不能满足你的需要——例如，你希望消息包含一些额外的字段，但是你仍然希望使用旧消息格式的代码，不必担心！！！protobuf不需要修改已有代码就可以非常简单的来更新一个消息类型。有如下规则：
* 不要改变任何已存在字段的数字标签
* 任何新加的字段都应该是optional或repeated。这意味着任何使用旧代码序列化后的消息可以使用新的代码解析，它不会缺失任何required字段。你应该为这些字段设置默认值，以便新的代码能够和老的代码进行交互。相应的，新的代码创建的消息也能够被老的代码解析：老的二进制简单的在解析的过程中忽略新的字段。但是，解析的过程中新的字段并不会被丢弃，如果重新被序列化，新的字段仍然会被序列化，因此，如果消息重新发送给新的程序，新的字段仍然能够被正确解析。
* 非必须字段能够被移除——它的数字标签不能在你的新消息类型中重新使用。为了防止出现类似的情况，你可以重新命名这个字段，如添加前缀"OBSOLETE_"或者将这个数字标签保留。
* 非必须字段可以转换成扩展，反之亦然——要保证类型和数字标签一样。
* int32, uint32, int64, uint64和bool是兼容的——这意味着你能够将一个字段从其中一种类型转换到另一种类型，而不会破坏向前或向后兼容。如果一个数字从一个不一致的类型解析出来，将会出现类似C++中的问题（例如，一个64-bit数字按照32-bit数字读，其会截断成32-bit的数字）。
* sint32和sint64是兼容的，但是与其他整数类型不兼容。
* string和bytes（只要bytes是有效的UTF-8）是兼容的。
* 嵌入消息和bytes（如果bytes包含一个消息版本的编码）是兼容的。
* fixed32和sfixed32是兼容的，fixed64和sfixed64是兼容的。
* optional和repeated是兼容的。Given serialized data of a repeated field as input, clients that expect this field to be optional will take the last input value if it's a primitive type field or merge all input elements if it's a message type field.
* 更改默认值一般是可以的，但要记住，默认值不会通过网络发送，因此，如果程序收到一个特定字段没有设置的消息，这个程序会从程序的协议版本中读取默认值。它不会看到发送者代码中的默认值。
* enum is compatible with int32, uint32, int64, and uint64 in terms of wire format (note that values will be truncated if they don't fit), but be aware that client code may treat them differently when the message is deserialized. Notably, unrecognized enum values are discarded when the message is deserialized, which makes the field's has.. accessor return false and its getter return the first value listed in the enum definition, or the default value if one is specified. In the case of repeated enum fields, any unrecognized values are stripped out of the list. However, an integer field will always preserve its value. Because of this, you need to be very careful when upgrading an integer to an enum in terms of receiving out of bounds enum values on the wire.
* In the current Java and C++ implementations, when unrecognized enum values are stripped out, they are stored along with other unknown fields. Note that this can result in strange behavior if this data is serialized and then reparsed by a client that recognizes these values. In the case of optional fields, even if a new value was written after the original message was deserialized, the old value will be still read by clients that recognize it. In the case of repeated fields, the old values will appear after any recognized and newly-added values, which means that order will not be preserved.

## 扩展
扩展让你能够在消息中声明一个第三方可用的数字标签范围。其他人可以使用这些数字标签在新的proto文件中为你的消息定义新的字段而不需要修改原始文件，如：
```
message Foo {
	// ...
	extensions 100 to 199;
}
```

这是说在Foo中数字标签[100-199]是预留给扩展的。其他人现在导入你的proto文件后就能在他们自己的proto文件中使用特定的数字标签范围为Foo添加字段，如：
```
extend Foo {
	optional int32 bar = 126;
}
```
现在Foo有了一个名称为bar的optional字段。
当Foo编码之后，其二进制表示与用户直接在Foo中定义一个新的字段编码之后的表示是一样的。然而，其访问扩展字段和访问普通字段的方式是不一样的。在C++中：
```
Foo foo;
foo.SetExtension(bar, 15);
```
Foo类定义了一个模板化的访问器HasExtension(), ClearExtension(), GetExtension(), MutableExtension()和AddExtension()，All have semantics matching the corresponding generated accessors for a normal field. For more information about working with extensions, see the generated code reference for your chosen language.

扩展可以是除了oneof或map的任何类型任何类型。

### 嵌套扩展
你可以在其他消息中定义扩展：
```
message Baz {
	extend Foo {
		optional int32 bar = 126;
	}
	...
}
```
相应的：
```
Foo foo;
foo.SetExtension(Baz::bar, 15);
```
换句话说，bar定义在Baz的作用域中了。

This is a common source of confusion: Declaring an extend block nested inside a message type does not imply any relationship between the outer type and the extended type. In particular, the above example does not mean that Baz is any sort of subclass of Foo. All it means is that the symbol bar is declared inside the scope of Baz; it's simply a static member.

A common pattern is to define extensions inside the scope of the extension's field type – for example, here's an extension to Foo of type Baz, where the extension is defined as part of Baz:
```
message Baz {
	extend Foo {
		optional Baz foo_ext = 127;
	}
	...
}
```
但是，在其他类型定义中定义扩展是不必要的：
```
message Baz {
	...
}

// This can even be in a different file.
extend Foo {
	optional Baz foo_baz_ext = 127;
}
```
实际上，这种语法可以避免混乱。如果不熟悉扩展，用户很可能会将扩展误会成子类。

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
[4]: https://developers.google.com/protocol-buffers/docs/encoding
[5]: https://developers.google.com/protocol-buffers/docs/reference/overview
[6]: https://developers.google.com/protocol-buffers/docs/proto3