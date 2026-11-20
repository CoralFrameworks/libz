## Layout

This repository is a thin CoralFrameworks wrapper around
[zlib](https://github.com/madler/zlib). Upstream sources are no longer vendored
here; they are tracked as the `zlib` git submodule.

```
libz/
  coral/   CoralFrameworks build integration (static `z` target)
  zlib/    git submodule -> https://github.com/madler/zlib (pinned to v1.3.1)
```

After cloning, initialize the submodule so the sources are present:

```
git submodule update --init --recursive
```

Or clone with submodules in one step:

```
git clone --recurse-submodules https://github.com/CoralFrameworks/libz.git
```

`coral/CMakeLists.txt` compiles the zlib C sources into a single static `z`
target and exports the zlib headers (`zlib.h`, `zconf.h`). Consumers only need:

```cmake
add_subdirectory(<path-to>/libz/coral)
target_link_libraries(<target> PRIVATE z)
```

If the zlib sources are stored somewhere else, preset `ZLIB_DIR` before adding
the directory:

```cmake
set(ZLIB_DIR /path/to/zlib)
add_subdirectory(<path-to>/libz/coral)
```

## Updating

zlib is pinned to a release tag (currently `v1.3.1`). To update to a newer
upstream release:

```
git -C zlib fetch --tags
git -C zlib checkout v1.3.1   # or a newer release tag
git add zlib
git commit -m "Update zlib submodule to v1.3.1"
```

Please verify that the wrapper still builds before uploading a PR:

```
cmake -S coral -B build -DCMAKE_BUILD_TYPE=Release
cmake --build build --config Release
```
