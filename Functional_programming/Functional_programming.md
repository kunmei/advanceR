### 函数编程
Ｒ其是核心是一种函数式编程语言。这意味这它提供了对于函数创建和操作的很多工具。特别的，Ｒ中函数是第一等公民。你可以对
函数做任何事，如同你对向量做的一样:你可以将它们赋值给变量，储存在列表中，作为参数传递给其他函数，在函数内部创建它们，
甚至作为函数的结果返回。

本章从一个motivating example开始，删除冗余和重复的代码，用来清理和总结数据。然后你会学到函数编程的三个基石:匿名函数，
闭包(functions written by functions),和函数列表。这些将在结论中整合到一起，向你展示，如何从最简单的原语开始，建立
一整套用于数值积分的工具。这是函数性编程反复出现的主题:从小的，容易理解的建造基石，整合到更加复杂的结构，然后运用它们。

函数编程在下面两个章节还会继续讨论:functionals探讨将函数作为参数，返回向量的函数;function operators探讨将函数作为输入
并返回它们的函数。

##### 纲要
- 动机　使用一个通用的问题来推动函数编程: 在严肃的分析之前清理和整理数据。
- 匿名函数　向你展示你可能不知道的函数的另一面:你可以使用使用，而不需要给它们一个名字。
- 闭包 介绍闭包，一个由另一个函数写的函数。一个闭包可以获取它自己的参数，和定义在它父亲中的变量。
- 函数列表　向你展示如何将函数放到一个列表中，并且解释为什么你需要关系它
- 数值积分　通过一个例子的学习总结本章，使用匿名函数，闭包和函数列表创建一个灵活的工具用于数值积分。

##### 准备
你需要熟悉词法作用域的基本规则，lexical scoping中描述的。确保你已经安装了pryr包。

#### 动机
想象一下你已经加载了一个数据文件，像下面的一样，你可以使用-99去替换缺失值。你想要用NAs替换所有的-99s。

```
# Generate a sample dataset
set.seed(1014)
df <- data.frame(replicate(6, sample(c(1:10, -99), 6, rep = TRUE)))
names(df) <- letters[1:6]
df
#>    a  b c   d   e f
#> 1  1  6 1   5 -99 1
#> 2 10  4 4 -99   9 3
#> 3  7  9 5   4   1 4
#> 4  2  9 3   8   6 8
#> 5  1 10 5   9   8 6
#> 6  6  2 1   3   8 5
```
当你第一次写Ｒ代码的时候，你可能已经用copy-and-paste解决了上面那个问题:

```
df$a[df$a == -99] <- NA
df$b[df$b == -99] <- NA
df$c[df$c == -98] <- NA
df$d[df$d == -99] <- NA
df$e[df$e == -99] <- NA
df$f[df$g == -99] <- NA
```
copy-and-paste的一个主要问题就是容易产生错误。你能发现上面两个代码的问题吗?这些错误产生的不一致，是因为我们没有对
期望的行为有一个权威的描述,(replace  − 99 with NA)。重复一个操作很容易产生bug，因为他将很难改变代码。举个例子，
如果对于缺失值的替换从-99改变到9999,你需要在多个地方进行修改。

为了防止bug，并且使代码更加的灵活，原则上需要采取“do not repeat yourself”。由Dave Thomas and Andy Hunt, 
这些“pragmatic programmers”，变形而来，这个原则是这样称述的:每一块知识在一个系统里面都要有一个单一的，明确的，
权威的表示。函数编程工具是有用的，因为它们提供了工具去减少重复。

我们可以用函数编程的思想重写上面的方法，修复单个向量中的缺失值:

```
fix_missing <- function(x) {
  x[x == -99] <- NA
  x
}
df$a <- fix_missing(df$a)
df$b <- fix_missing(df$b)
df$c <- fix_missing(df$c)
df$d <- fix_missing(df$d)
df$e <- fix_missing(df$e)
df$f <- fix_missing(df$e)
```
这个减少了可能发生错误的范围，但是没有消除它们:你不在会偶然地将-99打成了-98，但是你仍然可能混淆变量的名字。下面的步骤
就是通过两个函数的结合去移除这个错误。一个函数是，fix_missing()，修复单个的向量;另外一个是lapply()，针对数据框中的
每一列去做一些事。

