# sse2neon
[![Build Status](https://travis-ci.com/DLTcollab/sse2neon.svg?branch=master)](https://travis-ci.com/DLTcollab/sse2neon)

A C/C++ header file that converts Intel SSE intrinsics to Arm/Aarch64 NEON intrinsics.

## Introduction

`sse2neon` is a translator of Intel SSE (Streaming SIMD Extensions) intrinsics
to [Arm NEON](https://developer.arm.com/architectures/instruction-sets/simd-isas/neon),
shortening the time needed to get an Arm working program that then can be used to
extract profiles and to identify hot paths in the code.
The header file `sse2neon.h` contains several of the functions provided by Intel
intrinsic headers such as `<xmmintrin.h>`, only implemented with NEON-based counterparts
to produce the exact semantics of the intrinsics.

## Mapping and Coverage

Header file | Extension |
---|---|
`<mmintrin.h>` | MMX |
`<xmmintrin.h>` | SSE |
`<emmintrin.h>` | SSE2 |
`<pmmintrin.h>` | SSE3 |
`<tmmintrin.h>` | SSSE3 |
`<smmintrin.h>` | SSE4.1 |
`<nmmintrin.h>` | SSE4.2 |
`<wmmintrin.h>` | AES  |

`sse2neon` aims to support SSE, SSE2, SSE3, SSSE3, SSE4.1, SSE4.2 and AES extension.

In order to deliver NEON-equivalent intrinsics for all SSE intrinsics used widely,
please be aware that some SSE intrinsics exist a direct mapping with a concrete
NEON-equivalent intrinsic. However, others lack of 1-to-1 mapping, that means the
equivalents are implemented using several NEON intrinsics.

For example, SSE intrinsic `_mm_loadu_si128` has a direct NEON mapping (`vld1q_s32`),
but SSE intrinsic `_mm_maddubs_epi16` has to be implemented with 13+ NEON instructions.

## Usage

- Put the file `sse2neon.h` in to your source code directory.

- Locate the following SSE header files included in the code:
```C
#include <xmmintrin.h>
#include <emmintrin.h>
```
  {p,t,s,n,w}mmintrin.h should be replaceable, but the coverage of these extensions might be limited though.

- Replace them with:
```C
#include "sse2neon.h"
```

- Explicitly specify platform-specific options to gcc/clang compilers.
  * On ARMv8-A targets, you should specify the following compiler option: (Remove `crypto` and/or `crc` if your architecture does not support cryptographic and/or CRC32 extensions)
  ```shell
  -march=armv8-a+fp+simd+crypto+crc
  ```
  * On ARMv7-A targets, you need to append the following compiler option:
  ```shell
  -mfpu=neon
  ```

## Compile-time Configurations

Considering the balance between correctness and peformance, `sse2neon` recognizes the following compile-time configurations:
* `SSE2NEON_PRECISE_MINMAX`: Enable precise implementation of `_mm_min_ps` and `_mm_max_ps`. Turned off by default. If you need consistent results such as NaN special cases, define the macro as `1` before including `sse2neon.h`.

## Run Built-in Test Suite

`sse2neon` provides a unified interface for developing test cases. These test
cases are located in `tests` directory, and the input data is specified at
runtime. Use the following commands to perform test cases:
```shell
$ make check
```

You can specify GNU toolchain for cross compilation as well.
[QEMU](https://www.qemu.org/) should be installed in advance.
```shell
$ make CROSS_COMPILE=aarch64-linux-gnu- check # ARMv8-A
```
or
```shell
$ make CROSS_COMPILE=arm-linux-gnueabihf- check # ARMv7-A
```

:warning: **Warning: The test suite is based on the little-endian architecture.**

### Add More Test Items
Once the conversion is implemented, the test can be added with the following steps:

* File `tests/impl.h`

  Add the intrinsic under `#define INTRIN_FOREACH(TYPE)` macro. The naming convention
  should be `mm_xxx`.
  Place it in the correct classification with the alphabetical order.
  The classification can be referenced from [Intel Intrinsics Guide](https://software.intel.com/sites/landingpage/IntrinsicsGuide/#).

* File `tests/impl.cpp`
    ```c
    result_t test_mm_xxx()
    {
        // The C implementation
        ...

        // The Neon implementation
        ret = _mm_xxx();

        // Compare the result of two implementations and return either
        // TEST_SUCCESS, TEST_FAIL, or TEST_UNIMPL
        ...
    }
    ```

## Coding Convention
Use the command `$ make indent` to follow the coding convention.

## Adoptions
Here is a partial list of open source projects that have adopted `sse2neon` for Arm/Aarch64 support.
* [Apache Impala](https://impala.apache.org/) is a lightning-fast, distributed SQL queries for petabytes of data stored in Apache Hadoop clusters.
* [Apache Kudu](https://kudu.apache.org/) completes Hadoop's storage layer to enable fast analytics on fast data.
* [Catcoon](https://github.com/i-evi/catcoon) is a [feedforward neural network](https://en.wikipedia.org/wiki/Feedforward_neural_network) implementation in C.
* [dab-cmdline](https://github.com/JvanKatwijk/dab-cmdline) provides entries for the functionality to handle Digital audio broadcasting (DAB)/DAB+ through some simple calls.
* [FoundationDB](https://www.foundationdb.org) is a distributed database designed to handle large volumes of structured data across clusters of commodity servers.
* [iqtree_arm_neon](https://github.com/joshlvmh/iqtree_arm_neon) is the Arm NEON port of [IQ-TREE](http://www.iqtree.org/), fast and effective stochastic algorithm to infer phylogenetic trees by maximum likelihood.
* [kram](https://github.com/alecazam/kram) is a wrapper to several popular encoders to and from PNG/[KTX](https://www.khronos.org/opengles/sdk/tools/KTX/file_format_spec/) files with [LDR/HDR and BC/ASTC/ETC2](https://developer.arm.com/solutions/graphics-and-gaming/developer-guides/learn-the-basics/adaptive-scalable-texture-compression/single-page).
* [libscapi](https://github.com/cryptobiu/libscapi) stands for the "Secure Computation API", providing  reliable, efficient, and highly flexible cryptographic infrastructure.
* [MMseqs2](https://github.com/soedinglab/MMseqs2) (Many-against-Many sequence searching) is a software suite to search and cluster huge protein and nucleotide sequence sets.
* [OBS Studio](https://github.com/obsproject/obs-studio) is software designed for capturing, compositing, encoding, recording, and streaming video content, efficiently.
* [OpenXRay](https://github.com/OpenXRay/xray-16) is an improved version of the X-Ray engine, used in world famous S.T.A.L.K.E.R. game series by GSC Game World.
* [parallel-n64](https://github.com/libretro/parallel-n64) is an optimized/rewritten Nintendo 64 emulator made specifically for [Libretro](https://www.libretro.com/).
* [PlutoSDR Firmware](https://github.com/seanstone/plutosdr-fw) is the customized firmware for the [PlutoSDR](https://wiki.analog.com/university/tools/pluto) that can be used to introduce fundamentals of Software Defined Radio (SDR) or Radio Frequency (RF) or Communications as advanced topics in electrical engineering in a self or instructor lead setting.
* [Pygame](https://www.pygame.org) is cross-platform and designed to make it easy to write multimedia software, such as games, in Python.
* [srsLTE](https://github.com/srsLTE/srsLTE) is an open source SDR LTE software suite.
* [Surge](https://github.com/surge-synthesizer/surge) is an open source digital synthesizer.
* [XMRig](https://github.com/xmrig/xmrig) is an open source CPU miner for [Monero](https://web.getmonero.org/) cryptocurrency.

## Related Projects
* [SIMDe](https://github.com/nemequ/simde): fast and portable implementations of SIMD
  intrinsics on hardware which doesn't natively support them, such as calling SSE functions on ARM.
* [SSE2NEON.h : A porting guide and header file to convert SSE intrinsics to their ARM NEON equivalent](https://codesuppository.blogspot.com/2015/02/sse2neonh-porting-guide-and-header-file.html)
* [CatBoost's sse2neon](https://github.com/catboost/catboost/blob/master/library/cpp/sse/sse2neon.h)
* [ARM\_NEON\_2\_x86\_SSE](https://github.com/intel/ARM_NEON_2_x86_SSE)
* [SSE2NEON - High Performance MPC on ARM](https://github.com/rons1404/biu-cybercenter-proj-sse2neon)
* [AvxToNeon](https://github.com/kunpengcompute/AvxToNeon)
* [POWER/PowerPC support for GCC](https://github.com/gcc-mirror/gcc/blob/master/gcc/config/rs6000) contains a series of headers simplifying porting x86_64 code that
   makes explicit use of Intel intrinsics to powerpc64le (pure little-endian mode that has been introduced with the [POWER8](https://en.wikipedia.org/wiki/POWER8)).
    - implementation: [xmmintrin.h](https://github.com/gcc-mirror/gcc/blob/master/gcc/config/rs6000/xmmintrin.h), [emmintrin.h](https://github.com/gcc-mirror/gcc/blob/master/gcc/config/rs6000/emmintrin.h), [pmmintrin.h](https://github.com/gcc-mirror/gcc/blob/master/gcc/config/rs6000/pmmintrin.h), [tmmintrin.h](https://github.com/gcc-mirror/gcc/blob/master/gcc/config/rs6000/tmmintrin.h), [smmintrin.h](https://github.com/gcc-mirror/gcc/blob/master/gcc/config/rs6000/smmintrin.h)

## Reference
* [Intel Intrinsics Guide](https://software.intel.com/sites/landingpage/IntrinsicsGuide/)
* [Arm Neon Intrinsics Reference](https://developer.arm.com/architectures/instruction-sets/simd-isas/neon/intrinsics)
* [Neon Programmer's Guide for Armv8-A](https://developer.arm.com/architectures/instruction-sets/simd-isas/neon/neon-programmers-guide-for-armv8-a)
* [NEON Programmer's Guide](https://static.docs.arm.com/den0018/a/DEN0018A_neon_programmers_guide_en.pdf)
* [qemu/target/i386/ops_sse.h](https://github.com/qemu/qemu/blob/master/target/i386/ops_sse.h): Comprehensive SSE instruction emulation in C. Ideal for semantic checks.

## Licensing

`sse2neon` is freely redistributable under the MIT License.
