# Upgrade Plan: asset-manager (20260320075640)

- **Generated**: 2026-03-20 07:56
- **HEAD Branch**: main
- **HEAD Commit ID**: dc8ae57

## Available Tools

**JDKs**
- JDK 1.8.0_482: /usr/local/sdkman/candidates/java/8.0.482-tem/bin (current project JDK, used in Step 2 baseline)
- JDK 21: **<TO_BE_INSTALLED>** (required by Step 3 onwards for target Java 21)

**Build Tools**
- Maven 3.9.14: /usr/local/sdkman/candidates/maven/3.9.14/bin (compatible with Java 21)
- Maven Wrapper: 3.9.9 (in `.mvn/wrapper/maven-wrapper.properties`) — compatible with Java 21, no upgrade needed

## Guidelines

> Note: You can add any specific guidelines or constraints for the upgrade process here if needed, bullet points are preferred.

## Options

- Working branch: appmod/java-upgrade-20260320075640
- Run tests before and after the upgrade: true

## Upgrade Goals

- Upgrade Java from 8 to 21
- Upgrade Spring Boot from 2.7.18 to 3.5.x

### Technology Stack

| Technology/Dependency | Current | Min Compatible | Why Incompatible |
| --------------------- | ------- | -------------- | ---------------- |
| Java | 8 | 21 | User requested |
| Spring Boot | 2.7.18 | 3.5.x | User requested |
| Spring Framework | 5.3.x | 6.2.x | Spring Boot 3.5 requires Spring Framework 6.2+ |
| Hibernate | 5.6.x | 6.4.x | Spring Boot 3.5 requires Hibernate 6.4+ |
| javax.persistence ⚠️ EOL | javax.* namespace | N/A | Replaced by jakarta.persistence in Spring Boot 3.x |
| javax.servlet ⚠️ EOL | javax.servlet.* | N/A | Replaced by jakarta.servlet in Spring Boot 3.x |
| javax.annotation.PostConstruct ⚠️ EOL | javax.annotation.* | N/A | Replaced by jakarta.annotation in Spring Boot 3.x |
| javax.imageio | javax.imageio.* | N/A | Part of JDK standard library — stays as javax.imageio (not a Jakarta EE package) |
| Lombok | 1.18.x | 1.18.26 | Spring Boot 3.5 BOM provides compatible version |
| AWS SDK v2 | 2.25.13 | 2.25.13 | Compatible — no migration needed |
| PostgreSQL JDBC | managed | managed | Spring Boot BOM manages compatible version |
| maven-compiler-plugin | (managed) | 3.11.0 | Older versions cannot compile Java 21 bytecode |
| maven-surefire-plugin | (managed) | 3.1.0 | Older versions may fail with Java 21 module system |

### Derived Upgrades

- Upgrade Spring Framework from 5.3.x to 6.2.x (Spring Boot 3.5 requires Spring Framework 6.2)
- Upgrade Hibernate from 5.6.x to 6.4.x (Spring Boot 3.5 requires Hibernate 6.4)
- Migrate javax.persistence.* → jakarta.persistence.* in both web and worker modules (Spring Boot 3.x uses Jakarta EE)
- Migrate javax.servlet.* → jakarta.servlet.* in web module (Spring Boot 3.x uses Jakarta EE)
- Migrate javax.annotation.PostConstruct → jakarta.annotation.PostConstruct in web and worker modules
- Upgrade maven-compiler-plugin to 3.13+ (required for Java 21 target/source)
- Upgrade maven-surefire-plugin to 3.2+ (recommended for Java 21)
- Spring Boot 3.5 BOM will automatically manage compatible versions of Lombok, PostgreSQL JDBC, Jackson, etc.

## Upgrade Steps

- **Step 1: Setup Environment**
  - **Rationale**: Install JDK 21 required for Java 21 target compilation and final validation.
  - **Changes to Make**:
    - [ ] Install JDK 21 via SDKMAN
  - **Verification**:
    - Tool: `#appmod-list-jdks` to confirm JDK 21 is available
    - Expected: JDK 21 listed at a valid path

---

- **Step 2: Setup Baseline**
  - **Rationale**: Establish pre-upgrade compile and test results as acceptance criteria.
  - **Changes to Make**:
    - [ ] Run baseline compilation with JDK 8
    - [ ] Run baseline tests with JDK 8
  - **Verification**:
    - Command: `JAVA_HOME=/usr/local/sdkman/candidates/java/8.0.482-tem ./mvnw clean test-compile -q && ./mvnw clean test -q`
    - JDK: /usr/local/sdkman/candidates/java/8.0.482-tem
    - Expected: Document baseline compile and test pass rate

