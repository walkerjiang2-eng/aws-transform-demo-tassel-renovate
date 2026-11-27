# Native Library Resolution Report

## Resolution Details

### cisd:base 18.09.0 — libunix / libnativedata (finding 01M818BH0WJ96PGGMN1FZPKZEG, low)

Pulled in transitively by `cisd:jhdf5:19.04.1` (build.gradle.kts).

**What was checked in this run**

| Check | Result |
|-------|--------|
| `https://maven.scijava.org/content/repositories/public/cisd/base/maven-metadata.xml` | Only version published: `18.09.0` (latest = release = 18.09.0). No newer version to upgrade to. |
| `unzip -l base-18.09.0.jar` natives | `native/unix/{arm-Linux,amd64-Linux,x86_64-Mac OS X}`, `native/nativedata/{arm-Linux,amd64-Linux,x86_64-Mac OS X,amd64-Windows}` |
| `file native/*/arm-Linux/*.so` | `ELF 32-bit LSB shared object, ARM, EABI5` — 32-bit ARM, **not** aarch64 |
| Source in project | No (third-party jar) |

**Resolution: accept the built-in pure-Java fallbacks (no build change).**
There is no aarch64-Linux build of libunix/libnativedata in any published
version, and the library source is not part of this project. Verified on
linux/arm64 (eclipse-temurin:21-jdk, `os.arch=aarch64`):

```
NativeData.isUseNativeLib=false
Unix.isOperational=false
NativeData roundtrip ok=true      # int[] -> byte[] -> int[] via pure-Java path
```

- `NativeData` byte-order conversion uses its pure-Java implementation; results are correct.
- `ch.systemsx.cisd.base.unix.Unix` reports "not operational" (Unix file-metadata helpers unavailable). TASSEL does not call `Unix` directly.

## Pure Java Fallbacks

| Library | Native component | Fallback | Verified on aarch64 |
|---------|------------------|----------|---------------------|
| cisd:base 18.09.0 | libnativedata | Pure-Java `NativeData` conversion | Yes — round-trip correct |
| cisd:base 18.09.0 | libunix | None needed; `Unix.isOperational()` returns false | Yes |
| TASSEL `DoubleMatrixFactory` | optional `libTasselBlas` (not shipped in repo) | Existing EJML fallback on `UnsatisfiedLinkError` | Yes — `DEFAULT=ejml` |

## Related Observation (not a finding assigned to this run): cisd:jhdf5 19.04.1 libjhdf5

The cisd:base finding says to resolve it together with the jhdf5 fix. No jhdf5
finding was provided to this run, but the check below shows it is the one real
ARM64 native gap in the project:

| Check | Result |
|-------|--------|
| scijava `cisd/jhdf5/maven-metadata.xml` | Latest = `19.04.1` (already used). No newer version. |
| `unzip -l jhdf5-19.04.1.jar` natives | `arm-Linux` (ELF 32-bit ARM EABI5), `amd64-Linux`, `amd64-Windows`, `x86_64-Mac OS X`, `aarch64-Mac OS X`. **No `aarch64-Linux`.** |
| Maven Central search `a:jhdf5` | Only `org.scala-saddle:jhdf5:2.9` (old, unrelated packaging) |
| Upstream source | https://sissource.ethz.ch/sispub/jhdf5 (tag 19.04.1) |

On linux/arm64, any HDF5 access fails with
`java.lang.UnsupportedOperationException: No suitable HDF5 native library found for this platform.`
This affects TASSEL's HDF5 genotype features (`*.h5` import/export) and makes
two HDF5 tests fail for an ARM64 reason (see 06-build-test-results.md). It has
no pure-Java fallback.

**User action required:** build `libjhdf5.so` for aarch64-Linux from the
JHDF5 19.04.1 source distribution (it bundles HDF5 1.10 and the JNI sources),
then supply it at runtime with either
`-Dnative.libpath.jhdf5=/path/to/libjhdf5.so` or
`-Dnative.libpath=/dir` where the file is at `/dir/jhdf5/aarch64-Linux/libjhdf5.so`
(property names are read by `ch.systemsx.cisd.base.utilities.NativeLibraryUtilities`).