lapply()采用了三个输入:x,一个列表; f,　一个函数; and ..., 传递给f()函数的其他参数。它会把这个函数应用到列表中的每
一个元素上面，然后返回一个新的列表。lapply(x, f, ...)等同于下面的循环:

```
out <- vector("list", length(x))
for (i in seq_along(x)) {
  out[[i]] <- f(x[[i]], ...)
}
```
真实的lapply()会更加的复杂，因为为了效率它是用Ｃ语言实现的，但是算法的本质是相同的。lapply() 被称作一个functional，
因为它把函数作为一个参数传入。Functional是函数编程一个重要的部分。你将会在functionals学到更多。

我们可以将lapply()用到这个问题，因为数据框是一个列表。我们只需要一些小干净的小技巧确保我们返回了一个数据框，而不是列表。
不将lapply()的结果赋值给df，我们重新把它们赋值给df[]。Ｒ通常的规则确保我们拿到了一个数据框，而不是一个列表。（如果这点
很惊讶，你需要去阅读subsetting and assignment章节。将这些知识整合到一起可以得到:

```
fix_missing <- function(x) {
  x[x == -99] <- NA
  x
}
df[] <- lapply(df, fix_missing)
```
相对于复制和粘贴，这段代码有五点优势:
- 更加的简介
- 如果缺失值需要改变，只需要在一个地方进行更新
- 可以在任意数量的列上面进行工作，不会偶然缺失一列
- 不会偶然地区别对待某一列
- 将这个技巧扩展到列的一个子集是容易的

```
df[1:5] <- lapply(df[1:5], fix_missing)
```
主要的思想就是函数组合。采用两个简单的函数，一个是对一列做一些操作，另一个是修复缺失值，在每一列中结合它们从而修复缺失值。
写可以独立理解的简单函数，然后将它们组合一个强大的技巧。

如果不同的列对缺失值采用不同的代码怎么办?你可能会忍不住复制和粘贴:

```
fix_missing_99 <- function(x) {
  x[x == -99] <- NA
  x
}
fix_missing_999 <- function(x) {
  x[x == -999] <- NA
  x
}
fix_missing_9999 <- function(x) {
  x[x == -999] <- NA
  x
}
```
像之前一样，这个容易产生bug。我们可以采用闭包，创建和返回函数的函数。闭包可以允许我们基于一个模板创建一个函数。

```
missing_fixer <- function(na_value) {
  function(x) {
    x[x == na_value] <- NA
    x
  }
}
fix_missing_99 <- missing_fixer(-99)
fix_missing_999 <- missing_fixer(-999)

fix_missing_99(c(-99, -999))
#> [1]   NA -999
fix_missing_999(c(-99, -999))
#> [1] -99  NA
```
##### 额外的参数
在这个例子里面，你可能会说你需要添加另外一个参数:

