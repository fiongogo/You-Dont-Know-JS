# You Don't Know JS: Scope & Closures
# Chapter 1: What is Scope?
第 1 章 作用域是什么

One of the most fundamental paradigms of nearly all programming languages is the ability to store values in variables, and later retrieve or modify those values. In fact, the ability to store values and pull values out of variables is what gives a program *state*.

几乎所有编程语言最基本的功能之一，就是能够储存变量当中的值，并且能在之后对这个值进行访问或修改。事实上，正是这种储存和访问变量的值的能力将状态带给了程序。

Without such a concept, a program could perform some tasks, but they would be extremely limited and not terribly interesting.
若没有了状态这个概念，程序虽然也能够执行一些简单的任务，但它会受到高度限制，做不到非常有趣。

But the inclusion of variables into our program begets the most interesting questions we will now address: where do those variables *live*? In other words, where are they stored? And, most importantly, how does our program find them when it needs them?
但是将变量引入程序会引起几个很有意思的问题，也正是我们将要讨论的：这些变量住在哪里？换句话说，它们储存在哪里？最重要的是，程序需要时如何找到它们？

These questions speak to the need for a well-defined set of rules for storing variables in some location, and for finding those variables at a later time. We'll call that set of rules: *Scope*.
这些问题说明需要一套设计良好的规则来存储变量，并且之后可以方便地找到这些变量。这套规则被称为作用域。

But, where and how do these *Scope* rules get set?
但是，究竟在哪里而且怎样设置这些作用域的规则呢？

## Compiler Theory 编译原理

It may be self-evident, or it may be surprising, depending on your level of interaction with various languages, but despite the fact that JavaScript falls under the general category of "dynamic" or "interpreted" languages, it is in fact a compiled language. It is *not* compiled well in advance, as are many traditionally-compiled languages, nor are the results of compilation portable among various distributed systems.
尽管通常将JavaScript归类为“动态”或“解释执行”语言，但事实上它是一门编译语言。这个事实对你来说可能显而易见，也可能你闻所未闻，取决于你接触过多少编程语言，具有多少经验。但与传统的编译语言不同，它不是提前编译的，编译结果也不能在分布式系统中进行移植。

But, nevertheless, the JavaScript engine performs many of the same steps, albeit in more sophisticated ways than we may commonly be aware, of any traditional language-compiler.
尽管如此，JavaScript引擎进行编译的步骤和传统的编译语言非常相似，在某些环节可能比预想的要复杂。

In traditional compiled-language process, a chunk of source code, your program, will undergo typically three steps *before* it is executed, roughly called "compilation":
在传统编译语言的流程中，程序中的一段源代码在执行之前会经历三个步骤，统称为“编译”。

1. **Tokenizing/Lexing:** breaking up a string of characters into meaningful (to the language) chunks, called tokens. For instance, consider the program: `var a = 2;`. This program would likely be broken up into the following tokens: `var`, `a`, `=`, `2`, and `;`. Whitespace may or may not be persisted as a token, depending on whether it's meaningful or not.
分词/词法分析（Tokenizing/Lexing）
这个过程会将由字符组成的字符串分解成（对编程语言来说）有意义的代码块，这些代码块被
称为词法单元（token）。例如，考虑程序var a = 2;。这段程序通常会被分解成为下面这些词法
单元：var、a、=、2 、;。空格是否会被当作词法单元，取决于空格在这门语言中是否具有意义。

    **Note:** The difference between tokenizing and lexing is subtle and academic, but it centers on whether or not these tokens are identified in a *stateless* or *stateful* way. Put simply, if the tokenizer were to invoke stateful parsing rules to figure out whether `a` should be considered a distinct token or just part of another token, *that* would be **lexing**.
    分词（tokenizing）和词法分析（Lexing）之间的区别是非常微妙、晦涩的，主要差异在于词法单元的识别是通过有状态还是无状态的方式进行的。简单来说，如果词法单元生成器在判断a是一个独立的词法单元还是其他词法单元的一部分时，调用的是有状态的解析规则，那么这个过程就被称为词法分析。

