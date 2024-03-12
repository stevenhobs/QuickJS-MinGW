# QuickJS Engine on Windows

Synchronize and update the official master branch source code, using Mingw64 + CMake + Ninja for building.

## Support

- quickjs shared lib
- qjs binary
- qjsc binary

## My Build Environment

- Windows11 23H2
- Mingw64(mcf thread) gcc version 13.2.0
- VSCode latest version + extension for CMake
- CMake version >= 3.28.3
- Ninja version >= 1.11.1

## Notice

- This wrap project doesn't complete. I just do this for smallest build environment on Windows System.
- Some configurations do not yet test. If you changed , maybe the build would fail. For example CONFIG_LTO
- Dude, this is just a toy. Be cautious about the usage of products.

## Thanks

- QuickJS Offical <https://bellard.org/quickjs/>
- Mingw-builds-binaries <https://github.com/niXman/mingw-builds-binaries>
- dlfcn-win32 <https://github.com/dlfcn-win32/dlfcn-win32>
- Personal Blog <https://www.cnblogs.com/wunaozai/p/17850789.html>
