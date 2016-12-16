### 面向对象
本章节是认识和操作R对象的一个简明指南。R有三种面向对象的系统（加上基本类型），所以可能有点吓人。本章节的目的不是让你成为所有四种对象的专家，而是帮助你确定你是在操作哪个系统，并帮助你有效地使用它们。

任何面向对象系统，主要就是类和方法的概念。一个类定义了对象的行为，通过描述它们的属性和同其他类的关系。当选择方法时，类同样被使用，方法表现的不同取决于它们输入的类型。类通常是层级组织的：如果一个子
类不存在一个方法，然后它的父类将被使用。子类继承父类的行为。

R的三种面向对象的系统不同于类和方法是如何定义的：

- **S3** 实现面向对象编程的方式是面向对象的泛型函数。这不同于很多编程语言，像Java，C++和C，实现面向对象的消息传递。对于消息传递，消息（方法）发给对象，对象决定哪种方法执行。特别的，对象调用方法有特殊的形式，
  通常出现在方法或者消息的前面，例如：canvas.drawRect("blue")。S3是不同的。尽管计算仍然是由方法引出，一个称为泛型函数的特殊形式将决定调用哪种方法，例如：drawRect(canvas, "blue")。S3是一种非
  常简单的系统，它没有对类进行正式的定义。
  
- **S4** 对象同S3工作类似，但是更加正式。相对于S3，S4有两种主要的不同。S4有正式的类定义，描述了每个类的表示和继承，有辅助的函数去定义泛型和方法。S4同时有多重分发，意味着泛型函数可以根据类包含参数
  的多少决定选择哪种方法，而不仅仅是一个。
  
- **引用类** 简称为RC，同S3和S4非常不同。RC实现了面向对象的消息传递，因此方法是属于类的，而不是函数。$用于分割对象和方法，因此方法调用的形式如canvas$drawRect("blue"). RC对象是可变的：它们不使用R通常
  的修改复制的语境，而是随时修改。这点让引用类非常难以解释，但是可以让它们解决S3和S4难以解决的问题。
  
还有另外一个系统，不是那么的面向对象，但是提到很有必要：
- **基本类型** 内部的c级别的类型，位于其他面向对象系统之下。基本类型大多数通过C代码进行操作，但是它们很重要，因为它们为其他面向对象的系统提供了构建模块。

下面轮流介绍每一个系统，从基本类型开始。你将学习如何识别一个对象属于哪一个面向对象系统，方法如何分发工作，以及如何创建对象，类，泛型，和方法。本章节将针对如何使用每个系统做一些总结标注。

###### 预备
安装pryr包，install.packages("pryr")， 可以提供一些有用的函数去检查面向对象的属性。

###### 纲要
- 基本类型 关于R中基本的对象系统。只用R-core可以增加新的类到这个系统中，但是知道它很重要，因为它是其他三个系统的基础。
- S3 介绍S3对象系统的基础。它是最简单也是最常使用的。
- S4 讨论更加正式和严格的S4系统
- RC 告诉你R中最新的面向对象的系统：引用类，或者简称为RC
- 选择一个系统 当你开始建立一个新的对象的时候，对于使用何种系统给出建议。

#### 基本类型
在每一个R对象底下是一个C的结构，其描述了一个对象是如何保存在内存中的。这个结构包括这个对象的组成部分，用于内存管理的信息，以及最重要的部分，类型。这是R对象的基础类型。基础类型并不是一个真正的对象系统，因为只有
R-core团队才能创建新的类型。因此，新的基础类型将很少添加：最近一次改动是在2011年，加了两个你在R中从来没有见过的奇特类型，但是对于诊断内存问题很有用（NEWSXP和FREESXP）。在这之前，最后添加的是一种针对S4对象的
基础类型（S4SXP），2005年添加。

