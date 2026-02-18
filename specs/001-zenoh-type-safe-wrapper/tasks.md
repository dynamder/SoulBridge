---

description: "Task list for Zenoh Type-Safe Wrapper feature implementation"
---

# Tasks: Zenoh Type-Safe Wrapper

**Input**: Design documents from `/specs/001-zenoh-type-safe-wrapper/`
**Prerequisites**: plan.md, spec.md (required), research.md, data-model.md, quickstart.md

**Tests**: The feature specification does not explicitly request TDD. Tests are included as verification after implementation.

**Organization**: Tasks are grouped by user story to enable independent implementation and testing of each story.

## Format: `[ID] [P?] [Story] Description`

- **[P]**: Can run in parallel (different files, no dependencies)
- **[Story]**: Which user story this task belongs to (e.g., US1, US2, US3)
- Include exact file paths in descriptions

---

## Phase 1: Setup (Project Initialization)

**Purpose**: Initialize and configure the Rust workspace for new crates

- [ ] T001 Create crates/soul_macros/Cargo.toml with nom, miette, quote, syn dependencies
- [ ] T002 Create crates/soul_msgs/Cargo.toml with serde, zenoh-ext dependencies
- [ ] T003 [P] Add workspace dependencies to root Cargo.toml if needed
- [ ] T004 Run cargo build to verify all dependencies resolve

---

## Phase 2: Foundational (.smsg Parser & Code Generation Infrastructure)

**Purpose**: Core infrastructure that MUST be complete before ANY user story can be implemented

**⚠️ CRITICAL**: No user story work can begin until this phase is complete

- [ ] T005 Define .smsg syntax IR types in soul_macros/src/ir.rs (SmsgFile, MessageDef, Field, FieldType, PrimitiveType)
- [ ] T006 Implement .smsg lexer in soul_macros/src/parser/lexer.rs using nom
- [ ] T007 Implement .smsg parser in soul_macros/src/parser/smsg.rs (parse message definitions)
- [ ] T008 [P] Add SmsgParseError type in soul_macros/src/error.rs with miette integration
- [ ] T009 Create code generation trait in soul_macros/src/codegen/mod.rs
- [ ] T010 Implement Rust struct code generator in soul_macros/src/codegen/struct_gen.rs
- [ ] T011 [P] Add Serialize/Deserialize derive code generation in soul_macros/src/codegen/derive_gen.rs
- [ ] T012 Create #[smsg] attribute macro entry point in soul_macros/src/lib.rs
- [ ] T013 Create example .smsg file in crates/soul_msgs/smsg/chat.smsg for testing

**Checkpoint**: Foundation ready - user story implementation can now begin in parallel

---

## Phase 3: User Story 1 - Define Type-Safe Message Types (Priority: P1) 🎯 MVP

**Goal**: Implement .smsg file parsing and proc macro code generation for Rust struct definitions

**Independent Test**: A developer can create a .smsg file with message definitions and use the attribute macro to generate Rust structs with full IDE support including autocomplete and type checking.

### Implementation for User Story 1

- [ ] T014 [P] [US1] Add module declarations in crates/soul_macros/src/lib.rs
- [ ] T015 [US1] Implement #[smsg("path")] attribute macro to parse .smsg file and generate code
- [ ] T016 [US1] Test .smsg parser with chat.smsg example
- [ ] T017 [US1] Verify generated ChatMessage struct has fields: sender(String), content(String), timestamp(i64)
- [ ] T018 [US1] Add nested message type support (Position in RobotState)
- [ ] T019 [US1] Add array type support (float64[], int32[3])
- [ ] T020 [US1] Verify error messages show correct line/column for parse errors

**Checkpoint**: At this point, User Story 1 should be fully functional and testable independently

---

## Phase 4: User Story 2 - Use Existing Publish with Type-Safe Messages (Priority: P1)

**Goal**: Integrate .smsg generated types with existing Publisher API

**Independent Test**: A developer can use existing publisher with new message types defined in .smsg files.

### Implementation for User Story 2

- [ ] T021 [P] [US2] Add zenoh-ext Serialize impl to generated code in codegen/derive_gen.rs
- [ ] T022 [US2] Test publishing ChatMessage from .smsg using existing Publisher API
- [ ] T023 [US2] Verify type mismatch at compile time when wrong type is published

**Checkpoint**: At this point, User Stories 1 AND 2 should both work independently

---

## Phase 5: User Story 3 - Use Existing Subscribe with Type-Safe Messages (Priority: P1)

**Goal**: Integrate .smsg generated types with existing Subscriber API

**Independent Test**: A developer can subscribe to a topic and receive callbacks with fully-typed message objects from .smsg definitions.

### Implementation for User Story 3

- [ ] T024 [P] [US3] Add zenoh-ext Deserialize impl to generated code in codegen/derive_gen.rs
- [ ] T025 [US3] Test subscribing to ChatMessage using existing Subscriber API
- [ ] T026 [US3] Verify callback receives correct ChatMessage type

**Checkpoint**: At this point, User Stories 1, 2, AND 3 should all work independently

---

## Phase 6: User Story 4 - Query and Queryable with Type Safety (Priority: P2)

**Goal**: Implement type-safe querier and queryable for request-response patterns

**Independent Test**: A developer can declare a queryable that handles specific request/response types and a querier that sends requests with typed responses.

### Implementation for User Story 4

