# C++ 算法竞赛本机配置

这是一套在 Windows + VSCode 上刷 C++ 算法题的轻量配置：MSYS2 提供 GCC/GDB，VSCode 负责编辑和调试，CPH 负责抓取和批量运行题目样例。

## 目录

```text
competitive-programming/
├── .vscode/          # VSCode 编译、调试和 IntelliSense 配置
├── templates/        # 写题模板
├── contests/         # 按比赛或日期保存题目
└── stress-test/      # 预留给暴力解、数据生成器和对拍脚本
```

## 环境

当前本机使用 MSYS2 UCRT64、GCC 16.2.0、GDB 17.2、VSCode C/C++ 扩展和 Competitive Programming Helper（CPH）。MSYS2 工具链位于 `C:\msys64\ucrt64\bin`，该目录已经加入用户 PATH。

重新打开终端后检查：

```powershell
g++ --version
gdb --version
```

## 开始写题

用 VSCode 打开本目录，在 `contests` 下按比赛或日期新建文件夹，例如 `contests/2026-08-28-cf/A.cpp`。可以从 `templates/template.cpp` 复制模板。任务使用 `${file}`，题目文件不需要放在根目录。

## 编译和调试

`Ctrl+Shift+B` 可选择构建任务：

| 任务 | 参数 | 用途 |
| --- | --- | --- |
| Debug Build | C++17、O0、调试信息、`LOCAL`、ASan/UBSan | 进阶调试 |
| Debug Build (No Sanitizer) | C++17、O0、调试信息、`LOCAL` | 当前机器可用的 F5 调试任务 |
| Submit Simulation Build | C++17、O2、`-Wall -Wextra` | 提交前模拟评测 |
| Compat Check C++14 | C++14、O2、`-Wall -Wextra` | 检查老评测机兼容性 |

按 `F5` 会编译并调试当前文件。输出文件按源文件命名，避免不同题目互相覆盖。

当前 GCC 安装缺少 `libasan` 和 `libubsan`，所以带 sanitizer 的 `Debug Build` 暂时无法链接；F5 使用不带 sanitizer 的调试任务，不影响正常刷题。

## CPH

CPH 不读取 `tasks.json`，工作区已单独配置为使用 `C:\msys64\ucrt64\bin\g++.exe`，参数为 `-std=c++17 -O2 -Wall -Wextra`。安装浏览器端 Competitive Companion 后，可以从 Codeforces 等题目页面抓取样例并批量测试。

## 上传到 GitHub

先在 GitHub 网页新建一个空仓库，例如 `competitive-programming`，不要勾选 README、`.gitignore` 或 License。然后在本目录执行：

```powershell
cd D:\competitive-programming
git init
git add .
git commit -m "Initial competitive programming setup"
git branch -M main
git remote add origin https://github.com/<你的用户名>/competitive-programming.git
git push -u origin main
```

之后更新配置：

```powershell
git add .
git commit -m "Update VSCode configuration"
git push
```

配置中的编译器路径是本机路径。其他 Windows 用户若安装位置不同，需要把 `.vscode` 中的 `C:\msys64\ucrt64\bin` 改成自己的 UCRT64 路径。
