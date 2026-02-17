# Feature Specification: Zenoh Type-Safe Wrapper Library

**Feature Branch**: `001-zenoh-type-safe-wrapper`  
**Created**: 2026-02-18  
**Status**: Draft  
**Input**: User description: "建立一个zenoh的wrapper库，它的目的是增强zenoh的类型安全性，并建立一个统一的消息定义（类ROS的消息定义风格）和代码生成机制（这个wrapper后续会支持多种编程语言，所以需要从一个统一的消息定义，生成各个语言的数据类型代码，以便开发者可以使用ide的功能，例如自动补全）。对本次的rust包装，消息定义是一个.smsg 文件，并以attribute proc macro的方式生成对应的代码。你需要包装的zenoh功能包括publish, subscribe, querier, queryable, liveliness."

## Clarifications

### Session 2026-02-18

- Q: Implementation scope after user confirms publish, subscribe, and basic liveliness are already implemented → A: Focus on .smsg message definition system + querier/queryable integration
- Q: AI application development context → A: Chatbot-like applications, not sophisticated tensor operations
- Q: Node scale and deployment context → A: Up to 50 nodes, mainly on local machine (distributed node system via Zenoh)
- Q: Liveliness implementation details → A: Liveliness in Zenoh is a special pub/sub (token-based), already implemented using Zenoh's built-in liveliness API with internal SoulLivelinessMsg type (no user-defined types needed)

## User Scenarios & Testing *(mandatory)*

### User Story 1 - Define Type-Safe Message Types (Priority: P1)

As a developer, I want to define message types in a .smsg file using a ROS-like syntax, so that I can have type-safe message definitions that work across multiple programming languages.

**Why this priority**: This is the foundation of the entire system. Without standardized message definitions, the type safety and code generation features cannot exist. The existing publish/subscribe/liveliness implementation will be enhanced to use this new type system.

**Independent Test**: A developer can create a .smsg file with message definitions and use the attribute macro to generate Rust structs with full IDE support including autocomplete and type checking.

**Acceptance Scenarios**:

1. **Given** a .smsg file containing `message ChatMessage { string user_id, string content, int64 timestamp }`, **When** the attribute macro is applied, **Then** Rust struct `ChatMessage` is generated with fields `user_id`, `content`, `timestamp` of types `String`, `String`, `i64`
2. **Given** nested message types in .smsg, **When** processed by the macro, **Then** corresponding nested Rust structs are generated
3. **Given** an invalid .smsg file with syntax errors, **When** the macro processes it, **Then** clear compilation errors are shown pointing to the error location

---

### User Story 2 - Use Existing Publish with Type-Safe Messages (Priority: P1)

As a developer, I want to use the existing publish functionality with the new type-safe message system, so that I can leverage the already-implemented publish API while gaining type safety benefits.

**Why this priority**: Publish is already implemented. This story focuses on integrating it with the new .smsg type system.

**Independent Test**: A developer can use existing publisher with new message types defined in .smsg files.

**Acceptance Scenarios**:

1. **Given** a registered message type `ChatMessage` from .smsg, **When** I declare a publisher for topic "chat/message" with type `ChatMessage`, **Then** I can only publish `ChatMessage` objects to that topic
2. **Given** an active publisher with .smsg type, **When** I call `put(data)` with correct type, **Then** the message is sent over Zenoh
3. **Given** an active publisher, **When** I attempt to publish wrong type, **Then** compilation fails with type mismatch error

---

### User Story 3 - Use Existing Subscribe with Type-Safe Messages (Priority: P1)

As a developer, I want to use the existing subscribe functionality with the new type-safe message system, so that I can receive strongly-typed messages using the already-implemented subscribe API.

**Why this priority**: Subscribe is already implemented. This story focuses on integrating it with the new .smsg type system.

**Independent Test**: A developer can subscribe to a topic and receive callbacks with fully-typed message objects from .smsg definitions.

**Acceptance Scenarios**:

1. **Given** a registered message type `ChatMessage` from .smsg, **When** I subscribe to "chat/message" with type `ChatMessage`, **Then** callback receives `ChatMessage` objects
2. **Given** a subscription with a handler, **When** messages arrive on the topic, **Then** they are deserialized into the correct type before being passed to handler
3. **Given** a subscription to a topic with unknown type, **Then** error is raised indicating type not registered

