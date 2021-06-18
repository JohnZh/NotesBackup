# 添加 protobuf 到项目

## build.gradle of project:

```groovy
buildscript {

    ext.protobufVersion = '0.8.12'
		...
		dependencies {
    	classpath "com.google.protobuf:protobuf-gradle-plugin:$protobufVersion"
		}
}
```



## 添加 java lib module 配置 build.gradle

```groovy
plugins {
  ...
  id 'com.google.protobuf'
}

protobuf {
    protoc {
        // You still need protoc like in the non-Android case
        artifact = 'com.google.protobuf:protoc:3.0.0'
    }
    plugins {
        javalite {
            // The codegen for lite comes as a separate artifact
            artifact = 'com.google.protobuf:protoc-gen-javalite:3.0.0'
        }
    }
    generateProtoTasks {
        all().each { task ->
            task.builtins {
                // In most cases you don't need the full Java output
                // if you use the lite output.
                remove java
            }
            task.plugins {
                javalite { }
            }
        }
    }
}

sourceSets{ // 生成 java 文件的具体地址
    main.java.srcDirs += "${protobuf.generatedFilesBaseDir}/main/javalite"
  	// main.proto.srcDir = 修改 .proto 文件的地址，默认是 main/proto/
}
// 具体在 build/generated/source/proto/main/javalite/

dependencies {
    ...
    implementation 'com.google.protobuf:protobuf-lite:3.0.0'
}
```



## protobuf 例子

- .proto 文件，实际上为一种数据结构的描述文件
- 之后 protobuf compiler 会生成实现了编码和解析的类，该类完成了 protobuf 数据与二进制数据格式的变化
- 生成的类提供了 getter/setter 用于构成 protobuf 的字段，同时负责实现 protocol buf 像单元读写一样的的细节
- 非常重要的，protobuf 支持格式扩展，代码可以读取那些用老格式 encode 的数据

```protobuf
syntax = "proto2";

package tutorial;

option java_multiple_files = true;
option java_package = "com.example.tutorial.protos";
option java_outer_classname = "AddressBookProtos";

message Person {
  optional string name = 1;
  optional int32 id = 2;
  optional string email = 3;

  enum PhoneType {
    MOBILE = 0;
    HOME = 1;
    WORK = 2;
  }

  message PhoneNumber {
    optional string number = 1;
    optional PhoneType type = 2 [default = HOME];
  }

  repeated PhoneNumber phones = 4;
}

message AddressBook {
  repeated Person people = 1;
}
```

- java_package: 显示的定义包名，防止命名冲突
- java_outer_classname：生成文件的最外层包装类名
- java_multiple_files：多个 java 文件，取代一个java 文件，其他都是内部类的形式
- `bool, int32, float, double, string`：标准字段类型
- ‘=1’，‘=2’ 在每个元素上代表唯一的‘tag’，被用于二进制编码上。tag 1-15 需要 1 个字节进行编码少于那些更大的数字（>=16）
  - 因此，这是一种优化，使用 1-15用于常用的或者 repeated 的元素；16 及其以上用以不常用的元素和 optional 的元素
  - 每个 repeated 元素需要 re-encode tag 数字，因此 repeated 是优化的特殊的参与者
- 每个元素必须注以下修饰符：
  - optional：字段可以设置也可以不设置。如果不设置，默认值会别使用。例子中你可以定义自己的默认值（PhoneType）。不然，系统的默认值会被使用，数字是 0，字符串是空字符串，bool 是 false。内嵌的 message，默认值始终是 message 的 ‘defalut instance’ 或者 ‘prototype’
  - repeated：这个字段可以被重复多次。重复值的顺序会被保存在 proto buffer 里。把 repeated 字段想象成一个动态尺寸的数组
  - required：值必须别提供的字段。不然，这个 message 会被认为是“未初始化”。构造一个“未初始化” message 会抛出一个 RuntimeException。解析一个未初始化的 message 会抛出 IOException。除此之外，required 字段就像 optional field

**注意：proto3 已经没有 required，google内部也是强烈不支持 required，因为 required is forever。被标记为 required 的字段后面如果需要改为 optional 会让你遇到很多问题**



## 编译 protocol buffers

[download the package](https://developers.google.com/protocol-buffers/docs/downloads)

```shell
protoc -I=$SRC_DIR --java_out=$DST_DIR $SRC_DIR/addressbook.proto
```

AS build 一下就好

.proto style: [style guide](https://developers.google.com/protocol-buffers/docs/style)

protocol compiler generates for any particular field definition: [Java generated code reference](https://developers.google.com/protocol-buffers/docs/reference/java-generated)



# 扩展 Protocol Buffer

new buffer 向后兼容，old buffer 向前兼容。需要准守下面的规则：

- 不改变现有字段的 tag

- 不能添加和删除 required 的字段

- 可以删除 optional 和 repeated 的字段

- 可以添加新的 optional 和 repeated 的字段，但是必须使用新鲜的 tag（从未使用过的，甚至是以前已经删除的字段也没有使用过）




# 集成 Protobuf 3.8.0+

3.0.0 - 3.7.+ 的集成就是上述代码，3.8.0 后需要改两个地方：

1. ```groovy
   dependencies {implementation 'com.google.protobuf:protobuf-lite:3.0.0'}
   改为：
   dependencies {implementation 'com.google.protobuf:protobuf-javalite:3.8.0'}
   ```

2. 代码生成不再需要插件，但是要保持 protoc 的版本和 javalite 一致：

   ```groovy
   protobuf {
       protoc {
           // You still need protoc like in the non-Android case
           artifact = 'com.google.protobuf:protoc:3.8.0'
       }
       generateProtoTasks {
    				all().each { task ->
               task.builtins {
                   java { option 'lite' }
               }
           }
       }
   }
   ```

   参考：https://jitpack.io/p/google/protobuf-gradle-plugin
