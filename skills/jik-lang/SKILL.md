---
name: jik-lang
description: 编写、审查和调试 Jik 语言及 .jik 文件；处理 jik check/run/build/tran/test、区域内存、模块导入、标准库、C 互操作、编译器及语言测试时使用。
---

# Jik 语言开发

Jik 是静态类型、编译到 C 的语言，复合值使用区域管理内存。先查当前版本文档与已有示例，再写代码；不要套用其他语言的语法或假定标准库 API 存在。本技能以 Jik 源码仓库的 `docs/`、`examples/`、`test/jik/` 和源码为依据；以下文档和示例路径均相对于 Jik 源码仓库根目录，而非 skill 或使用者项目目录。编写时参考的仓库提交：`003fa39`；遇到差异以当前实现和测试为准。

在其他项目使用已安装的 Jik 时，先运行 `jik version` 和 `jik help topics`，再用 `jik help <主题>` 查询对应版本的文档，例如 `jik help syntax`、`jik help regions` 或 `jik help testing`。`jik env` 可显示配置的 `root` 等路径，但不要假定该目录包含源码、文档文件或示例；只有确认文件存在后才使用下面的仓库相对路径。

## 文档导航

- 语言概览与主题索引：`docs/overview.md`；精确语法：`docs/grammar.md`。
- 运行与 CLI：`docs/overview/02-running-jik-code.md`、`docs/cli.md`。
- 模块：`docs/overview/03-files-modules-and-imports.md`；基础语法、类型和变量：`docs/overview/04-basic-syntax-and-structure.md` 至 `docs/overview/06-variables-and-assignment.md`。
- struct、区域、enum、option、variant、容器、控制流、错误处理、FFI：依次查 `docs/overview/07-structs.md` 至 `docs/overview/17-ffi-c-interop.md`；内建函数和标准库见后续章节。
- 标准库 API：`docs/jiklib/`；对应实现：`jiklib/`。按需阅读具体模块，不要猜函数签名。
- 可运行的主题示例及说明：`examples/README.md`；边界条件和诊断：`test/jik/`，特别是 `test/jik/test_regions.jik`、`test/jik/test_compiler_errors.jik`。
- 编译器构建与自举：`docs/development.md`；已知限制：`docs/known-issues.md`。

## 工作流程

1. 定位任务涉及的文档、相近示例和测试；修改编译器时还要检查 `src/jik/` 的相关实现。新语言功能放在自托管编译器 `src/jik/`，不要随意修改冻结的 `src/seed/` 或其测试。
2. 使用已安装版本时直接运行 `jik`。如果在 Jik 源码仓库中需要构建编译器，从仓库根目录运行 `make`，用随附 C seed 构建根目录的 `./jik`；需要 Make 和 Clang 或 GCC，选用 GCC 可执行 `make CC=gcc`。
3. 对目标程序先用 `jik check <目标文件>` 快速检查，再按需要用 `jik run <目标文件>` 执行。在 Jik 源码仓库根目录中也可用 `./jik check examples/hello.jik` 和 `./jik run examples/hello.jik` 验证仓库示例。`tran` 生成 C，`build` 生成可执行文件；运行和构建需要 C 编译器。参数及输出路径查 `jik help <命令>` 或源码仓库中的 `docs/cli.md`。
4. 为更改补充针对性测试。应用程序可使用 `jik test`（文件、目录和选项见 CLI 文档）；仓库编译器回归使用 `make test`，不是把 `test/jik/tests.jik` 当作普通测试目录。涉及 seed 或自举时，按 `docs/development.md` 选择 `make test-seed` 或 `make test-bootstrap-full`。
5. 报告实际执行的检查、结果及未覆盖的验证。避免运行交互式示例或清理生成物，除非任务需要。

## 与常见语言的差异：写代码前先检查

Jik 是新语言，外观相似不代表语法或语义相同。不要将 Python、Rust、Go、C 或 JavaScript 的惯用写法直接翻译过来；遇到不确定的语法，先查对应章节或 `examples/`，再用 `jik check` 验证。