2. **Parsing:** taking a stream (array) of tokens and turning it into a tree of nested elements, which collectively represent the grammatical structure of the program. This tree is called an "AST" (<b>A</b>bstract <b>S</b>yntax <b>T</b>ree).
解析/语法分析（Parsing）这个过程是将词法单元流（数组）转换成一个由元素逐级嵌套所组成的代表了程序语法结构的
树。这个树被称为“抽象语法树”（Abstract Syntax Tree，AST）。

    The tree for `var a = 2;` might start with a top-level node called `VariableDeclaration`, with a child node called `Identifier` (whose value is `a`), and another child called `AssignmentExpression` which itself has a child called `NumericLiteral` (whose value is `2`).var a = 2;的抽象语法树中可能会有一个叫作VariableDeclaration的顶级节点，接下来是一个叫作Identifier（它的值是a）的子节点，以及一个叫作AssignmentExpression的子节点。AssignmentExpression节点有一个叫作NumericLiteral（它的值是2）的子节点。

3. **Code-Generation:** the process of taking an AST and turning it into executable code. This part varies greatly depending on the language, the platform it's targeting, etc.
代码生成将AST转换为可执行代码的过程称被称为代码生成。这个过程与语言、目标平台等息息相关。

    So, rather than get mired in details, we'll just handwave and say that there's a way to take our above described AST for `var a = 2;` and turn it into a set of machine instructions to actually *create* a variable called `a` (including reserving memory, etc.), and then store a value into `a`.抛开具体细节，简单来说就是有某种方法可以将var a = 2;的AST转化为一组机器指令，用来创建一个叫作a的变量（包括分配内存等），并将一个值储存在a中。

    **Note:** The details of how the engine manages system resources are deeper than we will dig, so we'll just take it for granted that the engine is able to create and store variables as needed.
    关于引擎如何管理系统资源超出了我们的讨论范围，因此只需要简单地了解引擎可以根据需要创建并储存变量即可。

The JavaScript engine is vastly more complex than *just* those three steps, as are most other language compilers. For instance, in the process of parsing and code-generation, there are certainly steps to optimize the performance of the execution, including collapsing redundant elements, etc.比起那些编译过程只有三个步骤的语言的编译器，JavaScript引擎要复杂得多。例如，在语法分析
和代码生成阶段有特定的步骤来对运行性能进行优化，包括对冗余元素进行优化等。

So, I'm painting only with broad strokes here. But I think you'll see shortly why *these* details we *do* cover, even at a high level, are relevant.因此在这里只进行宏观、简单的介绍，接下来你就会发现我们介绍的这些看起来有点高深的内容
与所要讨论的事情有什么关联。

For one thing, JavaScript engines don't get the luxury (like other language compilers) of having plenty of time to optimize, because JavaScript compilation doesn't happen in a build step ahead of time, as with other languages.首先，JavaScript引擎不会有大量的（像其他语言编译器那么多的）时间用来进行优化，因为与其他
语言不同，JavaScript的编译过程不是发生在构建之前的。

For JavaScript, the compilation that occurs happens, in many cases, mere microseconds (or less!) before the code is executed. To ensure the fastest performance, JS engines use all kinds of tricks (like JITs, which lazy compile and even hot re-compile, etc.) which are well beyond the "scope" of our discussion here.
对于JavaScript来说，大部分情况下编译发生在代码执行前的几微秒（甚至更短！）的时间内。在我
们所要讨论的作用域背后，JavaScript引擎用尽了各种办法（比如JIT，可以延迟编译甚至实施重编
译）来保证性能最佳。

Let's just say, for simplicity's sake, that any snippet of JavaScript has to be compiled before (usually *right* before!) it's executed. So, the JS compiler will take the program `var a = 2;` and compile it *first*, and then be ready to execute it, usually right away.
简单地说，任何JavaScript代码片段在执行前都要进行编译（通常就在执行前）。因此，JavaScript编
译器首先会对var a = 2;这段程序进行编译，然后做好执行它的准备，并且通常马上就会执行它。

## Understanding Scope理解作用域

The way we will approach learning about scope is to think of the process in terms of a conversation. But, *who* is having the conversation?我们学习作用域的方式是将这个过程模拟成几个人物之间的对话。那么，由谁进行这场对话呢？

### The Cast演员表

Let's meet the cast of characters that interact to process the program `var a = 2;`, so we understand their conversations that we'll listen in on shortly:首先介绍将要参与到对程序var a = 2;进行处理的过程中的演员们，这样才能理解接下来将要听
到的对话。

