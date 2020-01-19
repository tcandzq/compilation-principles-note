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
上下文无关语法**context-free grammars** (简称**grammars**)或者**BNF**(Backus-Naur Form)
1. **grammar**以简洁的方式声明了编程语言的语法。与语法图不同，**grammars**的表达非常紧凑；
2. grammar可以作为很好的文档；
3. 即使您从头开始手动编写解析器，**grammar**也是一个很好的起点。通常，您可以通过遵循一组简单的规则将**grammar**转换为代码。
4. 解析器生成器(**parser generators**)接受**grammar**作为输入，并根据**grammar**自动生成解析器。

下图就是一个算术表达式“7 * 4 / 2 * 3”的grammar描述：
<div align=center>
<img src="../compilation-principles-note/lsbasi/assert/lsbasi_part4_bnf1.png"/>
</div>

**grammer**由一系列规则(**rules**)组成，也称为结果(**productions**)。下图的**grammar**有两个**rules**。

<div align=center>
<img src="../compilation-principles-note/lsbasi/assert/lsbasi_part4_bnf2.png"/>
</div>


规则*non-terminal*称为*production*的**head**或者 **left-hand side** 、分号和一系列*terminals*或者*non-terminals*组成，它们称为*productio*n的**body**或者**right-hand side**。

<div align=center>
<img src="../compilation-principles-note/lsbasi/assert/lsbasi_part4_bnf3.png"/>
</div>

在上图中grammar ，像MUL, DIV, 和 INTEGER这样的tokens 称为**terminals**，像*expr* 和*factor* 这样的变量称为**non-terminals**。**Non-terminals** 通常由一系列**terminals** 或者**non-terminals**混合组成：

<div align=center>
<img src="../compilation-principles-note/lsbasi/assert/lsbasi_part4_bnf4.png"/>
</div>

第一条*rule*左边的*non-terminal*符号称为**start symbol**。在我们*grammar*的例子中，*start symbol*是*expr*。
<div align=center>
<img src="../compilation-principles-note/lsbasi/assert/lsbasi_part4_bnf5.png"/>
</div>

你可以将上述的*rule expr*读作："expr是一个可选的factor，后面跟着一个乘法或除法运算符，后面可以跟着另一个factor，然后后面跟着一个乘法或除法运算符，后面跟着另一个factor，依此类推"

在本文中*factor*只是一个整数。

如果从*grammer* 得到某个算术表达式，那么它就不支持该表达式，解析器在试图识别该表达式的时候将生成语法错误。

下图展示了grammer得到表达式"3"：
<div align=center>
<img src="../compilation-principles-note/lsbasi/assert/lsbasi_part4_derive1.png"/>
</div>

下图展示了grammer得到表达式3 * 7：
<div align=center>
<img src="../compilation-principles-note/lsbasi/assert/lsbasi_part4_derive2.png"/>
</div>

下图展示了grammer得到表达式 3 * 7 / 2：
<div align=center>
<img src="../compilation-principles-note/lsbasi/assert/lsbasi_part4_derive3.png"/>
</div>


现在遵循下面这些grammar 转为源代码的指南，我们能够逐字的将grammer转换为可以运行的解析器。
- grammer中定义的每一条规则**R**成为一个同名的方法，对该规则的引用成为方法的调用**R():**。
  方法体内遵循和规则体内一样的指导原则；
- 可选符号(a1 | a2 | aN) 成为 ***if-elif-else*** 语句；
- 可选分组符号(...)*成为**while**语句，可以循环多次；
- 对token **T** 的引用变成对方法`eat.eat(T)`的调用。eat方法的运行方式是，如果token T与当前的*lookahead* token一致，就使用T，然后从lexer获取一个新的token，并将新Token赋值给内部变量*current_token*。

以上指南的图形展示如下：
<div align=center>
<img src="../compilation-principles-note/lsbasi/assert/lsbasi_part4_rules.png"/>
</div>

现在按照上面的指南把下面的grammer转换为代码。
<div align=center>
<img src="../compilation-principles-note/lsbasi/assert/lsbasi_part4_bnf1.png"/>
</div>

grammer中有两条rules：一个是expr rule，另一个是factor rule。以factor rule(也称为production)开始。根据指南1，需要创建一个名为*factor*的方法，该方法只有一个对eat方法的调用来使用INTEGER token(根据指南4)。
```python
def factor(self):
    self.eat(INTEGER)
```
rule *expr*成为*expr*方法(根据指南1)。rule 内部以对*factor*的引用开始，该引用变成对方法factor()的调用。可选分祖符(...)*变成while循环，*(MUL | DIV)*变成 *if-elif-else*语句。把这些片段组合起来就可以得到如下的 *expr*方法：
```python
def expr(self):
    self.factor()

    while self.current_token.type in (MUL, DIV):
        token = self.current_token
        if token.type == MUL:
            self.eat(MUL)
            self.factor()
        elif token.type == DIV:
            self.eat(DIV)
            self.factor()
```

这是同一个*expr* rule的语法分析图：
<div align=center>
<img src="../compilation-principles-note/lsbasi/assert/lsbasi_part4_sd.png"/>
</div>


[calc4](lsbasi/calc4.py)可以处理包含整数和任意数量乘法、除法的算术表达式。它把词法分析器( lexical analyzer )重构为了一个单独的类Lexer，并更新了类Interpreter，该类以Lexer的实例为入参。



## Let’s Build A Simple Interpreter. Part 5.


## Let’s Build A Simple Interpreter. Part 6.

## Let’s Build A Simple Interpreter. Part 7: Abstract Syntax Trees

## Let’s Build A Simple Interpreter. Part 8.

## Let’s Build A Simple Interpreter. Part 9.

## Let’s Build A Simple Interpreter. Part 10.

## Let’s Build A Simple Interpreter. Part 11.

## Let’s Build A Simple Interpreter. Part 12.

## Let’s Build A Simple Interpreter. Part 13: Semantic Analysis.

## Let’s Build A Simple Interpreter. Part 14: Nested Scopes and a Source-to-Source Compiler.

## Let’s Build A Simple Interpreter. Part 15.

## Let’s Build A Simple Interpreter. Part 16: Recognizing Procedure Calls

## Let’s Build A Simple Interpreter. Part 17: Call Stack and Activation Records