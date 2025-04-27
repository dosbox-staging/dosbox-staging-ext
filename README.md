# dosbox-staging-ext

This repository contains various external dependencies (typically dynamic
libraries) required by [DOSBox Staging](https://github.com/dosbox-staging/dosbox-staging).

Certain dependencies can take a very long time to compile on Windows. For
example, FluidSynth and Slirp usually take 20-30 minutes to compile from
scratch due to their dependence on `glib` which requires vcpkg to build an
entire MSYS2 toolchain first. The situation is better on macOS and Linux, but
compiling these two from scratch still takes about 5-7 minutes. The solution
is to build these rarely changing artifacts outside the DOSBox Staging CI
build loop in this repo, and publish the resulting dynamic library binaries as
a downloadable artifact. The [DOSBox Staging build
workflows](https://github.com/dosbox-staging/dosbox-staging/actions) then
download these and injects the dynamic libraries into the final release
packages. So technically this is a form of caching.

CI build times are actually the smaller issue here; the bigger problem is long
local build times when the local vcpkg cache gets invalidated (e.g., when
upgrading Visual Studio).


## vcpkg dependencies

Make sure the CI runner images in the [DOSBox
Staging](https://github.com/dosbox-staging/dosbox-staging) repo and this repo
are always in sync.

Similarly, the [vcpkg
baseline](https://github.com/dosbox-staging/dosbox-staging/blob/main/vcpkg.json)
in the DOSBox Staging repo and this repo should be the same. This might not be
strictly necessary, but it can lead to problems if they differ.

The `deps-*.txt` files contain the set of dynamic libraries we need to inject
into the final release packages. The exact file names might change when
upgrading the dependencies, so make sure to check them when upgrading and
adjust the `deps-*.txt` files accordingly.

This is how to check the list of transitive dependencies required by a dynamic
library on each platform:

### Windows

```
dumpbin /dependents fluidsynth-3.dll
```

Note `dumpbin` does *not* perform a recursive lookup; you'll need to traverse
the chain of transitive vcpkg dependencies manually.

The tool is located in your Visual Studio tools folder, e.g.:

```
c:\Program Files\Microsoft Visual Studio\2022\Community\VC\Tools\MSVC\14.43.34808\bin\Hostx64\x64\dumpbin.exe
```

### macOS

This will list all transitive dynamic dependencies recursively:

```
otool -L libfluidsynth.3.dylib
```

### Linux

This will list all transitive dynamic dependencies recursively:

```
ldd libfluidsynth.so.3
```
