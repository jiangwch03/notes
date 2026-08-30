# Python `import` 的本质与 Monkey Patch

## 一、`import` 到底做了什么

Python 中所有 `import` 语句的本质都是一句话：

> **把某个对象的引用，绑定到当前模块的命名空间里。**

模块本身是一个 Python 对象，模块里所有的"名字"（变量名、函数名、类名）都存放在它的 `__dict__` 中。`import` 只是往这个 `__dict__` 里加了一条 `名字 → 对象` 的映射。

### 1. `import xxx`

```python
import fastapi
```

等价于：

- 加载 `fastapi` 模块对象（如未加载）
- 在当前模块命名空间中绑定：`fastapi → <module 'fastapi'>`

### 2. `from xxx import yyy`

```python
from fastapi.openapi.docs import get_redoc_html
```

等价于：

- 加载 `fastapi.openapi.docs` 模块（如未加载）
- 取出该模块里名为 `get_redoc_html` 的属性
- 在当前模块命名空间中绑定：`get_redoc_html → <function get_redoc_html>`

### 3. `from xxx import yyy as zz`

```python
from fastapi.openapi.docs import get_redoc_html as redoc
```

只是把绑定名换成 `redoc`，引用对象不变。

---

## 二、关键认知：导入的是"引用"，不是"拷贝"

`import` 拿到的是对象的**引用**（指针），不是值的副本。

```python
# a.py
data = [1, 2, 3]

# b.py
from a import data
data.append(4)        # 修改的是同一个 list 对象
print(a.data)         # [1, 2, 3, 4]
```

但要注意"引用绑定"和"对象本身"的区别：

```python
# b.py
from a import data
data = [9, 9, 9]      # 仅改变 b 里 data 这个名字的指向
print(a.data)         # [1, 2, 3]   ← a 里的 data 没受影响
```

---

## 三、模块属性的两种来源

模块里出现的名字，来源有两种，但**对外效果完全一样**：

| 来源 | 写法 | 是否存在于 `module.__dict__` |
| --- | --- | --- |
| 模块内定义 | `def foo(): ...` | ✅ |
| 从外部导入 | `from x import foo` | ✅ |

所以即使一个函数不是在某文件里 `def` 出来的，只要它被 `import` 进来，**它就是这个模块的属性**，可以通过 `module.foo` 访问，也可以被赋值替换。

---

## 四、Monkey Patch 的原理

由于"模块属性可被赋值替换"，就有了 monkey patch：

```python
import fastapi.applications as applications
applications.get_redoc_html = my_func   # 替换模块里这个名字的指向
```

**关键**：模块内部的代码引用名字时，是从**该模块自己的命名空间**里查找的。

```python
# fastapi/applications.py
from fastapi.openapi.docs import get_redoc_html

class FastAPI:
    def setup(self):
        return get_redoc_html(...)   # 查的是 applications 模块里的 get_redoc_html
```

所以：

- ✅ 改 `applications.get_redoc_html` → FastAPI 内部调用会走到新函数
- ❌ 改 `fastapi.openapi.docs.get_redoc_html` → 对 `applications` 内部毫无影响（因为它早已把引用复制到自己命名空间）

> **Monkey patch 的黄金法则：要 patch 谁内部的调用，就改谁那个模块里的引用，而不是改原产地。**

---

## 五、常见易错点

### 1. `from xxx import yyy` 之后再改原模块没用

```python
# x.py
val = 1
def show(): print(val)

# main.py
from x import show, val
import x
x.val = 100
show()        # 1   ← show 内部访问的是 x 模块里的 val，没问题
print(val)    # 1   ← main 里的 val 是早期拷贝的引用，仍指向旧的 1
```

### 2. 循环导入

`A 导 B、B 导 A` 时，谁先 import 就先把对方加载到一半的状态绑过来，后续访问到未定义的属性就会报错。解决方式：把 import 移到函数内部，或重新设计依赖结构。

### 3. `import x` vs `from x import y` 的可测试性差异

```python
# 写法 A
import time
def now(): return time.time()    # 测试时可 mock time.time

# 写法 B
from time import time
def now(): return time()         # 已经在本模块绑定了引用，mock time.time 没用
```

写法 A 更利于 mock / patch，因为它每次都通过模块查找名字。

---

## 六、一句话总结

> **`import` = 把别处的对象引用，绑定到当前模块命名空间的字典里。**
>
> 之后所有的"模块属性"行为、monkey patch 行为、mock 行为，都只是在操作这本"字典"。
