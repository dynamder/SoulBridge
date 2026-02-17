# Research Report: Zenoh Type-Safe Wrapper

**Date**: 2026-02-18
**Feature**: Zenoh Type-Safe Wrapper with .smsg Message Definition System

---

## 1. .smsg文件格式规范

### Decision
采用ROS风格的.smsg消息定义格式，参考ROS 1/2 .msg规范。

### Rationale
- ROS是成熟的机器人消息定义标准，有广泛的应用基础
- 语法简洁、易读，与其他开发者已有的经验兼容
- 支持嵌套类型、数组、基本数据类型

### Alternatives Considered
- Protobuf: 功能强大但需要额外工具链，不够轻量
- FlatBuffers: 适合高性能场景，但语法复杂
- 自定义格式: 需要从头设计，风险高

### .smsg语法规范

**基本类型映射**:
| .smsg类型 | Rust类型 |
|-----------|----------|
| int8 | i8 |
| int16 | i16 |
| int32 | i32 |
| int64 | i64 |
| uint8 | u8 |
| uint16 | u16 |
| uint32 | u32 |
| uint64 | u64 |
| float32 | f32 |
| float64 | f64 |
| bool | bool |
| string | String |

**语法示例**:
```smsg
# 简单消息
message ChatMessage {
    string user_id
    string content
    int64 timestamp
}

# 嵌套消息
message Position {
    float64 x
    float64 y
    float64 z
}

message RobotState {
    string name
    Position position
    int32 status
}

# 数组类型
message SensorData {
    float64[] readings
    int32[3] dimensions
}
```

---

## 2. Nom解析器实现

### Decision
使用nom 8.x作为.smsg文件的解析库，采用组合子(Combinator)方式构建解析器。

### Rationale
- nom是Rust生态中成熟的解析器组合子库
- 零依赖(仅使用标准库)，轻量高效
- 错误处理友好，支持精确的错误定位
- 与过程宏集成良好

### Alternatives Considered
- pest: 语法更清晰但依赖更多
- lalrpop: 功能强大但学习曲线陡峭
- 手写递归下降: 灵活性高但工作量大

### 实现要点
- 使用`nom::IResult`返回类型
- 实现自定义错误类型，包含行号和列号信息
- 使用`nom::error::VerboseError`进行详细错误报告

---

## 3. Proc宏实现

### Decision
使用`#[proc_macro_attribute]`属性宏应用于模块，解析.smsg文件并在模块内生成Rust结构体。

### Rationale
- 属性宏可以应用于模块级别，适合批量生成
- 模块级别生成可以避免命名冲突
- 与现有Rust生态(如serde)的工作方式一致

### Alternatives Considered
- `#[proc_macro_derive]`: 只能应用于struct/enum，不适合批量生成
- `#[proc_macro]`: 函数式宏，灵活性太高会增加复杂度

### 实现结构
```rust
#[smsg("path/to/messages.smsg")]
mod messages {}
```

### 代码生成要求
1. 每个.smsg消息生成对应的Rust struct
2. 自动实现`serde::Serialize`和`serde::Deserialize`
3. 自动实现`zenoh_ext::Serialize`和`zenoh_ext::Deserialize`
4. 使用`#[derive(Debug, Clone)]`

---

## 4. Zenoh Query/Queryable API

### Decision
使用Zenoh原生的Query/Reply API进行封装，提供类型安全的Querier和Queryable。

### Rationale
- Zenoh 1.7已提供完整的Query/Queryable支持
- 与现有Publish/Subscribe封装风格保持一致
- 类型安全与.smsg消息系统集成

### Zenoh Query API使用

**Queryable (服务端)**:
```rust
// Zenoh API
let queryable = session.declare_queryable("key/expression").await?;
while let Ok(query) = queryable.recv_async().await {
    query.reply("key/expression", "value").await?;
}
```

**Querier (客户端)**:
```rust
// Zenoh API
let replies = session.get("key/expression").await?;
while let Ok(reply) = replies.recv_async().await {
    println!(">> Received {:?}", reply.result());
}
```

### 类型安全封装设计

```rust
// Querier<Req, Resp>
pub struct Querier<Req, Resp>
where
    Req: zenoh_ext::Serialize + Debug + 'static,
    Resp: zenoh_ext::Deserialize + Send + Sync + 'static;

// Queryable<Req, Resp>  
pub struct Queryable<Req, Resp>
where
    Req: zenoh_ext::Deserialize + Send + Sync + 'static,
    Resp: zenoh_ext::Serialize + Debug + 'static;
```

---

## 5. 现有Pub/Sub集成

### Decision
.smsg生成的消息类型直接与现有的`Publisher<T>`和`Subscriber<T>`兼容。

### Rationale
- 现有API已使用泛型`T: zenoh_ext::Serialize/Deserialize`
- .smsg生成的结构体会自动实现这些trait
- 无需修改现有代码

### 兼容性要求
```rust
// .smsg生成的消息需要实现这些trait
impl zenoh_ext::Serialize for GeneratedMessage { ... }
impl zenoh_ext::Deserialize for GeneratedMessage { ... }
```

---

## 6. 错误报告

### Decision
使用miette库在soul_macros中提供美化的错误报告。

### Rationale
- miette是Rust生态中美化错误报告的标准库
- 与thiserror兼容
- 支持精确的错误位置和源码片段显示
- 支持Shell和GUI环境的错误显示

### Alternatives Considered
- anyhow: 适合应用程序错误，不适合编译时错误
- syn的compile_error: 只能显示简单文本
- thiserror: 功能有限

---

## 7. 总结

| 技术选型 | 决策 | 原因 |
|----------|------|------|
| 消息格式 | ROS风格.smsg | 成熟标准，语法简洁 |
| 解析器 | nom 8.x | 轻量，零依赖，高性能 |
| Proc宏 | 属性宏 | 适合模块级代码生成 |
| Query/Queryable | Zenoh原生API封装 | 保持一致性 |
| 错误报告 | miette | 编译时错误美化 |
| 序列化 | zenoh_ext | 与现有系统兼容 |

---

## 8. 后续工作

1. **Phase 1**: 实现.smsg解析器和代码生成器
2. **Phase 1**: 创建示例消息定义文件
3. **Phase 1**: 实现Querier和Queryable封装
4. **Phase 2**: 集成测试和文档

---

## References

- [ROS msg specification](http://wiki.ros.org/msg)
- [nom parser combinators](https://github.com/strong-roots-capital/nom)
- [Zenoh Rust API](https://docs.rs/zenoh/1.7.2/zenoh/)
- [Rust procedural macros](https://doc.rust-lang.org/reference/procedural-macros.html)
- [miette error reporting](https://docs.rs/miette)