1. *Engine*: responsible for start-to-finish compilation and execution of our JavaScript program.引擎 从头到尾负责整个JavaScript程序的编译及执行过程。

2. *Compiler*: one of *Engine*'s friends; handles all the dirty work of parsing and code-generation (see previous section).编译器
引擎的好朋友之一，负责语法分析及代码生成等脏活累活（详见前一节的内容）。

3. *Scope*: another friend of *Engine*; collects and maintains a look-up list of all the declared identifiers (variables), and enforces a strict set of rules as to how these are accessible to currently executing code.作用域 引擎的另一位好朋友，负责收集并维护由所有声明的标识符（变量）组成的一系列查询，并实施
一套非常严格的规则，确定当前执行的代码对这些标识符的访问权限。

For you to *fully understand* how JavaScript works, you need to begin to *think* like *Engine* (and friends) think, ask the questions they ask, and answer those questions the same.
为了能够完全理解JavaScript的工作原理，你需要开始像引擎（和它的朋友们）一样思考，从它们的
角度提出问题，并从它们的角度回答这些问题。

### Back & Forth 对话

When you see the program `var a = 2;`, you most likely think of that as one statement. But that's not how our new friend *Engine* sees it. In fact, *Engine* sees two distinct statements, one which *Compiler* will handle during compilation, and one which *Engine* will handle during execution.当你看见var a = 2;这段程序时，很可能认为这是一句声明。但我们的新朋友引擎却不这么看。事
实上，引擎认为这里有两个完全不同的声明，一个由编译器在编译时处理，另一个则由引擎在运行
时处理。

So, let's break down how *Engine* and friends will approach the program `var a = 2;`.下面我们将var a = 2;分解，看看引擎和它的朋友们是如何协同工作的。

The first thing *Compiler* will do with this program is perform lexing to break it down into tokens, which it will then parse into a tree. But when *Compiler* gets to code-generation, it will treat this program somewhat differently than perhaps assumed.编译器首先会将这段程序分解成词法单元，然后将词法单元解析成一个树结构。但是当编译器开
始进行代码生成时，它对这段程序的处理方式会和预期的有所不同。

A reasonable assumption would be that *Compiler* will produce code that could be summed up by this pseudo-code: "Allocate memory for a variable, label it `a`, then stick the value `2` into that variable." Unfortunately, that's not quite accurate.可以合理地假设编译器所产生的代码能够用下面的伪代码进行概括：“为一个变量分配内存，将其
命名为a，然后将值2保存进这个变量。”然而，这并不完全正确。

*Compiler* will instead proceed as:事实上编译器会进行如下处理。

1. Encountering `var a`, *Compiler* asks *Scope* to see if a variable `a` already exists for that particular scope collection. If so, *Compiler* ignores this declaration and moves on. Otherwise, *Compiler* asks *Scope* to declare a new variable called `a` for that scope collection.1. 遇到var a，编译器会询问作用域是否已经有一个该名称的变量存在于同一个作用域的集合中。
如果是，编译器会忽略该声明，继续进行编译；否则它会要求作用域在当前作用域的集合中声明一
个新的变量，并命名为a。

2. *Compiler* then produces code for *Engine* to later execute, to handle the `a = 2` assignment. The code *Engine* runs will first ask *Scope* if there is a variable called `a` accessible in the current scope collection. If so, *Engine* uses that variable. If not, *Engine* looks *elsewhere* (see nested *Scope* section below).接下来编译器会为引擎生成运行时所需的代码，这些代码被用来处理a = 2这个赋值操作。引擎运行时会首先询问作用域，在当前的作用域集合中是否存在一个叫作a的变量。如果否，引擎就会使用这个变量；如果不是，引擎会继续查找该变量（查看1.3节）。

If *Engine* eventually finds a variable, it assigns the value `2` to it. If not, *Engine* will raise its hand and yell out an error!
如果引擎最终找到了a变量，就会将2赋值给它。否则引擎就会举手示意并抛出一个异常！

To summarize: two distinct actions are taken for a variable assignment: First, *Compiler* declares a variable (if not previously declared in the current scope), and second, when executing, *Engine* looks up the variable in *Scope* and assigns to it, if found.
总结：变量的赋值操作会执行两个动作，首先编译器会在当前作用域中声明一个变量（如果之前没
有声明过），然后在运行时引擎会在作用域中查找该变量，如果能够找到就会对它赋值。

