# AGENTS.md

## Cursor Cloud specific instructions

### Overview

Bitcoin Armory is a C++17/Python3 desktop Bitcoin wallet client. It has three main C++ binaries (`ArmoryDB`, `CppBridge`, `BIP150KeyManager`) built via autotools, plus a Python3/Qt GUI (`ArmoryQt.py`).

### Build Dependencies Location

Third-party libraries are built from source into `/workspace/deps/`:
- **libbtc**: `/workspace/deps/libbtc` — Bitcoin crypto library
- **libwebsockets**: `/workspace/deps/libwebsockets` — WebSocket library (build dir: `build/`)
- **c20p1305_cffi**: `/workspace/deps/c20p1305_cffi` — ChaCha20-Poly1305 CFFI extension

The built CFFI `.so` is copied to `/workspace/armoryengine/`.

### Building

The C++ build uses autotools with an out-of-tree build directory at `/workspace/build/`:

```
sh autogen.sh
mkdir -p build && cd build
../configure --enable-tests --enable-debug \
  --with-own-libbtc=/workspace/deps/libbtc \
  --with-own-lws=/workspace/deps/libwebsockets/build
make -j$(nproc)
```

Binaries land in `/workspace/build/` (`ArmoryDB`, `CppBridge`, `BIP150KeyManager`).

### Running Tests

Test binaries are at `/workspace/build/cppForSwig/gtest/`. Run them directly:

```
./gtest/BIP151RekeyTest
./gtest/ContainerTests
./gtest/UtilsTests
./gtest/SignerTests
./gtest/WalletTests
./gtest/SupernodeTests
./gtest/CppBlockUtilsTests
./gtest/ZeroConfTests
./gtest/BridgeTests
```

**Note**: `make check` has a known issue where it tries to compile `.capnp.c++` files as standalone executables. Run test binaries directly instead.

**Note**: Some test failures (e.g. `BIP150_151Test.checkData_150_151*`, `ContainerTests.BlockingQueue_Concurrent`) appear to be pre-existing in the codebase and are not caused by the build environment.

### Running ArmoryDB

ArmoryDB requires Bitcoin Core blockchain data to be useful. For testing purposes you can start it in regtest mode:

```
./build/ArmoryDB --regtest --datadir=/tmp/armory_regtest/datadir \
  --dbdir=/tmp/armory_regtest/dbdir \
  --satoshi-datadir=/tmp/armory_regtest/blocks
```

It will prompt for a peers store password on first run. The "could not find first file index" error is expected without actual blockchain data files.

### Running ArmoryQt (GUI)

Requires a display server. The GUI is launched with:

```
python3 ArmoryQt.py
```

It auto-spawns `CppBridge` and `ArmoryDB`. The `CppBridge` binary must be accessible in the build output path. PySide6 and qtpy Python packages must be installed.

### Key Gotchas

- CMake picks up Clang by default on this VM; force GCC with `CC=gcc CXX=g++` when building c20p1305_cffi.
- The root-level `CMakeLists.txt` is for Windows (MSVC/MINGW) builds only; Linux uses autotools.
- `CppBridge` cannot be run standalone — it requires environment variables set by `ArmoryQt.py`.
- The `c20p1305` CFFI shared library must be in `armoryengine/` for ArmoryQt to start.
