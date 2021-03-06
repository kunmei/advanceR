### 函数
函数是R中基础结构单元:为了掌握本书中许多更加高级的技术，你需要对函数是如何工作有很好的基础。你可能已经创建了很多R的函数，并且熟悉它们是如何工作的基础。本章节的目的就是将你对函数已经存在的，不正式的理解转变成对函数是什么以及它们是如何工作的严格的理解。在本章节，你将看到一些有趣的技巧和技术，但是你想要学习的将会更加重要，因为它们是更加高级技术的基础。

理解R最重要的事情就是R中的函数本身就是一个对象。你可以同操作任何其他类型的对象一样去操作函数。这个主题将在functional programming函数式编程中详细的介绍。

##### 纲要
- 函数组成 描述一个函数三个主要的组成部分
- 词法域 教你R中如何通过名字找到值，词法域的过程
- 每个操作是一个函数调用 说明R中发生的每一件事都是一个函数调用的结果，即使看起来不像
- 函数参数 讨论了三种方式给函数提供参数，如何给一个参数列表取调用函数，以及延迟计算的影响
- 特殊的调用 描述了函数的两种特殊类型:中缀和替换函数
- 返回值 讨论了函数如何以及何时返回值，以及在函数退出时一个函数是如何运行的

##### 预备知识
仅仅需要pryr包，用来探索当修改向量的时候将发生什么。

#### 函数组成部分
所有R函数有三个部分:
- 函数体body()， 函数内部的代码
- 函数形参formals()， 参数列表，控制你将如何调用函数
- 环境environment()， 函数变量位置

当你在R中打印函数的时候，会显示这三个重要的成分。如果环境没有显示，意味着函数是在全局环境中创建的。

```
f <- function(x) x^2
f
#> function(x) x^2

formals(f)
#> $x
body(f)
#> x^2
environment(f)
#> <environment: R_GlobalEnv>
```
body(), formals(), and environment()的赋值形式也可以用来修改函数。

像R中所有的对象一样，函数也可以持有任意数目的额外的属性。在base R中使用的一个属性是"srcref"，source reference的简称，指向创建函数的源代码。不像body()，它包括代码注释和其他格式。你还可以为函数增加属性。举个例子，你可以设置class()和增加一个常用的print()方法。

##### 原语函数
函数有三个组成部分的规则存在一个例外。原语函数，例如sum()，直接用.Primitive() 调用C代码，而且不包含R代码。因此它们的formals(), body(), and environment()都是NULL。

```
sum
#> function (..., na.rm = FALSE)  .Primitive("sum")
formals(sum)
#> NULL
body(sum)
#> NULL
environment(sum)
#> NULL
```
原语函数只会在base包里面发现。因为它们在底层操作，它们是非常高效的（原语替代函数不需要做复制），并且针对参数匹配有不同的规则（比如switch和call）。然而，你会造成和R中所有其他函数的行为不同。因此R core团队一般会避免取创建它们，除非必须要使用它们。

#### 词法作用域
Scoping作用域是R中管理如何查看一个符号所对应值的规则集合。在下面的例子中显示了R从符号x到它的值10的作用域的规则。

```
x <- 10
x
#> [1] 10
```
理解作用域可以让你:
- 通过组合函数构建工具，将会在函数编程中介绍
- 否决通常的求值计算规则，做非标准的求值，将会在 non-standard evaluation介绍

R中有两种类型的作用域:词法作用域，在语言层面自动实现，另一个是动态作用域，在动态交互的时候用于选择函数去保存类型。在这里，我们讨论词法作用域，因为它和函数创建密切相关。动态作用域将会在
scoping issues里面具体描述。

词法作用域查找符号值基于当函数被创建时，它们是如何封装的，而不是它们被调用时是如何封装的。通过词法域，你不需要知道函数是如何被调用的，从而找出变量值是哪里被查找的。你仅仅需要查找函数的定义。