```
fix_missing <- function(x, na.value) {
  x[x == na.value] <- NA
  x
}
```
这是可以理解的方案，但是不是在每个方案中都工作的很好。我们在[MLE](http://adv-r.had.co.nz/Functionals.html#functionals-math)
中看到更多引人入胜的用法。

现在考虑一个相关问题。一旦你清理了数据，你可能想要为每一个变量计算一些数值统计。你可以写下面的代码:

```
mean(df$a)
median(df$a)
sd(df$a)
mad(df$a)
IQR(df$a)

mean(df$b)
median(df$b)
sd(df$b)
mad(df$b)
IQR(df$b)
```
但是，你最好定义和移除重复的项。花一两分钟思考一下如何解决这些问题，再继续阅读时。

一种方式就是一个汇总函数，然后把它应用到每一列:

```
summary <- function(x) {
  c(mean(x), median(x), sd(x), mad(x), IQR(x))
}
lapply(df, summary)
```
这是很好的开始，但是仍然有一些重复。如果我们使汇总函数更现实一点，会看的更容易:

```
summary <- function(x) {
 c(mean(x, na.rm = TRUE),
   median(x, na.rm = TRUE),
   sd(x, na.rm = TRUE),
   mad(x, na.rm = TRUE),
   IQR(x, na.rm = TRUE))
}
```
所有这五个函数都被同样的参数调用了五次，重复了五次。重复的代码会使我们的代码比较脆弱。这很容易产生bug，
也很难适应不断变化的需求。

为了移除重复的代码，你可以利用另一个函数编程的技巧:将函数保存在列表中。

```
summary <- function(x) {
  funs <- c(mean, median, sd, mad, IQR)
  lapply(funs, function(f) f(x, na.rm = TRUE))
}
```
本章节将会更细节地讨论这些技巧。但是当你可以学习它们的时候，你需要学习最简单的函数编程工具，匿名函数。

####　匿名函数
在R中，函数本身就是对象。它们不是自动地和名字绑定。不像其他很多函数，例如C, C++, Python, and Ruby，R没有一个特殊的语法
去创建一个有名字的函数:当你创建一个函数的时候，你使用正规的赋值操作符去给它一个名字。如果你没有给函数名字，你将得到一个匿名
函数。

当不值得去赋予一个函数名字的时候，你可以使用一个匿名函数:

```
lapply(mtcars, function(x) length(unique(x)))
Filter(function(x) !is.numeric(x), mtcars)
integrate(function(x) sin(x) ^ 2, 0, pi)
```
像R中很多其他函数一样，匿名函数有formals(), a body(), and a parent environment():

```
formals(function(x = 4) g(x) + h(x))
#> $x
#> [1] 4
body(function(x = 4) g(x) + h(x))
#> g(x) + h(x)
environment(function(x = 4) g(x) + h(x))
#> <environment: R_GlobalEnv>
```
你可以调用一个匿名函数，不需要给它名字，但是代码有点难以阅读，因为你以使用两种不同方式的括号:第一个去调用一个函数，第二个是为了
明确你是调用一个匿名函数本身，而不是调用一个匿名函数内部的函数。

```
# This does not call the anonymous function.
# (Note that "3" is not a valid function.)
function(x) 3()
#> function(x) 3()

# With appropriate parenthesis, the function is called:
(function(x) 3)()
#> [1] 3

# So this anonymous function syntax
(function(x) x + 3)(10)
#> [1] 13

# behaves exactly the same as
f <- function(x) x + 3
f(10)
#> [1] 13
```
你可以使用命名的参数调用匿名函数，但是这么做暗示你的函数需要一个名字。
 
匿名函数一个最普通的应用是创建一个闭包，由其他函数创建的函数。闭包将在下一个章节介绍。

#### 闭包
“An object is data with functions. A closure is a function with data.” — John D. Cook

匿名函数的一个使用是创建不值得命名的小的函数。另一个重要的应用是创建闭包，由其他函数写的函数。闭包得到它们的名字是因为它们
enclose（封闭）了父函数的环境，并且可以取得它的所有变量。这是有用的，因为它允许我们有两层参数:父亲层参数控制着操作，子层
参数真正做事情。

下面的例子使用这种思想产生了幂函数的族，父亲函数是power()，创建了两个子函数（square()和cube()）。

```
power <- function(exponent) {
  function(x) {
    x ^ exponent
  }
}

square <- power(2)
square(2)
#> [1] 4
square(4)
#> [1] 16

cube <- power(3)
cube(2)
#> [1] 8
cube(4)
#> [1] 64
```
当你打印一个闭包的时候，你将不会看到特别有用的信息。

```
square
#> function(x) {
#>     x ^ exponent
#>   }
#> <environment: 0x3a50680>
cube
#> function(x) {
#>     x ^ exponent
#>   }
#> <environment: 0x37f10f8>
```
这是因为函数本身并没有改变。不同的是函数的封闭环境，environment(square)。查看环境内容的一种方式是将其转换成一个列表:

```
as.list(environment(square))
#> $exponent
#> [1] 2
as.list(environment(cube))
#> $exponent
#> [1] 3
```
另外一种方式查看闭包是使用pryr::unenclose()。这个函数用它们的值替代了定义在闭包环境中的变量。

```
library(pryr)
unenclose(square)
#> function (x) 
#> {
#>     x^2
#> }
unenclose(cube)
#> function (x) 
#> {
#>     x^3
#> }
```
一个闭包的父环境是创建它函数的执行环境,如下面的代码所示:

```
power <- function(exponent) {
  print(environment())
  function(x) x ^ exponent
}
zero <- power(0)
#> <environment: 0x4197fa8>
environment(zero)
#> <environment: 0x4197fa8>
```
函数返回一个值之后，执行环境通常就消失了。然而，函数捕获了它们的封闭环境。这意味着当函数a返回函数b的时候，函数b捕获和保存了函数a的
执行环境，并且它不消失。（这对内存使用有着重要的后果，具体细节看memory usage)

