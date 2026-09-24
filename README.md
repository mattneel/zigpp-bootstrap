# zigpp-bootstrap

Builds [Zig++](https://github.com/mattneel/zigpp) from source for any target,
starting from a C++ compiler, together with the LLVM, Clang, and LLD libraries
it links against. It is [zig-bootstrap](https://codeberg.org/ziglang/zig-bootstrap)
for Zig++, and it makes the devkits that Zig++'s CI and releases build with.

## Versions

 * LLVM, Clang, LLD 23.1.2, from the release source of
   [llvmorg-23.1.2](https://github.com/llvm/llvm-project/releases/tag/llvmorg-23.1.2),
   checked against its SHA-256 and patched with the patches in `patches/`
 * zlib 1.3.1 and zstd 1.5.2, copied from zig-bootstrap
 * Zig++: the `zig` submodule, built as the version in `zig-version`

Unlike zig-bootstrap, this repository does not copy the LLVM sources: `./build`
downloads the release source once, into `downloads/`, and patches it into
`llvm-project/`.

### Patches

The patches are zig-bootstrap's patches for LLVM 23, one per file:

 * LLVM: support the .lib extension for static zstd
 * LLVM: don't pass -static when building executables
 * LLVM: OpenBSD `llvm-config` logic
 * Clang: ignore the examples directory
 * Clang: disable building libclang-cpp.so
 * Clang: remove the `nvptx-arch` and `amdgpu-arch` symlinks
 * Clang: remove broken scan-build manpage install logic
 * LLD: add an additional include directory for Zig's libunwind
 * LLD: respect `LLD_BUILD_TOOLS=OFF`
 * LLD: skip building docs
 * LLD: OpenBSD `findMajMinShlib()` logic

zlib carries zig-bootstrap's patch that deletes the ability to build a shared
library.

## Host System Dependencies

 * A C++ compiler that can build LLVM, Clang, and LLD (GCC 5.1+ or Clang)
 * CMake 3.20 or later, and Ninja or another build system that CMake supports
 * curl, tar with xz support, patch, and sha256sum or shasum
 * POSIX system (sh, mkdir, cd)
 * Python 3

## Build Instructions

```
git clone --recursive https://github.com/mattneel/zigpp-bootstrap
cd zigpp-bootstrap
./build <arch>-<os>-<abi> <mcpu>
```

`<arch>-<os>-<abi>` is a Zig target, and `<mcpu>` a `-mcpu` value of Zig:
`baseline` for a generic CPU of the architecture, or `native`. The Zig++
distribution for the target is then in `out/zig-<arch>-<os>-<abi>-<mcpu>/`.

Set `CMAKE_GENERATOR=Ninja` for a faster build, and `CMAKE_BUILD_PARALLEL_LEVEL`
to limit the parallel jobs. The first run builds LLVM, Clang, and LLD twice:
once for the host, to build Zig++ and the LLVM tools that the cross build
needs, and once for the target, with Zig++ as the cross compiler. Later runs
for other targets reuse the host build.

`ZIG_SRC` builds another Zig++ checkout instead of the submodule, and
`ZIG_VERSION` sets the version string that it is built as.

## Devkits

```
./devkit <arch>-<os>-<abi> <mcpu>
```

packs the target's libraries, headers, and Zig++ into
`out/devkit/zig+llvm+lld+clang-<arch>-<os>-<abi>-<version>.tar.xz` (a `.zip` for
Windows). Zig++'s CI downloads these from the releases of this repository:

```
./publish
```

uploads every devkit of the version in `zig-version` from `out/devkit`, with a
`SHA256SUMS` file, as the release `devkit-<version>`. Zig++'s
`.github/scripts/devkit.sh` names the version that CI uses.

## Updating

 * Zig++: move the `zig` submodule to the new commit and write its version,
   `0.17.0-dev.<commits since 0.16.0>+zigpp.<commit>`, to `zig-version`.
 * LLVM: set `LLVM_VERSION` and `LLVM_SHA256` in `build`, and update the
   patches in `patches/` from zig-bootstrap's branch for that LLVM version.