---

- **Step 3: Upgrade Spring Boot to 3.5.x and Java to 21**
  - **Rationale**: Core framework upgrade from Spring Boot 2.7.18 → 3.5.x with Java 8 → 21. Spring Boot 3.5 is the target — since current project is already on Spring Boot 2.7.18 (the best 2.x intermediate), a direct upgrade to 3.5.x is feasible. The BOM will automatically pull in Hibernate 6.4, Spring Framework 6.2, and their compatible dependencies.
  - **Changes to Make**:
    - [ ] Update `spring-boot-starter-parent` to `3.5.0` in root `pom.xml`
    - [ ] Update `<java.version>` from `8` to `21` in root `pom.xml`
    - [ ] Add explicit `maven-compiler-plugin` version 3.13.0 with Java 21 source/target
    - [ ] Add explicit `maven-surefire-plugin` version 3.2.5
  - **Verification**:
    - Command: `JAVA_HOME=/usr/local/sdkman/candidates/java/current ./mvnw clean test-compile -q` (tests may fail; fix in next step)
    - JDK: JDK 21 (installed in Step 1)
    - Expected: Compilation may fail due to javax→jakarta namespace issues — will be fixed in Step 4

---

- **Step 4: Migrate javax→jakarta Namespace**
  - **Rationale**: Spring Boot 3.x uses Jakarta EE 10 which replaces all `javax.*` (EE) packages with `jakarta.*`. Must migrate: `javax.persistence.*`, `javax.servlet.*`, `javax.annotation.PostConstruct`. Note: `javax.imageio.*` is part of the JDK standard library and does NOT change.
  - **Changes to Make**:
    - [ ] Migrate `javax.persistence.*` → `jakarta.persistence.*` in `web/src/main/java` and `worker/src/main/java`
    - [ ] Migrate `javax.servlet.*` → `jakarta.servlet.*` in `web/src/main/java`
    - [ ] Migrate `javax.annotation.PostConstruct` → `jakarta.annotation.PostConstruct` in web and worker modules
    - [ ] Fix any remaining compilation errors from API changes in Spring Boot 3.x / Hibernate 6.x
  - **Verification**:
    - Command: `JAVA_HOME=/usr/local/sdkman/candidates/java/current ./mvnw clean test-compile -q`
    - JDK: JDK 21
    - Expected: Compilation SUCCESS for both main and test sources

---

- **Step 5: Final Validation**
  - **Rationale**: Verify all upgrade goals are met, all code compiles, and all tests pass.
  - **Changes to Make**:
    - [ ] Verify `spring-boot-starter-parent` is 3.5.x and `java.version` is 21
    - [ ] Resolve any remaining TODOs and temporary workarounds
    - [ ] Run full test suite and fix ALL failures (iterative fix loop until 100% pass)
  - **Verification**:
    - Command: `JAVA_HOME=/usr/local/sdkman/candidates/java/current ./mvnw clean test`
    - JDK: JDK 21
    - Expected: Compilation SUCCESS + 100% tests pass

## Key Challenges

- **Jakarta EE Namespace Migration**
  - **Challenge**: Multiple source files across `web` and `worker` modules use `javax.persistence.*`, `javax.servlet.*`, and `javax.annotation.*` that must become `jakarta.*` in Spring Boot 3.x.
  - **Strategy**: Perform targeted find-and-replace across all Java source files. Note that `javax.imageio.*` (JDK standard library) stays unchanged.

- **Hibernate 5→6 Breaking Changes**
  - **Challenge**: Hibernate 6 has removed deprecated APIs and changed query/schema behavior. Entity annotations are now in `jakarta.persistence` namespace.
  - **Strategy**: The namespace migration handles most issues. If custom queries or schema validation errors appear, fix during Final Validation.

- **Spring Boot 3.x Servlet API Changes**
  - **Challenge**: `WebMvcConfig.java` uses `javax.servlet.http.HttpServletRequest/Response` which must become `jakarta.servlet`.
  - **Strategy**: Straightforward namespace substitution with no API signature changes.

## Plan Review

- All 5 steps are complete and sequential with no gaps.
- Java 8 → 21 and Spring Boot 2.7.18 → 3.5.x are the stated user goals.
- The plan uses a direct upgrade (no intermediate Spring Boot version needed) since the project is already on Spring Boot 2.7.18, the ideal 2.x bridge version.
- No known unfixable limitations. AWS SDK v2 (2.25.13) is Jakarta-compatible and requires no changes.
