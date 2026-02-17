# 快速开始: Zenoh Type-Safe Wrapper

本指南将帮助您快速上手使用SoulBridge的Zenoh类型安全包装库。

## 前置要求

- Rust 1.85+ (Edition 2024)
- 已安装zenoh依赖的运行时环境

## 1. 安装

将以下依赖添加到您的`Cargo.toml`:

```toml
[dependencies]
soul_node = { path = "crates/soul_node" }
soul_msgs = { path = "crates/soul_msgs" }
soul_macros = { path = "crates/soul_macros" }
zenoh = { workspace = true }
zenoh-ext = { workspace = true }
tokio = { workspace = true }
```

## 2. 定义消息类型

创建一个`.smsg`文件，例如`messages/chat.smsg`:

```smsg
message ChatMessage {
    string sender
    string content
    int64 timestamp
}

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
```

## 3. 使用Proc宏生成代码

在您的代码中使用`#[smsg]`属性宏:

```rust
use soul_macros::smsg;
use soul_node::SoulNode;
use zenoh_ext::{Serialize, Deserialize};
use serde::{Serialize, Deserialize};

// 导入生成的消息
#[smsg("messages/chat.smsg")]
mod chat_messages {}
// 生成以下结构:
// - ChatMessage
// - Position  
// - RobotState
```

## 4. 发布消息

```rust
#[tokio::main]
async fn main() -> Result<(), Box<dyn std::error::Error>> {
    // 初始化节点
    let node = SoulNode::new("publisher_node", None).await?;
    
    // 注册主题
    let topic_id = node.register_topic::<_, ChatMessage>("chat/messages")?;
    
    // 声明发布者
    let publisher = node.declare_publisher(topic_id).build().await?;
    
    // 发布消息
    let message = ChatMessage {
        sender: "user1".to_string(),
        content: "Hello, World!".to_string(),
        timestamp: chrono::Utc::now().timestamp(),
    };
    
    publisher.put(&message).await?;
    
    Ok(())
}
```

## 5. 订阅消息

```rust
#[tokio::main]
async fn main() -> Result<(), Box<dyn std::error::Error>> {
    let node = SoulNode::new("subscriber_node", None).await?;
    
    let topic_id = node.register_topic::<_, ChatMessage>("chat/messages")?;
    
    // 使用回调订阅
    let subscriber = node
        .declare_subscriber(topic_id)
        .callback(|sample: ChatMessage| {
            println!("Received from {}: {}", sample.sender, sample.content);
        })
        .build()
        .await?;
    
    // 保持运行
    tokio::signal::ctrl_c().await?;
    
    Ok(())
}
```

## 6. 查询服务 (Querier/Queryable)

### 服务端 (Queryable)

```rust
#[tokio::main]
async fn main() -> Result<(), Box<dyn std::error::Error>> {
    let node = SoulNode::new("service_node", None).await?;
    
    // 注册请求和响应主题
    let req_topic = node.register_topic::<_, ChatRequest>("chat/service")?;
    let resp_topic = node.register_topic::<_, ChatResponse>("chat/service")?;
    
    // 声明Queryable
    let queryable = node
        .declare_queryable(req_topic, resp_topic)
        .build()
        .await?;
    
    println!("Service started, waiting for queries...");
    
    // 处理查询
    while let Ok(query) = queryable.recv_async().await {
        let request: ChatRequest = query.deserialize()?;
        
        let response = ChatResponse {
            response: format!("Hello, {}!", request.user_id),
            confidence: 0.95,
            timestamp: request.timestamp,
        };
        
        query.reply(response).await?;
    }
    
    Ok(())
}
```

### 客户端 (Querier)

```rust
#[tokio::main]
async fn main() -> Result<(), Box<dyn std::error::Error>> {
    let node = SoulNode::new("client_node", None).await?;
    
    let querier = node
        .declare_querier::<ChatRequest, ChatResponse>("chat/service")
        .build()
        .unwrap();
    
    let request = ChatRequest {
        user_id: "user123".to_string(),
        query: "What time is it?".to_string(),
        timestamp: chrono::Utc::now().timestamp(),
    };
    
    let responses = querier.get().await?;
    
    for response in responses {
        println!("Response: {}", response.response);
    }
    
    Ok(())
}
```

## 7. Liveliness (节点发现)

```rust
#[tokio::main]
async fn main() -> Result<(), Box<dyn std::error::Error>> {
    let node = SoulNode::new("node_with_liveliness", None).await?;
    
    // 声明liveliness token
    let token = node
        .liveliness()
        .declare_token("my_node")
        .await?;
    
    // 订阅其他节点的liveliness
    let ll_topic = node.register_liveliness_topic("nodes")?;
    let subscriber = node
        .liveliness()
        .declare_subscriber(ll_topic)
        .callback(|sample| {
            println!("Node status changed: {:?}", sample);
        })
        .await?;
    
    // 保持运行
    tokio::signal::ctrl_c().await?;
    
    Ok(())
}
```

## 8. 错误处理

```rust
use thiserror::Error;

#[derive(Error, Debug)]
pub enum MyError {
    #[error("Topic registration failed: {0}")]
    TopicRegistration(#[from] TopicRegistryError),
    
    #[error("Publisher build failed: {0}")]
    PublisherBuild(#[from] PublisherBuildError),
    
    #[error("Serialization error: {0}")]
    Serialization(#[from] zenoh_ext::ZSerializeError),
}
```

## 9. 调试

启用日志:

```rust
use log::info;

fn main() {
    // 初始化日志
    env_logger::Builder::from_env(env_logger::Env::default().default_filter_or("info"))
        .init();
    
    // 您的代码
}
```

或使用SoulBridge内置日志:

```rust
soul_node::init_soul_bridge_loggiing();
```

## 下一步

- 查看完整的API文档: `cargo doc --open`
- 运行示例: `cargo run --example`
- 查看测试用例: `cargo test`
