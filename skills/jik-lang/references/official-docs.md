# Jik 官方文档导航与开发要点

来源：[官方文档首页](https://jik-lang.org/docs/index.html)，查阅于 2026-11-02。官网文档可能随 beta 版本变化；它是在线参考，不保证与已安装编译器一致。优先用 `jik version`、`jik help topics` 和对应版本的帮助、源码与测试确认；官网不可用时可使用本地帮助和技能中已有内容。

## 按任务查找

| 任务 | 官方文档 |
|---|---|
| 命令、参数、编译器选择 | [CLI 参考](https://jik-lang.org/docs/cli.html)、[运行代码](https://jik-lang.org/docs/overview/02-running-jik-code.html) |
| 查精确语法、运算符和产生式 | [语法参考](https://jik-lang.org/docs/grammar.html)、[基础语法](https://jik-lang.org/docs/overview/04-basic-syntax-and-structure.html) |
| 模块、导入与包 | [文件、模块和导入](https://jik-lang.org/docs/overview/03-files-modules-and-imports.html) |
| 值、类型、变量、赋值 | [值和类型](https://jik-lang.org/docs/overview/05-values-and-types.html)、[变量和赋值](https://jik-lang.org/docs/overview/06-variables-and-assignment.html) |
| 结构体与区域生命周期 | [结构体](https://jik-lang.org/docs/overview/07-structs.html)、[内存管理](https://jik-lang.org/docs/overview/08-memory-management.html) |
| 枚举、可选值、变体 | [枚举](https://jik-lang.org/docs/overview/09-enumerations.html)、[Option](https://jik-lang.org/docs/overview/10-options.html)、[Variant](https://jik-lang.org/docs/overview/11-variants.html) |
| 容器与表 | [Vector](https://jik-lang.org/docs/overview/12-vectors.html)、[Dictionary](https://jik-lang.org/docs/overview/13-dictionaries.html)、[Table](https://jik-lang.org/docs/overview/14-tables.html) |
| 分支、循环、错误处理 | [控制流](https://jik-lang.org/docs/overview/15-control-flow.html)、[错误处理](https://jik-lang.org/docs/overview/16-error-handling.html) |
| 调用 C 或检查编译兼容性 | [C 互操作](https://jik-lang.org/docs/overview/17-ffi-c-interop.html)、[已知限制](https://jik-lang.org/docs/known-issues.html) |
| 内建函数与标准库 API | [内建函数](https://jik-lang.org/docs/overview/18-builtins.html)、[标准库入口及模块文档](https://jik-lang.org/docs/overview/19-standard-library.html) |
| 修改编译器、自举、仓库测试 | [开发说明](https://jik-lang.org/docs/development.html) |

## 排查重点

- **区域错误**：先找出每个复合值的所属 region 和预期存活时间。受同区域规则约束的调用中，非 `foreign` 复合参数和 `Region` 参数决定复合返回值的区域；局部值不能直接返回。编译器能证明区域安全的函数有例外，具体判断查内存管理章节。
- **跨区域数据**：`foreign` 用于允许不参与复合值存储的参数来自其他区域；它不表示不可变，也不允许返回以该参数为根的复合值。需要保留数据时优先直接分配到目标区域，或在 `copy` 支持的类型范围内复制；不要假定它支持任意嵌套结构的深拷贝。
- **错误处理**：`throws func` 标记可能失败的函数；`try ... except ... end` 本地处理失败，而抛错函数中的 `try` 声明可以传播失败。`must` 和后缀 `!` 要求调用成功，不是恢复机制。按错误处理章节核对允许的位置与具体语义。
- **C 互操作**：`extern` struct 是不透明句柄，字段不能在 Jik 中直接访问；通过 extern 函数构造和访问。检查传入 C 的数据生命周期、资源释放和平台兼容性，区域安全不应被假定为能自动约束任意 C 代码。
- **平台和构建**：官网列出的测试平台为 Windows x64 和 Linux x64；使用 Clang 或 GCC，MSVC 不受支持。生成的 C 可能依赖 GCC/Clang 扩展并产生警告，部分 C 类型的对齐要求可能无法由 region 内存满足。移植或 FFI 前重新查当前限制。
- **定位标准库**：先在标准库入口找到具体模块，再核对该版本文档或实现中的签名、错误行为和 region 参数；不要从其他语言猜测 API。

完整、已验证的入门程序另见本技能的 [examples.md](examples.md)。以上要点不代替规范，尤其不要仅根据速查内容推断未列出的类型、API 或生命周期规则。