在R中，几乎所有的函数都是一个闭包。所有的函数都记住了它们被创建时的环境，如果这个函数是你自己写的，那就是全局环境;如果是其他人写的，
那就是包环境。唯一的意外就是原语函数，直接调用C代码，没有相联系的环境。

闭包在制造函数工厂时是有用的，并且是R中管理可变状态的一种方式。

##### 函数工厂
一个函数工厂是制造一个新的函数的工厂。我们已经看到函数工厂的两个例子了，missing_fixer()和power()。你可以通过参数调用它们，描述了
期望的行为，并且返回了一个为你工作的函数。对于missing_fixer()和power()，使用一个函数工厂代替一个具有多个参数的单一函数收益不是那么
大的。下面的情况函数工作是最有用的:
- 不同的程度是更加复杂的，带有更多的参数和更加复杂的函数体。
- 当一些函数产生的时候，一些工作只需要被做一次。

函数工作特别适用于最大似然问题，并且你可以在mathematical functionals看到更加引人注目的使用。

##### 可变的状态
在两个水平上面有变量允许你在函数调用之间维持状态。这是可能的，因为当执行环境每次被刷新的时候，封闭环境是不变的。在不同的水平管理变量的关键
是使用双箭头赋值操作符。不像通常的单箭头赋值操作符总是在当前的环境中赋值，双箭头操作符将持续查找父环境的链知道找到匹配的名字。（Binding names to values）
对于这如何工作有更细节的描述。

同时，一个静态的父环境和<<-使得在不同函数调用中维持状态变得可能。下面的雷子展示了一个计数器，记录了一个函数被调用了多少次。new_counter每一次跑，它都创建了
一个函数，在这个环境中初始化了计数器i，然后创建了一个新的函数。

```
new_counter <- function() {
  i <- 0
  function() {
    i <<- i + 1
    i
  }
}
```
新的函数是一个闭包，它的封闭环境是当new_counter()执行时所创建的环境。函数执行环境是临时的，但是闭包可以获取它被创建时的环境。在下面的例子里，闭包counter_one()
和counter_two()执行时，每一个都取到它们子集的封闭环境，因此它们能获取不同的计数。

```
counter_one <- new_counter()
counter_two <- new_counter()

counter_one()
```
计数函数通过在它们的本地环境不修改变量的方法逃避了重新初始化的限制。因为变化是在不变的父环境中或者称为封闭环境中发生，它们在函数调用过程中被保存了。

如果你不使用闭包会发生什么?如果你使用<-而不是<<-会发生什么?预测用下面的变体去替换new_counter()会发生什么，执行下面的代码并检查你的预测。

```
i <- 0
new_counter2 <- function() {
  i <<- i + 1
  i
}
new_counter3 <- function() {
  i <- 0
  function() {
    i <- i + 1
    i
  }
}
```
在父亲环境中修改值是一个重要的技巧，因为这是R中产生可变状态的一种方式。可变状态通常是难的，因为每次你看起来像是修改一个对象，实际上你是在创建然后修改
一个复制。然而，如果你的确需要可变的对象，你的代码通常不是简单的，使用引用类通常是更好的选择，在RC中描述的。

闭包的功能是和functionals和function operators中更高级的思想是紧密耦合的。在这两章中你将看到更多的闭包。下面的章节讨论Ｒ中函数编程的第三个技巧:
在列表中存储函数的能力。

