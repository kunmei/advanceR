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
流失预警










