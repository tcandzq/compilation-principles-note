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

- 由tokens中的字符构成的序列称为`语义(lexeme)`；
- 分析tokens序列结构的过程，换句话说在tokens序列中识别短语(phrase)的过程称为`解析(parsing)`，解释器或者编译器中执行这个过程的部分称为`解析器(parser)`。

例如，[calc2](../compilation-principles-note/lsbasi/calc2.py)的expr函数通过 *get_next_token* 方法来分析输入字符串的结构，判断是什么类型的`短语(phase)`,例如是加法还是减法，然后解释已经识别的短语，生成算数表达式的结果。

> 所以，*expr*是解释器的一部分，解析(parsing)和解释(interpreting )都在这里发生。

[calc2](../compilation-principles-note/lsbasi/calc2.py)相对于[calc1](../compilation-principles-note/lsbasi/calc1.py)增加了以下功能：
- 处理了输入字符串中的空格；
- 可以接受多个数字的输入，但依然只抽取前面两个数字进行运算；
- 增加了减法运算
  
代码改动如下：
- 对*get_next_token*方法进行了重构。pos指针向后移动的逻辑被advance方法分解了；
- 新增了两个方法*skip_whitespace*用来来忽略空格，*integer*用来处理多个数字的输入；
- *expr*修改后可以识别不仅可以识别INTEGER -> PLUS -> INTEGER，还可以识别INTEGER -> MINUS -> INTEGER。

[参考](https://ruslanspivak.com/lsbasi-part2/)

  

## Let’s Build A Simple Interpreter Part 3.

[calc3](lsbasi/calc3.py)新增解析和运行含有任意个数量的加号或者减号的算术表达式，例如：“7 - 3 + 2 - 1”。

`语法分析图(syntax diagram)`是某种编程语言语法规则的图形展示；
语法分析图的主要有两个目的：
- 图形化的表示一种编程语言的规范
- 用来帮助我们编写解析器，可以通过简单的规则将关系图映射到代码中。

我们已经知道在tokens流中识别短语的过程称为**parsing**。解释器或者编译器执行这个工作的部分称为解析器(**parser**)。解析也称为语法分析(**syntax analysis**)，因此语法解析器也可以恰当的被称为语法分析器(**syntax analyzer**)。
[clac3](lsbasi/calc3.py)中的解析器代码如下：
```python
def term(self):
    self.eat(INTEGER)

def expr(self):
    # set current token to the first token taken from the input
    self.current_token = self.get_next_token()

    self.term()
    while self.current_token.type in (PLUS, MINUS):
        token = self.current_token
        if token.type == PLUS:
            self.eat(PLUS)
            self.term()
        elif token.type == MINUS:
            self.eat(MINUS)
            self.term()
```
解析器本身不解释任何东西，如果它识别了一个表达式，它就没有任何异常；如果没有识别，就会抛出语法异常。
由于解析器需要计算表达式的值，因此对`term`方法进行了修改，使其返回一个整数值。同时对expr方法进行了修改，使其在适当的位置执行加法和减法，并返回解释结果：
```python
def term(self):
    """Return an INTEGER token value"""
    token = self.current_token
    self.eat(INTEGER)
    return token.value

def expr(self):
    """Parser / Interpreter """
    # set current token to the first token taken from the input
    self.current_token = self.get_next_token()

    result = self.term()
    while self.current_token.type in (PLUS, MINUS):
        token = self.current_token
        if token.type == PLUS:
            self.eat(PLUS)
            result = result + self.term()
        elif token.type == MINUS:
            self.eat(MINUS)
            result = result - self.term()

    return result

```

## Let’s Build A Simple Interpreter. Part 4.
上下文无关语法context-free grammars (简称grammars)或者**BNF**(Backus-Naur Form)
1. grammar以简洁的方式声明了编程语言的语法。与语法图不同，grammars非常紧凑；
2. grammar可以作为很好的文档；
3. 即使您从头开始手动编写解析器，grammar也是一个很好的起点。通常，您可以通过遵循一组简单的规则将语法转换为代码。
4. 解析器生成器(parser generators)接受grammar作为输入，并根据grammar自动生成解析器。


grammer由一系列规则(*rules*)组成，也称为结果(*productions*)。

规则由非终端符(*non-terminal*)组成，称为一个*production*的**head**或者 **left-hand side** 


## Let’s Build A Simple Interpreter. Part 5.


## Let’s Build A Simple Interpreter. Part 6.