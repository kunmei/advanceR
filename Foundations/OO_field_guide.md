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

你还可以用一个非S3的对象调用S3的泛型。非内部非S3泛型将分发给基础类型的隐式类。（内部的泛型不是因为性能原因这样做。决定一个基础类型的隐式类是复杂的，如下面的函数所示：

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
##### S4
S4和S3工作类似，不过更加正式和严格。方法仍然属于函数，不是类，但是：
- 类有正式的定义，描述了它们的领域和继承结构，父类。
- 方法分发可以基于泛型函数的多个参数进行，而不仅仅是一个。
- 有一个特殊的操作算子，@， 用来提取S4对象的槽（字段）

所有S4相关的代码都保存在爱methods这个包里面。当你交互式地执行R的时候，这个包总是有的，但是当用批模式跑R的时候不一定得到。这是这个原因，无论你什么时候使用S4，
明确的library(methods)是一个好的注意。

S4是一个丰富且复杂的系统。用几页纸肯定不能完全解释清楚。这里我将聚焦与S4底层主要的思想，从而你可以搞笑地使用已经存在的S4对象。学习更多，这里有一些推荐：
- [S4 system development in Bioconductor](http://www.bioconductor.org/help/course-materials/2010/AdvancedR/S4InBioconductor.pdf)
- John Chambers’ Software for Data Analysis
- Martin Morgan’s answers to S4 questions on stackoverflow

##### 识别对象，泛型函数和方法
识别S4对象，泛型和方法是容易的。你可以识别一个S4的对象，因为str()把它描述成了一个正式的类，isS4()会返回TRUE，pryr::otype()返回"S4"。S4的泛型和方法是容易定义的，因为它们是带有很好类的定义的S4对象。

在基础包里面(stats, graphics, utils, datasets and base)中没有经常使用的S4类，因此我们利用stats4这个包开始创建S4的对象，它提供了一些S4的类和最大似然估计相联系的方法。

```
library(stats4)

# From example(mle)
y <- c(26, 17, 13, 12, 20, 5, 9, 8, 5, 4, 8)
nLL <- function(lambda) - sum(dpois(y, lambda, log = TRUE))
fit <- mle(nLL, start = list(lambda = 5), nobs = length(y))

# An S4 object
isS4(fit)
#> [1] TRUE
otype(fit)
#> [1] "S4"

# An S4 generic
isS4(nobs)
#> [1] TRUE
ftype(nobs)
#> [1] "s4"      "generic"

# Retrieve an S4 method, described later
mle_nobs <- method_from_call(nobs(fit))
isS4(mle_nobs)
#> [1] TRUE
ftype(mle_nobs)
#> [1] "s4"     "method"
```
使用is()带一个参数可以列出一个对象继承的所有类。使用is()带两个参数可以检验一个对象是否从一个特殊的类继承。

```
is(fit)
#> [1] "mle"
is(fit, "mle")
#> [1] TRUE
```
你可以用getGenerics()列出所有S4的泛型，用getClasses()列出所有的S4的类。这个列表包括针对S3类和基础类型的垫片类。你可以用showMethods()列出所有的S4方法，可选择的通过泛型或者是通过类。提供where = search()限制搜索的环境也是好的注意。

##### 定义类和创建对象
在S3中，你可以将任意对象转变成一个特殊类型的对象，仅仅设置类属性。S4是更加严格的：你必须用setClass()定义类的表达方式，用new()的方式创建一个新的对象。你可以用特殊的语法：class?className, e.g., class?mle，查找一个类的文档。

一个S4类有三个主要的特性：
- **名字** 一个字母-数字类定义，按照惯例，S4类的名字采用大驼峰。
- **槽**(slots)的名字列表,定义了槽的名字和允许类。举个例子，一个人的类可能是由一个字符型的名字和一个数值型的年龄定义：list(name = "character", age = "numeric")。
- 继承类的字符类型，或者在S4的术语中，叫做**contains**。你可以针对多重继承提供多个类，但是这是一个高级的技术，增加了复杂性。

在slots和contains，你可以使用S4类，或者是用setOldClass()注册的S3类，或者是基础类型的隐式类。你slots你还可以使用ANY这个特殊咧，从而不限制输入。

S4类可以有其他的可选择的特性，例如一个validity方法，测试一个对象是否是固定的，和一个prototype对象定义默认的slot值。具体可以看?setClass。

下面的例子创建了一个拥有name和age字段的Person类，和一个Employee类继承于Person。这个
Employee类继承Person的槽和方法，并且添加一个额外的slot，boss。我们可以通过用一个类的名字，slot值的名字-值对来调用new()来创建对象。

```
setClass("Person",
  slots = list(name = "character", age = "numeric"))
setClass("Employee",
  slots = list(boss = "Person"),
  contains = "Person")

alice <- new("Person", name = "Alice", age = 40)
john <- new("Employee", name = "John", age = 20, boss = alice)
```
大多数S4类会有一个构造函数，名字同类的名字相同：如果存在，使用它代替直接调用new()。

取得一个S4对象的槽使用@或者slot():

```
alice@age
#> [1] 40
slot(john, "boss")
#> An object of class "Person"
#> Slot "name":
#> [1] "Alice"
#> 
#> Slot "age":
#> [1] 40
```
(@ 等同于 $, slot()等同于 [[.)
如果一个S4对象包含（继承于）一个S3类或者一个基础类型，会有一个特殊的.Data槽，包含了潜在的基础的类型或者是S3对象：

```
setClass("RangedNumeric",
  contains = "numeric",
  slots = list(min = "numeric", max = "numeric"))
rn <- new("RangedNumeric", 1:10, min = 1, max = 10)
rn@min
#> [1] 1
rn@.Data
#>  [1]  1  2  3  4  5  6  7  8  9 10
```
因为R是一个交互式的编程语言，你可以在任意时刻创建一个新类或者重新定义已经存在的类。当你交互式操作S4的时候，这可能是一个问题。如果你修改一个类，确保还要重新创建这个类的任何对象，否则你将遇到非法的对象。

##### 创建新的方法和泛型
S4提供了特殊的函数用来创建泛型和方法。setGeneric()创建一个新的泛型或者将一个已经存在的函数转变成泛型。setMethod()利用泛型的名字，方法使用到的类，以及实现该方法的一个函数类创建一个S4的method。举个例子，union()，通常使用在向量上，我们可以使它使用到数据框上面。

```
setGeneric("union")
#> [1] "union"
setMethod("union",
  c(x = "data.frame", y = "data.frame"),
  function(x, y) {
    unique(rbind(x, y))
  }
)
#> [1] "union"
```
如果你要从零开始创建一个新的接口，你需要提供一个函数调用standardGeneric():

```
setGeneric("myGeneric", function(x) {
  standardGeneric("myGeneric")
})
#> [1] "myGeneric"
```
S4中的standardGeneric()等同于UseMethod()。
##### 方法分发
如果一个S4的泛型分发在只有一个父类的单个类上时，S4的方法分发就等同于S3的分发。最主要的不同是如何取设置默认值:S4使用特殊的类ANY去匹配任意的类型，"missing"去匹配一个缺失值。和S3一样，S4也有组泛型，用?S4groupGeneric可以查看文档,和一种调用父类的方法，callNextMethod()。

如果通过多个参数，或者你的类具有多重继承的时候，方法分发将变得更加的复杂。规则描述在?Methods里面，但是它们是复杂的，并且很难去预测该调用哪种方法。因为这个原因，我强烈建议避免多重继承和多重分发，除非绝对有必要。

最后，有两种方法可以找到给定一个特定的泛型调用，哪种方法被调用:

```
# From methods: takes generic name and class names
selectMethod("nobs", list("mle"))

# From pryr: takes an unevaluated function call
method_from_call(nobs(fit))
```
#### RC
引用类，或者简称RC是R中最新的面向对象系统。它们在2.12版本中被介绍。它们从基本上不同于S3和S4，是因为:
- RC方法属于对象，而不是函数
- RC的对象是可变的：通常的R copy-on-modify语义是不适用的

这些特性使得RC对象表现的更像其他编程语言中的对象，例如Python，Ruby，Java和C#。引用类是用R代码实现的:它们是一个特殊的S4对象包装在一个环境中。

##### 定义类和创建对象
因为在R的基础包里面没有任何引用类，我们从创建一个开始。RC类描述有状态的对象是最好使用的，对象可以随着时间改变，因此我们创建一个简单的类去模拟一个银行账户。

创建一个新的RC类和创建一个新的S4类是类似的，但是你需要使用setRefClass()代替setClass()。第一和唯一要求的参数是字母数字形式的name。当你用new()去创建一个新的RC对象时，使用setRefClass()返回的对象是生成新的对象是一种好的风格。（你可以用在S4对象上，但基本看不到）

```
Account <- setRefClass("Account")
Account$new()
#> Reference class object of class "Account"
```
setRefClass()同样接收一个名字-类型对的列表来定义类的fields。等同于S4的槽。同时额外传给new()的参数将初始化字段的值。你可以得到和设置字段的值通过$:

```
Account <- setRefClass("Account",
  fields = list(balance = "numeric"))

a <- Account$new(balance = 100)
a$balance
#> [1] 100
a$balance <- 200
a$balance
#> [1] 200
```
除了为字段提供一个类的名字，你还可以提供一个获取方法。当你获取或者设置字段的时候，你可以增加定制的行为。具体看?setRefClass。

注意RC对象是可变的，例如，它们有引用语义，不是copied-on-modify:

```
b <- a
b$balance
#> [1] 200
a$balance <- 0
b$balance
#> [1] 0
```
由于这个原因，RC对象有一个copy()的方法，可以得到一个对象的复制:

```
c <- a$copy()
c$balance
#> [1] 0
a$balance <- 100
c$balance
#> [1] 0
```
一个对象如果没有通过methods定义的行为可能不是有用的。RC方法和类联系，并且能随时修改它的字段。在下面的例子里，注意你可以通过名字得到字段的值，并且可以用<<-修改它们。你可以在Environments中学到更多关于<<-的知识。

```
Account <- setRefClass("Account",
  fields = list(balance = "numeric"),
  methods = list(
    withdraw = function(x) {
      balance <<- balance - x
    },
    deposit = function(x) {
      balance <<- balance + x
    }
  )
)
```
你可以用取得字段同样的方式取调用一个RC的方法:

```
a <- Account$new(balance = 100)
a$deposit(100)
a$balance
#> [1] 200
```
最后，setRefClass()重要的参数还有contains。这是所要继承的父RC类的名字。下面的雷子创建了一个新的银行账号，返回一个错误从而阻止账户金额低于0。


```
NoOverdraft <- setRefClass("NoOverdraft",
  contains = "Account",
  methods = list(
    withdraw = function(x) {
      if (balance < x) stop("Not enough money")
      balance <<- balance - x
    }
  )
)
accountJohn <- NoOverdraft$new(balance = 100)
accountJohn$deposit(50)
accountJohn$balance
#> [1] 150
accountJohn$withdraw(200)
#> Error in accountJohn$withdraw(200): Not enough money
```
所有的引用类最终都继承于envRefClass。它提供了有用的方法，例如copy()，callSuper()（调用父的字段），field()（通过名字得到字段的值)， export()（等同于as()）,show()（覆盖print）。可以在setRefClass()的继承章节查看详细内容。

##### 识别对象和方法
你可以识别RC对象因为它们是继承于“refClass”(is(x, "refClass"))的S4对象 (isS4(x)) 。pryr::otype()将返回"RC"。RC方法也是S4对象，在类refMethodDef中。

##### 方法分发
在RC中方法分发是容易的，因为方法是和类联系在一起的，不是函数。当你调用
x$f()的时候，R会在类x中寻找方法，然后在它的父类中，再到父类的父类，以此类推。在一个方法的内部，你可以直接用callSuper(...)调用父亲的方法。

#### 选择一个系统
在一门语言中，三种面向对象系统是很多的，但是对于多数的R编程来说，S3已经足够了。在R中你经常可以创建相当简单的对象和方法，针对已经存在的泛型函数，例如print()， summary()和plot()。S3是非常适合这个任务的，而且在我已经写过的R代码中绝大多数面向对象的代码都是S3。S3是有点古怪的，但是它可以用最小的代码完成工作。

如果你要创建有相互联系对象的更加复杂的系统，S4可能会更加合适。一个好的例子就是 Douglas Bates and Martin Maechler写的Matrix的包。它被设计用来高效地储存和计算很多不同类型的稀疏矩阵。在1.1.3版本中，它定义了102个类和20ge泛型函数。这个包写的很好，注释得也很好，并且附带的简介vignette("Intro2Matrix", package = "Matrix")可对这个包的结构给出了很好的介绍。在Bioconductor包中S4也被广泛的使用，需要给不同生物之间复杂的相互关系进行建模。Bioconductor提供了很多学习S4的好的资源。如果你已经掌握了S3，S4是相对容易是掌握的，思想都是一样的，只是更加的正式，更加的严格，和更加的详细。

如果你已经在主流的面向对象语言中编程过，RC将看起来非常自然。但是因为它们通过可变的状态所引起的副作用，将很难去理解。举个例子，当你在R中调用f(a, b)的时候，你可能会假设a和b将不会被修改。但是当a和b是一个RC对象，它们可以被随时的修改。更通俗的讲，当使用RC对象的时候，你需要尽可能地减少副作用，仅仅是可变状态是必须的时候才使用它们。大多数函数仍然应该是功能型的，并且没有副作用。这将使得代码是容易推出的，并且对于其他R语言编程者来说是容易理解的。



