### Compiler Speak编译器有话说

We need a little bit more compiler terminology to proceed further with understanding.为了进一步理解，我们需要多介绍一点编译器的术语。

When *Engine* executes the code that *Compiler* produced for step (2), it has to look-up the variable `a` to see if it has been declared, and this look-up is consulting *Scope*. But the type of look-up *Engine* performs affects the outcome of the look-up.编译器在编译过程的第二步中生成了代码，引擎执行它时，会通过查找变量a来判断它是否已声明
过。查找的过程由作用域进行协助，但是引擎执行怎样的查找，会影响最终的查找结果。

In our case, it is said that *Engine* would be performing an "LHS" look-up for the variable `a`. The other type of look-up is called "RHS".在我们的例子中，引擎会为变量a进行LHS查询。另外一个查找的类型叫作RHS。

I bet you can guess what the "L" and "R" mean. These terms stand for "Left-hand Side" and "Right-hand Side".我打赌你一定能猜到“L”和“R”的含义，它们分别代表左侧和右侧。

Side... of what? **Of an assignment operation.**什么东西的左侧和右侧？是一个赋值操作的左侧和右侧。

In other words, an LHS look-up is done when a variable appears on the left-hand side of an assignment operation, and an RHS look-up is done when a variable appears on the right-hand side of an assignment operation.
换句话说，当变量出现在赋值操作的左侧时进行LHS查询，出现在右侧时进行RHS查询。


Actually, let's be a little more precise. An RHS look-up is indistinguishable, for our purposes, from simply a look-up of the value of some variable, whereas the LHS look-up is trying to find the variable container itself, so that it can assign. In this way, RHS doesn't *really* mean "right-hand side of an assignment" per se, it just, more accurately, means "not left-hand side".
讲得更准确一点，RHS查询与简单地查找某个变量的值别无二致，而LHS查询则是试图找到变量
的容器本身，从而可以对其赋值。从这个角度说，RHS并不是真正意义上的“赋值操作的右侧”，更准
确地说是“非左侧”。

Being slightly glib for a moment, you could also think "RHS" instead means "retrieve his/her source (value)", implying that RHS means "go get the value of...".你可以将RHS理解成retrieve his source value（取到它的源值），这意味着“得到某某的值”。



Let's dig into that deeper.

When I say:
让我们继续深入研究。
考虑以下代码：

```js
console.log( a );
```

The reference to `a` is an RHS reference, because nothing is being assigned to `a` here. Instead, we're looking-up to retrieve the value of `a`, so that the value can be passed to `console.log(..)`.
其中对a的引用是一个RHS引用，因为这里a并没有赋予任何值。相应地，需要查找并取得a的值，这
样才能将值传递给console.log(..)。


By contrast:
相比之下，例如：

```js
a = 2;
```

The reference to `a` here is an LHS reference, because we don't actually care what the current value is, we simply want to find the variable as a target for the `= 2` assignment operation.
这里对a的引用则是LHS引用，因为实际上我们并不关心当前的值是什么，只是想要为= 2这个赋
值操作找到一个目标。

**Note:** LHS and RHS meaning "left/right-hand side of an assignment" doesn't necessarily literally mean "left/right side of the `=` assignment operator". There are several other ways that assignments happen, and so it's better to conceptually think about it as: "who's the target of the assignment (LHS)" and "who's the source of the assignment (RHS)".
LHS和RHS的含义是“赋值操作的左侧或右侧”并不一定意味着就是“=赋值操作符的左侧
或右侧”。赋值操作还有其他几种形式，因此在概念上最好将其理解为“赋值操作的目标是谁
（LHS）”以及“谁是赋值操作的源头（RHS）”。

Consider this program, which has both LHS and RHS references:

```js
function foo(a) {
	console.log( a ); // 2
}

foo( 2 );
```

The last line that invokes `foo(..)` as a function call requires an RHS reference to `foo`, meaning, "go look-up the value of `foo`, and give it to me." Moreover, `(..)` means the value of `foo` should be executed, so it'd better actually be a function!

There's a subtle but important assignment here. **Did you spot it?**