##### 函数列表
在Ｒ中，函数可以保存在列表中。这使得可以更方便地操作相似函数族，同样的道理，数据框是为了更方便地操作相似的向量组。

我们从简单的基准例子开始。想象一下你在比较计算算术平均不同方式的性能。你可以将每一种方法储存在列表中做到它:

```
compute_mean <- list(
  base = function(x) mean(x),
  sum = function(x) sum(x) / length(x),
  manual = function(x) {
    total <- 0
    n <- length(x)
    for (i in seq_along(x)) {
      total <- total + x[i] / n
    }
    total
  }
)
```
从列表中调用函数是直接的。你抽取它然后调用它:

```
x <- runif(1e5)
system.time(compute_mean$base(x))
system.time(compute_mean[[2]](x))
system.time(compute_mean[["manual"]](x))
```
调用每一个函数，可以使用lapply()。我们需要一个匿名函数或者一个新的命名函数，因为没有内置的函数去处理这种情景。

```
lapply(compute_mean, function(f) f(x))
call_fun <- function(f, ...) f(...)
lapply(compute_mean, call_fun, x)
```
为了统计每个函数的时间，我们需要组合lapply() and system.time():

```
lapply(compute_mean, function(f) system.time(f(x)))
```
函数列表的另一个用途是用多种方式汇总一个对象。为了实现它，我们可以将每个汇总函数放到一个列表中，然后用lapply()
执行它们:

```
x <- 1:10
funs <- list(
  sum = sum,
  mean = mean,
  median = median
)
lapply(funs, function(f) f(x))
```
如果我们希望我们的汇总函数自动去除缺失值该怎么办呢？一种方式是制造一系列匿名函数，通过合适的参数取调用我们的汇总函数:

```
funs2 <- list(
  sum = function(x, ...) sum(x, ..., na.rm = TRUE),
  mean = function(x, ...) mean(x, ..., na.rm = TRUE),
  median = function(x, ...) median(x, ..., na.rm = TRUE)
)
lapply(funs2, function(f) f(x))
```
然而，这个将产生一系列重复。除了不同的函数名字，每个函数基本是相同的。一个更好地办法是修改我们的lapply()函数调用去包含
额外的参数:

```
lapply(funs, function(f) f(x, na.rm = TRUE))
```
##### 移动函数列表到全局环境
有时候你可能想要创建函数列表，不需要使用特殊的语法就可以得到。举个例子，想象一下你想要创建HTML代码，通过将每一个tag映射到
R函数中。下面的例子使用了一个函数工厂创建可函数，针对标签(paragraph), (bold), and (italics)。

我已经将函数放到一个列表中去了因为我不想它们随时可以得到。在一个已经存在R函数和一个HTML标签中存在冲突的风险是高的。但是将
它们放在一个列表中将会使代码更加冗余:

```
html$p("This is ", html$b("bold"), " text.")
```
取决于我们想要这种效果持续多久，消除html$的使用有三种选择:
- 对于非常短暂的效果，你可以使用with():

```
　　with(html, p("This is ", b("bold"), " text."))
```
- 对于长期的效果，你可以将函数attach()到搜索路径上面，然后当你们完成了，可以detach():

```
  　attach(html)
  　p("This is ", b("bold"), " text.")
  　detach(html)
```
- 最后你可以使用list2env()函数将函数复制到全局环境中。你可以在完成之后，删除这些函数。

```
　　list2env(html, environment())
  　p("This is ", b("bold"), " text.")
　　rm(list = names(html), envir = environment())
```
我推荐第一种用法，使用with(),因为这使得代码在特殊的上下文时执行时非常明确，并且知道上下文是什么。

##### 案例学习:数值积分
为了总结本章，我将优先使用函数开发一个简单的数值积分工具。开发这个工具的每一步都是秉着减少重复和
使得这个方法更加普适的原则。

数值积分背后的思想很简单:使用一些小的组件近似找出曲线下面的面积。两个最简单的方法是中值点和梯形规则。
中值规则用三角形近似曲线。梯形规则使用梯形。每一次我们从a到b对一个函数进行积分。对于这个例子，我们
使用0到pi对sinx进行积分。


































