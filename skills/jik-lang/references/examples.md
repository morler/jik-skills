# Jik 示例速查

以下完整程序参考 [Jik 官方示例](https://jik-lang.org/examples.html) 改写，分别保存为独立 `.jik` 文件。示例 API 和语义可能随版本变化；编写时应对照当前版本的 `jik help` 和源码示例。以下程序已用 `jik version: 0.1.0-beta.1` 的 `jik check` 和 `jik run` 验证（2026-11-02）。

## 范围、分支和循环

范围上界不包含；`continue` 跳过当前轮，`break` 结束循环。

```jik
func main():
    total := 0
    for n = 1, 5:
        if n == 2:
            continue
        elif total + n > 5:
            break
        else:
            total += n
        end
    end
    println("total: ", total)
end
```

输出：`total: 4`。

## 字典与 Option

字典读取返回 Option；确认是 `Some` 后再用 `?` 取值。不要依赖字典遍历顺序。

```jik
func main():
    stock := {"pens": 2, "books": 3}
    stock["pens"] = 4
    count := stock["pens"]
    if count is Some:
        println("pens: ", count?)
    end
    if stock["erasers"] is None:
        println("erasers missing")
    end
end
```

输出依次为 `pens: 4`、`erasers missing`。

## 区域和复合返回值

调用者传入目标区域，构造值使用 `[r]`；`_` 表示调用者的局部区域。嵌套的复合值也分配到目标区域。

```jik
struct Notebook:
    notes: Vec[String]
end

func new_notebook(r: Region) -> Notebook:
    return Notebook{notes = ["first"]}[r]
end

func add_note(book: Notebook):
    note := "second"[.book]
    push(book.notes, note)
end

func main():
    book := new_notebook(_)
    add_note(book)
    println(book.notes)
end
```

## 抛错与处理

`try` 块在 `except` 中处理失败；在抛错函数中，声明右侧的 `try` 可向调用者传播失败。`must` 仅适用于调用者确定成功的情况。

```jik
throws func validate_count(count):
    if count < 0:
        fail("negative count")
    end
    return count
end

throws func half_count(count):
    valid := try validate_count(count)
    return valid / 2
end

func main():
    try half := half_count(-1):
        println("half: ", half)
    except:
        println("error: ", error_msg())
    end
    println("valid: ", must validate_count(4))
end
```

输出依次为 `error: negative count`、`valid: 4`。

## 测试套件

这是官网 `testing_demo.jik` 风格的普通可运行程序，使用 `jik run`；不要与 `jik test` 的测试发现机制混淆。

```jik
use "jik/testing" as test

func square(x):
    return x * x
end

func main():
    ts := test::suite_new()
    test::suite_assert(ts, square(3) == 9, site())
    test::suite_assert(ts, square(-4) == 16, site())
    test::suite_finish(ts)
end
```

更多主题（包括本地模块、多文件示例、字符串、向量、文件系统和 FFI）见[官方示例目录](https://jik-lang.org/examples.html)及对应版本的源码仓库 `examples/`。运行依赖文件或交互的示例前，先检查其前置条件和副作用。
