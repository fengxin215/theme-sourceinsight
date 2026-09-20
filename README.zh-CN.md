# Source Insight 配色 · VS Code

使用 source insight 写 C 语言的老铁们，终于可以在 VS Code 中使用 Source Insight 配色了。
界面沿用 VS Code 内置 **Light Modern**，仅替换代码文本配色，无粗体。

## 目录结构

```
theme-sourceinsight/
├── package.json                        # 扩展清单
└── themes/
    └── source-insight-light.json       # 主题本体
```

## 前置条件

- Linux + VS Code）
- C 语言开发需要网络（下载 clangd 二进制）

---

## 第 1 步：安装主题

把整个目录复制到 VS Code 的本地扩展目录（注意：目标目录名必须保持 `theme-sourceinsight`）：

```bash
cp -r /path/to/theme-sourceinsight ~/.vscode/extensions/theme-sourceinsight
```

重启 VS Code 然后：选择 **Source Insight Light** 主题

> 到这一步颜色已生效，但 C/Go 的"定义/引用区分着色"需要语言服务配合，继续往下。

## 第 2 步：C/C++ 支持（clangd）

### 2.1 安装 clangd 扩展

```bash
code --install-extension llvm-vs-code-extensions.vscode-clangd
```

### 2.2 安装 clangd 二进制

```bash
curl -fL -o /tmp/clangd.zip \
  https://github.com/clangd/clangd/releases/download/18.1.3/clangd-linux-18.1.3.zip
unzip -q /tmp/clangd.zip -d ~/.local/
~/.local/clangd-linux-18.1.3/bin/clangd --version   # 应输出版本号
```

> 注意：解压出来的目录名是 `clangd-linux-18.1.3`（中划线）。
> 如果有 root 权限，也可以 `sudo apt install clangd`，并把下述 `clangd.path` 写成 `clangd`（走 PATH）。

### 2.3 配置 settings.json

~/.config/Code/User/sttings.json，加入：

```json
"clangd.path": "/home/<你的用户名>/.local/clangd-linux-18.1.3/bin/clangd",
```

如果同时装了微软的 C/C++ 扩展（cpptools），需要关掉它的智能感知，避免和 clangd 打架
（**调试功能保留，仍用 cpptools**）：

```json
"C_Cpp.intelliSenseEngine": "disabled",
"C_Cpp.enhancedColorization": "disabled"
```

### 2.4（可选，推荐）提升 C 工程解析精度

CMake 工程配置时加一个参数，生成 `compile_commands.json` 放到工程根目录：

```bash
cmake -DCMAKE_EXPORT_COMPILE_COMMANDS=ON ...
```

没有它 clangd 会用回退参数"猜"，着色仍可用，只是头文件相关的诊断偶有误报。

## 第 3 步：Go 支持（gopls）

### 3.1 安装 Go 扩展

```bash
code --install-extension golang.go
```

gopls 二进制不用手动装——扩展首次打开 .go 文件时会自动装到 `~/go/bin/`。

### 3.2 关键设置

**gopls v0.22.0 起有一个回归：语义 token 默认返回空包**，表现为函数调用处不上分类色。
必须在 settings.json 里显式打开（注意：老教程里的 `uiSemanticTokens` 选项已改名，
用了会报 "unexpected setting"）：

```json
"gopls": {
    "semanticTokens": true
}
```

改完后重启vscode。

## 第 4 步（可选）：推荐设置

```json
"editor.inlayHints.enabled": "off"      // 关掉调用处灰色的 a: b: 参数名提示
```

## 完整 settings.json 参考

```json
{
    "workbench.colorTheme": "Source Insight Light",
    "clangd.path": "/home/<你的用户名>/.local/clangd-linux-18.1.3/bin/clangd",
    "C_Cpp.intelliSenseEngine": "disabled",
    "C_Cpp.enhancedColorization": "disabled",
    "gopls": { "semanticTokens": true },
    "editor.inlayHints.enabled": "off"
}
```

---

对照颜色表：

| 元素 | 颜色 |
|---|---|
| 注释 | 紫 |
| 关键字 / 运算符 / `.` `->` | 绿 |
| `if` `for` `return` 等控制流 | 藏青 |
| `#include` `#define`（含 `#`） | 绿 |
| 字符串 | 橙（深烧橙 `#a63d00`） |
| 数字 / `NULL` | 红 |
| 函数 / 类型 / 变量 / 宏**定义处** | 藏青 |
| 函数调用、宏引用、成员访问 | 绿 |
| 局部变量、参数**引用处** | 青 |
| 全局变量引用（仅 C） | 紫 |
| 枚举成员 | 红 |
| `() {} [] , ;` | 紫 |

Go 侧同理：函数定义藏青、调用绿、变量青、包名深灰（命名空间引用同为深灰）、常量红。

## 已知限制（VS Code 平台限制，非配置问题）

1. **字符串浅黄底**：VS Code 不渲染 token 级背景色，SI 的 `#ffffbb` 字符串底色无法还原
2. **常数宏红色**：clangd 不区分"常量型宏"（`#define N 3`）与函数宏，引用统一按宏的绿色处理
   （SI 中常量宏引用为红色）
3. **Go 全局变量**：gopls 的语义 token 无"全局/局部"标记，Go 的包级变量与局部变量同为青色
   （C 侧全局变量为紫色，正常）
4. 全部无粗体，对应source insight的mono模式

## 卸载

```bash
rm -rf ~/.vscode/extensions/theme-sourceinsight
```

并删除 settings.json 中上述相关配置项。

## 示例
![演示](assets/sample.gif)
