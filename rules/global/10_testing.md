---
trigger: always_on
name: 10_testing
---

# Testing

## Purpose

Define mandatory software testing standards.

---

## Scope

Applies to all applications, services, libraries, and APIs.

---

## Principles

- Test Early
- Test Automatically
- Repeatability
- Reliability
- Fast Feedback

---

# Rules

## MUST

### TEST-001

Every production feature MUST be testable.

### TEST-002

Automated tests MUST be executable without manual intervention.

### TEST-003

Every bug fix MUST include a regression test.

### TEST-004

Unit tests MUST be isolated.

### TEST-005

Integration tests MUST verify external integration.

### TEST-006

Tests MUST be deterministic.

### TEST-007

Tests MUST be repeatable.

### TEST-008

Test data MUST be isolated.

### TEST-009

Failing tests MUST block release.

### TEST-010

Tests MUST have clear assertions.

### TEST-011

Tests MUST have meaningful names.

### TEST-012

Tests MUST clean up created resources.

### TEST-013

Critical business logic MUST be covered by automated tests.

### TEST-014

Security-sensitive functionality MUST be tested.

### TEST-015

API contracts MUST be verified.

### TEST-030

Tests MUST achieve Coverage Level 2 (Branch Coverage). Every decision node (if, switch, loop) MUST have its TRUE and FALSE paths executed at least once.

### TEST-031

Unit tests for functions MUST apply Basis Path Testing (Tom McCabe). Calculate Cyclomatic Complexity `V(G) = P + 1` (where P is the number of predicate nodes) and ensure `C` basic paths are independently tested.

### TEST-032

Input data analysis MUST be performed at each node. Apply Equivalence Partitioning (for Happy Cases), Boundary Value Analysis (for Edge Cases), and test Negative Cases (e.g., incorrect types, empty, out of bounds).

### TEST-033

Gradle projects MUST configure `testLogging` to emit `passed`, `skipped`, and `failed` events in the console for CI/CD integration.
```groovy
tasks.named('test') {
	useJUnitPlatform()
	testLogging {
		events "passed", "skipped", "failed"
	}
}
```

### TEST-034

CI/CD deployment pipelines (e.g., GitHub Actions `deploy.yml`) MUST execute the test suite (e.g., `./gradlew test`) before building the final artifact.
They MUST also integrate a Test Reporter Action (e.g., `EnricoMi/publish-unit-test-result-action`) and upload the HTML report as an artifact to provide a visual summary of test results on the CI/CD interface.

---

## MUST NOT

### TEST-016

Do not ignore failing tests.

### TEST-017

Do not depend on execution order.

### TEST-018

Do not share mutable state between tests.

### TEST-019

Do not write flaky tests.

### TEST-020

Do not use production data.

### TEST-021

Do not disable tests permanently.

---

## SHOULD

### TEST-022

Keep tests independent.

### TEST-023

Keep tests fast.

### TEST-024

Prefer integration tests for critical workflows.

### TEST-025

Mock only external dependencies.

### TEST-026

Continuously execute automated tests.

---

## MAY

### TEST-027

Perform load testing.

### TEST-028

Perform chaos testing.

### TEST-029

Perform mutation testing.

---

# Test Categories

- Unit Test
- Integration Test
- Contract Test
- End-to-End Test
- Performance Test
- Security Test

---

# Anti-patterns

- Flaky Test
- Slow Test Suite
- Shared State
- Manual Verification
- Missing Regression Test
- Assertion-less Test
- Disabled Test

---

# Checklist

- [ ] Unit Test
- [ ] Integration Test
- [ ] Regression Test
- [ ] Repeatable
- [ ] Independent
- [ ] Deterministic
- [ ] Fast
- [ ] Clear Assertions

---

# References

- Google Testing Blog
- Martin Fowler – Test Pyramid
- OWASP Testing Guide
- Testing on the Toilet