# Data Model: Zenoh Type-Safe Wrapper

## 1. 消息类型定义 (.smsg)

### 1.1 SmsgFile
表示一个完整的.smsg文件。

| 字段 | 类型 | 说明 |
|------|------|------|
| messages | `Vec<MessageDef>` | 文件中定义的所有消息 |

### 1.2 MessageDef
表示单个消息定义。

| 字段 | 类型 | 说明 |
|------|------|------|
| name | `String` | 消息名称 (PascalCase) |
| fields | `Vec<Field>` | 消息字段列表 |
| line | `usize` | 定义所在行号 |
| col | `usize` | 定义所在列号 |

### 1.3 Field
表示消息的单个字段。

| 字段 | 类型 | 说明 |
|------|------|------|
| name | `String` | 字段名称 (snake_case) |
| field_type | `FieldType` | 字段类型 |
| line | `usize` | 定义所在行号 |
| col | `usize` | 定义所在列号 |

### 1.4 FieldType
表示字段的数据类型。

| 变体 | 说明 |
|------|------|
| Primitive(PrimitiveType) | 基本类型 (int8, string, float64等) |
| Array(Box<FieldType>, Option<usize>) | 数组类型，可选固定长度 |
| Nested(String) | 嵌套消息类型 |

### 1.5 PrimitiveType
支持的基本数据类型。

| 类型名 | Rust对应 | 默认值 |
|--------|----------|--------|
| int8 | i8 | 0 |
| int16 | i16 | 0 |
| int32 | i32 | 0 |
| int64 | i64 | 0 |
| uint8 | u8 | 0 |
| uint16 | u16 | 0 |
| uint32 | u32 | 0 |
| uint64 | u64 | 0 |
| float32 | f32 | 0.0 |
| float64 | f64 | 0.0 |
| bool | bool | false |
| string | String | String::new() |

---

## 2. 运行时实体

### 2.1 SoulNode
主入口点，保持不变。

| 方法 | 说明 |
|------|------|
| `register_topic<T>(key_path) -> TypedTopicId<T>` | 注册主题 |
| `declare_publisher(topic_id) -> PublisherBuilder<T>` | 声明发布者 |
| `declare_subscriber(topic_id) -> SubscriberBuilder<T>` | 声明订阅者 |
| `liveliness() -> SoulLiveliness` | 声明liveliness |
| `declare_querier<Req, Resp>(key_path) -> QuerierBuilder<Req, Resp>` | [NEW] 声明查询者 |
| `declare_queryable<Req, Resp>(key_path) -> QueryableBuilder<Req, Resp>` | [NEW] 声明可查询服务 |

### 2.2 Querier<Req, Resp>
类型安全的查询发送者。

| 字段 | 类型 | 说明 |
|------|------|------|
| topic_id | `TypedTopicId<Req>` | 主题ID |
| zenoh_inner | `zenoh::query::Querier` | Zenoh内部实现 |

| 方法 | 说明 |
|------|------|
| `get(&self) -> SoulQuerierGetBuilder<Req, Resp>` | 发送查询 |
| `undeclare(self)` | 注销查询者 |

### 2.3 Queryable<Req, Resp>
类型安全的查询处理者。

| 字段 | 类型 | 说明 |
|------|------|------|
| topic_id | `TypedTopicId<Req>` | 主题ID |
| zenoh_inner | `zenoh::query::Queryable` | Zenoh内部实现 |

| 方法 | 说明 |
|------|------|
| `recv_async(&self) -> SoulQueryRecvFut<Req>` | 接收查询请求 |
| `undeclare(self)` | 注销查询服务 |

### 2.4 SoulQueryRecvFut
查询请求的异步future。

| 字段 | 类型 | 说明 |
|------|------|------|
| z_recv | `RecvFut<'a, Query>` | Zenoh接收future |

---

## 3. Builder模式

