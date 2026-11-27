# ARM64 Compatibility Validation — Summary

## Project Overview
TASSEL 5.2.96 (`net.maizegenetics`) is a single-module Gradle (Kotlin DSL)
project with Java and Kotlin sources and a JDK 21 toolchain. It is deployed on
hosts (no Dockerfile, launcher script or CI configuration in the repository).
Two low-severity findings were provided:

| Finding ID | Title | Resolution |
|------------|-------|------------|
| 01M818BH0WJ96PGGMN1FZPKZEG | Transitive cisd:base natives (libunix, libnativedata) unconfirmed for aarch64 | Confirmed: there is no aarch64-Linux native in the only published version (18.09.0). Pure-Java fallbacks accepted and verified on aarch64. No build change possible. |
| 01M818BH70CRPH4HGK9Z254EWG | No Graviton-tuned JVM flags defined | Added `-XX:ReservedCodeCacheSize=64M -XX:InitialCodeCacheSize=64M -Xss1m` to the existing `test` `jvmArgs` in build.gradle.kts and validated them on aarch64 JDK 21. `-XX:-TieredCompilation` was not applied because it needs benchmarking. |

## Exit Criteria Checklist
| # | Criterion | Status | Notes |
|---|-----------|--------|-------|
| 1 | Compiles on ARM64 without errors | ✅ PASS | `clean build -x test` passes on linux/arm64. This needs a validation-only exclusion of the unresolvable `openchart:openchart:1.4.2`, which is pre-existing and affects every architecture |
| 2 | Native libraries load on ARM64 | ⚠️ PARTIAL | cisd:base: pure-Java fallbacks work. TasselBlas: EJML fallback works. cisd:jhdf5: no aarch64-Linux libjhdf5.so exists and there is no fallback (not a finding of this run) |
| 3 | Blocking dependency updates applied | ✅ PASS | No MUST UPGRADE dependencies in the findings. cisd:base cannot be upgraded (one version only) |
| 4 | No ARM64-specific runtime errors | ⚠️ PARTIAL | HDF5 operations fail on aarch64-Linux (libjhdf5 missing). Everything else runs |
| 5 | Container image builds on ARM64 | N/A | No Dockerfile |
| 6 | Test suite executes on ARM64 | ✅ PASS | 19 tests ran, 16 failed, 1 skipped. Same as baseline |
| 7 | Test failures classified | ✅ PASS | 14 INFRA (missing dataFiles), 2 ARM64 (HDF5 native), executor exit PRE-EXISTING |
| 8 | No ARM64-related test failures | ❌ FAIL | AddReferenceAlleleToHDF5PluginTest and MigrateHDF5FromT4T5Test fail from missing aarch64-Linux libjhdf5. They also fail on x86_64 because test data is missing |
| 9 | Application starts on ARM64 | ✅ PASS | TasselPipeline starts and exits 0 with the Graviton flags |
| 10 | JVM flags accepted without errors | ✅ PASS | `java <flags> -version` exit 0 on aarch64 |

## Changes Made
- `build.gradle.kts`: Graviton JVM flags added to `tasks.test.jvmArgs` (05-jvm-configuration.md).
- No dependency, source or native changes. See 02-native-library-report.md, 03-dependency-compatibility-report.md and 04-code-scan-findings.md.

## Remaining Concerns
1. **cisd:jhdf5 19.04.1 has no aarch64-Linux native library.** This blocks HDF5 features on Graviton. It is not one of this run's findings. The user must build `libjhdf5.so` for aarch64-Linux from https://sissource.ethz.ch/sispub/jhdf5 (tag 19.04.1) and supply it with `-Dnative.libpath.jhdf5=<path>`.
2. **Pre-existing build break:** `openchart:openchart:1.4.2` (transitive of forester 1.039) cannot be resolved from Maven Central or scijava. This affects all architectures and was not changed here.
3. **Test data:** most tests need the `dataFiles/` directory, which is not in the repository.
4. **User responsibility:** benchmark on Graviton (including `-XX:-TieredCompilation`), run extended stability tests, and set up CI for ARM64.
