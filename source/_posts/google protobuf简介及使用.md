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

### 字段的类型
在上边的例子中，我们使用了两种类型：string和int32，除此之外你还可以声明其他的一些基本类型，如bool等，同时还可以是一些自定义的类型，如枚举和一个pb消息。

### 分配标签
每一个字段后，都分配了一个独一无二的数字标号，这个标号是用来在二进制格式中区分某一个字段用的。因此，一个标号只能使用一次，也就是说即使你将旧的字段删除，也不能再重用为之分配的标签了，需要重新指定一个没使用过得标签号。

标签号的值在1-15之前的，编码后仅仅使用一个字节（包括标识号和字段类型）。标签号范围在16-2047占用2个字节，因此你应该将1-15分配给频率使用很高的字段，可以给使用频繁的字段预留1-15的标签号。

标签号最小为1，最大为2^29-1（536870911），数字19000到19999 (FieldDescriptor::kFirstReservedNumber到FieldDescriptor::kLastReservedNumber)是预留给ProtoBuf实现的，因此你也不能使用这些标签号。

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
确保两个用户不会使用同一个数字标签为消息添加不同的扩展是很重要的，将扩展意外的解析成错误的类型能够造成数据损坏。你可以在定义消息的时候指定扩展数字标签的范围。如果你想使用很大的数字，可以使用关键字max代替：
```
message Foo {
	extensions 1000 to max;
}
```
max最大是2^29-1或536870911。

除此之外，你还要避免使用保留数字标签从19000到19999 (FieldDescriptor::kFirstReservedNumber到FieldDescriptor::kLastReservedNumber)，你能够在指定扩展数字范围时指定这些数字，但是编译器会阻止你真正使用这些保留的数字。

## Oneof
如果你的消息有很多可选的字段，但是同一时间只能表现为一个消息，那么你可以使用oneof特性来实现。

oneof像所有的可选字段使用共享的内存区域，同时只能有一个可选字段被设置。设置任一oneof的消息，会自动的清除其他消息，你可以使用case()或WhichOneof()检查那个消息被设置了。

### 使用Oneof
使用oneof非常简单：
```
message SampleMessage {
	oneof test_oneof {
		string name = 4;
		SubMessage sub_message = 9;
	}
}
```
你可以在oneof定义中添加任何类型的字段而不能使用required、optional或repeated关键字。

在生成的代码中，oneof字段拥有和optional字段一样的getter和setter方法。另外还有检查哪一个字段被设置的api，可以参考[这里][7]。
### Oneof特性
* 设置一个oneof字段，会自动将其他字段清空。因此，如果你设置了许多字段，只有最后一个设置的字段是有效的
```
SampleMessage message;
message.set_name("name");
CHECK(message.has_name());
message.mutable_sub_message();   // Will clear name field.
CHECK(!message.has_name());
```
* 如果解析器遇到了多个oneof成员，只有最后一个遇到的成员在解析的消息中有效
* 扩展不支持oneof
* oneof不能是repeated的
* 反射API能够在oneof字段上使用
* 如果你使用C++，确保你的代码不会崩溃。下面简单的代码因为使用了被set_name已经清除的sub_message，会导致崩溃
```
SampleMessage message;
SubMessage* sub_message = message.mutable_sub_message();
message.set_name("name");      // Will delete sub_message
sub_message->set_...            // Crashes here
```
* 在C++中，如果你让oneof消息使用Swap函数，这两个消息会拥有对方的值，如下，msg1会有sub_message的值，msg2会有name的值
```
SampleMessage msg1;
msg1.set_name("name");
SampleMessage msg2;
msg2.mutable_sub_message();
msg1.swap(&msg2);
CHECK(msg1.has_sub_message());
CHECK(msg2.has_name());
```

### 向后兼容问题
在消息中添加或删除oneof字段时，要特别小心。如果检查到字段的结果是None或NOT_SET，可能意味着消息还没有被设置或者在不同版本的oneof消息中被设置了。没法知道一个不知道的字段是不是oneof消息的成员，这种情况就没有办法区分。

### 标签重用问题
* 将可选字段移动到或者移出oneof：序列化或反序列化消息的时候可能会丢失一些信息（一些字段会被清空）。
* 删除一个oneof字段并且将它加回来：序列化或反序列化消息的时候可能会清空你当前设置的字段。
* 分割或合并oneof消息：这和移动optional字段类似。

