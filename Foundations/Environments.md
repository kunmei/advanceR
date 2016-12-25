### 环境
环境是强化作用域的数据结构。本章节将深入探讨环境，深入地描述它的数据结构，使用它去提供你对在词法作用域中所描述的四中词法作用域规则的理解。

环境本身也是一个非常有用的数据机构，因为它们有引用语义。当你修改一个环境变量中的绑定的时候，环境不是被复制的；它随时被修改，引用语义通常
不是需要的，但是是及其有用的。

##### 纲要
- 环境变量基础　介绍一个环境的基础特性，并展示如何创建一个新的。
- 环境中的递归　修改Ｒ中作用域规则的深度，展示它们是如何和每个函数的四中类型的环境所对应的
- 绑定名字到值　描述了名字必须遵循的规则，展示绑定一个名字到变量的一些变化
- 具体的环境　讨论了环境本身作为一个有用的数据机构，以及作为在作用域中独立角色的三个问题

#### 环境变量基础
一个环境的工作是联系，或者绑定变量名称的集合到值的集合。你可以把一个变量认为是一包变量名称。

每个变量名称指向一个存储在内存中的对象

```
e <- new.env()
e$a <- FALSE
e$b <- "a"
e$c <- 2.3
e$d <- 1:3
```
对象不是存活在环境中，因此不同的名字可以指向同一个对象:

```
e$a <- e$d
```
同时它们也可以指向具有同样值的不同的对象

```
e$a <- 1:3
```
如果一个对象没有变量名称指向它，它将被垃圾回收器自动删除。这个过程将在gc章节详细的描述。

每一个环境都有一个父亲，另外一个变量。在图形中，我将会用一个小的黑圆圈代表指向其父亲的点。
父亲环境用来实现词法作用域:如果一个名字没有在环境中发现，Ｒ将寻找其父亲环境。唯一没有父亲的
一个环境是:空的环境。

我们可以使用家庭去隐喻环境。一个环境的祖父是其父亲的父亲，祖先包括所有父环境变量直到空环境。
很少取谈论一个环境的孩子，因为没有向后的链接:给定一个环境没有办法找到它的孩子。

通俗的来说: 一个环境同列表是类似的，但有四个重要的例外:
- 环境中的每一个对象都有一个独特的名字
- 一个环境中的对象不是有序的（例如，询问一个环境中第一个对象是没有意义的）
- 每个环境都有一个父亲
- 环境有引用语义

更技术的来说:一个环境是由两个部分组成的,frame(框架)，包含了名字和对象的绑定（表现地更像是一个名字的列表），
和其父亲环境。不幸的是，在Ｒ中，"frame"使用不是一致的。例如, parent.frame()并没有给出一个环境的父frame。
而是，它会给你调用环境。这将会在calling environment中详细讨论。

有四种特殊的环境:
- globalenv(),或者称为全局环境，是交互工作空间。这是你通常工作的环境。全局环境的父亲是你用library()和
require()加载的最后一个包。
- baseenv(),或者称为基础环境，是base包里面的环境。它的父亲是空的环境。
- emptyenv(),或者称为空的环境，是所有环境最终的祖先，是唯一没有父亲的祖先。
- environment(), 指当前的祖先。

search()会列出全局变量所有的父亲。这被称作搜索路劲，因为在这些变量中的所有对象都可以在最顶层的交互空间被
找到。每一个加载的包都有一个环境，你任何你attach()的其他对象。它还包括一个特殊的叫做Autoloads的环境，
仅仅是当需要加载包对象的时候（例如大的数据集）用来节约资源。

你可以用as.environment()取到搜索列表中的任何环境

```
search()
#> [1] ".GlobalEnv"        "package:stats"     "package:graphics" 
#> [4] "package:grDevices" "package:utils"     "package:datasets" 
#> [7] "package:methods"   "Autoloads"         "package:base"     

as.environment("package:stats")
#> <environment: package:stats>
```
在搜索列表中的globalenv()和baseenv()环境，以及emptyenv() 通过下图相连接。每次你通过library()加载一个新包
的时候，将会被插到全局环境和之前在搜索路径顶部的包之间。

手动创建一个环境，需要使用new.env()。你可以用ls()列出环境结构中所有绑定，用parent.env()查看它的父亲。

