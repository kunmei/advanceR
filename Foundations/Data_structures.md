## 数据结构
本章节总结在基础R中最重要的数据结构。在这之前你可能使用过它们中的许多，但是你可能还没有深刻地思考它们是如何相互联系的。在这篇简单的概述中，我不会深入讨论单个的类型。反而，我将向你们展示它们作为一个整体是如何组合在一起的。如果你需要更多的细节，你可以在R的help文档中找到它们。

R的基础数据结构可以根据它们的维度以及它们是否是同质的或者是异质的进行分类。这就产生了在数据分析中经常使用到的五种数据类型。

||同质的|非同质的|
|:-:|:-:|:-:|
|一维|原子向量|列表|
|二维|矩阵|数据框|
|多维|数组||

几乎所有替他类型都是在这些基础上建立的。在面向对象指南中，你将看到一些更加复杂的对象是如何通过这些建立的。需要注意的是，R中没有零维和变量类型。单个的数字或者字符串，也许你会认为它们是标量，其实它们是长度为一的向量。给定一个对象，str() 是最好的方法去了解它是由什么数据结构所组成的。str()是structure是缩写，它对任何一个R数据结构给出了简洁的，便于人们阅读的描述。

##### 小测试
做个小测试检验你是否需要阅读该章节。如果你快速知道答案，你可以自然地跳过该章节。在可以在回答部分查看答案。
1. 除了内容， 一个向量的三个特质是什么？
2. 原子向量的四种常见类型是什么？两种不常见的类型是什么？
3. 属性是什么？你如何得到它们并且设置它们？
4. 列表和原子向量的区别是什么？ 数据框和矩阵的区别是什么？
5. 你能得到是一个矩阵的列表吗？ 数据框的某一列能否是一个矩阵？

### 大纲
**Vectors** 向你们介绍原子向量和列表，R中的一维数据结构
**Attributes** 讨论一下属性，R中灵活的元数据规范。这里你讲学习到因子，一种通过对原子向量设定属性而创造的重要的数据结构
**Matrices and arrays** 介绍矩阵和数组，分别储存二维和高维数据的数据结构
**Data frames** 介绍数据框，R中储存数据的最重要的一种数据结构。数据框结构结合了列表和矩阵的行为，很好地满足统计数据的需要

#### 向量（Vectors）
R中最基础的数据结构是向量，向量有两种类型（come in two flavours）：原子向量和列表。它们有三个共同的特性：
- **类型** Typeof()，是什么
- **长度** length()，包含多少个元素
- **属性** attributes()，额外的任意元数据

它们在它们元素的类型上面不同，原子向量的所有元素必须是同一个类型，然而列表的类型可以是不同的类型。
**请注意**：is.vector() 不是验证一个对象是否是向量。仅仅当一个对象是一个除了名字（names）不包含其他属性的向量的时候，它才返回TRUE。用is.atomic(x) || is.list(x)去验证一个对象是否真的是一个向量。

##### 原子向量
我们将详细讨论原子向量中四种常见的类型，分别是：逻辑类型，整数类型，双精度型（通常也叫做数值类型）和字符类型。还有两种罕见的类型我们不做深入地讨论：复数类型（complex）和raw原子向量通常用c()进行创建，combine的缩写：

```
dbl_var <-c(1, 2.5, 4.5)
# With the L suffix, you get an integer rather than a double
int_var <-c(1L, 6L, 10L)
# Use TRUE and FALSE (or T and F) to create logical vectors
log_var <-c(TRUE, FALSE, T, F)
chr_var <-c("these are", "some strings")
```
原子向量总是平的，即使你使用嵌套的c()：

```
c(1, c(2, c(3, 4)))
#> [1] 1 2 3 4# the same asc(1, 2, 3, 4)
#> [1] 1 2 3 4
```
缺失值指定为NA，是一个长度为1的逻辑向量。如果你在c()里面使用NA值， NA值总是被强制转换成正确的类型。或者你可以用NA_real_ （一个双精度向量）、NA_integer_ 和NA_character_.创建指定类型的NA。

##### 类型和测试
给定一个向量，你可以用typeof()查看它的类型，或者用is函数去查看其是否是指定的类型。is.character(), is.double(), is.integer(), is.logical(), 或者更通用的is.atomic().

```
int_var <- c(1L, 6L, 10L)
typeof(int_var)
#> [1] "integer"
is.integer(int_var)
#> [1] TRUE
is.atomic(int_var)
#> [1] TRUE

dbl_var <- c(1, 2.5, 4.5)
typeof(dbl_var)
#> [1] "double"
is.double(dbl_var)
#> [1] TRUE
is.atomic(dbl_var)
#> [1] TRUE
```
**请注意**：is.numeric() 是数值类型vector的通用检验。对double和integer类型的向量都返回TRUE。不是针对double向量的特别检验，其通常被称为数值型。