在词法作用域中的lexical不是对应于通常的英语定义(“of or relating to words or the vocabulary of a language as distinguished from its grammar and construction”)，来自于计算机
科学中的词法分析，是将代码翻译成编程语言可以理解的文本部分的处理过程的一部分。

R中实现词法作用域主要有四种原则:
- 名字屏蔽
- 函数和变量
- 新的开始
- 动态查找

你可能已经知道许多这些原则，但是你可能还没有明确地想清楚。在查看答案之前，通过思考示例代码检验你的知识。

##### 名字屏蔽
下面的例子揭示了词法作用域中最主要的原则，预测输出结果应该没问题。

```
f <- function() {
  x <- 1
  y <- 2
  c(x, y)
}
f()
rm(f)
```
如果名字没有在函数的内部定义，R将会在上层查找:

```
x <- 2
g <- function() {
  y <- 1
  c(x, y)
}
g()
rm(x, g)
```
如果一个函数定义在另一个函数里面，同样的规则:在当前函数里面查找，然后是函数的定义处，以此类推，直到全局环境，再到其他已经加载的包。先看一下下面的代码，
然后在执行代码确认输出。

```
x <- 1
h <- function() {
  y <- 2
  i <- function() {
    z <- 3
    c(x, y, z)
  }
  i()
}
h()
rm(x, h)
```
规则同样适用于闭包，通过其他函数创建的函数。闭包将会在函数编程里面更加细节的描述。这里我们将仅仅看它们是如何同作用域相联系的。下面的函数，j()返回一个函数。
当我们调用这个函数的时候，你认为这个函数会返回什么呢？

```
j <- function(x) {
  y <- 2
  function() {
    c(x, y)
  }
}
k <- j(1)
k()
rm(j, k)
```
这里看起来有点奇妙。（当这个函数已经被调用了之后，R是如何知道y的值是多少呢）。这是因为k保存了它被定义时候的环境，而且这个环境包含了y的值。环境给你如何深入提供
了一些指示，并且可以找出每个函数保存在环境中的值。

##### 函数和变量
不管相关值的类型，适用同样的原则-寻找函数同寻找变量一模一样:

```
l <- function(x) x + 1
m <- function() {
  l <- function(x) x * 2
  l(10)
}
m()
#> [1] 20
rm(l, m)
```
对于函数，这个规则有一个小小的调整。如果你在明显想要一个函数的上下文中使用一个名字，（例如，f(3)），当搜寻的时候，R将忽视不是函数的对象。在下面的例子中，n会是不同的
值取决于R是在寻找一个函数还是一个变量。

```
n <- function(x) x / 2
o <- function() {
  n <- 10
  n(n)
}
o()
#> [1] 5
rm(n, o)
```
然而，对函数和其他对象使用相同的名字将会让代码产生混淆，最好避免。
##### 新的开始

```{r}
j <- function() {
  if (!exists("a")) {
    a <- 1
  } else {
    a <- a + 1
  }
  print(a)
}
j()
rm(j)
```
你可能会惊讶函数每次返回同样的值1.这是因为每个一个函数被调用，一个新的环境就会被创建。一个函数没法知道它上一次被调用的时候发生了什么。每一次调用都是完全独立的。
（可以在可变状态中寻找一些方式去避免它）

##### 动态查找
词法作用域决定了哪里去寻找值，而不是何时取寻找它们。R寻找值是在函数执行的时候，而不是它被创建的时候。这意味着函数的输出可能是不同的，取决于它环境之外的对象:

```
f <- function() x
x <- 15
f()
#> [1] 15

x <- 20
f()
#> [1] 20
```
你通常想要避免这种行为，因为这意味这函数不再是自我控制的。这是一个常见错误，如果你在你的代码中有一个拼写错误，当你创建这个函数的时候不会有错误，甚至当你执行这个
函数的时候，也不会发生错误，取决于在全局环境中定义了什么变量。