| 容易沿用的习惯 | Jik 写法和注意事项 |
|---|---|
| Python 的 `def`、仅靠缩进结束代码块；C/Rust 的 `{ ... }` 函数体 | 使用 `func main(): ... end`。`if`、`for`、`while` 等也以 `:` 开头、`end` 结束；缩进供阅读，不决定代码块。 |
| `let` / `var` 声明，或把 `:=` 当成赋值 | `n := 1` 声明并推断类型，`n: int = 1` 显式声明，后续用 `n = 2` 赋值；已声明名字不要再次用 `:=`。未初始化的显式类型变量使用该类型的默认值。 |
| C/JS 的 `&&`、`||`、`!` 表示逻辑运算 | 使用 `and`、`or`、`not`；不要把逻辑非写成 `!`，Jik 的后缀 `!` 与错误处理有关。 |
| C/JS 的 `else if` 或 Python 风格省略结束符 | 使用 `elif condition:`，整个条件块以一个 `end` 结束。条件表达式的示例为 `a if condition else b`。 |
| C 的 `for (i = 0; ...)`、Python 的 `range()` | 数值范围示例为 `for i = 0, 3:`，上界不包含；遍历容器可用 `for index, value in values:`。具体迭代形式查控制流章节。 |
| Rust 的 `::` 用于引入模块，或 Python 的 `import` | 先 `use "jik/string"`，再以 `string::split(...)` 引用；本地导入相对于导入它的文件。 |
| C/JS 对象字面量或 Rust struct 初始化的 `field: value` | Jik struct 初始化使用 `Task{title = "..."}`，字段访问用 `task.title`；`{}` 等也有其他语言构造含义，不要随意套用。 |
| 把 `?` 当 Rust 的错误传播，或认为字典下标直接返回值 | 字典读取返回 `Option`；先 `if result is Some:`，再用 `result?` 提取。抛错函数用 `throws func`、`try`/`except` 等机制；详见 `examples/error_handling.jik`。 |
| 以 GC、引用计数或 Rust 借用直觉返回局部复合值 | 调用者提供 `Region`，返回值分配在目标 region；`_` 用于传递调用者局部 region，不能写成分配标注 `[_]`。先查内存管理章节。 |
| 字符串下标按 Unicode 字符或码点计数 | Jik 字符串索引和切片按字节处理；涉及非 ASCII 文本时查 `examples/strings.jik` 和字符串文档。 |

以下是完整程序，而非伪代码；同时展示声明、分支、范围和 option 检查：

```jik
func main():
    stock := {"pens": 2}
    count := stock["pens"]
    if count is Some:
        for i = 0, count?:
            println("pen ", i)
        end
    else:
        println("missing")
    end
end
```

提交前特别检查：块是否都有 `end`、声明是否误写成赋值、运算符是否照搬其他语言、复合返回值的 region 是否正确、option 是否先检查，以及 API 签名是否已查证。不要通过改写成另一种语言的语法来“修复”Jik 诊断。

## 核心约定与常见陷阱

- 每个 `.jik` 文件是一个模块；`use "jik/io"` 导入标准库，使用 `io::` 访问。普通导入相对于导入文件，`pkg/` 导入依赖 `JIK_PKG_PATH`；路径统一使用 `/`。
- 函数和控制流使用 `:` 和 `end`；声明可用 `:=` 推断类型。具体语法参照示例，不要假定 C/Rust 的大括号语法。
- `String`、容器、struct 等复合值由 region 管理。函数局部 region 在返回时销毁；返回复合值通常应接受 `Region` 参数，调用者传 `_` 表示调用者局部 region，在构造字面量时用 `[r]` 指定目标。不要返回局部分配的复合值，也不要把不兼容生命周期的复合值随意存入容器。
- `_` 是传递局部 region 的参数，不是分配标注：`[_]` 无效。`[.x]` 表示使用复合值 `x` 所在 region。复合字面量后第一个 `[...]` 是分配语法；先绑定变量再下标访问。详见内存管理章节和 `examples/regions_basic.jik`。
- 字典查询返回 option；提取值前检查。枚举/变体匹配、错误传播和 FFI 各有专门规则，分别查对应文档与示例，不要假定它们与其他语言相同。
- `jik run` 的编译选项在程序参数分隔符 `--` 之前；之后的参数传给程序。仅在明确接受风险时使用 `--unsafe-no-bounds-checks`。

## 更新和验证技能

文档或编译器变动后，对照对应版本的 `jik help` 与可用的源码文档更新此文件；用 `jik check <目标文件>` 和 `jik run <目标文件>` 验证相关示例，并按更改范围运行测试。在 Jik 源码仓库根目录可使用 `./jik check examples/hello.jik` 和 `./jik run examples/hello.jik`。语言语义以实际编译器及正反例为准；技能不是完整规范。