```
is.numeric(int_var)
#> [1] TRUE
is.numeric(dbl_var)
#> [1] TRUE
```
##### 强制转换
一个原子向量的所有类型必须是同一个类型。因此当你尝试去组合不同的类型，它们将被强制转换成最灵活的类型。类型灵活程度从低到高的排序是：logical, integer, double, and character.
举个例子，组合字符和整数将产生字符。

```
str(c("a", 1))
#>  chr [1:2] "a" "1"
```
当一个逻辑向量被强制转成整型或者双精度型，TRUE变成1， FALSE变成0. 这个结合sum() 和 mean() 非常好用。

```
x <- c(FALSE, FALSE, TRUE)
as.numeric(x)
#> [1] 0 0 1

# Total number of TRUEs
sum(x)
#> [1] 1

# Proportion that are TRUE
mean(x)
#> [1] 0.3333333
```
强制转换通常是自动发生的。大多数数学函数(+, log, abs, etc.)将转成双精度和整型，大多数(&, |, any, etc) 逻辑操作将转成逻辑类型。如果强制转换的过程中丢失了信息，你通常会得到一个警告信息。
为了避免混淆，可以用as.character(), as.double(),as.integer(), or as.logical()进行强制转换。

##### 列表
列表不同于原子向量，是因为它们的元素可以是任意类型，包括一个列表。创建一个列表用list()而不是c()。

```
x <- list(1:3, "a", c(TRUE, FALSE, TRUE), c(2.3, 5.9))
str(x)
#> List of 4
#>  $ : int [1:3] 1 2 3
#>  $ : chr "a"
#>  $ : logi [1:3] TRUE FALSE TRUE
#>  $ : num [1:2] 2.3 5.9
```
列表有时候被称为循环的向量。因为一个列表可以包含其他列表。这个使得它们同原子向量有根本区别。

```
x <- list(list(list(list())))
str(x)
#> List of 1
#>  $ :List of 1
#>   ..$ :List of 1
#>   .. ..$ : list()
is.recursive(x)
#> [1] TRUE
```
c() 可以组合不同的列表到一个。如果将一个原子向量和列表进行组合，c()将在组合它们之前将向量强制转成列表。比较下面list()和c()结果的不同。

```
x <- list(list(1, 2), c(3, 4))
y <- c(list(1, 2), c(3, 4))
str(x)
#> List of 2
#>  $ :List of 2
#>   ..$ : num 1
#>   ..$ : num 2
#>  $ : num [1:2] 3 4
str(y)
#> List of 4
#>  $ : num 1
#>  $ : num 2
#>  $ : num 3
#>  $ : num 4
```
列表的typeof() 是list。可以用is.list()检验一个list，也可以用as.list()强制转换成一个list。你可以用unlist()将一个list转成原子向量。如果一个list中包含不同的类型。unlist()使用同c()相同的转换规则。
在R中很多更加复杂的数据结构是用list进行构建的。举个例子，数据框和线性模型对象（由lm()生成）都是list：

```
is.list(mtcars)
#> [1] TRUE

mod <- lm(mpg ~ wt, data = mtcars)
is.list(mod)
#> [1] TRUE
```
##### 属性
所有的对象都可以有任意的额外的属性,用于储存关于该对象的元信息。属性可以被看做是一个有名字的列表。可以用attr()单独获得属性，或者利用attribute()得到所有属性。

```
y <- 1:10
attr(y, "my_attribute") <- "This is a vector"
attr(y, "my_attribute")
#> [1] "This is a vector"
str(attributes(y))
#> List of 1
#>  $ my_attribute: chr "This is a vector"
```
函数structure()返回一个属性经过修改的新的对象：

```
structure(1:10, my_attribute = "This is a vector")
#>  [1]  1  2  3  4  5  6  7  8  9 10
#> attr(,"my_attribute")
#> [1] "This is a vector"
```
默认的，当修改一个向量的时候，大多数属性将丢失：

```
attributes(y[1])
#> NULL
attributes(sum(y))
#> NULL
```
下面最重要的三种属性将不会丢失：

- **名字** 一个字符向量给每一个变量赋予一个名字，在names进行描述。
- **维度** 被使用将向量转换成矩阵和数组，在matrices and arrays进行描述。
- **类** 被使用去实现S3对象系统，在S3中进行描述。