### 3.1 QuerierBuilder<Req, Resp>
```rust
pub struct QuerierBuilder<'a, Req, Resp>
where
    Req: zenoh_ext::Serialize + Debug + 'static,
    Resp: zenoh_ext::Deserialize + Send + Sync + 'static;

impl<'a, Req, Resp> QuerierBuilder<'a, Req, Resp> {
    pub fn build(self) -> Result<Querier<Req, Resp>, QuerierBuildError>;
}
```

### 3.2 QueryableBuilder<Req, Resp>
```rust
pub struct QueryableBuilder<'a, Req, Resp>
where
    Req: zenoh_ext::Deserialize + Send + Sync + 'static,
    Resp: zenoh_ext::Serialize + Debug + 'static;

impl<'a, Req, Resp> QueryableBuilder<'a, Req, Resp> {
    pub async fn build(self) -> Result<Queryable<Req, Resp>, QueryableBuildError>;
}
```

### 3.3 SoulQueryReplyBuilder
用于构造查询响应的builder。

```rust
pub struct SoulQueryReplyBuilder<'a, Resp>
where
    Resp: zenoh_ext::Serialize + Debug + 'static;

impl<'a, Resp> SoulQueryReplyBuilder<'a, Resp> {
    pub fn key_expr(mut self, key: KeyExpr<'static>) -> Self;
    pub async fn await(self) -> Result<(), zenoh::Error>;
}
```

---

## 4. 错误类型

### 4.1 SmsgParseError
解析.smsg文件时的错误。

| 变体 | 说明 |
|------|------|
| FileNotFound(String) | 文件未找到 |
| ParseError(String, usize, usize) | 解析错误，包含位置信息 |
| InvalidType(String) | 无效的类型 |
| DuplicateMessage(String) | 重复的消息定义 |

### 4.2 QuerierBuildError
创建Querier时的错误。

| 变体 | 说明 |
|------|------|
| TopicNotFound(TopicId) | 主题未找到 |
| ZenohError(zenoh::Error) | Zenoh错误 |

### 4.3 QueryableBuildError
创建Queryable时的错误。

| 变体 | 说明 |
|------|------|
| TopicNotFound(TopicId) | 主题未找到 |
| ZenohError(zenoh::Error) | Zenoh错误 |

---

## 5. 消息类型注册表

### 5.1 TypeRegistry
用于运行时类型查找的注册表(可选，用于高级功能)。

| 方法 | 说明 |
|------|------|
| `register(name, type_info)` | 注册类型 |
| `lookup(name) -> Option<TypeInfo>` | 查找类型 |
| `list() -> Vec<String>` | 列出所有类型 |

---

## 6. 使用示例

### 6.1 定义消息 (.smsg文件)
```smsg
message ChatRequest {
    string user_id
    string query
    int64 timestamp
}

message ChatResponse {
    string response
    float64 confidence
    int64 timestamp
}
```

### 6.2 使用代码
```rust
// 服务端 - 声明Queryable
let req_topic = node.register_topic::<_, ChatRequest>("chat/service").unwrap();
let resp_topic = node.register_topic::<_, ChatResponse>("chat/service").unwrap();
let queryable = node.declare_queryable(req_topic, resp_topic).build().await.unwrap();

// 处理请求
while let Ok(query) = queryable.recv_async().await {
    let request: ChatRequest = query.deserialize().unwrap();
    let response = ChatResponse {
        response: format!("Hello, {}!", request.user_id),
        confidence: 0.95,
        timestamp: request.timestamp,
    };
    query.reply(response).await.unwrap();
}

// 客户端 - 声明Querier
let querier = node.declare_querier::<ChatRequest, ChatResponse>("chat/service").build().unwrap();
let response = querier.get().await.unwrap();
```

---

## 7. 验证规则

1. **类型匹配**: 发布者和订阅者必须使用相同的消息类型
2. **字段命名**: .smsg中使用snake_case，生成的Rust struct使用snake_case
3. **类型转换**: .smsg基本类型必须映射到对应的Rust基本类型
4. **嵌套类型**: 嵌套消息必须在同一个.smsg文件中定义
5. **数组边界**: 定长数组必须指定长度
