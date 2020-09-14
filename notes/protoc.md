# Protoc buffer 



> 一个 message 中的 number 用来标识二进制消息中对应的字段。一旦 message 投入使用，number 不应该再被修改。

> number 0~15 占一个字节，16~2047 占两个字节。

> number 取值范围: 1~(2**29-1), 同时不可以使用 19000 ～ 19999 (保留范围)

  

## 声明字段规则：

 - required: 谨慎使用，如果以后希望这个字段改为 optional，所有使用这个 message 的服务都需要修改,对于必传项的验证尽量编写自定义的验证逻辑，而不是在 message 中定义
          
 - optional: 可以通过 [default=...] 为 optional 字段设置默认值

 - repeated: 可以理解为 array，有序，可为空,历史原因造成的 repeated 字段编码效率不高，新版本添加 [packed=true] 提高编码效率
          
 - reserved: 保留字段

 - enum: 枚举类型，枚举值常量必须在 32 位整数范围内， 负数效率不高，不建议使用



```protobuf
syntax = "proto3"

message SearchRequest {
    required string query = 1;
    optional int32 page_number = 2;
    optional int32 result_per_page = 3 [default = 10];
    repeated string tag = 4 [packed=true];
    reserved 9, 12 to 20, 50 to max, "foo";
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



## 更新 message 定义

- 不要修改任何字段的 field number.
- 任何新添加的字段都应该定义为 `optional` || `repeated`，这样可以保证老版本的 message 也可以被更新后的 message 解析,同时新的 message 也可以被旧版本的 message 接受者解析，新添加的字段会被忽略，但是不会被丢弃。所以当 message 再次被序列化到一个新的 message 接受者中时，新添加的字段还是可以被读取到
- 非 `required` 字段可以删除。可以给字段添加 'OBSOLETE_' 前缀，或者使用 reserved 保留该字段，以防止以后重用该字段
- 非 `required` 字段只要类型和 field number 不变，就可以字段改为扩展、
- int32,uint32,int64,uint64,bool 是相互兼容的，所以这些类型可以相互转换，而不会影响兼容性
- sint32 和 sint64 兼容，但不和其他数字类型兼容
- string 和 byte 兼容，只要是有效的 UTF8 字符
...