这些属性都有一个特定的访问函数去得到和设定值。当用到这些属性的时候，使用names(x), dim(x), 和class(x), 而不是attr(x, "names"), attr(x, "dim"), 和attr(x, "class")。

##### 名字
你可以用三种方式对向量进行命名：
- 在创建向量的时候： x <- c(a = 1, b = 2, c = 3)。
- 适当地修改一个已经存在的向量：x <- 1:3; names(x) <- c("a", "b", "c")。
- 创建一个向量的修改备份：x <- setNames(1:3, c("a", "b", "c"))。

名字不是必须是唯一的。然而，字符构造子集，在subsetting中描述的，是最重要的原因去使用名字，当名字是唯一的时候，也是最有用的。

并不是一个向量的所有元素都需要一个名字。如果一些名字丢失了，names()将为这些元素返回空的字符。如果所有的名字都是丢失的，names()将返回NULL值。

```
y <- c(a = 1, 2, 3)
names(y)
#> [1] "a" ""  ""

z <- c(1, 2, 3)
names(z)
#> NULL
```
你可以通过unname(x)去创建一个新的向量，或者用names(x) <- NULL在适当地时候移除名字。

##### 因子
使用属性的一个重要的原因是去定义因子。因子是一个只包含预定义值的向量，被使用去储存类别数据。因子是通过使用两个属性，在整数型向量上建立的：
class(), “factor”, 这个属性使得它们与正规的整数向量表现不同， levels()定义了允许值的集合。

```
x <- factor(c("a", "b", "b", "a"))
x
#> [1] a b b a
#> Levels: a b
class(x)
#> [1] "factor"
levels(x)
#> [1] "a" "b"

# You can't use values that are not in the levels
x[2] <- "c"
#> Warning in `[<-.factor`(`*tmp*`, 2, value = "c"): invalid factor level, NA
#> generated
x
#> [1] a    <NA> b    a   
#> Levels: a b

# NB: you can't combine factors
c(factor("a"), factor("b"))
#> [1] 1 1
```
当你知道一个变量可能出现的值的时候，因子是有用的，即使你看不到在给定全部数据中的所有值。当一些组不包含观察值的时候，用因子去替代字符型向量是明显的。

```
sex_char <- c("m", "m", "m")
sex_factor <- factor(sex_char, levels = c("m", "f"))

table(sex_char)
#> sex_char
#> m 
#> 3
table(sex_factor)
#> sex_factor
#> m f 
#> 3 0
```
有时候当一个数据框是从文件中直接读取的，其中一个列是本来认为应该是数值型的向量，却产生了因子。这是因为列中包含非数值型的值，通常是一个缺失值，被一个特殊的方式表示，例如.或者-。
为了解决这个问题，强制将一个向量从因子转成字符向量，然后再转成双精度型的向量。（这个过程之后，检查一个缺失值）。当然，一个更好的选择是在第一时间发现什么原因导致了这个问题，然后
去修正它。使用read.csv()中的na.strings参数通常是一个好的选择。

```
# Reading in "text" instead of from a file here:
z <- read.csv(text = "value\n12\n1\n.\n9")
typeof(z$value)
#> [1] "integer"
as.double(z$value)
#> [1] 3 2 1 4
# Oops, that's not right: 3 2 1 4 are the levels of a factor, 
# not the values we read in!
class(z$value)
#> [1] "factor"
# We can fix it now:
as.double(as.character(z$value))
#> Warning: NAs introduced by coercion
#> [1] 12  1 NA  9
# Or change how we read it in:
z <- read.csv(text = "value\n12\n1\n.\n9", na.strings=".")
typeof(z$value)
#> [1] "integer"
class(z$value)
#> [1] "integer"
z$value
#> [1] 12  1 NA  9
# Perfect! :)
```
不幸的，在R中大多数数据加载函数都自动地将字符型向量转成因子。这是次优的。因为对于这些函数，没有办法知道集合所有可能的水平或者它们最优的次序。可以使用参数stringsAsFactors = FALSE
去抑制这种行为，然后手动地利用你对数据的理解将字符型向量转成因子。一个全局的操作，options(stringsAsFactors = FALSE)，可以去控制这个行为但是不推荐使用。当和其他代码结合时，改变
这个全局设置可能造成无法预期的结果（来自其他包或者加载了其他代码）。同时全局设置也会造成代码比较难以理解，因为增加了代码的行数，并且你需要去理解一个单独的代码是如何工作的。

尽管因子通常表现起来或看起来像是字符型向量，它们实际上是整数。当把它们看成是字符串的时候要小心。一些字符串函数，例如gsub()和grepl()将把因子强制转换成字符，然而其他（例如nchar()）将
抛出一个错误，还有其他（例如c())将使用潜在的整数值。因为这种原因，如果你需要使用字符一样的行为，通常最好的方式是明确的将因子转换成字符型向量。在R的早期版本，使用因子相对于使用字符型
向量具有内存优势，但是现在不存在了。