## Maps
如果你想在你数据定义中使用map，protobuf提供了一个简单的语法：
```
map<key_type, value_type> map_field = N;
```
key_type可以是整数或者string类型（因此除了浮点数的原生类型都可以），value_type可以是任何类型。

例如，你想创建一个和字符串关联的Project变量projects，可以使用如下的语法：
```
map<string, Project> projects = 3;
```
更多的api可以参考[这里][7]。

### Map特性
* 扩展不支持map
* map不能是repeated、optional或required
* 二进制格式的顺序或map迭代的顺序是未定义的，因此你不能依赖map的items以特定的顺序进行遍历
* 当proto转换成text格式的时候，map会按照key排序，key是数字的时候按照数字大小排序
* 当从二进制解析或合并的时候，如果map的key重复了，只有最后一个value会被用到。当从text格式解析的时候，遇到重复的键值，会解析失败

### 向后兼容问题
map在二进制格式下与下边的语法一样，因此protobuf实现虽然不支持map，但仍然能后处理你的数据：
```
message MapFieldEntry {
	key_type key = 1;
	value_type value = 2;
}

repeated MapFieldEntry map_field = N;
```

## 包
你可以在proto文件中加入可选的package指定包名称，这样可以避免消息的名称出现冲突。
```
package foo.bar;
message Open { ... }
```

为消息添加包名之后需要添加包名来取到特定的消息：
```
message Foo {
	...
	required foo.bar.Open open = 1;
	...
}
```

在不同的语言中使用不同的语法来拿到包中的消息：
* 在C++中，包使用名称空间实现，例如Open在名称空间foo::bar中
* 在Java中，可以像Java中的包一样使用，除非你在proto文件中额外指定了option java_package选项
* 在Python中，包直接被忽略了，因为Python使用源代码在文件系统中的位置来组织模块

### 包和名称解析
在protobuf中的类型名称解析和C++中类似，首先在最内层的作用域搜寻，然后是次内层，等等。每一个包是它父包的内一层。一个前导的点表示从最外边的作用域开始搜寻，如.foo.bar.Baz。

protobuf编译器通过导入proto文件来解析所有的类型名。即使生成代码的语言有着不同的作用域规则，它也能引用到每一种类型。

## 定义服务
如果你想在RPC（Remote Procedure Call）系统中使用你的消息，你可以在proto文件中定义一个RPC服务的接口，protobuf编译器会自动生成这些服务接口的代码。例如，你想定义一个接收SearchRequest和返回SearchResponse方法的RPC服务，你可以在proto文件中类似这样定义：
```
service SearchService {
  rpc Search (SearchRequest) returns (SearchResponse);
}
```

默认的，protobuf编译器会生成一个名为SearchService的抽象接口，并且有一个相应的“桩”实现。这个实现将所有的请求转发给一个必须实现抽象接口的RpcChannel。例如，你想实现一个将序列化消息通过HTTP发送给服务器的RpcChannel。换句话说，生成的代码提供了一个使用基于protobuf类型安全的RPC调用，而你不需要关注特定的RPC实现，因此，在C++中，你可以像这样写代码：
```
using google::protobuf;

protobuf::RpcChannel* channel;
protobuf::RpcController* controller;
SearchService* service;
SearchRequest request;
SearchResponse response;

void DoSearch()
{
	// You provide classes MyRpcChannel and MyRpcController, which implement
	// the abstract interfaces protobuf::RpcChannel and protobuf::RpcController.
	channel = new MyRpcChannel("somehost.example.com:1234");
	controller = new MyRpcController;

	// The protocol compiler generates the SearchService class based on the
	// definition given above.
	service = new SearchService::Stub(channel);

	// Set up the request.
	request.set_query("protocol buffers");

	// Execute the RPC.
	service->Search(controller, request, response, protobuf::NewCallback(&Done));
}

void Done()
{
	delete service;
	delete channel;
	delete controller;
}
```

