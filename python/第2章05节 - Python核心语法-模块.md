# Python 核心语法：模块与包

> **学习目标：** 按职责拆分代码，通过导入复用功能，并理解模块执行入口。

## 模块与导入

一个 `.py` 文件就是模块，可定义变量、函数、类和可执行代码。拆分模块能降低复杂度并提高复用性。

```python
# circle.py
def area(radius):
    return 3.14 * radius ** 2

# main.py
import circle
print(circle.area(10))
```

常用导入写法有 `import module`、`import module as alias`、`from module import name`。

> **重点：** 推荐通过 `module.function()` 调用以避免同名冲突。`from module import *` 会污染命名空间，通常不应使用；`__all__` 只影响星号导入，并非安全机制。

## 入口与包

模块直接运行时 `__name__ == "__main__"`，被导入时 `__name__` 是模块名。测试和演示代码应放入入口判断，避免导入时意外执行。

```python
if __name__ == "__main__":
    print(area(10))
```

包用于组织多个模块。课程中常见的 `__init__.py` 可明确包边界、初始化或控制导出；从项目根目录运行程序可减少相对导入和同名文件导致的问题。