##### 矩阵和数组
对一个原子向量添加一个dim()的属性，将使其表现的像一个多维的数组。数组的一个特殊例子是矩阵，其具有两个维度。矩阵通常用作为统计数学机器的一部分。数组更罕见，但是值的注意。

矩阵和数组可以分别通过matrix()和array()进行创建，或者使用dim()的赋值行为。

```
# Two scalar arguments to specify rows and columns
a <- matrix(1:6, ncol = 3, nrow = 2)
# One vector argument to describe all dimensions
b <- array(1:12, c(2, 3, 2))

# You can also modify an object in place by setting dim()
c <- 1:6
dim(c) <- c(3, 2)
c
#>      [,1] [,2]
#> [1,]    1    4
#> [2,]    2    5
#> [3,]    3    6
dim(c) <- c(2, 3)
c
#>      [,1] [,2] [,3]
#> [1,]    1    3    5
#> [2,]    2    4    6
```
length()和names()有高维的扩展：
- length()在矩阵中扩展成nrow()和ncol()，在数组中扩展成dim()
- name()在矩阵中扩展成rownames()和colnames()，在数组中扩展成dimnames()，一个字符向量的列表

```
length(a)
#> [1] 6
nrow(a)
#> [1] 2
ncol(a)
#> [1] 3
rownames(a) <- c("A", "B")
colnames(a) <- c("a", "b", "c")
a
#>   a b c
#> A 1 3 5
#> B 2 4 6

length(b)
#> [1] 12
dim(b)
#> [1] 2 3 2
dimnames(b) <- list(c("one", "two"), c("a", "b", "c"), c("A", "B"))
b
#> , , A
#> 
#>     a b c
#> one 1 3 5
#> two 2 4 6
#> 
#> , , B
#> 
#>     a  b  c
#> one 7  9 11
#> two 8 10 12
```
c()在矩阵中扩展成cbind()和rbind()，在数组中扩展成abind()（由abind包提供）。你可以用t()对矩阵进行转置。对数组来说用的是aperm()。

你可以用is.matrix()和is.array()来测试一个对象是矩阵还是数组，用dim()查看长度。as.matrix()和as.array()可以将一个已经存在的向量
转成矩阵或者数组。

向量并不是唯一的一维数据结构。你可以有一个一行或者一列的矩阵，或者一个维度的数组。它们可能打印出来相似，但表现的不一样。不同并不是很重要，
但当你从一个函数得到一个奇怪的输出的时候，你就知道它们存在的必要了。（tapply()通常会产生这样的输出）。同时，可以使用str()揭示这种不同。

```
str(1:3)                   # 1d vector
#>  int [1:3] 1 2 3
str(matrix(1:3, ncol = 1)) # column vector
#>  int [1:3, 1] 1 2 3
str(matrix(1:3, nrow = 1)) # row vector
#>  int [1, 1:3] 1 2 3
str(array(1:3, 3))         # "array" vector
#>  int [1:3(1d)] 1 2 3
```
当原子向量通常转成向量的时候，维度属性仍然可以加列表上使其变成列表矩阵或者列表数组。

```
l <- list(1:3, "a", TRUE, 1.0)
dim(l) <- c(2, 2)
l
#>      [,1]      [,2]
#> [1,] Integer,3 TRUE
#> [2,] "a"       1
```
这些都是相对复杂的数据结构，但是当你想把对象弄成网格状的结构，将会有用的。举个例子，如果你在一个时空网格上面训练模型，你就可以用一个三维的数组就保存这样一个网格结构的模型。

##### 数据框
数据框是R中存储数据最常用的方式，如果你系统地使用将使得数据分析变得更容易。从内部结构看，数据框是一个等长度向量的列表。这使得它是一个二维的结构，所有它兼具矩阵和列表的特性。
这意味着可以用names(), colnames()和rownames(),尽管names()和colnames()是一回事。数据框的长度是底层的长度，因此ncol()返回数据框的长度；nrow()返回数据框的行数。

在构造子集会描述，你可以构造一个一维或者二维的数据框，一维表现得像一个列表，二维表现得像一个矩阵。

##### 强制转换
你可以用data.frame()创建一个数据框，它是将一个向量作为输入：

