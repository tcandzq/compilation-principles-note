# compilation-principles-note
编译原理学习笔记

## Let’s Build A Simple Interpreter Part 1.
`编译器( compiler)`或者`解释器(interpreter)`就是将一种高级语言表示的源程序翻译成另一种语言的程序。

- 如果一个翻译器不需要把源程序先翻译成机器语言就能处理并执行它，那么它就是`解释器`

- 如果一个翻译器把一段源程序翻译成机器语言，那么它就是`编译器`。

当我们在命令行中输入表达式3+5，解释器会得到字符串“3+5”。为了让解释器真正明白该如何处理这个字符串，它需要首先将输入的“3+5”分解成称为`tokens`的组件。
- `token`是具有类型和值的对象。例如：字符串“3”就是一个`token`，它的类型是INTEGER，对应的值是3。

- 将输入的字符串分解为`token`的过程称为词法分析`(lexical analysis)`

首先，解释器需要做的第一步是读取输入的字符串，并将其转化为`tokens`流。
- 解释器中实现这个功能的部分称为`词法分析器(lexical analyzer)`或者简称`lexer`。在其他资料中可能称为 `scanner` 或者`tokenizer`。但其实做的都是同一件事，就是将输入的字符串转换为`tokens流`。

[参考](https://ruslanspivak.com/lsbasi-part1/)  


## Let’s Build A Simple Interpreter Part 2.
### 概念

- 由tokens中的字符构成的序列称为语义(lexeme)；
- 挖掘tokens序列中的结构的过程称为

[calc2](../compilation-principles-note/lsbasi/calc2.py)相对于[calc1](../compilation-principles-note/lsbasi/calc1.py)增加了以下功能：
- 处理了输入字符串中的空格；
- 可以接受多个数字的输入，但依然只抽取前面两个数字进行运算；
- 增加了减法运算
  
代码改动如下：
- 对*get_next_token*方法进行了重构。pos指针向后移动的逻辑被advance方法分解了；
- 新增了两个方法*skip_whitespace*用来来忽略空格，*integer*用来处理多个数字的输入；
- *expr*修改后可以识别不仅可以识别INTEGER -> PLUS -> INTEGER，还可以识别INTEGER -> MINUS -> INTEGER。



  