所有的服务类会实现一个服务的接口，它提供了一个不需要在编译时知道方法名或它的输入类型和输出类型而调用特定接口的方式。在服务端，这可以用来实现一个可以注册服务的RPC服务器。
```
using google::protobuf;

class ExampleSearchService : public SearchService
{
public:
	void Search(protobuf::RpcController* controller, const SearchRequest* request,
			SearchResponse* response, protobuf::Closure* done)
	{
		if (request->query() == "google") {
			response->add_result()->set_url("http://www.google.com");
		} else if (request->query() == "protocol buffers") {
			response->add_result()->set_url("http://protobuf.googlecode.com");
      	}
		done->Run();
	}
};

int main(int argc, char argv[])
{
	// You provide class MyRpcServer.  It does not have to implement any
	// particular interface; this is just an example.
	MyRpcServer server;

	protobuf::Service* service = new ExampleSearchService;
	server.ExportOnPort(1234, service);
	server.Run();

	delete service;
	return 0;
}
```

除此之外，你可以使用[ gRPC][8]：一个谷歌开发的语言和平台无关的开源RPC系统。gRPC可以很好的和protobuf一起工作，通过一个特殊的protobuf编译器插件从proto文件生成相应的RPC代码。然而，因为客户端和服务器使用的proto2和proto3生成代码之间潜在的兼容性问题，推荐你使用proto3来定义RPC服务。你可以在[Proto3 Language Guide][9]这里找到更多关于proto3的语法。如果你想在gRPC中使用proto2，你需要使用version 3.0.0或更高版本的protobuf编译器和库。

除了gRPC，有许多不断基于protobuf的正在开发中的第三方RPC项目。你可以在这里找到这些列表：[third-party add-ons wiki page][10]。

## 选项
在一个单独的proto文件中可以声明一些选项。选项不会改变一些整体声明的含义，但是可能会影响一些它在特定上下文的处理。全部的选项在google/protobuf/descriptor.proto中。

一些选项是文件级的选项，意味着你应该将这些选项写在top-level的作用域，而不是在任何消息，枚举或服务的定义中。一些选项是消息级别的定义，意味着应该写在消息定义里。一些选项是字段级别的选项，意味着它们应该写在字段定义中。选项也可以应用在枚举类型，枚举值，服务类型和服务方法，然而这些选项对它们没有用。

这里是很多常用的选项：
* java_package（文件选项）：你想在生成的java类中使用的包名。如果没有在proto文件中指定java_package选项，那么将会使用默认的proto包（在proto文件中使用package关键字指定）。然而，proto的包通常不用来代替Java包，因为proto的包不期望是一个反向域名。如果不生成Java代码，这个选项不起作用。
```
option java_package = "com.example.foo";
```
* java_outer_classname（文件选项）：这个类名是你生成的Java外部类的名字（也是文件名字）。如果不指定java_outer_classname，类名由proto文件的名字驼峰方式组成（如foo_bar.proto 的名字FooBar.java）。如果不生成Java代码，这个选项不起作用。
```
option java_outer_classname = "Ponycopter";
```
* optimize_for (文件选项): 可以设置为SPEED，CODE_SIZE或者LITE_RUNTIME。这个对C++和Java代码生成器（或其他第三方代码生成器）的起作用的方式如下：
  * SPEED（默认）：protobuf编译器为序列化，解析和在消息类型上执行其他通用的操作而生成代码。这个代码性能是很高的。
  * CODE_SIZE：在这个选项下protobuf编译器会生成较小的类，它们会依赖共享的，基于反射的代码来实现序列化、解析和变量的其他操作。生成的代码比SPEED小，但是操作的速度会变慢。类仍然会实现和SPEED模式下一样的公共API。这个模式比较适合那些包含了很多数量的proto文件，而又对它们的性能没有很高要求的场景。
  * LITE_RUNTIME：protobuf编译器会生成在运行时依赖"lite"库（用libprotobuf-lite替代libprotobuf）的类。lite运行时库比全量的库小很多（大概小一个数量级），但是它去除了一些类似描述器和反射的特性。这对运行在特定平台如移动电话上的应用格外有用。编译器会生成与SPEED模式速度一样的方法。生成的类在每种语言中仅仅会实现MessageLite接口中的方法，它提供了Message全部方法的子集。