在数据结构章节，解释了最常用的基础类型，（原子向量和列表），但是基础类型还包括函数，环境和其他更奇特的对象，例如名字，调用，promises，这些你将在这本书的后面学习到。你可以用typeof()查看一个对象的基础类型。
不幸的是基础类型的名字在R中不会一致的使用，类型和其对应的is函数可以使用不同的名字：

```
# The type of a function is "closure"
f <- function() {}
typeof(f)
#> [1] "closure"
is.function(f)
#> [1] TRUE

# The type of a primitive function is "builtin"
typeof(sum)
#> [1] "builtin"
is.primitive(sum)
#> [1] TRUE
```
你可能已经听说过mode()和storage.mode()。我建议你忽略这些函数，因为它们仅仅是typedof()返回名字的别名，存在仅仅是为了兼容S语言。如果你想要了解他们具体是做什么
的，你可以阅读他们的源码。

针对不同的基础类型，表现不同的函数几乎总是用C写的，使用switch语句可以进行分发（例如：switch(TYPEOF(x))）。即使你从来不写C代码，了解基础类型也是重要的，因为其他每件事都是建立在它们之上的：S3对象能在任何基础类型上面建立，S4对象使用一个特殊的基础类型，RC
对象是S4对象和环境（另外一个基础类型）的结合。检查一个对象是否是一个完全的基础类型：is.object(x)返回FALSE，并且没有S3，S4或者RC对象的行为。

##### S3
S3是R中第一个也是最简单的面向对象系统。它是基础和统计包中唯一的面向对象系统，也是CRAN包中使用最多的。S3是非正式的和特别的，但是它的简单中有某种优雅：你不能取走它的和人部分，同时它仍然是一个有用的面向对象系统。

###### 认识对象，泛型和方法
你遇到的大多数对象都是S3对象。但是不幸的是没有简单的方法去测试在基础R中一个对象是否是S3对象。可以使用最接近就是is.object(x) & !isS4(x)，例如它是一个对象但不是S4。一个简单的方式是使用pryr::otype():

```
library(pryr)

df <- data.frame(x = 1:10, y = letters[1:10])
otype(df)    # 一个数据框是一个S3类型
#> [1] "S3"
otype(df$x)  # A numeric vector isn't
#> [1] "base"
otype(df$y)  # A factor is
#> [1] "S3"
```
在S3中，方法属于函数，称为泛型函数，或者简称为泛型。S3方法不属于对象或者类。这个不同于很多其他编程语言，但是一种正当的面向对象的方式。

查看一个函数是否是S3泛型，你可以用UseMethod()查看它的源码：这个函数可以找出正确的调用方法，函数分发的过程。同otype()类似，pryr同样提供ftype()描述对象系统，如下所示：

