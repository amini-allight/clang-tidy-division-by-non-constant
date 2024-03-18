# clang-tidy: division by non-constant

A Clang tidy that finds all divisions by non-constant values in your C/C++ codebase. It's useful for hunting down possible runtime division-by-zero exceptions if you haven't been proactive about guarding them in your codebase before. Many thanks to Kristen Shaker for her [wonderful talk](https://www.youtube.com/watch?v=torqlZnu9Ag) explaining how to do this, go check it out!

## Usage

This tidy is distributed as a patch file against [llvm-project](https://github.com/llvm/llvm-project) commit [0c423af59c971ddf1aa12d94529edf8293608157](https://github.com/llvm/llvm-project/commit/0c423af59c971ddf1aa12d94529edf8293608157) although it should work against future commits too.

Execute the following commands to apply it and build a custom version of `clang-tidy`:

```sh
git clone --depth 1 "https://github.com/llvm/llvm-project"
cd llvm-project
patch -p0 < /path/to/clang-tidy-division-by-non-constant/division-by-non-constant.patch
mkdir -p build
cd build
cmake -DCMAKE_BUILD_TYPE=RelWithDebInfo -DLLVM_ENABLE_PROJECTS="clang;clang-tools-extra" ../llvm
make -j$(nproc) clang-tidy
```

Then to use it on your CMake-based project:

```sh
export PATH=/path/to/llvm-project/build/bin:$PATH
cmake -DCMAKE_C_CLANG_TIDY="clang-tidy;-checks=bugprone-division-by-non-constant" -DCMAKE_CXX_CLANG_TIDY="clang-tidy;-checks=bugprone-division-by-non-constant" .
make -j$(nproc)
```

## Known Issues

The tidy can be overzealous when the right-hand side of the division operation involves function calls, for example:

```cpp
static const int x = 2;

int y = 4 / pow(x, 2);
```

This will be registered as a division by a non-constant value even though the right-hand side is constant in practice.

## Credit & License

Developed by Amini Allight and licensed under the Apache License 2.0 with LLVM exceptions, matching the license used by the LLVM project itself.