```
option optimize_for = CODE_SIZE;
```
* cc_generic_services, java_generic_services, py_generic_services (文件选项): 它们分别决定编译器在生成C++、Java和Python代码时是否基于服务定义生成抽象服务代码。因为历史原因，它们默认是true。然而，2.3.0版本（2010年1月）认为更好实现RPC系统的方法是提供代码生成插件去为每一个系统去生成代码，而不是依赖抽象服务。
```
// This file relies on plugins to generate service code.
option cc_generic_services = false;
option java_generic_services = false;
option py_generic_services = false;
```
* cc_enable_arenas (文件选项)：Enables [arena allocation][11] for C++ generated code
* message_set_wire_format (消息选项)：如果设置成true，这个消息使用一个不同的意图兼容以前在Google内部使用被叫做MessageSet的二进制格式。在Google之外的使用者或许永远也不会用到这个选项。消息必须被定义为如下形式：
```
message Foo {
  option message_set_wire_format = true;
  extensions 4 to max;
}
```
* packed(字段选项)：如果设置为true，在基本数字类型的repeated字段上会使用更加紧凑的编码。使用这个选项没有缺点。2.3.0之前的版本遇到packed数据会忽略它，因此，改变一个存在的字段不可能不打破wire的兼容性。在2.3.0之后的版本，这个改变是安全的，因为解析器总是能接受这两种格式，但是要在使用老版本protobuf的程序中小心的处理这个字段。
```
repeated int32 samples = 4 [packed=true];
```
* deprecated (字段选项)：如果设置为true，表示这个字段是废弃的，不应该在新的代码中使用这个字段。在多数语言中这个选项没有实际的用处，在Java中，这会变成一个@Deprecated的注解。在未来，其他特定语言的生成器也许会生成在字段访问器上添加废弃符号，在真正编译代码的时候如果尝试使用这个字段，则发出一个警告。如果这个字段没有被任何人使用，而且希望新用户不会使用这个字段，可以考虑将其放置在保留语句中。
```
optional int32 old_field = 6 [deprecated=true];
```

### 自定义选项
Protobuf甚至可以允许你定义和使用你自己的选项。注意到这个高级特性大多数人并不需要用到。选项例如FileOptions或FieldOptions在google/protobuf/descriptor.proto文件中定义，因此，定义自己的选项可以简单的扩展这些消息。例如：
```
import "google/protobuf/descriptor.proto";

extend google.protobuf.MessageOptions {
  optional string my_option = 51234;
}

message MyMessage {
  option (my_option) = "Hello world!";
}
```
这里我们通过扩展MessageOptions定义了一个消息选项。当我们使用这个选项时，这个选项的名字必须使用括号括起来，表示这是一个扩展的选项。我们可以在C++中类似这样来读取my_option的值：
```
string value = MyMessage::descriptor()->options().GetExtension(my_option);
```
MyMessage::descriptor()->options()返回MyMessage类型的MessageOptions，读取自定义选项，就像读取其他扩展字段。
相应的，在Java中可以写成这样：
```
String value = MyProtoFile.MyMessage.getDescriptor().getOptions()
	.getExtension(MyProtoFile.myOption);
```
在Python类似：
```
value = my_proto_file_pb2.MyMessage.DESCRIPTOR.GetOptions()
	.Extensions[my_proto_file_pb2.my_option]
```
自定义选项可以用在使用protobuf构造的任何结构中，下边是例子：
```
import "google/protobuf/descriptor.proto";

extend google.protobuf.FileOptions {
	optional string my_file_option = 50000;
}
extend google.protobuf.MessageOptions {
	optional int32 my_message_option = 50001;
}
extend google.protobuf.FieldOptions {
	optional float my_field_option = 50002;
}
extend google.protobuf.EnumOptions {
	optional bool my_enum_option = 50003;
}
extend google.protobuf.EnumValueOptions {
	optional uint32 my_enum_value_option = 50004;
}
extend google.protobuf.ServiceOptions {
	optional MyEnum my_service_option = 50005;
}
extend google.protobuf.MethodOptions {
	optional MyMessage my_method_option = 50006;
}

option (my_file_option) = "Hello world!";

message MyMessage {
	option (my_message_option) = 1234;

	optional int32 foo = 1 [(my_field_option) = 4.5];
	optional string bar = 2;
}

enum MyEnum {
	option (my_enum_option) = true;

	FOO = 1 [(my_enum_value_option) = 321];
	BAR = 2;
}

message RequestType {}
message ResponseType {}

service MyService {
	option (my_service_option) = FOO;

	rpc MyMethod(RequestType) returns(ResponseType) {
		// Note:  my_method_option has type MyMessage.  We can set each field
		//   within it using a separate "option" line.
		option (my_method_option).foo = 567;
		option (my_method_option).bar = "Some string";
  }
}
```
注意如果你希望使用在一个package中定义的选项，你必须在选项名字前加上package名，就像使用类型那样：
```
// foo.proto
import "google/protobuf/descriptor.proto";
package foo;
extend google.protobuf.MessageOptions {
	optional string my_option = 51234;
}

// bar.proto
import "foo.proto";
package bar;
message MyMessage {
	option (foo.my_option) = "Hello world!";
}
```