```
mean
#> function (x, ...) 
#> UseMethod("mean")
#> <bytecode: 0x21b6ea8>
#> <environment: namespace:base>
ftype(mean)
#> [1] "s3"      "generic"
```
一些S3的泛型，例如[, sum()和cbind()，不调用UseMethod()因为它们是用C实现的。另外，它们调用C的函数DispatchGroup()和DispatchOrEval()。在C代码中进行方法分发的函数称为内部的泛型，用?"internal generic"可以查看文档。ftype() 可以知道这个特殊的例子。

给定一个类，S3泛型的作用是调用正确的S3的方法。你可以通过名字识别S3的方法，形如generic.class()。举个例子，mean()泛型的时间类型的函数，叫做mean.Date()，print()泛型的因子方法，叫做print.factor()。

这也是现在大多数编程指南中不鼓励在函数名字中使用.的原因：它看起来像是S3的方法。举个例子，t.test()是针对test对象的t方法吗？类似的，在类名字中使用.也会带来疑惑：print.data.frame()是data.frames的print()方法，还是针对frames的print.data()方法？pryr::ftype()知道这些特殊情况，因此你可以用它找出一个函数是S3方法还是泛型：

```
ftype(t.data.frame) # data frame method for t()
#> [1] "s3"     "method"
ftype(t.test)       # generic function for t tests
#> [1] "s3"      "generic"
```
你可以使用methods()查看属于一个泛型的所用方法：

```
methods("mean")
#> [1] mean.Date     mean.default  mean.difftime mean.POSIXct  mean.POSIXlt 
#> see '?methods' for accessing help and source code
methods("t.test")
#> [1] t.test.default* t.test.formula*
#> see '?methods' for accessing help and source code
```
(除了定义在base包中的方法，很多S3的方法是不可见的：使用getS3method()可以查看它们的源码）

给定一个类别，你可以列出所用提供其方法的泛型：

```
methods(class = "ts")
#>  [1] aggregate     as.data.frame cbind         coerce        cycle        
#>  [6] diffinv       diff          initialize    kernapply     lines        
#> [11] Math2         Math          monthplot     na.omit       Ops          
#> [16] plot          print         show          slotsFromS3   time         
#> [21] [<-           [             t             window<-      window       
#> see '?methods' for accessing help and source code
```
没有办法列出所用S3的类，这个你将在下面的部分学习到。

##### 定义类和创建对象
S3是一个简单和特别的系统：它没有对类正式的定义。去创建一个类实例对象，你仅仅使用一个已经存在的基础对象还是设定它的类属性就行了。你可以通过structure()函数进行创建，或者在之后使用class<-()函数：

```
# Create and assign class in one step
foo <- structure(list(), class = "foo")

# Create, then set class
foo <- list()
class(foo) <- "foo"
```
S3对象通常建立在列表或者带有属性的原子向量之上（你可以在attribute章节进行查看）。你还可以将函数转成S3对象。其他一些基础类型不常用，要么在R中很多见到，要么就是不太适合同属性一起操作。

你可以通过class(x)查看任意对象的类别。或者查看你一个对象是否从一个特别的类别继承，使用inherits(x, "classname")。
```
class(foo)
#> [1] "foo"
inherits(foo, "foo")
#> [1] TRUE
```
S3对象的类可以是一个向量，描述了按具体程度从高到低的行为。举个例子，glm()对象的类是c("glm", "lm")，暗示了广义线性模型继承了线性模型的行为。类名通常小写，你应该避免使用.。否则，使用对于多单词的类名来说是使用下划线还是大驼峰会产生混淆。

大多数S3类提供一个构造函数：
```
foo <- function(x) {
  if (!is.numeric(x)) stop("X must be numeric")
  structure(list(x), class = "foo")
}
```
如果有你可以使用它（像factor()和data.frame()）。这个确保你可以使用正确的内容创建一个类。构造器函数通常同类有着相同的名字。

除了开发者提供构造函数，S3不检查正确性。这意味着你可以改变一个已存在对象的类：

```
# Create a linear model
mod <- lm(log(mpg) ~ log(disp), data = mtcars)
class(mod)
#> [1] "lm"
print(mod)
#> 
#> Call:
#> lm(formula = log(mpg) ~ log(disp), data = mtcars)
#> 
#> Coefficients:
#> (Intercept)    log(disp)  
#>      5.3810      -0.4586

# Turn it into a data frame (?!)
class(mod) <- "data.frame"
# But unsurprisingly this doesn't work very well
print(mod)
#>  [1] coefficients  residuals     effects       rank          fitted.values
#>  [6] assign        qr            df.residual   xlevels       call         
#> [11] terms         model        
#> <0 rows> (or 0-length row.names)
# However, the data is still there
mod$coefficients
#> (Intercept)   log(disp) 
#>   5.3809725  -0.4585683
```
如果你使用其他面向对象的函数，这是可能会让你不习惯。但是令人惊讶的是，这种灵活性会很少造成问题：尽管你能改变对象的类型，但你从不应该。R语言不会保护你自己：你可以轻松地拿石头砸自己的脚。
只要你不朝你的脚开枪，你就不会有问题。

##### 创建新的方法和泛型
添加一个新的泛型，可以用UseMethod()创建一个函数。UseMethod()输入两个变量：一个是泛型函数的名字，一个是方法分发的使用。如果你忽略了第二个参数，它将根据第一个参数取分发函数。将泛型的任意参数传递给UseMethod()是没必要的，也是不应该的。UseMethod()通过黑科技自己找到它们。
```
f <- function(x) UseMethod("f")
```
一个没有方法的泛型是没有用的。添加一个方法，你仅仅需要用一个正确的名字去创建一个正规的函数：

```
f.a <- function(x) "Class a"

a <- structure(list(), class = "a")
class(a)
#> [1] "a"
f(a)
#> [1] "Class a"
```
添加方法到已经存在的接口也是同样的方式：

```
mean.a <- function(x) "a"
mean(a)
#> [1] "a"
```
正如你所看见的，方法返回一个与接口相匹配的类是没有做检查的。你的方法不违背已经存在代码的期望，取决你自己。

##### 方法分发
S3的方法分发是相对简单的。UseMethod()创建了一个函数名字的向量，例如paste0("generic", ".", c(class(x), "default"))，然后轮流查找每一个。默认的类为不知道的类提供了一个了候补的方法。

```
f <- function(x) UseMethod("f")
f.a <- function(x) "Class a"
f.default <- function(x) "Unknown class"

f(structure(list(), class = "a"))
#> [1] "Class a"
# No method for b class, so uses method for a class
f(structure(list(), class = c("b", "a")))
#> [1] "Class a"
# No method for c class, so falls back to default
f(structure(list(), class = "c"))
#> [1] "Unknown class"
```
组泛型方法加起来可能有一些复杂。组泛型可以实现通过一个函数为多个泛型提供方法。四个组泛型和它们包括的函数是：
- Math: abs, sign, sqrt, floor, cos, sin, log, exp, …
- Ops: +, -, *, /, ^, %%, %/%, &, |, !, ==, !=, <, <=, >=, >
- Summary: all, any, sum, prod, min, max, range
- Complex: Arg, Conj, Im, Mod, Re

组泛型是相对比较高级的技巧，并且超过了本章节的范围，你可以通过?groupGeneric查看更多帮助。最重要的事情是意识到Math, Ops, Summary和Complex不是真正的函数，而是代表了函数的组。注意
在组泛型函数的内部，有一个特殊的变量.Generic提供了真正的泛型函数。

如果你有复杂的类层次结构，调用父方法有时是有用的。但是如何精确地定义是比较棘手的，如果当前的方法不存在，这个父方法将会被调用。同时，这是一个比较高级的技巧，你可以在?NextMethod中找到。

因为方法都是正常的R函数，你可以直接调用它们：

```
c <- structure(list(), class = "c")
# Call the correct method:
f.default(c)
#> [1] "Unknown class"
# Force R to call the wrong method:
f.a(c)
#> [1] "Class a"
```
然而，改变一个对象的类是危险的，你不应该这么做。请不要朝自己的脚上开枪。直接调用方法的唯一理由是有时候你可以通过去除方法分发得到性能的提升。详细看**性能**章节。

你还可以用一个非S3的对象调用S3的泛型。非内部非S3泛型将分发给基础类型的隐式类。（内部的泛型不是因为性能原因这样做。决定一个基础类型的隐式类是复杂的，如下面的函数
所示：

```
iclass <- function(x) {
  if (is.object(x)) {
    stop("x is not a primitive type", call. = FALSE)
  }

  c(
    if (is.matrix(x)) "matrix",
    if (is.array(x) && !is.matrix(x)) "array",
    if (is.double(x)) "double",
    if (is.integer(x)) "integer",
    mode(x)
  )
}
iclass(matrix(1:5))
#> [1] "matrix"  "integer" "numeric"
iclass(array(1.5))
#> [1] "array"   "double"  "numeric"
```















