---
title: 读本好书 《Python 进阶》
tags:
  - python
url: 492.html
id: 492
categories:
  - DEV
date: 2018-10-28 00:00:00
---

前
-

坚持自己的读书计划吧，不断地学习新知识和历练自己的能力水平。来填充自己平时的空白，的确是一件很舒服的事情不是吗。

没必要去羡慕，静下心来做好自己的东西就好了。很多东西一旦拥有了，也许就不会珍惜了，得不到的永远在骚动，被偏爱的却有恃无恐。

* * *

平时总是有很多想法，偶尔的写写 XD。这次的书，篇幅很短，所以也很顺利的在一周之内读的差不多了。正值周末，来写写笔记，和一些使用的demo 。不然就忘得差不多了。

简介
--

*   书名：Intermediate Python
*   作者：yassoob
*   ISBN：N/A
*   开源书籍

本书是一本开源书籍，开源的精神真的感染了我们每个人，我写书，大家来翻译，来排版，来查错。大家都是自愿的一起去实现同一个目标，就算功底不行，提个 issue 也好呢。

> 感谢英文原著作者 @yasoob《[Intermediate Python](https://github.com/yasoob/intermediatePython)》，有了他才有了这里的一切
> 
> 中译版 [Python 进阶](https://github.com/eastlakeside/interpy-zh)

* * *

这是一本技术类书籍，介绍了很多 python 的较为进阶的语法和使用指南， 这里就直接做个总结吧。

\*args 和 \*\*kwargs
-------------------

`*args` 和 `**kwargs` 实现了函数里面的不定参数。比如 python 里面的 `print()`函数。我们就一次性打印多个参数。比如以下实例代码

    def args(*argvs):
        for i in argvs:
            print(i)
    
    args('1',2,'test')

可见，实际上我们的多个参数使用 `*args` 在进入函数之后，会被组装成一个 `tuple` 元组。我们可以对其进行迭代。

**值得注意的**是 `*args` 并不是固定的，这个只是默认的约定，如上述代码里面的 argvs 一样的是可以成功运行。

* * *

`**kwargs` 这个名字看起来不含辨识，实际上如其功能，这个是传递的是不定长度的键值对 **key-value**，KV ≈ KW ？

问：dict 字典本身不就是不定长度的吗？ 一样可以直接作为参数传入，

答：看示例代码的形式

    def args(**kwargs):
        print(kwargs.key())
        print(kwargs.values())
    
    args(a=123, b=456, c=789)

这个传入的形式是不是相当的眼熟，没错，我们的pymysql 里面的进行连接的函数一样的使用的是这种形式。所以对于我们的启发是。

python的函数可以同时使用 常规参数，和这种可变长字典的形式，这样可以用于我们多出的参数， 也增加了程序的鲁棒性。

* * *

如果同时的去使用 这两种形式，其顺序如下

    some_func(fargs, *args, **kwargs)

这里还有个很有趣的用法，叫做 **猴子补丁(monkey patch)** 指的是程序再起运行的时候进行动态的 hot patch。就像 Django 一样，可以动态的修改路由不必停止其整个进程。

PDB
---

PDB 可以实现对 py 文件进行简单的调试：

    python -m pdb my_script.py

##### 命令列表：

*   `c`: 继续执行
*   `w`: 显示当前正在执行的代码行的上下文信息
*   `a`: 打印当前函数的参数列表
*   `s`: 执行当前代码行，并停在第一个能停的地方（相当于单步进入）
*   `n`: 继续执行到当前函数的下一行，或者当前行直接返回（单步跳过）

生成器
---

生成器，可以抽象成一个 函数，输入有对应的输出。生成我们的结构，而不一个大大的列表，用来存内容，显得 太不 pythonic。

> 迭代器是一个让程序员可以遍历一个容器（特别是列表）的对象。

这里书里有的三个概念：

*   可迭代对象(Iterable) 比如列表，他是可迭代的
*   迭代器(Iterator) 用于迭代可迭代容器的一个对象
*   迭代(Iteration) 遍历课迭代对象的过程

* * *

> Python中任意的对象，只要它定义了可以返回一个迭代器的`__iter__`方法，或者定义了可以支持下标索引的`__getitem__`方法(这些双下划线方法会在其他章节中全面解释)，那么它就是一个可迭代对象。

这个也就解释了，当一个错误的类型被使用 `[1]` 进行索引报错的，是没有`__getitem__` 这个方法。

> 任意对象，只要定义了`next`(Python2) 或者`__next__`方法，它就是一个迭代器。就这么简单。现在我们来理解迭代(iteration)

这个 对象可以使用 next ，对容器的下一个继续迭代。

（这里大面积引用，因为地区是很简明扼要的。）

* * *

正如之前说，生成器是可以看作一个函数，每次都是有一个输出，而不是大量的一堆的数据。下面的代码便是实现了一个生成器

    def generator_function():
        for i in range(10):
            yield i
    for i in generator_function():
        print(i)

有了这样的方法，我们定义的生成器是迭代的，最大的好处，是我们不必一次性的生成大量的数据。like：

    def mass(n):
        a = []
        for i in range():
            a.append(i*i)
        return a        
    
    for i in mass(100):
        print(i)

这样的方法我们生成大量的数据的时候，会占据大量的数据缓冲，十分的不 pythonic，所以换个写法：

    def mass(n):
        for i in range(n):
            yield i*i   
    
    for i in mass(100):
        print(i)

这种写法，实现了一个迭代器，使用 for 结构，对其进行迭代。可是，其实际上的过程呢，看这里:

    print(type(mass))
    print(type(mass(10)))
    print(next(mass(10)))
    

可见构造的生成器函数在初始化之后成为了一个生成器，且其为一个课迭代对象，可以使用next 进行迭代。

* * *

迭代器，前面提到是可以对可迭代容器进行迭代的对象。我们常见的 `str` 本身是一个可迭代对象，但是不是迭代器。

    mStr = "hello"
    str_iter = iter(mStr)
    print(next(str_iter))

这样，就完成了一个迭代器的初始化，以及进行了迭代器的操作 。

Map，Filter，Reduce
-----------------

这几个东西，就是相当的 pythonic 了，实现对可迭代数据的批量操作。用的好的话，程序是十分优雅的~。

经常会有这样的代码，真的是蠢蠢的格式，和行为

    a = [1,2,3,4,5,6]
    x = []
    for i in range(len(a)):
        x.append(a[i]**2)       

那么怎么办呢， 这里就出现了 map 函数，实际上这函数在 JS 里面也是有的。

    a = [1,2,3,4,5,6]
    x = list(map(lambda x: x**2, a))

这样的简单的一句，就实现了对于整个数数组的批量操作，非常的优雅不是吗~。里面的 `lambda` 称之为 匿名函数，是个函数，但是没有吗名字，多用于临时的使用的函数，或者简单的单行函数。**当然我们可以给他个名字**

    a = [1,2,3,4,5,6]
    pow = lambda x: x**2
    x = list(map(pow, a))

* * *

这里还有个很厉害的操作，这里的 map 批量操作的对象甚至是可以是一个 函数的列表。比如实现，对一个列表的数据，进行多个不同的操作，并且可以容易的进行对比：操作如下：

    def multiply(x):
            return (x*x)
    def add(x):
            return (x+x)
    
    funcs = [multiply, add]
    for i in range(5):
        value = map(lambda x: x(i), funcs)
        print(list(value))

* * *

`filter` 正如其名，对数据进行过滤，再也不需要一个for 遍历之后，再 append 一个新的数组出来了。

    a = range(0,100)
    b = filter(lambda x:(x+1)% 2 == 0, a)
    print(b)

完美优雅的筛选出了0~99 直接的所有的奇数；

* * *

reduce 用于对一个可迭代对象内部的元素进行批量操作。比如列表求和，直到 求方方差之类的。

    mids = map(lambda x: x.replace('\n', ''), list(s))
    mids = set(mids)    # 直接转集合去重
    h5_list_1 = reduce(lambda x,y: x + '<option value="' + y +'">',mids)

一样进行了迭代，和兼并操作~可以得到这个一个进行拼接的一个 `<option></option`\> 的一个列表。

Set 集合
------

不是这里看到差点忘了 python 还有这种的数据超类型。近似于列表，但是集合元素不可重复。且可以进行集合的与或非运算哦。赶紧巩固一下初始化形势 `a = {1,2,3}`

**如何从 一个列表里面剔除重复的元素。** 集合提供了完美的解决方案！

    a = [1,1,1,1,2,2,2,2,3,3,4,3,5]
    print(set(a))

简直是。。。完美

三元运算符
-----

正如，一般的 JS/C 的一样的存在三元运算符 `(Q?A:B)`, python 也是存在这样的一个 三元运算符，恨意很方便的实现逻辑的简单的判断。

    A if statement else B

与其不一样的是其判别式在中间， 左真右假。

还有一种的变体，显得比较不直观：

    (a,b)[statement]

装饰器
---

记得第一次 遇上这个特性是在写 `solidity`的时候遇上的，通过 装饰器的结构，可以很容易的在一个方法的外面，再给他套上一层的功能。自己之前用过如下的一个：

    def time_me(fn):
        u'''
        函数耗时修饰器
        '''
        def _wrapper(*args, **kwargs):
            start = time.clock()
            ret = fn(*args, **kwargs)       # 这里记得加返回值，血坑
            print u"%s cost %s second"%(fn.__name__, time.clock() - start)
            return ret
        return _wrapper
    
    @time
    def main ():
        pass

这里就用到了之前的特性 `*args， **kwargs` 这个适用于传递我们的可变参数。在python 里面有句话 **一切皆对象** ，所以函数也如此，阅读了前面的修时器的代码，其实不难发现，其传入的是个函数，返回的也是个函数，这个返回的函数在我们的传入函数两边包裹了一些内容。实现了对函数的功能修饰。下面自己构造一个修饰器：

    a = range(0,100)
    
    def my_decorator(func):
        def func_warp(*args,**kwargs):
            return func(*args,**kwargs)
        return func_warp
    
    @my_decorator
    def app():
        return filter(lambda x:(x+1)% 2 == 0, a)
    print(app())

这个也行是最没有用的修饰器了，只是添加了调用返回的过程，但是没有任何作用，不过这个的确实现了一个修饰器的作用，我们成功的进行了一次修饰，而且捕获了他的返回值。

还是上面的代码，注意一个使用细节：

    print(my_decorator(app)())

如果 `@` 的符号出现的太突兀，我们可以使用这样的形式来使用修饰器，很好理解~，函数传入了我们的修饰器（函数），返回了经过修饰的函数，最后再进行调用。这样完成了整个的修饰过程~。

* * *

修饰器作为一个如此强大的功能，那么器具体的作用呢？其实有相当的多。比如上面给出的 一个函数计时器。在函数调用，以及函数返回 的过程来进行 计时，以及时间的打印。在 `Flask` 里面。修饰器的存在也是相当的频繁了，对于我们的路由进行直接的修饰。十分的方便。

Return 与 Global
---------------

这个是比较常用的特性，简单讲吧。

    global val  # 用于声明 全局的变量。

return 可以返回多个返回值，而不需要在返回前先把参数进行打包：

    return a,b
    a,b = c()
    
    return (a,b)
    res = c()
    res[0],res[1]

对象变动(Mutation)
--------------

这个问题是相当的有趣了，让我想到很久之前一个自己写出的一个bug，其代码示例示例如下

    x = 1
    y = [1]
    q=[]
    p=[]
    for i in (range(3)):
        x += 1;
        p.append(x)
        y[0] = y[0] + 1
        q.append(y)
    print(p,q)

为什么这样讲？看起来很简单的代码呀。实际上其结果是这样的：

    ([2, 3, 4], [[4], [4], [4]])

当时，挺震惊的~。 完全的不符合自己的直觉呀。实际上 这个就遇到了 深拷贝问题：

> Python 中的**复杂类型的传递，是通过引用**的。这个引用，就是我们最熟悉的 id （抽象化的地址）

怎么讲呢？ 上面的代码我们稍作修改：

    x = 1
    y = [1]
    q=[]
    p=[]
    for i in (range(3)):
        x += 1;
        p.append(x)
        print('x', id(x))   # < 打印ID
        y[0] = y[0] + i
        q.append(y) 
        print('y', id(y))   # < 打印 ID
    print(p,q)

其结果如下 ：

    这里我们得注意 id 的值
    ('x', 72836976L)    < 
    ('y', 80438856L)
    ('x', 72836952L)    < 
    ('y', 80438856L)
    ('x', 72836928L)    < 
    ('y', 80438856L)    

过程发现 ，y 的 id 在整个过程中 都是一样的，也就是地址是一样的，然而 x 的 id 随着值得递增，在不断变化！这个就是这里的核心问题。下面进行更进一步的探索！

    x = 1
    y = [1]
    q=[]
    p=[]
    
    def ccc(x,y):
        for i in (range(3)):
            print('x', id(x))
            x += 1;
            p.append(x)
            print('y', id(y))
            y[0] = y[0] + i
            q.append(y)
        print(p,q)
    print(id(x), id(y))
    ccc(x,y)

这里的命名有些随意了，不过注意主要是展示问题的 嘻嘻。这段代码的输出：

    (72116104L, 82077256L)
    ('x', 72116104L)    < 
    ('y', 82077256L)        <
    ('x', 72116080L)
    ('y', 82077256L)
    ('x', 72116056L)
    ('y', 82077256L)
    ([2, 3, 4], [[4], [4], [4]])

这里可以看到， 我们传入 的 int 的 ID 在函数传递后，是没有变化的，操作之后，发生了变化。然而，传入的数组的，整个过程的 ID 是没有发生变化的。所以可以确定，我们传入的是引用。所以，会导致了这样的问题产生。

* * *

问题的解决：这里自己之前遇到的解决方法是使用 **对象的拷贝**，作为另一个变量传入，而不是引用,代码如下：

    x = 1
    y = [1]
    q=[]
    p=[]
    
    def ccc(x,y):
        import copy
        for i in (range(3)):
            b = copy.deepcopy(y)
            b[0] = b[0] + i
            print('b', id(b))
            q.append(b)
    
        print(p,q)
    
    print(id(x), id(y))
    ccc(x,y)

这里使用了 python 的复制模块，可以实现 对象的拷贝，可以是强制的复制形参。得到的结果，就看起来正常的多：

    (74868616L, 81159752L)
    ('b', 81172168L)
    ('b', 81160136L)
    ('b', 81161160L)
    ([], [[1], [2], [3]])
    

* * *

内容压缩
----

*   `__slots__` 用于对类定义的时候，指定确定的参数。而不是
*   `virtualenv` 建立python的虚拟环境，避免依赖混乱

对象自省
----

> 自省(introspection)，在计算机编程领域里，是指在运行时来判断一个对象的类型的能力。它是Python的强项之一。

python 里面提供了自省的模块，我们可以使用 `dir` 可以列出了一个对象的所有的成员。

`__doc__` 这个成员变量，是一般类的说明。

`type` 用于查看一个对象的类型

`id`可以理解为 C 里面的地址

推导式 (comprehensions)
--------------------

推导式的功能是极其强大了，可以实现十分 pythonic 的写法和功能。使用这个功能可以很轻松的对连续冗余的 for 循环，进行压缩。这里就随便贴一句，不过可读性太差了。

    for i in filter(lambda l: re.match('.*404$', l['name']),[x for key in cts for x in cts[key]]):

python 的推导式的形式如下：

    variable = [out_exp for out_exp in input_list if out_exp == 2]

推导式在功能上和filter 有些相似，不过，推但是在进行运算的本身，实际上也带了生成的作用，

比如下面的，快速生成：

    squared = []
    for x in range(10):
        squared.append(x**2)
    
    squared = [x**2 for x in range(10)]
    squared = map(lambda x:x**2, range(10))

异常 Exception
------------

代码里面的异常处理是很重要的，特别是当代码的体量大了之后，异常的处理处理显得尤为重要，不好的异常处理习惯，可能直接导致后面的奇怪的问题得不到解决，之前有遇到的问题是，异常的捕获过于随意，然而又没有经过处理，所以导致了，出现问题直接没有任何会显得情况。

**注意如果存在，没有捕获到的异常的时候使用 raise 把异常上抛**

    except Exception as e:
        print e
        raise

* * *

**finally** 从句，用于代码段执行之后的 处理，在异常发生与否，都会被调用。

**try/else** 从句， 用于try 中没有异常触发的情况下调用。

lambda 表达式
----------

这里的 **lambda** 的意思就是我们的函数。这里就是匿名函数的意思，这个概念在 JS 里面是大量存在的 `a(function(){...})` 。其具体的形式如下：

    lambda x,x1,x2:...

其用处在实习单行的简单函数的时候，将会显得十分优雅：

    def add(a,b):
        return a + b
    reduce(add, nums)
    
    reduce(lambda x,y:x+y, nums)
    add1 = lambda x,y:x+y
    reduce(add1, nums)
    
    ############################################
    a = [(1, 2), (4, 1), (9, 10), (13, -3)]
    a.sort(key=lambda x: x[1])
    
    print(a)

`lambda` 表达书的用处韩式相当广泛的。

一行式
---

这里讲了 python 的单行的妙用吧。

功能

命令

**简易Web Server**

`python -m SimpleHTTPServer`

**漂亮的打印**

`from pprint import pprint`

**json 解析**

`cat file.json | python -m json.tool`

**单行命令**

`python -c`

**列表辗平**

`itertools.chain.from_iterable()` 传入一个二维列表

for 的 else 从句
-------------

一个循环的退出，存在两种情况：

1.  循环到最后的结束
2.  循环内部的 break

当我们需要进行分辨这两种情况的时候，就需要一个额外的标志位。这里如果使用 else 的话满清可以很轻易的实现这个功能：

    for item in iteration:
        ...
    else:
        # for 不是通过 break 结束的时候执行

Python 对 动态链接库的使用
-----------------

python 可以很容易的使用系统中的动态链接库 `.so` (Shared Object)

    //sample C file to add 2 numbers - int and floats
    
    int add_int(int, int);
    float add_float(float, float);
    
    int add_int(int num1, int num2){
        return num1 + num2;
    
    }
    
    float add_float(float num1, float num2){
        return num1 + num2;
    }

编译命令如下：

    $  gcc -shared -Wl,-soname,adder -o adder.so -fPIC add.c

之后可以直接在 Python 从进行引用：

    from ctypes import *
    
    adder = CDLL('./adder.so')
    
    res_int = adder.add_int(4,5)
    print "Sum of 4 and 5 = " + str(res_int)

Python/C API
------------

这部分，讲了使用 C 写python 的模块，可以有更好的性能。

python 协程
---------

协程的概念，可以理解为一个可以多次返回的函数。其关键字是 `yield` 这个关键字，我们在前面的生成器里面见过，协程至于生成器的不同，我已理解为一个进行 参数的输出的，另一个是进行输入的。

> [协程](https://eastlakeside.gitbooks.io/interpy-zh/content/Coroutines/) 参考原书内容

with/as 上下文管理结构
---------------

python 里面的：

    with open('some_file', 'w') as opened_file:
        opened_file.write('Hola!')

就是一个上下文管理结构， 指的是 一个结队操作中间夹杂其他代码的很好的解决方案。打开之后，在缩进结束之后，会进行自动的关闭