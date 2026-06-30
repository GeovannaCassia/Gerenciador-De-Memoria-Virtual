# Feature Specification: Virtual Memory Simulator

**Feature Branch**: `002-virtual-memory-simulator`

**Created**: 2026-06-29

**Status**: Draft

**Input**: User description: "Desenvolver um simulador de Gerenciamento de Memória Virtual em linguagem C a partir do projeto-base fornecido pelo professor. O simulador deve traduzir endereços lógicos para endereços físicos utilizando Tabela de Páginas, TLB e Memória Física, implementando paginação por demanda e substituição de páginas utilizando o algoritmo Aging (LRU aproximado)."

## User Scenarios & Testing *(mandatory)*

### User Story 1 - Translate logical addresses to physical addresses with demand paging (Priority: P1)

A student or tester provides a sequence of logical addresses and expects the simulator to resolve each one into a physical address by using page table lookup, TLB lookup, and demand paging backed by `BACKING_STORE.bin`.

**Why this priority**: This is the core functionality of the simulator and the feature required for correct end-to-end address translation.

**Independent Test**: A trace file of logical addresses can be executed and the resulting physical addresses and data bytes can be compared against expected output.

**Acceptance Scenarios**:

1. **Given** a valid logical address, **When** the simulator processes it, **Then** it uses only the lower 16 bits of the address and resolves the page number and offset.
2. **Given** a page table miss for a page that is not currently resident, **When** the simulator processes the logical address, **Then** it loads the page from `BACKING_STORE.bin`, updates physical memory and the page table, and continues translation.
3. **Given** a resident page, **When** the simulator processes the logical address, **Then** it returns the correct physical address and the byte stored at that address.

---

### User Story 2 - Use a TLB with FIFO replacement to reduce page table lookups (Priority: P2)

A simulator user expects repeated page references to benefit from a 16-entry TLB that returns hits for recently inserted page/frame mappings and replaces the oldest entry first.

**Why this priority**: TLB behavior is essential for matching the simulator's expected performance characteristics and validating hit/miss statistics.

**Independent Test**: A crafted address sequence exercises repeated page references and confirms TLB hit counts and FIFO eviction ordering.

**Acceptance Scenarios**:

1. **Given** repeated accesses to the same page, **When** the simulator processes the addresses, **Then** the TLB returns a hit after the first translation and counts the hit.
2. **Given** more than 16 distinct pages accessed without reuse, **When** the simulator inserts TLB entries, **Then** the TLB evicts the oldest entry first.

---

### User Story 3 - Apply Aging-based replacement when physical memory is full (Priority: P3)

When all 128 physical frames are occupied, the simulator must select a victim page to evict using the Aging algorithm, approximating least recently used behavior while respecting reference bits.

**Why this priority**: This ensures the memory simulator can continue translating addresses after physical memory becomes saturated and complies with the required replacement policy.

**Independent Test**: A memory trace that fills all frames and then continues with page references can be validated against expected page evictions and residency state.

**Acceptance Scenarios**:

1. **Given** a page fault with no free physical frames, **When** the simulator must load a new page, **Then** it selects the resident page with the smallest 8-bit aging counter.
2. **Given** an accessed resident page, **When** translation completes, **Then** the page's reference bit is set to 1 and its aging counter updates on the next aging cycle.
3. **Given** a page has been evicted, **When** a later access references that page, **Then** it is treated as a page fault and reloaded from `BACKING_STORE.bin`.

### Edge Cases

- What happens when a logical address is larger than 16 bits? The simulator must ignore the upper 16 bits and operate only on the lower 16 bits.
- What happens when the TLB is full and a new mapping is inserted? The simulator must evict the oldest TLB entry and invalidate stale page/table mappings only when the corresponding page is removed.
- What happens when a page is resident but its reference bit has not yet been incorporated into its aging counter? The aging algorithm must update counters on each maintenance step and then clear the reference bit.

## Requirements *(mandatory)*

### Functional Requirements

