# Upgrade Summary: asset-manager (20260320075640)

- **Completed**: 2026-03-20 08:14
- **Plan Location**: `.github/java-upgrade/20260320075640/plan.md`
- **Progress Location**: `.github/java-upgrade/20260320075640/progress.md`

## Upgrade Result

| Metric     | Baseline                        | Final                           | Status |
| ---------- | ------------------------------- | ------------------------------- | ------ |
| Compile    | ✅ SUCCESS                      | ✅ SUCCESS                      | ✅     |
| Tests      | 3/4 passed (1 pre-existing err) | 4/4 passed (100%)               | ✅     |
| JDK        | JDK 8 (1.8.0_482)               | JDK 21 (21.0.8)                 | ✅     |
| Build Tool | Maven Wrapper 3.9.9             | Maven Wrapper 3.9.9             | ✅     |

**Upgrade Goals Achieved**:
- ✅ Java 8 → 21
- ✅ Spring Boot 2.7.18 → 3.5.0
- ✅ Spring Framework 5.3.x → 6.2.x (via Spring Boot BOM)
- ✅ Hibernate 5.6.x → 6.4.x (via Spring Boot BOM)
- ✅ Jakarta EE namespace migration complete (javax.* → jakarta.*)

## Tech Stack Changes

| Dependency | Before | After | Reason |
| ---------- | ------ | ----- | ------ |
| Java | 8 | 21 | User requested |
| Spring Boot | 2.7.18 | 3.5.0 | User requested |
| Spring Framework | 5.3.x | 6.2.x | Spring Boot 3.5 requires Spring Framework 6.2 |
| Hibernate | 5.6.x | 6.4.x | Spring Boot 3.5 requires Hibernate 6.4 |
| Lombok | 1.18.x | 1.18.38 | Managed by Spring Boot 3.5 BOM |
| PostgreSQL JDBC | managed | 42.7.5 | Managed by Spring Boot 3.5 BOM |
| Jackson | 2.x | 2.19.0 | Managed by Spring Boot 3.5 BOM |
| maven-compiler-plugin | (managed) | 3.13.0 | Explicit version pinned for Java 21 |
| maven-surefire-plugin | (managed) | 3.2.5 | Explicit version pinned for Java 21 |
| javax.servlet | javax namespace | jakarta namespace | jakarta.servlet required by Spring Boot 3.x |
| javax.persistence | javax namespace | jakarta namespace | jakarta.persistence required by Spring Boot 3.x |
| javax.annotation | javax namespace | jakarta namespace | jakarta.annotation required by Spring Boot 3.x |

## Commits

| Step | Commit | Message |
| ---- | ------ | ------- |
| 3+4 | 4456a63 | Step 3+4: Upgrade Spring Boot 2.7.18→3.5.0, Java 8→21, migrate javax→jakarta |

## CVE Scan Results

| Severity | CVE | Dependency | Description | Action |
| -------- | --- | ---------- | ----------- | ------ |
| HIGH | CVE-2025-49146 | postgresql:42.7.5 | pgjdbc allows fallback to insecure authentication despite `channelBinding=require` configuration — MITM risk | Upgrade postgresql driver once a patched version is released. Workaround: set `sslMode=verify-full` in connection configuration. |

## Test Coverage

Test coverage collection skipped — project does not have JaCoCo configured. Test execution results: 4/4 passed (100%).

## Challenges

- **Jakarta EE Namespace Migration**: All `javax.persistence.*`, `javax.servlet.*`, and `javax.annotation.PostConstruct` usages required migration to `jakarta.*`. Additionally, `WebMvcConfigurerAdapter` and `HandlerInterceptorAdapter` were removed in Spring 6 and replaced with `WebMvcConfigurer` and `HandlerInterceptor` interfaces.
  - **Resolution**: Applied targeted find-and-replace across 5 source files. `javax.imageio.*` (JDK standard library) was correctly left unchanged.

- **Pre-existing Test Failure**: Baseline had 1 failing test — Mockito 4.x could not mock `ResponseInputStream` (a final class) from AWS SDK without inline mock-maker.
  - **Resolution**: Under JDK 21, Mockito 5.x (bundled with Spring Boot 3.5 BOM) self-attaches as a Java agent and enables inline mocking by default. All 4 tests now pass.

## Limitations

None — all upgrade goals achieved and all tests pass.

## Next Steps

1. **Address CVE-2025-49146**: Monitor for a patched `postgresql` JDBC driver release. In the meantime, configure `sslMode=verify-full` in `application.properties` if channel binding protection is required.
2. **Add JaCoCo**: Consider adding the JaCoCo Maven plugin to track test coverage metrics going forward.
3. **Suppress Mockito JDK 21 warnings**: Add `-XX:+EnableDynamicAgentLoading` JVM arg or configure `mockito-agent` explicitly to silence the dynamic agent loading warnings on JDK 21.