```
df <- data.frame(x = 1:3, y = c("a", "b", "c"))
str(df)
#> 'data.frame':    3 obs. of  2 variables:
#>  $ x: int  1 2 3
#>  $ y: Factor w/ 3 levels "a","b","c": 1 2 3
```
注意data.frame()默认将字符串转成因子。使用stringAsFactors = FALSE去抑制这个行为。

```
df <- data.frame(
  x = 1:3,
  y = c("a", "b", "c"),
  stringsAsFactors = FALSE)
str(df)
#> 'data.frame':    3 obs. of  2 variables:
#>  $ x: int  1 2 3
#>  $ y: chr  "a" "b" "c"
```
##### 验证和强制转换
因为数据框是一个S3类型的对象，它的类型反映了创建它的底层向量，即一个列表。检查一个对象是否是数据框，可以使用class()或者精确的用is.data.frame()。

```
typeof(df)
#> [1] "list"
class(df)
#> [1] "data.frame"
is.data.frame(df)
#> [1] TRUE
```
你可以用as.data.frame()强制将一个对象转成数据框。
- 一个向量将变成一个1列的数据框
- 一个列表将把每个元素转成一列，如果它们的长度不相等，将会报错
- 一个数组变成数据框，将保持同样的行和同样的列

##### 链接数据框
你可以使用cbind()和rbind()连接数据框

```
cbind(df, data.frame(z = 3:1))
#>   x y z
#> 1 1 a 3
#> 2 2 b 2
#> 3 3 c 1
rbind(df, data.frame(x = 10, y = "z"))
#>    x y
#> 1  1 a
#> 2  2 b
#> 3  3 c
#> 4 10 z
```
当连接列的时候，行数必须要保持统一，但行的名字可以忽略。当连接行的时候，列的数据和名字都要匹配。可以使用plyr::rbind.fill()将不同长度的列连接起来。

通过用cbind()将向量整合在一起创建数据框通常是错误的。不行是因为cbind()会创建一个矩阵，除非其中一个元素已经是一个数据框。直接用data.frame()进行创建：

```
bad <- data.frame(cbind(a = 1:2, b = c("a", "b")))
str(bad)
#> 'data.frame':    2 obs. of  2 variables:
#>  $ a: Factor w/ 2 levels "1","2": 1 2
#>  $ b: Factor w/ 2 levels "a","b": 1 2
good <- data.frame(a = 1:2, b = c("a", "b"),
  stringsAsFactors = FALSE)
str(good)
#> 'data.frame':    2 obs. of  2 variables:
#>  $ a: int  1 2
#>  $ b: chr  "a" "b"
```
cbind()的转换规则是复杂的，一个最好的方法去避免就是确保所有的数据类型都是一样的。

##### 特殊的列
因为数据框是一个向量的列表，所以对于一个数据框来说，可以有一列是一个列表

```
df <- data.frame(x = 1:3)
df$y <- list(1:2, 1:3, 1:4)
df
#>   x          y
#> 1 1       1, 2
#> 2 2    1, 2, 3
#> 3 3 1, 2, 3, 4
```
然而当一个列传入data.frame(),它尝试将列表中的每一个元素放进自己的列里面，所以无法创建：

```
data.frame(x = 1:3, y = list(1:2, 1:3, 1:4))
#> Error in data.frame(1:2, 1:3, 1:4, check.names = FALSE, stringsAsFactors = TRUE): arguments imply differing number of rows: 2, 3, 4
```
一个变通方案是使用I(),这个函数将使得data.frame()把这个列表当做一个单元：

```
dfl <- data.frame(x = 1:3, y = I(list(1:2, 1:3, 1:4)))
str(dfl)
#> 'data.frame':    3 obs. of  2 variables:
#>  $ x: int  1 2 3
#>  $ y:List of 3
#>   ..$ : int  1 2
#>   ..$ : int  1 2 3
#>   ..$ : int  1 2 3 4
#>   ..- attr(*, "class")= chr "AsIs"
dfl[2, "y"]
#> [[1]]
#> [1] 1 2 3
```
I()函数添加了一个AsIs的类到它的输入，但是这个通常可以忽略掉。

类似的，数据框的一列也可以是一个矩阵或者数组，只要其行数和数据框可以匹配。

```
dfm <- data.frame(x = 1:3, y = I(matrix(1:9, nrow = 3)))
str(dfm)
#> 'data.frame':    3 obs. of  2 variables:
#>  $ x: int  1 2 3
#>  $ y: 'AsIs' int [1:3, 1:3] 1 2 3 4 5 6 7 8 9
dfm[2, "y"]
#>      [,1] [,2] [,3]
#> [1,]    2    5    8
```
使用列表和数组列的时候要谨慎：因为很多操作数据框的函数假设数据框的所有列是一个原子向量。




























