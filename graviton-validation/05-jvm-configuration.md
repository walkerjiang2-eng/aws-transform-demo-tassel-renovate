# Graviton JVM Configuration

## Target Java Version
JDK 21 (Gradle toolchain `JavaLanguageVersion.of(21)`, `jvmTarget = JVM_21`).
Validated with Eclipse Temurin 21.0.13 on linux/arm64. The JDK distribution and version were not changed.

## Where Flags Are Applied
The only JVM flag configuration in the repository is the Gradle `test` task
`jvmArgs` in `build.gradle.kts`. The repository has no launcher script,
Dockerfile, systemd unit or Kubernetes manifest. The flags were added in that
existing location next to `-Xmx10g`:

```kotlin
jvmArgs = listOf<String>(
    "-Xmx10g",
    "-XX:ReservedCodeCacheSize=64M",
    "-XX:InitialCodeCacheSize=64M",
    "-Xss1m",
)
```

For production runs, add the same flags to the `java` command that launches `sTASSEL.jar`.

## Applied JVM Flags
| Flag | Applicable JDK Versions | Workload Type | Expected Impact |
|------|------------------------|---------------|-----------------|
| -XX:ReservedCodeCacheSize=64M | 11, 17, 21+ | General | Smaller, better-localised code cache on Graviton |
| -XX:InitialCodeCacheSize=64M | 11, 17, 21+ | General | Avoids code-cache growth at runtime |
| -Xss1m | 11, 17, 21+ | Multi-threaded (TASSEL plugins use thread pools) | Lower per-thread stack memory (aarch64 default is 2 MB) |
| -Xmx10g (pre-existing) | all | — | Unchanged |

## Flags NOT Applied (Version-Gated)
| Flag | Reason Not Applied |
|------|--------------------|
| -XX:-TieredCompilation | Helps only lock-contended workloads and can slow startup. Benchmark before adopting it (the finding also says to benchmark). |
| -XX:CompilationMode=high-only | Not recommended for JDK 21+ |
| -XX:CICompilerCount=2 | JDK 21: test first; no benchmark evidence |
| -XX:+UnlockDiagnosticVMOptions -XX:+UseAESCTRIntrinsics | Enabled by default on JDK 21+ |

## Flag Validation
Run on linux/arm64 (`uname -m` = aarch64), eclipse-temurin:21-jdk:
```
$ java -Xmx10g -XX:ReservedCodeCacheSize=64M -XX:InitialCodeCacheSize=64M -Xss1m -version
openjdk version "21.0.13" 2026-10-20 LTS
flags-exit=0
```
All flags were accepted. The Gradle test executor ran with these flags, and
`TasselPipeline` started with them (see 06-build-test-results.md).

## Configuration Profiles
There is one profile (the general flags above). `-XX:-TieredCompilation` can
be added for long-running, lock-contended analyses after benchmarking.
