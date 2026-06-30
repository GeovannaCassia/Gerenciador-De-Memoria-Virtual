# Tasks: Virtual Memory Simulator

**Input**: Design documents from `/specs/002-virtual-memory-simulator/`

---

## Phase 1: Setup (Shared Infrastructure)

**Purpose**: Prepare the implementation context and verify build/test readiness.

- [ ] T001 [P] Review `specs/002-virtual-memory-simulator/spec.md` and `specs/002-virtual-memory-simulator/plan.md` for implementation details
- [ ] T002 [P] Confirm existing `Makefile` and build environment support C99 on Windows and Linux
- [ ] T003 [P] Locate all TODO annotations in `src/main.c`, `src/memory.c`, `src/page_table.c`, `src/tlb.c`, and `src/statistics.c`

---

## Phase 2: Foundational (Blocking Prerequisites)

**Purpose**: Implement the core subsystems that all user stories depend on.

- [ ] T004 [P] Implement physical memory frame management in `src/memory.c` and `include/memory.h`
- [ ] T005 [P] Implement page table operations in `src/page_table.c` and `include/page_table.h`, including frame number, valid bit and page residency metadata
- [ ] T006 [P] Implement backing store page loading from `BACKING_STORE.bin` in `src/memory.c` using random access (`fopen`, `fseek`, `fread`, `fclose`)
- [ ] T007 Implement page number and offset extraction in `src/main.c`
- [ ] T008 Implement the logical-to-physical address translation pipeline, connecting page extraction, page table lookup and physical address calculation (without demand paging)

---

## Phase 3: User Story 1 - Translate logical addresses to physical addresses with demand paging (Priority: P1)

**Goal**: Deliver end-to-end address translation using page table lookup, demand paging, and backing store loading.

**Independent Test**: Verify a trace of logical addresses resolves to correct physical addresses and returned bytes using only the core translation path.

- [ ] T009 [US1] Implement page fault detection and coordination with backing store loading in `src/main.c`
- [ ] T010 [US1] Allocate a free physical frame and copy the loaded page into physical memory in `src/memory.c`
- [ ] T011 [US1] Update the page table after loading a page from `BACKING_STORE.bin` in `src/page_table.c`
- [ ] T012 [US1] Complete the translation flow in `src/main.c` so a page table hit returns `physical_address = frame * 256 + offset`
- [ ] T013 [US1] Ensure the existing output format for translated addresses and loaded bytes remains unchanged

---

## Phase 4: User Story 2 - Use a TLB with FIFO replacement to reduce page table lookups (Priority: P2)

**Goal**: Deliver a 16-entry TLB with FIFO replacement that accelerates repeated page lookups.

**Independent Test**: Confirm repeated page references hit the TLB and that the oldest entries are evicted after 16 distinct pages.

- [ ] T014 [US2] Implement TLB lookup, insertion and invalidation routines in `src/tlb.c` and `include/tlb.h`, returning HIT or MISS without consulting the page table
- [ ] T015 [US2] Implement FIFO eviction logic for the 16-entry TLB in `src/tlb.c`
- [ ] T016 [US2] Integrate TLB lookup into the translation flow in `src/main.c`
- [ ] T017 [US2] Update TLB hit counting and statistics integration in `src/statistics.c` and `src/main.c`
- [ ] T018 [US2] Invalidate stale TLB entries whenever a page is removed from physical memory

---

## Phase 5: User Story 3 - Apply Aging-based replacement when physical memory is full (Priority: P3)

**Goal**: Implement Aging page replacement so the simulator continues when all 128 frames are occupied.

**Independent Test**: Validate victim selection by having the simulator evict the resident page with the smallest 8-bit aging counter.

- [ ] T019 [US3] Add per-page aging counter and reference bit support to `src/page_table.c` and `include/page_table.h`
- [ ] T020 [US3] Mark the page reference bit whenever a resident page is accessed
- [ ] T021 [US3] Implement periodic aging counter updates by shifting counters, inserting the reference bit into the most significant bit and clearing the reference bit
- [ ] T022 [US3] Implement victim selection using the Aging algorithm when no free frames remain
- [ ] T023 [US3] Invalidate page table entries and corresponding TLB entries for evicted pages

---

## Phase 6: Polish & Cross-Cutting Concerns

**Purpose**: Final integration, statistics, validation, and cleanup.

- [ ] T024 Implement total references, page faults and TLB hits counters in `src/statistics.c` and `include/statistics.h`
- [ ] T025 Implement final Page Fault Rate and TLB Hit Rate calculations in `src/statistics.c`
- [ ] T026 Integrate statistics updates throughout address translation, page faults and TLB events in `src/main.c`
- [ ] T027 Run `make` (or `mingw32-make` on Windows) and execute the provided address traces to validate address translation and statistics
- [ ] T028 Validate calculated statistics against the expected results from the provided input traces
- [ ] T029 Perform code cleanup, resolve remaining TODOs and ensure the project compiles without warnings or errors

---

## Dependencies & Execution Order

- Phase 1 tasks are preparatory and can run in parallel.
- Phase 2 must be completed before implementing any user story.
- User Story 1 (Phase 3) is the MVP and enables all remaining functionality.
- User Story 2 depends on the complete translation flow from User Story 1.
- User Story 3 depends on the page table, physical memory management and page fault handling from User Story 1.
- Phase 6 depends on completion of all previous phases.

---

## Parallel Opportunities

- `T001`, `T002` and `T003` can run in parallel.
- `T004`, `T005` and `T006` can run in parallel because they implement independent foundational subsystems.
- `T014` and `T015` can be developed independently from `T019` and `T020` until integration.
- `T024` and `T025` can be implemented independently after the translation pipeline is stable.