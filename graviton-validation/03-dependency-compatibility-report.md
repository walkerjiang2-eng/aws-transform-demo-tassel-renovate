# Dependency Compatibility Report — Applied Versions

No dependency was flagged MUST UPGRADE in the findings provided to this run.
No dependency version was changed.

| Dependency | Scope | Current Version | Applied Version | Status | Evidence |
|------------|-------|-----------------|-----------------|--------|----------|
| cisd:base | transitive (via cisd:jhdf5:19.04.1) | 18.09.0 | 18.09.0 (unchanged) | Not upgradable; pure-Java fallbacks accepted | Only published version is 18.09.0 (scijava maven-metadata). Jar has no aarch64-Linux natives. See 02-native-library-report.md |
| cisd:jhdf5 | direct | 19.04.1 | 19.04.1 (unchanged) | Not upgradable (latest); no aarch64-Linux native | scijava maven-metadata latest = 19.04.1. Not a finding for this run; documented as remaining concern |

## Transitive Dependency Resolutions

No transitive overrides were applied. `cisd:base` has a single published
version, so no dependencyManagement / resolution-strategy override can provide
aarch64-Linux natives.

## Pre-existing, architecture-independent resolution issue (not changed)

`org.biojava.thirdparty:forester:1.039` declares `openchart:openchart:1.4.2`,
which its POM says comes from the retired JBoss third-party repository. It is
not on Maven Central or the scijava repository, so `./gradlew build` fails at
`:compileKotlin` on every architecture (it fails the same way on x86_64).
TASSEL source does not reference openchart. To validate, a Gradle init script
outside the repository (`-I exclude-openchart.init.gradle.kts`, which excludes
`openchart:openchart`) was used. The project build files were not changed for
this, because the problem is not related to ARM64.
