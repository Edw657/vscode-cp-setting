# VSCode C++ 算法竞赛配置

一套面向 Windows 的轻量竞赛环境。它把编译、调试、代码补全和样例测试集中到 VSCode 中，适合 Codeforces、洛谷、牛客等以单文件提交为主的 OJ。如果能帮到你，请不吝惜点一个小小的star thanks~

## 这套配置包含什么

### MSYS2 UCRT64

MSYS2 为 Windows 提供 GCC、GDB 等开发工具。这里使用官方推荐的 UCRT64 环境，它基于较新的 Windows UCRT 运行库。

工具链默认安装在：

```text
C:\msys64\ucrt64\bin
```

### GCC 16.2.0

GCC 负责把 `.cpp` 源文件编译为 Windows 下可运行的 `.exe`。编译参数分别写在 `.vscode/tasks.json` 和 CPH 设置中。

### GDB 17.2

GDB 是 VSCode 断点调试背后的调试器。按 `F5` 后，可以逐行执行代码、查看变量和调用栈。

### C/C++ 扩展

微软官方 C/C++ 扩展提供 IntelliSense、代码跳转、错误提示、格式化和 GDB 调试适配。

### CPH

Competitive Programming Helper 用于管理和批量运行样例。配合浏览器端 Competitive Companion，可以从 Codeforces 等网站把题目样例发送到 VSCode，免去反复复制输入输出。

### PATH

将 `C:\msys64\ucrt64\bin` 加入 PATH 后，终端才能直接找到 `g++` 和 `gdb`。重新打开 VSCode 后可检查：

```powershell
g++ --version
gdb --version
```

## 目录结构

```text
competitive-programming/
├── .vscode/
│   ├── tasks.json
│   ├── launch.json
│   ├── c_cpp_properties.json
│   ├── settings.json
│   └── extensions.json
├── templates/
│   └── template.cpp
├── contests/
└── stress-test/
```

- `templates` 保存写题模板。
- `contests` 按比赛、日期或题单整理代码。
- `stress-test` 预留给暴力解、数据生成器和对拍脚本。

例如：

```text
contests/
└── 2026-08-28-cf-div3/
    ├── A.cpp
    ├── B.cpp
    └── C.cpp
```

## VSCode 配置说明

### tasks.json

定义四种单文件构建任务。所有任务都编译当前编辑器中打开的 `${file}`，生成的程序放在源文件旁边。

| 任务 | 主要参数 | 输出文件 | 用途 |
| --- | --- | --- | --- |
| Debug Build | `-std=c++17 -O0 -g3 -DLOCAL -fsanitize=address,undefined` | `题名_debug.exe` | 带内存与未定义行为检查的调试构建 |
| Debug Build (No Sanitizer) | `-std=c++17 -O0 -g3 -DLOCAL` | `题名_debug.exe` | 当前机器实际使用的调试构建 |
| Submit Simulation Build | `-std=c++17 -O2 -Wall -Wextra` | `题名.exe` | 提交前模拟 OJ 编译环境 |
| Compat Check C++14 | `-std=c++14 -O2 -Wall -Wextra` | `题名_c14.exe` | 检查代码能否通过 C++14 编译 |

`-O0` 关闭优化，方便断点和变量观察；`-g3` 写入调试信息；`-O2` 用于接近评测环境的优化；`-Wall -Wextra` 提前暴露常见问题；`-DLOCAL` 只启用本地调试代码。

### launch.json

负责 `F5` 调试。它会先执行 `Debug Build (No Sanitizer)`，再用 UCRT64 的 GDB 启动当前文件对应的 `_debug.exe`。工作目录设置为源文件所在目录，因此相对路径读写也以题目目录为准。

### c_cpp_properties.json

负责 IntelliSense，不参与实际编译。它告诉 C/C++ 扩展使用 UCRT64 GCC、C++17 语法和 `windows-gcc-x64` 模式，使代码补全、头文件识别和语法检查与编译器保持一致。

### settings.json

工作区设置包括：

- 默认使用 UCRT64 的 `g++.exe`；
- C++ 使用 C++17，C 使用 C17；
- 保存 `.cpp` 时调用 clang-format 自动排版；
- 为 VSCode 终端补充 UCRT64 PATH；
- 单独指定 CPH 的编译器、参数和 3 秒超时。

CPH 有自己的编译流程，不会读取 `tasks.json`，所以它的参数需要单独配置：

```text
-std=c++17 -O2 -Wall -Wextra
```

### extensions.json

这是扩展推荐清单。其他人打开仓库时，VSCode 会推荐安装：

- `ms-vscode.cpptools`
- `divyanshuagrawal.competitive-programming-helper`

Code Runner 被列为不推荐，避免它与 `tasks.json` 各维护一套编译参数。

## 代码模板

`templates/template.cpp` 包含竞赛常用头文件、快速输入输出和 `LOCAL` 调试宏：

```cpp
#ifdef LOCAL
#define debug(x) cerr << #x << " = " << (x) << '\n'
#else
#define debug(x) ((void)0)
#endif
```

调试构建定义了 `LOCAL`，因此 `debug(x)` 会输出到标准错误；模拟提交构建不定义 `LOCAL`，这些调试语句会在编译时失效。

## 使用方法

1. 用 VSCode 打开整个仓库目录，而不是只打开某个 `.cpp` 文件。
2. 从 `templates/template.cpp` 复制一份到 `contests/比赛名/题号.cpp`。
3. 编写代码后按 `F5` 断点调试。
4. 使用 CPH 批量运行题目样例。
5. 提交前运行 `Submit Simulation Build`，确认代码在 `-O2` 且没有 `LOCAL` 的情况下仍能正常编译和运行。
6. 遇到只支持旧标准的比赛，再运行 `Compat Check C++14`。

构建任务使用 `${file}`，所以代码可以放在 `contests` 的任意子目录中，不必堆在仓库根目录。

## 关于 Sanitizer

`Debug Build` 原本用于启用 AddressSanitizer 和 UndefinedBehaviorSanitizer，帮助发现数组越界、释放后访问及部分未定义行为。

当前安装的 MSYS2 GCC 能识别 `-fsanitize` 参数，但链接阶段找不到 `libasan` 和 `libubsan`，因此这个任务暂时不可用。日常 `F5` 已绑定到 `Debug Build (No Sanitizer)`，正常编译和断点调试不受影响。

## 路径与移植

配置默认工具链位于 `C:\msys64\ucrt64\bin`。如果 MSYS2 安装在其他位置，需要修改 `.vscode` 中的 `compilerPath`、`command`、`miDebuggerPath` 和 PATH 配置。

`.gitignore` 已排除 `.exe`、`.o`、`.out` 和 CPH 本地数据，避免把构建产物和测试缓存提交到仓库。

## License

本项目采用 MIT License。
