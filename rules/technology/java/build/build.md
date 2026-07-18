---
trigger: always_on
---

# Java Build Rules

## Purpose

Define mandatory standards for building Java projects, managing Gradle dependencies, and ensuring build stability.

---

## Scope

Applies to all Java projects built using Gradle or Maven.

---

# Rules

## MUST

### BUILD-001

Use the Gradle Wrapper (`gradlew` or `gradlew.bat`) for all build operations. Never use globally installed Gradle directly.

### BUILD-002

Specify exact versions for all dependencies. Do not use dynamic versions (e.g. `1.+`, `latest`) or snapshot versions in production releases.

### BUILD-003

Declare explicit `group` and `version` properties in the build configuration.

### BUILD-004

Do not mix dependency management strategies. Use Spring Boot dependency management plugins uniformly.

### BUILD-005

Configure JVM toolchain to enforce a specific Java version (e.g., Java 25) aligned across all developers and CI/CD environments.

---

## MUST NOT

### BUILD-006

Do not commit IDE-specific build folders (e.g., `.gradle`, `build/`, `.idea/`, `out/`, `bin/`) to the repository.

### BUILD-007

Do not bypass unit tests during release builds (e.g., do not use `-x test` without explicit justification).

---

## SHOULD

### BUILD-008

Define separate configurations for testing and production dependencies to keep production artifacts slim.

### BUILD-009

Verify build and run local unit tests (`./gradlew test`) before pushing changes to the repository.

---

# Checklist

- [ ] Gradle Wrapper Used
- [ ] Explicit Dependency Versions (No dynamic/wildcard versions)
- [ ] Group and Version Defined
- [ ] IDE Build Artifacts Excluded (.gitignore)
- [ ] Unit Tests Executed

---

# References

- Gradle User Manual
- Spring Boot Gradle Plugin Reference Guide