- **FR-001**: The simulator MUST accept 32-bit logical addresses and use only the least significant 16 bits for translation.
- **FR-002**: The simulator MUST extract an 8-bit page number and an 8-bit offset from the 16-bit logical address using bitwise operations.
- **FR-003**: The simulator MUST read page contents from `BACKING_STORE.bin` using `fopen`, `fread`, `fseek`, and `fclose`.
- **FR-004**: The simulator MUST implement a page table with 256 entries, each storing a frame number and a validity bit.
- **FR-005**: The simulator MUST implement physical memory with 128 frames, each 256 bytes in size, managing free and occupied frames.
- **FR-006**: The simulator MUST handle page faults by loading missing pages from backing store and updating the page table, physical memory, and TLB.
- **FR-007**: The simulator MUST implement the Aging page replacement policy with an 8-bit counter and a reference bit for each resident page.
- **FR-008**: The simulator MUST increment reference bits on every access and age resident pages by shifting counters right and inserting the reference bit at the MSB.
- **FR-009**: The simulator MUST use a 16-entry TLB with FIFO replacement and valid bits for page-to-frame mappings.
- **FR-010**: The simulator MUST calculate total address references, page faults, and TLB hits during execution.
- **FR-011**: The simulator MUST compute page fault rate and TLB hit rate at the end of execution.
- **FR-012**: The simulator MUST preserve the existing project structure and public interfaces while implementing only the TODO-marked functionality.
- **FR-013**: The simulator MUST preserve the existing output format for final results.

### Non-Functional Requirements

- **NFR-001**: The implementation MUST use standard C99 and avoid external libraries.
- **NFR-002**: The code MUST remain compatible with Windows and Linux build environments.
- **NFR-003**: The implementation MUST preserve the existing project file layout and not introduce new top-level directories.
- **NFR-004**: The implementation MUST keep coupling low by separating page table, TLB, physical memory, and backing store logic.
- **NFR-005**: The implementation MUST only modify code areas marked with TODO comments when possible.

### Key Entities *(include if feature involves data)*

- **Logical Address**: 32-bit input value where only the 16 least significant bits are used for translation.
- **Page Number**: 8-bit value derived from the lower 16 bits of the logical address.
- **Offset**: 8-bit value derived from the lower 16 bits of the logical address.
- **Page Table Entry**: Structure containing a frame number and a valid bit.
- **Physical Frame**: 256-byte block that stores page contents loaded from backing store.
- **TLB Entry**: 16-entry mapping containing page number, frame number, and valid bit.
- **Aging Counter**: 8-bit counter associated with each resident page for replacement selection.
- **Reference Bit**: Flag set when a resident page is accessed and incorporated during aging updates.

## Success Criteria *(mandatory)*

### Measurable Outcomes

- **SC-001**: The simulator MUST translate at least 99% of provided test addresses correctly when compared to expected physical addresses and data bytes.
- **SC-002**: The simulator MUST compute page fault rate and TLB hit rate with the same format and values expected by the existing test harness.
- **SC-003**: The simulator MUST perform correct page replacement using Aging when physical memory is full, evicting the resident page with the lowest 8-bit counter.
- **SC-004**: The simulator MUST maintain TLB FIFO semantics, evicting the oldest entry when inserting into a full table.
- **SC-005**: The simulator MUST compile successfully on both Windows and Linux without requiring external dependencies.

## Assumptions

- The project already includes a trace-driven entry point and output format that the implementation should preserve.
- The simulator can store metadata for page residency and aging state per page table entry or through a dedicated resident-page structure.
- Page replacement decisions are evaluated only when a new page must be loaded and no physical frames are free.
- The backing store file is accessible at runtime as `BACKING_STORE.bin` and contains 256 pages of 256 bytes each.

## Out of Scope

- Adding new command-line interfaces outside the existing project harness.
- Changing the existing output formatting or statistics layout.
- Introducing new external dependencies or libraries.
- Implementing alternative page replacement policies other than Aging.
- Expanding the simulator to support more than 256 logical pages or more than 128 physical frames.