---

### User Story 4 - Query and Queryable with Type Safety (Priority: P2)

As a developer, I want to implement queryable services and send queries with type-safe request/response, so that I can build type-safe service-oriented communication for chatbot applications.

**Why this priority**: Querier and queryable enable request-response patterns which are essential for chatbot applications to handle user queries and generate responses.

**Independent Test**: A developer can declare a queryable that handles specific request/response types and a querier that sends requests with typed responses.

**Acceptance Scenarios**:

1. **Given** registered request type `ChatRequest` and response type `ChatResponse`, **When** I declare a queryable for "chat service" with these types, **Then** only requests of type `ChatRequest` are accepted
2. **Given** an active querier with request type `ChatRequest` and response type `ChatResponse`, **When** I send a query, **Then** I receive a response of type `ChatResponse`
3. **Given** a queryable that returns incorrect response type, **Then** compilation fails

---

### User Story 5 - Use Existing Liveliness for Node Discovery (Priority: P2)

As a developer, I want to use the existing liveliness functionality to monitor node presence, so that I can detect when nodes go online or offline using Zenoh's built-in token-based liveliness system.

**Why this priority**: Liveliness is already implemented using Zenoh's built-in liveliness API (token-based pub/sub). It uses internal SoulLivelinessMsg type and does not require user-defined .smsg types.

**Independent Test**: A developer can declare a liveliness token to announce node presence and subscribe to receive online/offline events.

**Acceptance Scenarios**:

1. **Given** a node declares a liveliness token, **When** another node subscribes to liveliness, **Then** callback receives `Online` event
2. **Given** a node with active liveliness token drops it, **When** subscribers are listening, **Then** callback receives `Offline` event
3. **Given liveliness token, **When **network** a node with disconnection occurs, **Then** subscribers are notified of the change

---

### Edge Cases

- What happens when message type is not registered before use?
- How does the system handle version mismatches between publisher and subscriber?
- What happens when .smsg file references a type that doesn't exist?
- How does the system behave when Zenoh session is lost and reconnected?

## Requirements *(mandatory)*

### Functional Requirements

- **FR-001**: System MUST provide a .smsg file format for defining message types with ROS-like syntax (field types and names)
- **FR-002**: System MUST generate Rust struct definitions from .smsg files via attribute proc macro
- **FR-003**: Existing publisher API MUST integrate with type-safe .smsg message types
- **FR-004**: Existing subscriber API MUST integrate with type-safe .smsg message types
- **FR-005**: System MUST provide type-safe querier API for sending queries with typed requests and responses
- **FR-006**: System MUST provide type-safe queryable API for handling typed requests
- **FR-007**: Liveliness API already implemented using Zenoh's built-in token-based pub/sub (no .smsg type integration needed)
- **FR-008**: System MUST provide error messages that indicate root cause and location in .smsg file
- **FR-009**: Message type registry MUST support lookup by type name and maintain type metadata

### Key Entities

- **MessageType**: Represents a user-defined message structure with field definitions
- **Field**: Individual data member with name and type
- **TypeRegistry**: Central repository storing all registered message types
- **Publisher<T>**: Type-safe publisher for message type T
- **Subscriber<T>**: Type-safe subscriber for message type T
- **Querier<Req, Resp>**: Type-safe querier with request type Req and response type Resp
- **Queryable<Req, Resp>**: Type-safe queryable handler for request type Req and response type Resp

## Success Criteria *(mandatory)*

### Measurable Outcomes

- **SC-001**: Developers can define a message type in .smsg and generate Rust code in under 5 seconds
- **SC-002**: Type mismatches between publisher and subscriber are caught at compile time
- **SC-003**: IDE autocomplete works for all generated message type fields
- **SC-004**: Querier and queryable support type-safe request/response using .smsg message definitions
- **SC-005**: Error messages from .smsg parsing point to exact line and column of issues
- **SC-006**: Existing publish/subscribe APIs work seamlessly with new .smsg type system; liveliness already functional
- **SC-007**: System supports up to 50 nodes on a single machine without performance degradation
- **SC-008**: Node discovery and liveliness updates propagate within 1 second