一种方式去检测这个问题是用codetools里面的findGlobals()函数。这个函数列出了一个函数所有的外部依赖:

```
f <- function() x + 1
codetools::findGlobals(f)
#> [1] "+" "x"
```
另外一种方式去解决这个问题是人工将这个函数的环境改成emptyenv()，一个不包含任何东西的环境。

```
environment(f) <- emptyenv()
f()
#> Error in f(): could not find function "+"
```
这种方式不工作是因为Ｒ依赖词法作用域去寻找每个对象，即使是+操作符。使得一个函数完全自我控制的不可能的，因为你必须依赖定义在基础包或者其他包中定义的函数。

你还可以使用这个技巧去做一些及其不推荐的做法。举个例子，因为Ｒ中所有标准的操作都是函数，你可以根据自己的选择重写它们。如果你曾经有点邪恶，当你的朋友离开的
电脑的时候执行下面的代码。

```
`(` <- function(e1) {
  if (is.numeric(e1) && runif(1) < 0.1) {
    e1 + 1
  } else {
    e1
  }
}
replicate(50, (1 + 2))
#>  [1] 3 3 3 3 3 3 4 3 3 3 3 3 3 3 3 3 3 3 3 3 3 3 3 3 3 3 3 3 3 3 3 3 3 3 3
#> [36] 3 3 3 3 3 3 3 3 3 3 3 3 3 3 3
rm("(")
```
这会产生一个特别致命的bug，10的几率，１会加到括号里面的数值计算。这也是启动一个干净的Ｒ进程的理由。

#### 每个操作都是一个函数调用
了解Ｒ中的计算，两条法则是有用的:
- 存在的每个东西都是对象
- 发生的每件事都是一个函数调用

前面对(的重新定义可以工作，是因为Ｒ中的每一个操作都是函数调用，不管它看起来是不是像一个函数。这些函数包括中缀操作符像+，控制流操作符像for, if, 和while，
取子集操作符像[]和$，甚至是花括号｛。下面例子中每一对陈述都是等同的。注意倒引号的使用，可以指函数或者已经保存或者非法名字的变量。

```
x <- 10; y <- 5
x + y
#> [1] 15
`+`(x, y)
#> [1] 15

for (i in 1:2) print(i)
#> [1] 1
#> [1] 2
`for`(i, 1:2, print(i))
#> [1] 1
#> [1] 2

if (i == 1) print("yes!") else print("no.")
#> [1] "no."
`if`(i == 1, print("yes!"), print("no."))
#> [1] "no."

x[3]
#> [1] NA
`[`(x, 3)
#> [1] NA

{ print(1); print(2); print(3) }
#> [1] 1
#> [1] 2
#> [1] 3
`{`(print(1), print(2), print(3))
#> [1] 1
#> [1] 2
#> [1] 3
```
重写这些特殊函数的定义是可能的，但是这是不好的注意。然而，有些场合这个可能是有用的:它允许你去做一些是，否则是无法完成的。
举个例子，这个特点可以使得dplyr包将Ｒ的表达式翻译成sql的表达式。Domain specific languages使用这种思想创建领域专用
语言使得你可以使用已经存在的Ｒ的构造器精确的表达新的内容。

把特殊的函数当成普通的函数通常是有用的。举个例子，我们可以使用sapply()，通过定义一个函数add()，将３加到列表的每个元素
之上。

```
add <- function(x, y) x + y
sapply(1:10, add, 3)
#>  [1]  4  5  6  7  8  9 10 11 12 13
```
我们可以使用内建的+函数得到同样的效果。

```
sapply(1:5, `+`, 3)
#> [1] 4 5 6 7 8
sapply(1:5, "+", 3)
#> [1] 4 5 6 7 8
```
注意`+` and "+"和的不同。第一个是被称为+的对象的值，第二个是包含字符+的字符串。第二个版本工作是因为sapply可以提供函数的名字
而不是函数本身。如果你读sapply()的源码，你可以看到第一行使用的是match.fun()去查找给定名字的函数。

一个更有用的应用是用subsetting结合lapply()或者sapply()。

```
x <- list(1:3, 4:9, 10:12)
sapply(x, "[", 2)
#> [1]  2  5 11

# equivalent to
sapply(x, function(x) x[2])
#> [1]  2  5 11
```
记住Ｒ中发生的每件事都是一个函数调用，在metaprogramming将会帮助你。

#### 函数参数
将一个函数的形参和实际的参数区分是有用的。形参是函数的一部分，然而每次你调用函数实际或者调用参数应该是不一样的。本章节讨论
调用函数是如何匹配到形参的，给定参数的列表是如何调用的，默认的参数是如何工作的和延迟计算的影响。

##### 调用函数
当调用函数的时候，你可以通过位置，通过完整的名字，或者通过部分名字来确定参数。参数首先通过确定的名字被匹配，然后是前缀匹配，
最后是位置匹配。

```
f <- function(abcdef, bcde1, bcde2) {
  list(a = abcdef, b1 = bcde1, b2 = bcde2)
}
str(f(1, 2, 3))
#> List of 3
#>  $ a : num 1
#>  $ b1: num 2
#>  $ b2: num 3
str(f(2, 3, abcdef = 1))
#> List of 3
#>  $ a : num 1
#>  $ b1: num 2
#>  $ b2: num 3

# Can abbreviate long argument names:
str(f(2, 3, a = 1))
#> List of 3
#>  $ a : num 1
#>  $ b1: num 2
#>  $ b2: num 3

# But this doesn't work because abbreviation is ambiguous
str(f(1, 3, b = 1))
#> Error in f(1, 3, b = 1): argument 3 matches multiple formal arguments
```
通常情况下，你仅仅是需要使用位置匹配第一个或者第二个参数，它们是最常用的，大多数读者也是可以知道它们是什么。对于很少使用的参数，避免使用
位置匹配，仅仅使用可读的缩写用于部分匹配。（如果你开发一个包，想要发布到CRAN上面，是不能使用部分匹配的，要使用完整的名字）。有名字的参数
应该总是出现在没有名字的参数之后。如果一个函数使用...，你仅能在...之后用完整的名字确定参数。

好的方式:
```
mean(1:10)
mean(1:10, trim = 0.05)
```
可能是过度的:

```
mean(x = 1:10)
```
下面是迷惑的:

```
mean(1:10, n = T)
mean(1:10, , FALSE)
mean(1:10, 0.05)
mean(, TRUE, x = c(1:10, NA))
```
##### 给定参数列表去调用函数
假设你有一个函数的列表:

```
args <- list(1:10, na.rm = TRUE)
```
如何才能将一个参数列表传给mean()。你需要do.call():

```
do.call(mean, list(1:10, na.rm = TRUE))
#> [1] 5.5
# Equivalent to
mean(1:10, na.rm = TRUE)
#> [1] 5.5
```
##### 默认参数和缺失参数
Ｒ中的函数参数可以有默认参数:

```
f <- function(a = 1, b = 2) {
  c(a, b)
}
f()
#> [1] 1 2
```
因为Ｒ中的参数是延迟计算的，默认值可以被定义在其他参数的基础上。

```
g <- function(a = 1, b = a * 2) {
  c(a, b)
}
g()
#> [1] 1 2
g(10)
#> [1] 10 20
```
默认参数甚至可以定义在函数内部创建的变量之上。在Ｒ基础函数中经常使用，但是我认为这是不好的尝试，因为如果你不完整的阅读源代码，
你不能知道参数的默认值是什么。

```
h <- function(a = 1, b = d) {
  d <- (a + 1) ^ 2
  c(a, b)
}
```
你可以使用missing()去判断一个函数是提供的了还是没有。

```
i <- function(a, b) {
  c(missing(a), missing(b))
}
i()
#> [1] TRUE TRUE
i(a = 1)
#> [1] FALSE  TRUE
i(b = 2)
#> [1]  TRUE FALSE
i(1, 2)
#> [1] FALSE FALSE
```
有时你需要增加一个非平凡的默认值，可能需要花费几行代码去计算。不用将代码插入到函数定义中，如果需要，可以用missing()去有条件的计算。
然而，如果没有仔细的阅读文档，这将很难知道哪些参数是要求的，哪些是可以选择的。因此，我经常把默认值设置成NULL,然后使用is.null()去
检查参数是否是提供的。

##### 延迟计算
默认的，R的函数参数是延迟计算的，只有当它们真正被使用的时候，才会计算。

```
f <- function(x) {
  10
}
f(stop("This is an error!"))
#> [1] 10
```
如果你想要一个参数一定被计算，需要使用force():

```
f <- function(x) {
  force(x)
  10
}
f(stop("This is an error!"))
#> Error in force(x): This is an error!
```
当用lapply()或者一个循环创建一个闭包的时候是重要的:

```
add <- function(x) {
  function(y) x + y
}
adders <- lapply(1:10, add)
adders[[1]](10)
#> [1] 11
adders[[10]](10)
#> [1] 20
```
当你第一次调用其中一个加法函数的时候，ｘ是延迟计算的。这时，循环结束，ｘ最终的值是10。因此所有的加法函数都会在输入上面加10，
这可能跟你想的不一样。人工强制计算可以修复这个问题:

```
add <- function(x) {
  force(x)
  function(y) x + y
}
adders2 <- lapply(1:10, add)
adders2[[1]](10)
#> [1] 11
adders2[[10]](10)
#> [1] 20
```
这段代码同下面的等同:

```
add <- function(x) {
  x
  function(y) x + y
}
```
因为强制函数被定义成force <- function(x) x。然而使用这个函数意味着你在进行强制计算，而不是意外输入了x。

默认参数是在函数内部被计算的。这意味这如果这个表达式依赖于当前的环境，结果将会不一样取决于你是否使用了
默认值或者是精确的提供了一个。

```
f <- function(x = ls()) {
  a <- 1
  x
}

# ls() evaluated inside f:
f()
#> [1] "a" "x"

# ls() evaluated in global environment:
f(ls())
#>  [1] "add"     "adders"  "adders2" "args"    "f"       "funs"    "g"      
#>  [8] "h"       "i"       "objs"    "path"    "x"       "y"
```
更技术的讲是，一个没有计算的参数被称为**promise**，或者是一个形实转换程序。一个约定是由两个部分组成的:
- 一个产生延迟计算的表达式。（可以通过substitute()取掉，non-standard evaluation可以看到更多细节）
- 表达式被创建时的环境和它将在哪里被计算。

表达式在它被创建的环境下被计算的时候，约定第一个被取到。这个值是被缓存的，因此后面取到被计算的约定将不会
重新计算（但是原来的表达式仍然和这个值有联系，因此substitute()仍然可以取到这个值。你可以使用pryr::promise_info()
找到关于一个约定更多的信息。这是使用一些C++的代码去对这个约定取信息的，而不需要计算它，使用纯Ｒ代码是做不到的。

在if表达式中延迟计算是有用的。下面第二个表达式只有当第一个为TRUE的时候才会被计算。如果不是，称述将返回一个错误，
因为NULL > 0是一个长度为0的逻辑向量，不是if的固定输入。

```
x <- NULL
if (!is.null(x) && x > 0) {

}
```
我们可以我们自己的"&&"操作符:

```
`&&` <- function(x, y) {
  if (!x) return(FALSE)
  if (!y) return(FALSE)

  TRUE
}
a <- NULL
!is.null(a) && a > 0
#> [1] FALSE
```
这个函数没有延迟计算是不会工作的，因为x和ｙ总是被计算的，甚至当a是NULL值的时候，也会测试a>0。

有时候你可以使用延迟计算去除if逻辑判断，举个例子,不用使用:

```
if (is.null(a)) stop("a is null")
#> Error in eval(expr, envir, enclos): a is null
```
你可以写:

```
!is.null(a) || stop("a is null")
#> Error in eval(expr, envir, enclos): a is null
```
##### ...
有一个特殊的参数叫做...。这个参数可以匹配任何灭有匹配到的参数，也可以很容易地传递到其他参数。如果你想收集参数去调用
另一个函数这是有用的，但是你不想提前阐述它们的可能的名字。...经常使用在S3的泛型，从而允许单个方法更加的灵活。

...一个复杂的使用者是基础的plot()函数。plot()是一个带有x,y和...的泛型参数。为了了解对于一个给定的函数...的意思，我们需要
阅读帮助文档:"被传递给方法的参数，比如图形参数"。plot()很多简单的调用最终是调用plot.default()，其中带有很多参数，
也有...。阅读文档揭示了...接受其他图像参数，列在帮助中用于par()。这允许我们去这样书写代码:

```
plot(1:5, col = "red")
plot(1:5, cex = 5, pch = 20)
```
去描述...,以一种容易的方式去工作，你可以使用list(...).（在capturing unevaluated dots中可以看到一些其他方式去描述...而不需要
计算这些参数）

```
f <- function(...) {
  names(list(...))
}
f(a = 1, b = 2)
#> [1] "a" "b"
```
使用...会有一定的代价，任何错误拼写的参数都不会抛出一个错误，在...之后的任意参数都必须提供完整的名字。这将使得拼写错误很同意被
忽视:

```
sum(1, 2, NA, na.mr = TRUE)
#> [1] NA
```
明确指定通常比隐式更加好一点，因此你可能不需要叫使用者去提供一系列额外的参数。如果你尝试使用...匹配很多额外的参数会是容易的。

##### 特殊的调用
Ｒ支持两种额外的语法调用特殊的函数类型:中缀和替换函数。

###### 中缀函数
Ｒ中的大多数函数都是前缀操作符:函数的名字出现在参数的前面。你还可以创建中缀函数，函数的名字出现在参数的中间，例如+号和-号。
所有用户创建的中缀函数都必须以%开始和结束。Ｒ中有下面已经定义好的中缀函数:%%, %*%, %/%, %in%, %o%, %x%。（内建的不需要
%的中缀运算符的完整列表是：::, :::, $, @, ^, *, /, +, -, >, >=, <, <=, ==, !=, !, &, &&, |, ||, ~, <-, <<-)

举个例子，我们可以创建一个将所有字符串连接起来的新的运算符:

```
`%+%` <- function(a, b) paste0(a, b)
"new" %+% " string"
#> [1] "new string"
```
注意当你创建这个函数的时候，你必须将名字放在倒引号里面，因为它是特殊的名字。这仅仅是一个给其他普通函数调用的语法糖;
对于Ｒ而言，这两种表达式是没有区别的。

```
"new" %+% " string"
#> [1] "new string"
`%+%`("new", " string")
#> [1] "new string"
```
或者的确在中间:

```
1 + 5
#> [1] 6
`+`(1, 5)
#> [1] 6
```
中缀函数的名字相对于常规的Ｒ函数更加灵活:它们可以包含任意的字符序列，除了“%”。在定义函数的时候，你需要转义任何特殊的字符，而在函数
调用的时候不需要。

```
`% %` <- function(a, b) paste(a, b)
`%'%` <- function(a, b) paste(a, b)
`%/\\%` <- function(a, b) paste(a, b)

"a" % % "b"
#> [1] "a b"
"a" %'% "b"
#> [1] "a b"
"a" %/\% "b"
#> [1] "a b"
```
Ｒ的默认优先级规则是中缀操作符是从左到右。

```
`%-%` <- function(a, b) paste0("(", a, " %-% ", b, ")")
"a" %-% "b" %-% "c"
#> [1] "((a %-% b) %-% c)"
```
有一个中缀函数我经常使用。是由 Ruby’s || logical or operator启发而来，尽管它在Ｒ中工作有点不一样，因为Ruby有一个更加灵活的定义关于在if
语句什么计算出来是TRUE。当另一个函数的输出是NULL的时候，上述方式作为一种默认方式是有用的。

```
`%||%` <- function(a, b) if (!is.null(a)) a else b
function_that_might_return_null() %||% default value
```
##### 替换函数
替换函数表现地可以随时修改它们的参数，有一个特殊的名字xxx<-。它们通常有两个参数(x和value)，尽管它们可以有更多，但是它们必须返回被修改的对象。
举个例子，下面的函数允许你修改一个向量的第二个元素。

```
`second<-` <- function(x, value) {
  x[2] <- value
  x
}
x <- 1:10
second(x) <- 5L
x
#>  [1]  1  5  3  4  5  6  7  8  9 10
```
当Ｒ计算second(x) <- 5的赋值，注意到<-的左边不是一个简单的名字，因此它会寻找一个叫做second<-的函数去做替换。

我说它们表现得像它们可以随时修改它们的参数，因为它们的确创建了一个被修改的复制。我们使用pryr::address()查看底层对象的内存地址。

```
library(pryr)
x <- 1:10
address(x)
#> [1] "0x2c29230"
second(x) <- 6L
address(x)
#> [1] "0x397efe0"
```
用.Primitive()的实现的内置函数也能随时改变:

```
x <- 1:10
address(x)
#> [1] "0x103945110"

x[2] <- 7L
address(x)
#> [1] "0x103945110"
```
意识到这种行为是非常重要的，因为它有重要的性能影响。

如果你想提供额外的参数，它们需要在x和value中间:

```
`modify<-` <- function(x, position, value) {
  x[position] <- value
  x
}
modify(x, 1) <- 10
x
#>  [1] 10  6  3  4  5  6  7  8  9 10
```
当你调用modify(x, 1) <- 10的时候，在Ｒ中秘密地转成了:

```
x <- `modify<-`(x, 1, 10)
```
这意味这你不能写如下的代码:

```
modify(get("x"), 1) <- 10
```
因为它会转成非法的代码:

```
get("x") <- `modify<-`(get("x"), 1, 10)
```
将替换和取子集结合起来通常是有用的:

```
x <- c(a = 1, b = 2, c = 3)
names(x)
#> [1] "a" "b" "c"
names(x)[2] <- "two"
names(x)
#> [1] "a"   "two" "c"
```
这种方式可行是因为表达式names(x)[2] <- "two"被计算，如同你写了如下的代码:

```
`*tmp*` <- names(x)
`*tmp*`[2] <- "two"
names(x) <- `*tmp*`
```
的确，它创建了一个本地的名为*tmp*的变量，不过之后删除了。

####　返回值
函数中最后被计算的表达式是返回值，调用函数的结果。

```
f <- function(x) {
  if (x < 10) {
    0
  } else {
    10
  }
}
f(5)
#> [1] 0
f(15)
#> [1] 10
```
通常来说，当你提前退出的时候，保存精确的return()字句的使用是一种好的风格，例如一个错误，或者是函数的简单情况。
这种编程的风格也可以减少缩进，也可以使得函数更容易理解，因为你能在本地分析原因。

```
f <- function(x, y) {
  if (!x) return(y)

  # complicated processing here
}
```
函数仅能返回一个对象。但是这不是一个限制，因为你能返回一个列表包含任意数量的对象。

最同意理解和分析的函数是纯函数:将同样的输入映射到同样的输出的函数，并对工作空间没有影响。换句话说，纯函数没有副作用:
除了它们的返回值，它们不影响任何返回值之外的世界。

Ｒ对一种类型的副作用有副作用:大多数Ｒ对象有copy-on-modify语义。因此修改一个函数参数不会改变原来的值:

```
f <- function(x) {
  x$a <- 2
  x
}
x <- list(a = 1)
f(x)
#> $a
#> [1] 2
x$a
#> [1] 1
```
(copy-on-modify规则有两个重要的例外:环境和引用类。它们可以随时被修改，因此操作它们需要格外的小心)

这个和Java等可以修改函数输入的语言是有显著的不同的。这种copy-on-modify行为有重要的性能后果，将会在profiling中深入讨论。
（注意Ｒcopy-on-modify语义的实现会造成一些性能后果。但不是普遍会这样。Clojure 是一种新的语言，其中使用了很多copy-on-modify
语义，但是有很少的性能后果。

大多数Ｒ基础函数是纯的，也有一些明显的意外:
- library(),加载一个包，因此修改了搜索路径。
- setwd(), Sys.setenv(), Sys.setlocale(),分别改变了工作目录，环境变量和语言环境。
- plot() 产生了图形输出
- write(), write.csv(), saveRDS()等将结果保存到硬盘上面
_ options() and par() 修改了全局设置。
- S4相关的函数会修改类和方法的全局表
- 随机数生生成器，每次执行，都会产生不同的数字。

通常最小化副作用的使用是好的注意，并且尽可能的，通过将纯函数和非纯函数分开去减少副作用的足迹。纯函数是容易测试的，因为所有你需要
关系的就是输入值和输出值，并且在不同的平台上或者不同版本的Ｒ上面工作很少会出现不同。举个例子,这个ggplot2激励原则的一个:大多数
对对象的操作代表了一个画图，然而仅仅是最后的print or plot调用才用真正画图而产生的副作用。

函数可以返回invisible的值，当你调用这个函数的时候，默认将不会打印出来。

```
f1 <- function() 1
f2 <- function() invisible(1)

f1()
#> [1] 1
f2()
f1() == 1
#> [1] TRUE
f2() == 1
#> [1] TRUE
```
你可以通过将它包在括号里强制看不见的值返回。

```
(f2())
#> [1] 1
```
返回看不见值最常用的函数是<-:

```
a <- 2
(a <- 2)
#> [1] 2
```
这将使得将一个值赋值给多个变量成为可能:

```
a <- b <- c <- d <- 2
```
因为它会被解析成这样:

```
(a <- (b <- (c <- (d <- 2))))
#> [1] 2
```
##### 退出
除了返回一个值，函数通过on.exit()退出的时候，可以设置其他触发器。当函数退出，需要确保对全局状态的改变被保存下来的时候，这通常是一种有用的方式。
不管函数是如何退出的，（一个明确的return, 一个错误，或者是简单地执行到了函数的底部）on.exit()里面的代码都会被执行。

```
in_dir <- function(dir, code) {
  old <- setwd(dir)
  on.exit(setwd(old))

  force(code)
}
getwd()
#> [1] "/home/travis/build/hadley/adv-r"
in_dir("~", getwd())
#> [1] "/home/travis"
```
基础的模型是简单的:
- 首先我们将目录设置到一个新的位置，然后通过setwd()的输出获取当前的位置。
- 然后我们使用on.exit()确保工作目录返回到之前的值，不管函数是如何退出的。
- 最后，我们明确强制代码的计算。（我们实际上是不需要force()的，但是可以方便读者知道我们在做什么）。

小心:如果你在函数内部使用很多on.exit()调用，确保设置add = TRUE。不幸的是，on.exit()默认是add = FALSE。因此你每次执行它时，它将重写已经存在的退出表达式。
因为on.exit()实现的方式，我们不可能创建一个变种，带有add = TRUE，因此当使用它时，你必须要小心。














































