```
e <- new.env()
# the default parent provided by new.env() is environment from 
# which it is called - in this case that's the global environment.
parent.env(e)
#> <environment: R_GlobalEnv>
ls(e)
#> character(0)
```
修改一个环境中绑定最简单的方式就是把它当做一个列表:

```
e$a <- 1
e$b <- 2
ls(e)
#> [1] "a" "b"
e$a
#> [1] 1
```
默认的，ls()只会显示不是以.开头的名字。使用all.names = TRUE将会把一个环境中所有绑定的名字显示出来。

```
e$.a <- 2
ls(e)
#> [1] "a" "b"
ls(e, all.names = TRUE)
#> [1] "a"  ".a" "b"
```
另一个有用的方式去查看环境是用ls.str()。它比str()更加有用是因为它把环境中的每个变量都展示出来了。像
ls()一样，它也有all.name的参数。

```
str(e)
#> <environment: 0x273f248>
ls.str(e)
#> a :  num 1
#> b :  num 2
```
给定一个名称，你可以用$, [[, or get()提取出它所绑定的值:
- $ and [[只会查找一个环境变量，如果没有名字绑定的值，将会返回NULL。
- get() 使用常规的作用域规则，如果没有绑定被发现，将抛出一个错误。

```
e$c <- 3
e$c
#> [1] 3
e[["c"]]
#> [1] 3
get("c", envir = e)
#> [1] 3
```
从环境中删除一个变量和列表工作起来有点不一样。在列表中，你可以通过设置其为NULL,将一个元素删除。在环境中，
将创建一个新的绑定到NULL,然而，可以用rm()删除一个绑定。

```
e <- new.env()

e$a <- 1
e$a <- NULL
ls(e)
#> [1] "a"

rm("a", envir = e)
ls(e)
#> character(0)
```
你可以用exists()判断一个环境中是否存在某个绑定。像get()一样，它默认的行为遵循词法
作用域规则，并且查看它的父亲环境。如果你不想要这种行为，你可以使用inherits = FALSE:

```
x <- 10
exists("x", envir = e)
#> [1] TRUE
exists("x", envir = e, inherits = FALSE)
#> [1] FALSE
```
比较两个环境，你必须使用identical()而不是==:

```
identical(globalenv(), environment())
#> [1] TRUE
globalenv() == environment()
#> Error in globalenv() == environment(): comparison (1) is possible only for atomic and list types
```
####　环境中的递归:
环境形成了一棵树，因此是比较方便写一个递归函数的。本章节将向你展示如果运用你对环境的新认识去理解有用的pryr::where()。给定一个名字，where()找到名字是在哪个环境中被定义的，使用R正常的作用域规则:

```
library(pryr)
x <- 5
where("x")
#> <environment: R_GlobalEnv>
where("mean")
#> <environment: base>
```
where()的定义是直接的。它有两个函数:查找的名字（一个字符串），一个需要寻找的环境。（我们将会在calling environments学到为什么parent.frame()是一个好的默认选择。

```
where <- function(name, env = parent.frame()) {
  if (identical(env, emptyenv())) {
    # Base case
    stop("Can't find ", name, call. = FALSE)
    
  } else if (exists(name, envir = env, inherits = FALSE)) {
    # Success case
    env
    
  } else {
    # Recursive case
    where(name, parent.env(env))
    
  }
}
```
有三种情况:
- 基本情况: 我们已经到达了一个空的环境，并且没有找到绑定。因此我们不能向前寻找了，抛出一个错误。
- 成功的情况: 名字存在在环境中，因此我们返回这个环境。
- 递归的情况: 名字没有在环境中找到，因此尝试在父亲环境中寻找。

在环境中使用递归是很自然的，因此where()提供一个很有用的模板。移除where()中很具体的内容，把结构显示得更加清楚:

```
f <- function(..., env = parent.frame()) {
  if (identical(env, emptyenv())) {
    # base case
  } else if (success) {
    # success case
  } else {
    # recursive case
    f(..., env = parent.env(env))
  }
}
```
##### 迭代和递归
使用一个迭代而不是递归是可能的。这个可能跑稍微快一点（因为我们移除了一些函数的调用）,但是我认为这是难以理解的。我把这个写进来，是因为如果你对递归函数不是很熟悉，你可能会比较容易知道发生了什么。

```
is_empty <- function(x) identical(x, emptyenv())

f2 <- function(..., env = parent.frame()) {
  while(!is_empty(env)) {
    if (success) {
      # success case
      return()
    }
    # inspect parent
    env <- parent.env(env)
  }

  # base case
}
```
#### 函数环境
大多数环境不是你用new.env()创建的，是使用函数创建的结果。本节讨论和一个函数相联系的四种环境:enclosing, binding, execution, and calling.

**enclosing**是函数被创建时候的环境。每个环境都有一个且唯一的enclosing环境。对于其他三种类型的环境，可能是０或者１，或者更加与每个函数相联系的环境。
- 通过<-绑定一个函数到名字定义了一个**binding**函数。
- 调用一个函数创建了一个短暂的执行环境，保存了在执行期间产生的变量。
- 每一个执行环境都能一个调用环境想联系，告诉你函数是在哪里被调用的。

下面的部分将解释为什么这些环境中每一个都是重要的，如何得到它们，以及如何使用它们。

##### The enclosing environment
当一个函数被创建的时候，它获得了一个创建它环境时的一个引用。这就是enclosing environment，并用作词法作用域。你可以得到一个函数的enclosing environment，通过调用environment()，其中一个函数作为它的第一个参数。

```
y <- 1
f <- function(x) x + y
environment(f)
#> <environment: R_GlobalEnv>
```
在下面的图形中，我将把函数描述成一个圆角矩阵。一个函数的enclosing environment，由一个小的黑色圆圈所给定。

##### Binding environments
上面那个图比较简单，因为函数没有名字。然而函数的名字是由绑定定义的。函数的绑定环境是所有有绑定的环境。下面的图较好地反应了这个关系，因为enclosing environment包含一个绑定从f到函数。

在这个例子里，封闭环境和绑定环境是一样的。如果你将一个函数赋值给一个不同的环境会不同。

```
e <- new.env()
e$g <- function() 1
```
封闭环境属于函数，而且从不改变，即使环境被移到另外一个环境。封闭环境决定了函数如何寻找它的值，绑定环境决定了我们如何寻找这个函数。

绑定函数和封闭环境之间的不同对于包的命名空间是重要的。包的命名空间保持包独立。举个例子，如果包Ａ使用mean()这个基础函数，如果包Ｂ创建了一个自己的mean()
函数将会发生什么？命名空间确保包Ａ继续使用基础base()函数，不会被包Ｂ影响。

命名空间是使用环境实现的，利用函数不需要存活在它们的封闭环境的优势。举个例子，例如基础函数sd(),它的绑定和封闭环境是不同的:

```
environment(sd)
#> <environment: namespace:stats>
where("sd")
#> <environment: package:stats>
```
sd()的定义使用var(),但是如果我们使用我们自己版本的var()，这将不会影响sd()。

```
x <- 1:10
sd(x)
#> [1] 3.02765
var <- function(x, na.rm = TRUE) 100
sd(x)
#> [1] 3.02765
```
这种方式可行，是因为每个包都有两个与他相关的函数:包环境和命名空间环境。包环境包含每个公开可以访问到的函数，并且被放置在搜索路径上面。命名空间环境包含所有的函数（包括内部的函数）,它的父亲环境是一个特殊的导入环境，包含包所需要所有函数的绑定。包中每一个对外暴露的函数被绑定到包环境，但是被命名空间环境所封闭。这个复杂的关系是由下面的图例所揭示的:
[详见这里](http://adv-r.had.co.nz/Environments.html)

当我们在控制台里面输入var，将在全局环境中被找到。当sd()寻找var(),将会在命名空间环境中第一个被找到，不会在globalenv()中寻找。

####　执行环境
当第一次跑下面的函数的时候将返回什么?第二次呢?

```
g <- function(x) {
  if (!exists("a", inherits = FALSE)) {
    message("Defining a")
    a <- 1
  } else {
    a <- a + 1
  }
  a
}
g(10)
g(10)
```
每次这个函数返回同一个值，因为新的开始原则，在the fresh start中描述了。每次一个函数被调用了，一个新的环境就被创建到主机执行中。执行环境的父亲是函数的封闭环境。一旦一个函数完成了，这个环境就会抛出。

让我们用一个简单的函数图形化描述一下。我用虚线在函数周围画了一个框，表示执行环境。

```
h <- function(x) {
  a <- 2
  x + a
}
y <- h(1)
```
当你在函数内部创建了一个函数，子函数的封闭环境就是父亲环境的执行环境，同时执行环境不再是短暂的。下面的例子揭示了一个函数工厂的想法，plus()。我们使用这个工厂创建了一个函数叫做plus_one()。plus_one()的封闭环境是plus()的执行环境，其中x绑定给值１。

```
plus <- function(x) {
  function(y) x + y
}
plus_one <- plus(1)
identical(parent.env(environment(plus_one)), environment(plus))
#> [1] TRUE
```
你可以在functional programming中学到更多关于函数工厂的知识。

#### 调用函数
看下面的代码，i()将返回什么?

```
h <- function() {
  x <- 10
  function() {
    x
  }
}
i <- h()
x <- 20
i()
```
在底层的ｘ是一个烫手的山芋:使用正常的作用域规则,h()会首先在它所定义的地方寻找，值为10。然而，询问在i()所调用的环境中ｘ的值是多少仍然是有意义的。在 h()所定义的环境中x的值10，而在h()所调用的环境中x的值是20。

我们可以使用parent.frame()得到这个环境。这个函数返回函数被调用时候的环境。我们也可以在这个环境中使用这个函数去查找一个变量名字的值。

```
f2 <- function() {
  x <- 10
  function() {
    def <- get("x", environment())
    cll <- get("x", parent.frame())
    list(defined = def, called = cll)
  }
}
g2 <- f2()
x <- 20
str(g2())
#> List of 2
#>  $ defined: num 10
#>  $ called : num 20
```
在更复杂的情景中，不只有一个父调用而是一系列调用最终都指向最初的函数，在最上层调用的。下面的代码产生了一个三层深的栈调用。开放式的箭头代表了每一个执行环境的调用环境。

```
x <- 0
y <- 10
f <- function() {
  x <- 1
  g()
}
g <- function() {
  x <- 2
  h()
}
h <- function() {
  x <- 3
  x + y
}
f()
#> [1] 13
```
注意每一个执行环境都有两个两个父亲:一个是调用环境，另一个是封闭环境。Ｒ正常的作用域规则仅仅使用封闭父亲；parent.frame()允许你拿到调用父亲。

在调用环境中查找变量而不是封闭环境，叫做动态作用域。很少有语言实现了冬天作用域。（Emacs Lisp是一个例外）。这是因为动态作用域使得一个函数是如何运作的变得很难解释。你不仅需要知道它是如何定义的，也是知道在被调用时候的上下文。动态作用域在开发用于援助交互式分析的函数是有用的。这是在non-standard evaluation主要讨论的话题之一。

#### 绑定一个名称到值
在环境中，赋值一个绑定一个名称到值的行为。它是作用域的配对物，决定了一个值是如何和名称联系起来的一系列规则。和大多数语言相比较，Ｒ中对于绑定名称到值有及其灵活的工具。实际上，你不仅可以绑定值到名称，你还可以绑定表达式，甚至是函数，因此每个你取到与名称相联系的值的时候，可能都会有一些不同。

在Ｒ中你可能已经使用过了上千次的正常的赋值。正常的赋值创建了一个名称和一个当前环境对象的绑定。变量名称经常是由字母，数字，.和下划线组成的，不能以_开头。如果你尝试使用一个不遵守这些规则的名字，你会得到一个错误:

```
_abc <- 1
# Error: unexpected input in "_"
```
保留字（像TRUE, NULL, if, and function）遵循这些规则，但是已经被保留用作其他目的:

```
if <- 10
#> Error: unexpected assignment in "if <-"
```
保留子的完整列表可以通过?Reserved.查看。

你可以通过倒引号重写这些常用的规则，并且使用任意序列的字符作为名称:

```
`a + b` <- 3
`:)` <- "smile"
`    ` <- "spaces"
ls()
#  [1] "    "   ":)"     "a + b"
`:)`
#  [1] "smile"
```
#### 引号
你还可以用单个或者双引号去创建非语法结构的绑定，但这是不推荐的。在赋值箭头的左边使用字符是Ｒ在支持倒引号之前的历史的产物。

常规的赋值箭头，<-，总是在当前环境中创建一个变量。深赋值箭头，<<-，不会在当前环境创建一个变量，而是在父亲搜寻从而替换一个已经存在的变量。你仍可以用assign()做深度的绑定:name <<- value等同于assign("name", value, inherits = TRUE)。

```
x <- 0
f <- function() {
  x <<- 1
}
f()
x
#> [1] 1
```
如果<<-没有发现一个已经存在的便量，它会在全局变量中创建换一个新的。这通常是不可取的，因为全局变量将在函数之间产生不明显的依赖关系。<<-通常会和闭包在一起工作，在Closures将会描述。

还有另外两种特殊的绑定，延迟的和活跃的迟的绑定将创建和保存一个约定，在需要的时候计算这个表达式。我们可以通过一个特殊的赋值算子"%<d-%"去创建一个延迟绑定，由pryr包所提供。

 ```
library(pryr)
system.time(b %<d-% {Sys.sleep(1); 1})
#>    user  system elapsed 
#>       0       0       0
system.time(b)
#>    user  system elapsed 
#>   0.000   0.000   1.001
 ```
 该算子是对基础delayedAssign()的一个封装，如果你需要更多的控制，你可以直接使用它。延迟的绑定被用于实现autoload()，将使得Ｒ表现的好像包数据是在内存中，即使只有当你请求数据时，才会从硬盘中加载。
- 活跃不是针对不变的对象进行绑定的，每次取值时都会重新计算:

 ```
x %<a-% runif(1)
x
#> [1] 0.1100141
x
#> [1] 0.1063044
rm(x)
 ```
 该算子是对makeActiveBinding()的封装。如果你想要更多的控制，你可以直接使用这个函数。活跃的绑定可以实现引用类的字段。
 
#### 明确的环境
不仅仅是强大的作用域，环境本身也是一个强大的数据结构，因为它们有引用的语义。不像Ｒ中大多数对象，当你修改一个环境的时候，它不是做了一个复制。举个例子，看这个modify()函数。

```
modify <- function(x) {
  x$a <- 2
  invisible()
}
```
如果你对一个列表这样操作,原本的列表将不会改变因为修改一个列表实际上是创建并修改一个复制。

```
x_l <- list()
x_l$a <- 1
modify(x_l)
x_l$a
#> [1] 1
```
然而，对一个环境这么操作，原来的环境就修改了:

```
x_e <- new.env()
x_e$a <- 1
modify(x_e)
x_e$a
#> [1] 2
```
如同你可以使用一个列表在函数之间传递数值，你还可以使用一个环境。当创建一个你自己的环境的时候，注意你应该将父亲环境设置成一个空的环境。这个确保你不会偶然地从其他地方继承了对象:

```
x <- 1
e1 <- new.env()
get("x", envir = e1)
#> [1] 1

e2 <- new.env(parent = emptyenv())
get("x", envir = e2)
#> Error in get("x", envir = e2): object 'x' not found
```
环境作为数据结构，解决单个通常的问题是有用的:
- 避免大数据的复制
- 在一个包里面管理状态
- 从名称里面搞笑地查找值

#### 避免复制
因为环境有引用语义，你将不会意外的创建一个复制。这对于大对象是一个有用的容器。在bioconductor包中是通用的技术，需要经常处理大的染色体对象。在R 3.1.0中已经是这个使用持续地显得不是那么重要，因为修改一个列表也不会造成深入的复制。之前，修改一个列表中单独的元素，都会造成每个元素被复制，如果一些元素很大将是很昂贵的操作。现在，修改一个列表将高效地复用这些已经存在的向量，节约很多时间。

#### 包的状态
在包中明确的环境是有用的，因为它们可以使你在函数调用中间维持状态。通常，包中的对象是锁住的，因此你不能直接修改它们。然而你可以这样做:

```
my_env <- new.env(parent = emptyenv())
my_env$a <- 1

get_a <- function() {
  my_env$a
}
set_a <- function(value) {
  old <- my_env$a
  my_env$a <- value
  invisible(old)
}
```
从setter函数中返回一个老的值是好的模型，因为使得用on.exit()函数重新设置之前的值变的容易。

#### 作为一个hashmap
一个hashmap是一个数据结构花费常数时间，O(1)去寻找一个对象，基于它的名称。环境可以默认提供这种行为，因此可以被用作模拟hashmap。看hash包完成实现了这种想法。











































