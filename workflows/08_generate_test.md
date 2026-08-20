---
description: Generate comprehensive tests for Java Spring Boot applications.
name: 08_generate_test
trigger: always_on
---

# Generate Test Workflow

## Purpose

Generate comprehensive tests for Java Spring Boot applications.

Tests must verify correctness without changing production code.

---

## Input

- Source Code
- API Blueprint (Optional)
- Existing Tests (Optional)

---

## Preconditions

The target code is available.

If the expected behavior cannot be determined,

STOP.

Ask the user.

---

## Rule Files

### Read Before Starting

- `rules/global/10_testing.md`
- `rules/technology/java/testing/testing.md`
- `rules/technology/java/core/java.md`
- `rules/templates/java/structure.md`

### Read If Related

- `rules/technology/java/persistence/persistence.md`
- `rules/technology/java/spring/spring-boot.md`
- `rules/technology/java/spring/spring-data.md`
- Infrastructure rules matching the test scope

---

## Steps

### Step 0 — Load Rules

Read all rule files listed in the Rule Files section above.

Understand testing standards from `10_testing.md` and `testing.md`.

Understand the project structure from `structure.md` to place test classes in the correct packages.

---

### Step 1 — Understand the Target

Identify

- Public Methods
- Business Rules
- Validation Rules

---

### Step 2 — Control Flow Graph & Complexity Analysis

Analyze the logical flow of the target function to satisfy TEST-031:

- Draw/list the Control Flow Graph (CFG) nodes and edges.
- Calculate Cyclomatic Complexity `V(G) = P + 1`.
- List exactly `V(G)` Basis Paths that cover all independent execution paths.

---

### Step 2.5 — Data Input & Edge Case Mapping

For each Basis Path identified, define the specific input data required to trigger it to satisfy TEST-032:

- **Happy Cases**: Use Equivalence Partitioning for expected inputs.
- **Edge Cases**: Use Boundary Value Analysis for limits.
- **Negative Cases**: Nulls, empty strings, incorrect types, strings exceeding max length.

---

### Step 3 — Determine Test Type

Based on the target component and `structure.md`

- Unit Test (Service, Validator, Mapper)
- Integration Test (Repository, external integrations)
- Controller Test (API endpoint testing)

---

### Step 4 — Generate Test Cases

Follow testing rules from `testing.md`. Ensure that each generated test method explicitly states which Basis Path it covers.

Ensure:

- Independent and repeatable execution.
- No `@Data` on test fixtures with entities per `PERSIST-028a`.
- Test naming follows `NAME-xxx` conventions and clearly indicates the scenario/path.
- Data input boundaries are clearly mocked or provided.

---

### Step 5 — Review Coverage

Check that the generated tests satisfy TEST-030 and TEST-031:

- Are there at least `V(G)` tests corresponding to the Basis Paths?
- Are all decision branches (True and False) covered (Coverage Level 2)?
- Are all boundary values and negative inputs handled?
- Are business rules and exceptions covered?

---

## Human Approval Gate

STOP if generating tests requires

- Changing Production Code
- Changing Business Logic
- Modifying APIs
- Changing Database Schema

Ask the user before proceeding.

---

## Output

- Test Classes
- Test Cases
- Coverage Summary
- Untested Areas

---

## Validation Checklist

- [ ] Rules loaded before test generation
- [ ] Testing rules from `testing.md` followed
- [ ] Test classes placed in correct packages per `structure.md`
- [ ] Happy path covered
- [ ] Validation covered
- [ ] Exceptions covered
- [ ] Edge cases covered
- [ ] Critical paths covered
- [ ] Production code unchanged