- [ ] T027 [P] [US4] Create Querier struct in crates/soul_node/src/query/querier.rs
- [ ] T028 [P] [US4] Create QuerierBuilder in crates/soul_node/src/query/querier.rs
- [ ] T029 [P] [US4] Create Queryable struct in crates/soul_node/src/query/queryable.rs
- [ ] T030 [P] [US4] Create QueryableBuilder in crates/soul_node/src/query/queryable.rs
- [ ] T031 [US4] Implement SoulQueryReplyBuilder for constructing responses
- [ ] T032 [US4] Add declare_querier method to SoulNode in crates/soul_node/src/base.rs
- [ ] T033 [US4] Add declare_queryable method to SoulNode in crates/soul_node/src/base.rs
- [ ] T034 [US4] Add query module to lib.rs exports
- [ ] T035 [US4] Test querier/queryable with ChatRequest/ChatResponse .smsg types

**Checkpoint**: At this point, User Stories 1-4 should all work independently

---

## Phase 7: User Story 5 - Use Existing Liveliness for Node Discovery (Priority: P2)

**Goal**: Integrate liveliness with type system (already implemented, verify integration)

**Independent Test**: A developer can declare a liveliness token to announce node presence and subscribe to receive online/offline events.

### Implementation for User Story 5

- [ ] T036 [US5] Verify existing liveliness implementation works with type-safe setup
- [ ] T037 [US5] Test node discovery and online/offline events

**Checkpoint**: All user stories should now be independently functional

---

## Phase 8: Polish & Cross-Cutting Concerns

**Purpose**: Improvements that affect multiple user stories

- [ ] T038 [P] Run cargo fmt and cargo clippy on all crates
- [ ] T039 Add unit tests for .smsg parser in soul_macros
- [ ] T040 [P] Add unit tests for Querier/Queryable in soul_node
- [ ] T041 Run integration tests (local_echo, liveliness) to verify nothing broke
- [ ] T042 Update quickstart.md with complete working examples

---

## Dependencies & Execution Order

### Phase Dependencies

- **Setup (Phase 1)**: No dependencies - can start immediately
- **Foundational (Phase 2)**: Depends on Setup completion - BLOCKS all user stories
- **User Stories (Phase 3+)**: All depend on Foundational phase completion
  - User stories can then proceed in parallel (if staffed)
  - Or sequentially in priority order (P1 → P2)
- **Polish (Final Phase)**: Depends on all desired user stories being complete

### User Story Dependencies

- **User Story 1 (P1)**: Can start after Foundational (Phase 2) - No dependencies on other stories
- **User Story 2 (P1)**: Can start after Foundational (Phase 2) - Depends on US1 completion (needs Serialize impl)
- **User Story 3 (P1)**: Can start after Foundational (Phase 2) - Depends on US1 completion (needs Deserialize impl)
- **User Story 4 (P2)**: Can start after Foundational (Phase 2) - Uses .smsg types, independent of US2/US3
- **User Story 5 (P2)**: Already implemented - verify integration

### Within Each User Story

- Models/System before integration
- Core implementation before integration tests
- Story complete before moving to next priority

### Parallel Opportunities

- All Setup tasks marked [P] can run in parallel
- All Foundational tasks marked [P] can run in parallel (within Phase 2)
- Once Foundational phase completes, User Stories 2, 3, 4 can start in parallel
- Models within a story marked [P] can run in parallel

---

## Parallel Example: User Story Implementation

```bash
# After Foundational (Phase 2) completes, launch in parallel:
# - US1: Complete .smsg system
# - US2: Publish integration (needs Serialize from US1)
# - US3: Subscribe integration (needs Deserialize from US1)
# - US4: Querier/Queryable (independent, uses .smsg types)

# For US4, these can run in parallel:
Task: "Create Querier struct in crates/soul_node/src/query/querier.rs"
Task: "Create Queryable struct in crates/soul_node/src/query/queryable.rs"
```

---

## Implementation Strategy

### MVP First (User Stories 1-3)

1. Complete Phase 1: Setup
2. Complete Phase 2: Foundational
3. Complete Phase 3: User Story 1
4. **STOP and VALIDATE**: Test .smsg parser and code generation
5. Complete Phase 4: User Story 2
6. Complete Phase 5: User Story 3
7. **STOP and VALIDATE**: Test publish/subscribe with .smsg types

### Incremental Delivery

1. Complete Setup + Foundational → Foundation ready
2. Add User Story 1 → Test → MVP!
3. Add User Story 2 → Test → Publish works with .smsg types
4. Add User Story 3 → Test → Subscribe works with .smsg types
5. Add User Story 4 → Test → Query/Reply works
6. Add User Story 5 → Test → Full feature set complete

### Parallel Team Strategy

With multiple developers:

1. Team completes Setup + Foundational together
2. Once Foundational is done:
   - Developer A: User Story 1 (.smsg system)
   - Developer B: User Story 2 & 3 (Pub/Sub integration)
   - Developer C: User Story 4 (Querier/Queryable)
3. Stories complete and integrate independently

---

## Notes

- [P] tasks = different files, no dependencies
- [Story] label maps task to specific user story for traceability
- Each user story should be independently completable and testable
- Commit after each task or logical group
- Stop at any checkpoint to validate story independently
- Avoid: vague tasks, same file conflicts, cross-story dependencies that break independence
