<!--
Sync Impact Report
===================
Version Change: 1.0.0 → 1.1.0 (MINOR - 新增原则)

Modified Principles:
- I. 代码质量优先 (无变化)
- II. 可测试性 (无变化)
- III. 最小可行产品（MVP） (无变化)
- IV. 拒绝过度设计 (无变化)
- V. 文档与国际化 (无变化)
+ VI. 类型安全 (新增)

Added Sections: None

Removed Sections: None

Templates Updated: N/A

Deferred Items: None
-->

# SoulBridge 宪法

## 核心原则

### I. 代码质量优先（非协商）
所有代码必须通过 `cargo fmt`、`cargo clippy` 和 `cargo test` 检查后方可提交。代码审查必须验证：正确的错误处理、清晰的命名、适当的文档注释。无警告编译是强制要求。

### II. 可测试性（非协商）
每个公共API必须有对应的单元测试。集成测试覆盖库间契约和关键业务流程。测试必须独立、可重复、无外部依赖（使用 mock 或 test doubles）。测试覆盖率应作为代码健康度的指标。

### III. 最小可行产品（MVP）
功能开发遵循 MVP 原则：优先交付核心价值，逐步迭代增强。每个用户故事必须可独立开发、测试、部署和演示。P1 优先级功能必须首先完成。

### IV. 拒绝过度设计
使用 YAGNI（You Aren't Gonna Need It）原则驱除未来可能需要的代码。保持简单：先解决当前问题，再考虑扩展。复杂性必须有明确的业务价值支撑，否则不予引入。

### V. 文档与国际化
所有规格文档（spec.md、quickstart.md 等）使用简体中文编写。代码注释使用英文（符合 Rust 社区惯例）。公共 API 必须有 `///` 文档注释，特别是 Builder 模式。

### VI. 类型安全（非协商）
充分利用 Rust 类型系统使不正确操作在编译时报错。使用强类型定义避免裸类型（如使用自定义类型代替 `String`、`u32`）。使用 `newtype` 模式封装外部数据。避免使用 `unsafe` 代码块，确需使用时应明确注释并添加测试验证。枚举应穷尽所有可能状态，避免 `#[default]` 掩盖逻辑遗漏。

## 技术栈约束

**语言与版本**: Rust Edition 2024  
** crates**: soul_msgs（消息类型）、soul_macros（过程宏）、soul_node（主库）  
**异步运行时**: tokio  
**通信中间件**: Zenoh  
**测试框架**: tokio test（multi_thread, worker_threads = 1）  

**依赖原则**:
- 新增依赖必须经过必要性论证
- 优先使用经过验证的稳定 crate
- 避免引入运行时反射或复杂元编程

## 开发工作流

**代码规范**:
- 遵循 `AGENTS.md` 中的代码风格指南
- 错误处理使用 thiserror + anyhow 组合
- 导入顺序：std → 外部 crate → 本地模块

**质量门禁**:
- `cargo fmt --check` 通过
- `cargo clippy -- -D warnings` 无警告
- `cargo test` 全部通过

**测试要求**:
- 单元测试位于源文件的 `mod test` 块
- 集成测试位于 `tests/` 目录
- 使用 `common` 模块共享测试工具

## 治理

**宪法优先**: 本宪法优先于所有其他开发实践。  
**修订程序**: 任何修订必须更新版本号、记录变更理由，并确保迁移计划完整。  
**合规检查**: 所有 PR 必须验证与宪法原则的一致性。  

**Version**: 1.1.0 | **Ratified**: 2026-02-18 | **Last Amended**: 2026-02-18
