+++
title = "Cpp 使用 conan 包管理工具"
date = "2025-12-01T00:00:00+08:00"
categories = ["折腾", "Cpp"]
tags = ["Cpp", "包管理工具", "conan", "cmake"]
+++

## 前提知识

### cmake 的基本流程

```sh
mkdir build
cmake -B build -S .         # 相当于 `cmake ..`
cmake --build build -j16    # 相当于 `make -j16`
```

### cmake preset

-   CMakePresets.json: 一般是要 git 跟踪的。
-   CMakeUserPresets.json: 一般是 gitignore 的。

```json
{
  "version": 6,
  "cmakeMinimumRequired": {
    "major": 3,
    "minor": 19,
    "patch": 0
  },
  "configurePresets": [
    {
      "name": "make-release",
      "displayName": "Release (Unix Makefiles)",
      "generator": "Unix Makefiles",
      "binaryDir": "${sourceDir}/build/release",
      "cacheVariables": {
        "CMAKE_BUILD_TYPE": "Release",
        "CMAKE_EXPORT_COMPILE_COMMANDS": true
      }
    },
    {
      "name": "make-debug",
      "displayName": "Debug (Unix Makefiles)",
      "generator": "Unix Makefiles",
      "binaryDir": "${sourceDir}/build/debug",
      "cacheVariables": {
        "CMAKE_BUILD_TYPE": "Debug",
        "CMAKE_EXPORT_COMPILE_COMMANDS": true
      }
    }
  ],
  "buildPresets": [
    {
      "name": "make-release",
      "configurePreset": "make-release",
      "jobs": 16
    },
    {
      "name": "make-debug",
      "configurePreset": "make-debug",
      "jobs": 16
    }
  ]
}
```

使用 cmake presets:

```sh
mkdir build

cmake --preset make-release -S .
cmake --build --preset make-release -j16
#cmake --build --preset make-release -j16 -- <target>
```

## 安装 conan2

```sh
sudo pacman -S conan

conan profile detect --force    # 生成 ~/.conan2/profiles/default 配置文件
```

## conan 的基本使用

### conan 添加第三方包

```py
def requirements(self):
    self.requires("fmt/11.0.2", options={"header_only": True})
    self.requires("boost/1.86.0", visible=False)   # 不传递给下游
    self.requires("openssl/3.0.13", options={"shared": True})
    self.test_requires("catch2/3.5.0")             # 仅测试阶段可见。conan test/create 时, 会使用这个依赖包。发布到 conan 仓库时, 不会依赖该包。
```

当一个软件依赖不同版本的包时:

```
myapp/1.0
├─ libA/1.0 ── fmt/9.1.0
└─ libB/1.0 ── fmt/10.0.0
```

解决方案:

```py
def requirements(self):
    self.requires("fmt/10.0.0", override=True)   # 强制使用 11.0.0
```

### 使用 conan2 的基本流程

新建一个 `cmake_exe` project:

```sh
mkdir demo_exe && cd demo_exe

conan new cmake_exe -d name=demo_exe -d version=0.1       # `conan new --help`
conan install . --build=missing     # 项目第一次运行 conan 时

cmake --preset conan-release
cmake --build --preset conan-release -j16

cmake --build --preset conan-release -- help    # 查看 makefile/ninja 有哪些 targets
```

新建一个 `cmake_lib` project:

```sh
mkdir demo_lib && cd demo_lib

conan new cmake_lib -d name=demo_lib -d version=0.1

# 接下来的流程和 `cmake_exe` 一样
# ...
```

## conan test/create package

```sh
conan editable add .    # 让缓存指向源码目录
conan editable list     # 可以看到缓存指向源码目录

conan export .                              # 先让其他人能 find_package。可用 `conan list` 查看
conan test test_package <pkg_name>/0.1      # 测试 header_lib

# 修改代码并 build 之后, 即可生效, 不用再 export。因为 pkg 的缓存是指向源码的。
cmake --build --preset conan-release -j16

# 测试完毕后
conan editable remove .
conan create . --build=missing            # export + 现场编译/打包 + 可选 test_package
```

## conan 编译 debug 版本

```sh
conan install . --build=missing -s build_type=Debug

cmake --preset conan-debug
cmake --build --preset conan-debug -j16
```

上面的编译, 编译出来的第三库也是 debug 版本的。有些时候如何机器存储比较小或者需要 adb 部署程序时, 是不适合的。<br/>
所以可以选择将第三方程序编译成 release 版本, 而自己的项目编译成 debug 版本:

```sh
# Ref: [Build the project with Debug while dependencies with Release](https://github.com/conan-io/conan/issues/13478#issuecomment-1475389368)
conan install . --build=missing -s "&:build_type=Debug" -s "build_type=Release"
```

## 查看包的信息

查看包的 options:

```sh
# Usage: <func> <pkg_name> <pgk_version>
conan_pkg_opts() {
  if [ -z "$1" ]; then
    echo "用法: conan_pkg_opts <pkg-name> [version]"
    echo "示例: conan_pkg_opts fmt"
    echo "示例: conan_pkg_opts fmt 11.0.2"
    return 1
  fi

  local pkg="$1"
  local ver="$2"

  if [ -z "$ver" ]; then
    # 不指定版本：匹配该包的所有版本
    conan graph info . --format=json 2>/dev/null | jq --arg pkg "$pkg" '
      .graph.nodes[]
      | select((.ref // "") | type == "string")
      | select(.ref | startswith($pkg + "/"))
      | {ref, options, default_options}
    '
  else
    # 指定版本：严格匹配 pkg/ver 或 pkg/ver#rev
    conan graph info . --format=json 2>/dev/null | jq --arg pkg "$pkg" --arg ver "$ver" '
      .graph.nodes[]
      | select((.ref // "") | type == "string")
      | select(.ref == ($pkg + "/" + $ver) or (.ref | startswith($pkg + "/" + $ver + "#")))
      | {ref, options, default_options}
    '
  fi
}
```

或者到 [conan center](https://conan.io/center) 查看

## conan 的常用命令

```sh
# conan {new|list|search|install|remove|export|test|editable|remote}

# 查看包的缓存路径
conan cache path <pkg_name>/<version>

# ## 导出缓存包到另外的机器
conan cache save header_lib/0.1:*   # 支持通配符
conan cache load header_lib-0.1__xxx.tgz
```

## vscode 使用 conan 项目

conanfile.py 指定 cmake 生成 command_compile.json。conanfile.py:

```python
    def generate(self):
        deps = CMakeDeps(self)
        deps.generate()
        tc = CMakeToolchain(self)
        tc.variables["CMAKE_EXPORT_COMPILE_COMMANDS"] = True    # 加上这行
        tc.generate()
```

```sh 
cmake --preset conan-release -S .
```

### 为 clangd 指定 command_compile.json 的位置

clangd 的配置指定。在项目的根目录, 创建 `.clangd` (推荐):

```
CompileFlags:
  CompilationDatabase: build/Release
```

OR. vscode 的项目配置指定。`.vscode/settings.json`:

```json
{
  "clangd.arguments": [
    "--compile-commands-dir=build/Release"
  ],
}
```

OR. 使用软链接的方式指定:

```sh
ln -s build/Release/compile_commands.json ./compile_commands.json
```