You may have missed the implied `a = 2` in this code snippet. It happens when the value `2` is passed as an argument to the `foo(..)` function, in which case the `2` value is **assigned** to the parameter `a`. To (implicitly) assign to parameter `a`, an LHS look-up is performed.

There's also an RHS reference for the value of `a`, and that resulting value is passed to `console.log(..)`. `console.log(..)` needs a reference to execute. It's an RHS look-up for the `console` object, then a property-resolution occurs to see if it has a method called `log`.

Finally, we can conceptualize that there's an LHS/RHS exchange of passing the value `2` (by way of variable `a`'s RHS look-up) into `log(..)`. Inside of the native implementation of `log(..)`, we can assume it has parameters, the first of which (perhaps called `arg1`) has an LHS reference look-up, before assigning `2` to it.

**Note:** You might be tempted to conceptualize the function declaration `function foo(a) {...` as a normal variable declaration and assignment, such as `var foo` and `foo = function(a){...`. In so doing, it would be tempting to think of this function declaration as involving an LHS look-up.

However, the subtle but important difference is that *Compiler* handles both the declaration and the value definition during code-generation, such that when *Engine* is executing code, there's no processing necessary to "assign" a function value to `foo`. Thus, it's not really appropriate to think of a function declaration as an LHS look-up assignment in the way we're discussing them here.

### Engine/Scope Conversation

```js
function foo(a) {
	console.log( a ); // 2
}

foo( 2 );
```

Let's imagine the above exchange (which processes this code snippet) as a conversation. The conversation would go a little something like this:

> ***Engine***: Hey *Scope*, I have an RHS reference for `foo`. Ever heard of it?

> ***Scope***: Why yes, I have. *Compiler* declared it just a second ago. He's a function. Here you go.

> ***Engine***: Great, thanks! OK, I'm executing `foo`.

> ***Engine***: Hey, *Scope*, I've got an LHS reference for `a`, ever heard of it?

> ***Scope***: Why yes, I have. *Compiler* declared it as a formal parameter to `foo` just recently. Here you go.

> ***Engine***: Helpful as always, *Scope*. Thanks again. Now, time to assign `2` to `a`.

> ***Engine***: Hey, *Scope*, sorry to bother you again. I need an RHS look-up for `console`. Ever heard of it?

> ***Scope***: No problem, *Engine*, this is what I do all day. Yes, I've got `console`. He's built-in. Here ya go.

> ***Engine***: Perfect. Looking up `log(..)`. OK, great, it's a function.

> ***Engine***: Yo, *Scope*. Can you help me out with an RHS reference to `a`. I think I remember it, but just want to double-check.

> ***Scope***: You're right, *Engine*. Same guy, hasn't changed. Here ya go.

> ***Engine***: Cool. Passing the value of `a`, which is `2`, into `log(..)`.

> ...

### Quiz

Check your understanding so far. Make sure to play the part of *Engine* and have a "conversation" with the *Scope*:

```js
function foo(a) {
	var b = a;
	return a + b;
}

var c = foo( 2 );
```

1. Identify all the LHS look-ups (there are 3!).

2. Identify all the RHS look-ups (there are 4!).

**Note:** See the chapter review for the quiz answers!

## Nested Scope

We said that *Scope* is a set of rules for looking up variables by their identifier name. There's usually more than one *Scope* to consider, however.

Just as a block or function is nested inside another block or function, scopes are nested inside other scopes. So, if a variable cannot be found in the immediate scope, *Engine* consults the next outer containing scope, continuing until found or until the outermost (aka, global) scope has been reached.

Consider:

```js
function foo(a) {
	console.log( a + b );
}

var b = 2;

foo( 2 ); // 4
```

The RHS reference for `b` cannot be resolved inside the function `foo`, but it can be resolved in the *Scope* surrounding it (in this case, the global).

So, revisiting the conversations between *Engine* and *Scope*, we'd overhear:

> ***Engine***: "Hey, *Scope* of `foo`, ever heard of `b`? Got an RHS reference for it."

> ***Scope***: "Nope, never heard of it. Go fish."

> ***Engine***: "Hey, *Scope* outside of `foo`, oh you're the global *Scope*, ok cool. Ever heard of `b`? Got an RHS reference for it."

