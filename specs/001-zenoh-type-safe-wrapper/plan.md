# Implementation Plan: Zenoh Type-Safe Wrapper

**Branch**: `001-zenoh-type-safe-wrapper` | **Date**: 2026-02-18 | **Spec**: specs/001-zenoh-type-safe-wrapper/spec.md
**Input**: Feature specification from `/specs/001-zenoh-type-safe-wrapper/spec.md`

## Summary

构建一个Zenoh类型安全包装库，支持通过.smsg文件定义ROS风格的消息类型，并通过过程宏自动生成Rust结构体。已实现的publish、subscribe、liveliness功能需要与新的类型系统集成，同时需要新增querier和queryable功能。

## Technical Context

**Language/Version**: Rust Edition 2024  
**Primary Dependencies**: 
- zenoh 1.7.2 (通信中间件)
- zenoh-ext 1.7.2 (序列化/反序列化)
- tokio 1.48.0 (异步运行时)
- nom 8.0.0 (.smsg文件解析)
- uuid 1.18.1 (唯一ID生成)
- thiserror 2.0.17 (错误处理)
- anyhow 1.0.101 (应用错误处理)
- miette 1.x (soul_macros中美化错误报告)
- serde 1.0.228 (序列化)
- log 0.4.28 (日志)

**Storage**: N/A (分布式通信中间件)  
**Testing**: cargo test (tokio test with multi_thread)  
**Target Platform**: 本地机器，支持最多50节点  
**Project Type**: Rust workspace (3 crates: soul_node, soul_macros, soul_msgs)  
**Performance Goals**: 节点发现和liveliness更新1秒内传播  
**Constraints**: <200ms p95延迟  
**Scale/Scope**: 最多50节点，单机部署

## Constitution Check

| 检查项 | 状态 | 说明 |
|--------|------|------|
| I. 代码质量优先 | PASS | cargo fmt, clippy, test已配置 |
| II. 可测试性 | PASS | 已有单元测试和集成测试 |
| III. 最小可行产品 | PASS | P1功能优先：.smsg定义+pub/sub集成 |
| IV. 拒绝过度设计 | PASS | MVP优先，逐步迭代 |
| V. 文档与国际化 | PASS | 简体中文spec，英文代码注释 |
| VI. 类型安全 | PASS | 充分利用Rust类型系统 |

## Project Structure

### Documentation (this feature)

```text
specs/001-zenoh-type-safe-wrapper/
├── plan.md              # This file
├── research.md          # Phase 0 output
├── data-model.md        # Phase 1 output
├── quickstart.md        # Phase 1 output
└── contracts/           # Phase 1 output
```

### Source Code (repository root)

```text
crates/
├── soul_node/           # Zenoh包装库（已有基础）
│   ├── src/
│   │   ├── lib.rs
│   │   ├── base.rs          # SoulNode核心
│   │   ├── pub_sub.rs       # 发布/订阅
│   │   ├── topic.rs         # 主题注册
│   │   └── query/           # [NEW] querier/queryable
│   └── tests/
│       ├── local_echo.rs
│       ├── liveliness.rs
│       └── common/mod.rs
├── soul_macros/        # 过程宏（需实现）
│   └── src/
│       ├── lib.rs           # smsg! 宏入口
│       ├── parser/          # .smsg解析器
│       └── codegen/         # 代码生成
└── soul_msgs/           # 消息定义（需实现）
	└── smsg/            # 消息的.smsg定义
    └── src/
        ├── lib.rs           # 导出生成的消息
        └── msg/             # 生成的代码
```

**Structure Decision**: Rust workspace with 3 crates following existing pattern. soul_node handles Zenoh wrapper, soul_macros handles .smsg parsing/codegen, soul_msgs stores predefined message types.

## Phase 0: Research Tasks

### 0.1 .smsg文件格式规范
- 确定ROS风格消息定义语法
- 定义支持的基本类型 (string, int8, int16, int32, int64, uint8, uint16, uint32, uint64, float32, float64, bool)
- 定义嵌套消息类型语法
- 定义数组/列表类型语法

### 0.2 Nom解析器实现
- 解析.smsg文件为中间表示(IR)
- 错误报告精确定位(行号、列号)
- 使用miette进行美化错误报告

### 0.3 Proc宏实现
- #[smsg("path/to/file.smsg")] 属性宏
- 模块级代码生成
- 生成Rust struct实现Serialize/Deserialize

### 0.4 现有pub/sub集成
- 理解现有Publisher<T>/Subscriber<T>泛型结构
- 确保.smsg生成的消息类型与现有API兼容
- zenoh_ext::Serialize/Deserialize集成

### 0.5 Querier/Queryable实现
- Zenoh query/queryable API封装
- 类型安全的请求/响应处理
- 与.smsg类型系统集成

## Phase 1: Implementation Plan

### 1.1 .smsg解析器 (soul_macros)
- 实现.smsg词法分析器
- 实现.smsg语法分析器
- 生成解析错误(带位置信息)

### 1.2 代码生成 (soul_macros)
- 从IR生成Rust struct
- 自动实现zenoh_ext::Serialize/Deserialize
- 支持嵌套类型

### 1.3 消息注册 (soul_msgs)
- 创建示例消息定义
- 验证代码生成流程

### 1.4 Querier/Queryable (soul_node)
- 实现Querier<T>类型
- 实现Queryable<T>类型
- 与TopicRegistry集成

## Complexity Tracking

> 无复杂度违规
