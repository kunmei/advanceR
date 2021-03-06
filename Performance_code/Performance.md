### Performance
R不是一个快速的语言。这不是偶然。R是为数据分析和统计而设计的。它不是为了电脑更容易地操作而设计的。尽管R相对于其他一些编程语言
来说是慢，但对于大多数目的，它是足够的。

本章的目的是给你对R的性能特点有一个更加深刻的理解。在本章节，你会学到R在灵活性和性能之间所做的一些平衡。下面的章节将向你展示
提升你代码速度的一些技巧。


##### 语言性能
在本小节，我将探索三种限制R语言性能的权衡:及其动态，在可变环境中查找名字，函数参数的延迟计算。我将用microbenchmark去揭示每一个
权衡，展示它是如何使GNU-R变慢的。我以GNU-R作为基准，因为不能以R语言作为基准（它不执行代码）。这意味着结果只是暗示这些设计决策的
成果，但是是有用的。我已经选择了三个例子去揭示这些权衡，这些是语言设计的关键:设计者必须平衡速度，灵活性和易实现。

如果你想更多地学习R语言的性能特性，以及它们是如何影响真实的代码的，我强烈建议Floreal Morandat, Brandon Hill, Leo Osvald, and Jan Vitek
写的“Evaluating the Design of the R Language” 。它使用了一个强大的方法论将一个修改的R解释器和在野外发现的代码集合整合在一起。

###### 极其动态
R是一个极其动态的编程语言。几乎所有的对象在对创建之后都可以被修改。下面列举一些例子，你可以:
- 改变函数体，函数参数和函数的环境。
- 改变一个泛型的s4方法。
- 向一个S3的对象添加新的属性，甚至是改变它的类
- 在本地环境的外面使用<<-修改对象

可能你唯一不能改变的就是在封闭命名空间中的对象，当你加载一个包时，它被创建了。

动态性的优势是你需要最小的前期规划。你可以随时修改你的注意，不需要重新开始去迭代你的解决方案。动态性的缺点是很难去预测给定一个函数调用，
将会发生什么。这是一个问题，因为越容易预测接下来将要发生，对于一个解释器或者编译器来说越容易做出一个优化。（如果你喜欢更多的细节，Charles Nutter
在On Languages, VMs, Optimization, and the Way of the World会扩展这个主意。如果一个解释器不能预测接下来将要发生什么，在找到正确的
一个之前需要考虑很多选择。举个例子，在R中下面的循环是慢的，因为Ｒ不知道ｘ总是一个整数。这意味着Ｒ在每一次迭代循环的时候总是去寻找正确的+方法。

```
x <- 0L
for (i in 1:1e6) {
  x <- x + 1
}
```
寻找正确方法的代价对于非原语函数是更高的。下面的微基准揭示了对于S3, S4, and RC方法分发的代价。对于每一个面向对象的系统，我都创建了一个泛型
和一个方法，然后调用这个泛型去看寻找和调用方法将会话费多长时间。我还统计了调用裸函数话费了多长时间，作为比较。