> ***Scope***: "Yep, sure have. Here ya go."

The simple rules for traversing nested *Scope*: *Engine* starts at the currently executing *Scope*, looks for the variable there, then if not found, keeps going up one level, and so on. If the outermost global scope is reached, the search stops, whether it finds the variable or not.

### Building on Metaphors

To visualize the process of nested *Scope* resolution, I want you to think of this tall building.

<img src="fig1.png" width="250">

The building represents our program's nested *Scope* rule set. The first floor of the building represents your currently executing *Scope*, wherever you are. The top level of the building is the global *Scope*.

You resolve LHS and RHS references by looking on your current floor, and if you don't find it, taking the elevator to the next floor, looking there, then the next, and so on. Once you get to the top floor (the global *Scope*), you either find what you're looking for, or you don't. But you have to stop regardless.

## Errors

Why does it matter whether we call it LHS or RHS?

Because these two types of look-ups behave differently in the circumstance where the variable has not yet been declared (is not found in any consulted *Scope*).

Consider:

```js
function foo(a) {
	console.log( a + b );
	b = a;
}

foo( 2 );
```

When the RHS look-up occurs for `b` the first time, it will not be found. This is said to be an "undeclared" variable, because it is not found in the scope.

If an RHS look-up fails to ever find a variable, anywhere in the nested *Scope*s, this results in a `ReferenceError` being thrown by the *Engine*. It's important to note that the error is of the type `ReferenceError`.

By contrast, if the *Engine* is performing an LHS look-up and arrives at the top floor (global *Scope*) without finding it, and if the program is not running in "Strict Mode" [^note-strictmode], then the global *Scope* will create a new variable of that name **in the global scope**, and hand it back to *Engine*.

*"No, there wasn't one before, but I was helpful and created one for you."*

"Strict Mode" [^note-strictmode], which was added in ES5, has a number of different behaviors from normal/relaxed/lazy mode. One such behavior is that it disallows the automatic/implicit global variable creation. In that case, there would be no global *Scope*'d variable to hand back from an LHS look-up, and *Engine* would throw a `ReferenceError` similarly to the RHS case.

Now, if a variable is found for an RHS look-up, but you try to do something with its value that is impossible, such as trying to execute-as-function a non-function value, or reference a property on a `null` or `undefined` value, then *Engine* throws a different kind of error, called a `TypeError`.

`ReferenceError` is *Scope* resolution-failure related, whereas `TypeError` implies that *Scope* resolution was successful, but that there was an illegal/impossible action attempted against the result.

## Review (TL;DR)

Scope is the set of rules that determines where and how a variable (identifier) can be looked-up. This look-up may be for the purposes of assigning to the variable, which is an LHS (left-hand-side) reference, or it may be for the purposes of retrieving its value, which is an RHS (right-hand-side) reference.

LHS references result from assignment operations. *Scope*-related assignments can occur either with the `=` operator or by passing arguments to (assign to) function parameters.

The JavaScript *Engine* first compiles code before it executes, and in so doing, it splits up statements like `var a = 2;` into two separate steps:

1. First, `var a` to declare it in that *Scope*. This is performed at the beginning, before code execution.

2. Later, `a = 2` to look up the variable (LHS reference) and assign to it if found.

Both LHS and RHS reference look-ups start at the currently executing *Scope*, and if need be (that is, they don't find what they're looking for there), they work their way up the nested *Scope*, one scope (floor) at a time, looking for the identifier, until they get to the global (top floor) and stop, and either find it, or don't.

Unfulfilled RHS references result in `ReferenceError`s being thrown. Unfulfilled LHS references result in an automatic, implicitly-created global of that name (if not in "Strict Mode" [^note-strictmode]), or a `ReferenceError` (if in "Strict Mode" [^note-strictmode]).

### Quiz Answers

```js
function foo(a) {
	var b = a;
	return a + b;
}

var c = foo( 2 );
```

1. Identify all the LHS look-ups (there are 3!).

	**`c = ..`, `a = 2` (implicit param assignment) and `b = ..`**

2. Identify all the RHS look-ups (there are 4!).

    **`foo(2..`, `= a;`, `a + ..` and `.. + b`**


[^note-strictmode]: MDN: [Strict Mode](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Functions_and_function_scope/Strict_mode)