最后一点：因为自定义选项是扩展，他们必须像其他字段或扩展那样指定标签号。在上边的例子中，我们使用了范围为50000-99999的字段号，这个范围时预留给独立组织内部使用的，因此你可以自由的在你自己的应用中使用。如果你想使用自定义选项在公共的应用中，你必须确保你的字段数字是全局唯一的。你可以向protobuf-global-extension-registry@google.com发送申请来获取一个全局唯一的字段数字，只需要提供你的项目名称和网站（如果可用的话）。通常，你只需要一个扩展数字，你可以声明多个选项在一个扩展消息中。
```
message FooOptions {
	optional int32 opt1 = 1;
	optional string opt2 = 2;
}

extend google.protobuf.FieldOptions {
	optional FooOptions foo_options = 1234;
}

// usage:
message Bar {
	optional int32 a = 1 [(foo_options).opt1 = 123, (foo_options).opt2 = "baz"];
	// alternative aggregate syntax (uses TextFormat):
	optional int32 b = 2 [(foo_options) = { opt1: 123 opt2: "baz" }];
}
```
因为不同的选项级别有着不同的字段数字空间，因此，你可以在不同的选项级别中使用相同的扩展数字。

## 生成你的消息类
为了生成与你定义的proto文件相应的Java，Python或C++代码，你需要运行protobuf编译器来作用在你的proto文件上。如果你还没有protobuf编译器，下载[安装包][12]，然后阅读README。
可以像这样调用protobuf编译器：
```
protoc --proto_path=IMPORT_PATH --cpp_out=DST_DIR --java_out=DST_DIR --python_out=DST_DIR path/to/file.proto
```
* IMPORT_PATH：指定了当使用import指令时寻找proto文件的路径。如果省略，就会搜寻当前路径。可以通过指定多次--proto_path来指定多个搜寻路径，他们会按照顺序搜寻。-I=IMPORT_PATH是--proto_path的简写。
* 你可以提供一个活多个输出目录：
  * --cpp_out：生成C++代码的目录。了解更多可以参考[这里][13]。
  * --java_out：生成Java代码的目录。了解更多可以参考[这里][14]。
  * --python_out：生成Python代码的目录。了解更多可以参考[这里][15]。

作为便利，如果这个目录是以.zip或.jar结尾的，编译器将会把文件输出到给定名字的zip文件中，对于.jar还会加入Java JAR需要的manifest文件。如果文件已经存在，将会覆盖写入。
* 你必须提供一个或多个proto文件作为输入。多个proto文件可以在一次提供，尽管名字是以当前目录的相对路径确定的，但是仍然需要的IMPORT_PATHs中包含，以确定它的位置。

## 参考
1) [Google Protocol Buffer 的在线帮助][1]
2) [Google Protocol Buffer 的使用和原理 - IBM][2]

[1]: https://developers.google.com/protocol-buffers
[2]: https://www.ibm.com/developerworks/cn/linux/l-cn-gpb
[3]: https://developers.google.com/protocol-buffers/docs/reference/overview
[4]: https://developers.google.com/protocol-buffers/docs/encoding
[5]: https://developers.google.com/protocol-buffers/docs/reference/overview
[6]: https://developers.google.com/protocol-buffers/docs/proto3
[7]: https://developers.google.com/protocol-buffers/docs/reference/overview
[8]: https://github.com/grpc/grpc-common
[9]: https://developers.google.com/protocol-buffers/docs/proto3
[10]: https://github.com/google/protobuf/blob/master/docs/third_party.md
[11]: https://developers.google.com/protocol-buffers/docs/reference/arenas
[12]: https://developers.google.com/protocol-buffers/docs/downloads.html
[13]: https://developers.google.com/protocol-buffers/docs/reference/cpp-generated
[14]: https://developers.google.com/protocol-buffers/docs/reference/java-generated
[15]: https://developers.google.com/protocol-buffers/docs/reference/python-generated