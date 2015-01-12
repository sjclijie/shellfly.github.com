---
layout: post
title: "(译) The Y combinator (Slight Return)"
date: 2015-01-07 20:52:02 +0800
comments: true
categories: 翻译
---

or:

<u>How to Succeed at Recursion Without Really Recursing</u>

    Tiger got to hunt,
    Bird got to fly;
    Lisper got to sit and wonder, (Y (Y Y))?
    
    Tiger got to sleep,
    Bird got to land;
    Lisper got to tell himself he understand.

      — Kurt Vonnegut, modified by Darius Bacon

<!--more-->

## 介绍

自从最近写了一篇关于Y combinator的文章后，我收到了很多有用的评论，所以我觉得应该再扩展一下，写一篇更完整的出来。这篇文章会讲得更深入，但我也希望它能更容易理解。你不需要先读完前一篇文章才能读懂这篇（实际上你没读过更好）。我需要的唯一背景知识就是一点点Scheme编程语言，包括递归和first-class函数，文章中我也会提到。欢迎（再次）评论。

## 为什么学习 Y combinator

在详细说明Y combinator到底是什么之前，我想先指出，为什么你，作为一个程序员，应该花时间来学习它。说实话，并没有多少实际的理由让你一定要学习Y combinator。虽然它的确有一点实际的作用，而且大部分是对于计算机语言学家们。尽管如此，我还是觉得你值得花时间来了解一点Y combinator的东西，因为：

1。 它是编程中最美的想法之一，如果你稍微有一点对于编程的审美，你一定会被Y combinator打动。

2。 它以一种非常质朴的方式展现出函数式编程的简单想法是多么惊人和强大。

1959年，英国科学家C。 P。 Snow做了一场有名的讲座，叫 **[The Two Cultures](http://en。wikipedia。org/wiki/The_Two_Cultures)** ，在讲座中他哀叹当时很多头脑聪明教育良好的人却对科学一无所知。他用热力学第二定律的知识作为可以认知科学和无法认知科学的人的分界线。我想我们同样可以用Y combinator的知识作为了解函数式编程（也就是说，掌握足够的函数式编程知识）和不了解函数式编程的程序员之间的分界线。有一些其他工具也可以代替Y combinator很好的工作（尤其是monads），但是Y combinator会完成的更漂亮。所以如果你渴望知道真正的Lambda本质，那就继续读下去。

顺便提一下，[Paul Graham](http://www。paulgraham。com/)(Lisp黑客，Lisp书籍作家，随笔作家，现在也是一名企业家)显然把Y combinator看的更重，他用[Y combinator](http://www。ycombinator。com/)来命名他的创业孵化器公司。Paul通过这些知识变的富有，也许有其他人也会，也许甚至就是你。


<hr />

## 一个谜题

## 阶乘

我们通过定义几个计算阶乘的函数来开始我们的Y combinator之旅，一个非负整数的阶乘就是从1一直乘到它自己，所以我们有：

<p class="indent">factorial 1 = 1
factorial 2 = 2 * 1 = 2
factorial 3 = 3 * 2 * 1 = 6
factorial 4 = 4 * 3 * 2 *1 = 24
</p>
等等。（我在这里写函数时没有用括号，所以 factorial 3 和通常写的factorial(3)是一个意思。就先依我吧）随着n的增长，阶乘增长的很快，20的阶乘等于2432902008176640000。0的阶乘被定义成1，事实证明在对于实际使用阶乘的场合，这是个非常合适的定义（比如解决组合数学问题的时候）。

## 递归定义阶乘函数

在编程语言里用一些像 `while` 和 `for` 这种循环控制语句，就能很容易地写出计算阶乘的函数（比如C或者Java）。不过写一个递归函数来计算阶乘也很简单，因为阶乘的定义本身就是递归的：

<p class="indent">factorial 0 = 1
factorial n = n * factorial (n - 1)
</p>

第二行适用于所有非0的 `n` ，事实上在计算机语言[Haskell](http://www。haskell。org/)里，这就是实际定义阶乘函数的方式。在我们要用的[Scheme](http://www。schemers。org/)语言里，这个函数会被写成这样：

``` Scheme
(define (factorial n)
  (if (= n 0)
      1
      (* n (factorial (- n 1)))))
```

在Scheme里所有东西都是用括号括起来的前缀表达式，所以像 `(- n 1)` 这种就表示一般在其他编程语言里写的 `n - 1` 。至于要这么写的原因超出了这篇文章要说的范围，就不细说了，但是要习惯这种写法并不难。

事实上，上面关于阶乘函数在Scheme里面的定义，可以写成下面这种更明确的方式:

``` Scheme
(define factorial
  (lambda (n)
    (if (= n 0)
        1
        (* n (factorial (- n 1))))))
```

关键字 `lambda` 只是简单的表示我们在定义的（lambda左边小括号和它对应的括号之间的全部内容）是一个函数。紧跟在 `lambda` 后面，括号中的是函数的*参数列表*，这里只有一个参数 `n` 。*函数体*跟在参数列表后面，这里由 `(if (= n 0) 1 (* n (factorial (- n 1))))` 组成。这种函数叫*匿名函数*。我们在这里给了匿名函数一个名字 `factorial` ，但这其实不是必须的，而且如果只准备用一次的话，一般不给它命名更方便。在Scheme和其他一些语言里面，匿名函数也被叫做*lambda表达式*。有许多其他语言也允许你定义匿名函数，包括Python，Ruby，Javascript，Ocaml和Haskell（但不幸的是，在C，C++和Java里不行）。下面的文章中我们会用到很多lambda表达式。

在Scheme里面，刚才给出的 `factorial` 定义和上一个定义其实是相同的，Scheme会在执行之前把第一个转换成第二个这种形式，所以Scheme里面所有的函数都是真正的lambda表达式。

注意这个函数体里有一个对 `factorial` （我们正在定义中的函数） 的调用，这就是为什么它是一个递归定义。我会称这种在函数体里使用函数名的定义为*显式递归定义*。（你可能会好奇 "隐式递归定义"会是什么样的，不过我不准备用这种表达方式，但是我脑子里想到的概念是通过一种非递归的方式生成递归函数——继续读下去！）

为了便于讨论，我们假设我们用的Scheme版本没有像C和Java中的 `for` 和 `while` 循环语句（尽管实际上，真正的Scheme实现里是有这些结构的，只是用了不同的名字），所以为了定义像 `factorial` 这样的函数，我们只能使用递归。这也是Scheme常被用作教学语言的原因之一，它强制学生学会递归的思考问题。

## 作为数据的函数和高阶函数

从很多方面来看Scheme都是一个很酷的语言，和我们这里讨论相关一个是，它允许你把函数当作"first-class"数据对象（我们一般是说成Scheme支持*first class 函数*），这就意味着在Scheme里，我们可以把一个函数当作另一个函数的参数，可以在执行一个函数后返回一个函数，也可以在需要的时候临时创建一个函数（用上面的 `lambda` 语句），这就是函数式编程的本质，在接下来的讨论里这一特征也会表现得更加显著。接受函数作为参数，或者返回一个函数作为结果的函数通常就被称为*高阶函数*。

## 消除显式递归

现在，谜题来了：如果告诉你在定义函数的时候不能递归的调用函数（例如，在上面的 `factorial` 函数的函数体的任何地方不准再使用 `factorial` ），但是你可以随意地使用first-class函数和高阶函数，你怎么在Scheme里定义 `factorial` 函数？这样你还能定义一个 `factorial` 函数吗？

答案是可以的，而且它将直接带我们进入Y combinator的世界。

<hr/>

## 什么是Y combinator，它又能做什么

Y combinator是一个高阶函数，它接受一个非递归函数作为参数，返回这个函数的递归版本。我们下面会逐步地了解用Y combinator从非递归函数生成递归函数的过程，不过这就是它最基本的定义。

说的更笼统一点，Y combinator给我们提供了一种方法，让我们在一个只支持first-class函数，但是没有内建递归的编程语言里完成递归。所以Y combinator给我们展示了一个语言完全可以定义递归函数，即使这个语言的定义一点也没提到递归。它给我们展示了一件美妙的事：仅仅函数式编程自己，就可以让我们做到我们从来不认为可以做到的事（而且还不止这一个例子）。

## 惰性求值还是严格求值（Lazy or strict evaluation）?

我们将会看到两大类计算机语言：使用*惰性求值*的和使用*严格求值*的。惰性求值的意思是，为了执行一个表达式，你只需要执行最终结果需要的部分。所以（比如）如果一个表达式中有一部分不需要求值（因为结果不依赖它）它就不会被执行。相反的，严格求值的意思是，在表达式的值确定前，表达式中的所有部分都会被执行（除了一些必要的例外，像 `if` 表达式就只能是惰性的这样它才能正常工作）。 实践中，惰性求值更宽泛，但是严格求值更可预测而且往往更有效率。大部分编程语言都使用严格求值。[Haskell](http://www。haskell。org/)使用惰性求值，这也是这个语言最有趣的部分。在下面这两种求值方式我们都会使用到。

## 只有一个Y combinator还是很多？

尽管我们经常说Y combinator，但在实际情况中其实有无数个Y combinator。我们现在只关心其中两个，一个惰性的一个严格的。我们需要两个是因为为惰性语言定义的Y combinator不能在严格的语言里工作。惰性的Y combinator通常也被称为*正则序Y combinator (normal-order Y combinator)* ，严格的Y combinator通常被称为*应用序Y combinator (applicative-orer Y combinator)*。基本上，*正则序*是"惰性"的另一种说法，而*应用序*是"严格"的另一种说法。

## 静态还是动态类型（Static or dynamic typing）?

编程语言里另一个大的分界线是*静态类型*和*动态类型*。静态类型语言里所有表达式的类型都在编译时确定，任何类型错误都会导致编译失败。动态类型语言在执行前不做任何类型检查，如果把函数应用在非法的类型参数上面（例如，尝试把数字和字符串相加），就会报一个错误出来。在常用的编程语言里，C，C++和Java是静态类型，Perl，Python和Ruby是动态类型。Scheme(我们要用来展示例子的语言)也同样是动态的。（也有那种同时横跨两种类型的语言，但是我们这里不会讨论到）

有人经常听说静态类型就是强类型，动态类型就是弱类型，但是这是错误的。强类型是指语言中的任何值有且只有一种类型，另一方面，弱类型是指有些值可以有多种类型。所以，像Scheme既是动态类型，也是强类型，但是C就是静态类型和弱类型的（因为你可以把指向一种类型对象的指针转换成指向另一种类型对象，而不需要改变指针的值）。这里我们只关心强类型语言。

在动态类型语言里定义Y combinator显得更简单，所以我也会这么做。在许多静态类型的语言里也可以定义Y combinator，但是（至少在我看到的例子里）这些定义一般都需要一些不常见的类型修改，因为Y combinator它自己没有一个直接的静态类型。关于这个问题已经超出了这篇文章要说的东西，所以下面不会再提了。

## 什么是 ”combinator“

Combinator就是一个*不使用自由变量的lambda 表达式*，我们上面已经看到了lambda表达式是什么(匿名函数)，但是自由变量是什么？不是*绑定变量*的变量就是自由变量。现在明白了吗？没有？好吧，让我解释一下。

我们看一些lambda表达式的例子和一些自由变量和绑定变量：

1. `(lambda (x) x)`
2. `(lambda (x) y)`
3. `(lambda (x) (lambda (y) x))`
4. `(lambda (x) (lambda (y) (x y)))`
5. `(x (lambda (y) y))`
6. `((lambda (x) x ) y )`

这些lambda表达式中函数体里的变量是自由变量还是绑定变量？我们会忽略lambda表达式中的参数，因为只有在函数体里的变量才有自由和绑定这种说法。对于这些变量，我们的答案是：

1. 函数体里的 `x` 是一个绑定变量，因为lambda表达式的参数也是 `x` 。这个lambda表达式没有其他参数。所以也就没有自由变量，所以它是combinator。
2. 函数体里的 `y` 是一个自由变量，所以这个lambda表达式不是combinator。
3. 除了函数的参数外，这里只有一个绑定变量 `x`（绑定于外层的lambda表达式），所以整个lambda表达式没有自由变量，是combinator。
4. 除了函数的参数外，这里有两个变量，最后面的 `x` 和 `y` ，两个都是绑定变量，是combinator。
5. 整个表达式不是一个lambda表达式，所以按照定义也不是combinator。不过这里的 `x` 是一个自由变量，后面的 `y` 是一个绑定变量。
6. 整个表达式也不是一个lambda表达式（它是一个函数应用），所以它也不是combinator。第二个 `x` 是一个绑定变量，`y` 是自由变量。

如果你好奇像 `factorial` 这样的递归函数是不是combinator:

```Scheme
 (define factorial
   (lambda (n)
   (if (= n 0)
       1
       (* n (factorial (- n 1))))))
```
你可以不考虑它的 `define` 部分，所以你真正想知道的是这个lambda表达式：
```Scheme
(lambda (n)
  (if (= n 0)
      1
      (* n (factorial (- n 1)))))
```
是不是一个combinator。因为这个lambda表达式里面 `factorial` 代表一个自由变量（ `factorial` 不在lambda表达式的参数里面），这个不是一个combinator。这对于下面要说的东西是很重要的一点。实际上 `=` ，  `*` ， `-` 这些都是自由变量，所以即使没有用 `factorial` ，这也不是一个combinator（更何况还有数字!）。
<hr />

## 回到那个谜题

## 抽象出递归的函数调用

回忆下我们之前的阶乘函数：
``` Scheme
(define factorial
  (lambda (n)
    (if (= n 0)
        1
        (* n (factorial (- n 1))))))
```
我们想做的是，想出一个可以同样工作的版本，但是不要函数体里那个碍眼的递归调用 `factorial`。

我们应该从哪里开始？如果我们可以保留函数的其他部分，不要递归调用，放一些其他东西在那就好了。可能看起来像这样：

```Scheme
(define sort-of-factorial
  (lambda (n)
    (if (= n 0)
        1
        (* n (<???> (- n 1))))))
```
这种形式仍然留给我们一个问题，我们应该在 `<???>` 这里放什么。在函数式语言里有一个行之有效的原则，如果你不确定在某个地方应该写一段什么代码，就把它抽象出来作为一个函数的参数。最简单的方式就是下面这样：

```Scheme
(define almost-factorial
  (lambda (f)
    (lambda (n)
      (if (= n 0)
          1
          (* n (f (- n 1)))))))
      
```

我们在这里把对 `factorial` 的调改成了 `f` ，然后把 `f` 变成了一个 `almost-factorial` 函数的参数。注意这里 `almost-factorial` 并不是阶乘函数，而是一个接受一个单一参数 `f` 的高阶函数（ `f` 最好是一个函数，否则 `(f (- n 1))` 就没有意义了），如果我们选择了正确的 `f` ，它将返回一个（希望)是阶乘函数( `(lambda (n) ...)` 部分)的函数。

有很重要的一点要意识到，这个方法并不是 `factorial` 特有的，我们可以把同样的方法应用在任何递归函数上面。例如，一个计算斐波那契数字的递归函数。斐波那契的递归定义如下：

<p class="indent">fibonacci 0 = 0
fibonacci 1 = 1
fibonacci n = fibonacci (n - 1) + fibonacci (n - 2)
<p>
(实际上，这是Haskell里斐波那契函数的定义)。在Scheme里面，我们可以把这个函数写成这样：

```Scheme
(define fibonacci
  (lambda (n)
    (cond ((= n 0) 0)
          ((= n 1) 1)
          (else (+ (fibonacci (- n 1) (fibonacci - n 2)))))))
```
（ `cond` 只是一个写嵌套 `if` 表达式的快捷方式）我们现在就可以像对 `factorial` 那样，移除这里的显示递归调用：

```Scheme
(define almost-fibonacci
  (lambda (f)
    (lambda (n)
      (cond ((= n 0) 0)
            ((= n 1) 1)
            (else (+ (f (- n 1)) (f (- n 2))))))))
```

可以看到，从递归函数到一个等价的非递归函数 `almost-` 是一个很机械的过程，把函数体里递归的函数重命名成 `f` ， 然后在原来的函数体外面包装一层 `(lambda (f) ...)`。

如果你有一直跟着我上面的做(不用管为什么，后面就知道了)，那么恭喜你， 就像尤达大师说的，你刚刚迈出了进入更大的世界的第一步。

## 剧透一下

也许我不该这么做，不过我想先剧透一下我们的目标。一旦我们定义了Y combinator，我们就可以使用 `almost-factorial` 定义阶乘函数， 就像这样：

```Scheme
(define factorial (Y almost-factorial))
```

`Y` 就是Y combinator，注意这里 `factorial` 的定义没有任何的显示递归在里面。同样地我们可以用 `almost-fabonacci` 来定义 `fibonacci`：

```Scheme
(define fibonacci (Y almost-fibonacci))
```

所以只要我们有合适的 `almost-` 函数（也就是，通过抽象递归调用，从递归函数衍生来的非递归函数），Y combinator就可以在我们需要时为我们完成递归。

继续阅读看看这里到底发生了什么，为什么这样是可行的。

## 从`almost-factorial`中恢复`factorial`

我们假设，为了讨论方便，我们已经有一个可以工作的阶乘函数（不管它是不是递归的）。我们叫这个假设的阶乘函数 `factorialA`。现在考虑下面的定义：

```Scheme
(define factorialB (almost-factorial factorialA))
```

问题：`factorialB` 可以计算阶乘吗？

为了回答这个问题，可以把 `almost-factorial` 的定义展开：

``` Scheme
(define factorialB
  ((lambda (f)
    (lambda (n)
      (if (= n 0)
          1
          (* n (f (- n 1))))))
    factorialA))
```

在函数体里用 `factorialA` 替换 `f` ，我们可以得到：

``` Scheme
(define factorialB
  (lambda (n)
    (if (= n 0)
        1
        (* n (factorialA (- n 1))))))
```

这个看起来已经很像递归版本的阶乘函数，但是并不是：`factorialA`  和 `factorialB` 不是一个函数。所以如果假设的 `factorialA` 函数可以工作的话，那么它就是一个非递归的阶乘函数。它真的可以计算阶乘吗？显然对于 `n = 0` 是可以计算的，因为 `(factorialB 0)` 就返回 `1` (0的阶乘)。如果 `n > 0` ， `(factorialB n)` 的值就是 `(* n (factorialA (- n 1)))` 。我们之前假设 `factorialA` 是可以计算阶乘的，所以 `(factorialA (- n 1))` 就是 `n - 1` 的阶乘，按照定义 `(* n (factorialA (- n 1)))` 就是 `n` 的阶乘。这就证明了只要 `factorialA` 可以计算阶乘， `factorialB`也可以计算阶乘。唯一的问题是我们并没有一个现成的 `factorialA` 。

如果你真的聪明的话，你可能会问，我们是不是可以这样做：

``` Scheme
(define factorialA (almost-factorial factorialA))
```

它的意思是这样的：假设 `factorialA` 是一个合法的阶乘函数。然后我们把它传给 `almost-factorial` ，返回的函数也将是一个合法的阶乘函数，那为什么不直接把这个返回函数也叫 `factorialA` 呢？

看起来我们好像造了一个永动机（或者应该说是永计算机），这里面一定有哪里错了。。。一定错了吗？

实际上，只要你使用的Scheme语言是惰性求值的，这个定义就可以正常的动作！标准的Scheme实现是严格求值的，所以它并不能工作（会陷入死循环）。如果你用[DrRacket](http://racket-lang.org/)作为Scheme解释器（你应该使用这个解释器，译者注：原文是DrScheme，现在已经改名成DrRacket，下面的都会替换掉，不在重复），然后打开"lazy Scheme"选项，上面的代码就可以实际的工作了（欢呼！）。我们下面会看到为什么，但是现在我想停留在标准（严格求值的）Scheme实现上用一种不同的方式继续探索这个问题。

让我们定义一些函数：

``` Scheme
(define identity (lambda (x) x))
(define factorial0 (almost-factorial indentity))
```

`identity` 是个很简单的函数：它接受一个参数，然后原封不动的返回（它也是一个combinator，希望你还记得）。我们只是用它来做为一个占位符，当我们需要把一个函数作为参数传递给另外一个函数时，如果我们不知道该传递什么函数，就先用 `identity` 。

`factorial0`更有趣。它是一个可以计算部分但不是所有阶乘的函数。特别地，它只能计算小于等于0的阶乘（也就是它只能计算0的阶乘，很快你就会知道为什么我要写小于等于）。让我们验证一下：

{% codeblock lang:Scheme %}
(factorial0 0)

==> ((almost-factorial identity) 0)

(((lambda (f)
         (lambda (n)
           (if (= n 0)
               1
               (* n (f (- n 1))))))
      identity)
     0)

==> ((lambda (n)
        (if (= n 0)
            1
            (* n (identity (- n 1)))))
     0)

==> (if (= 0 0)
        1
        (* n (identity (- n 1))))

==> 1
{% endcodeblock %}

它计算出来了，但是不幸的是，对于 `n > 0` 它并不能工作。比如对于`n = 1` 我们就有（跳过了一些步骤）：

``` Scheme
(factorial0 1)

==> (* 1 (identity (- 1 1)))
==> (* 1 (identity 0))
==> (* 1 0)
==> 0
```
得到的是错误的答案。

现在考虑一下这个`factorial0`的进化版本：

``` Scheme
(define factorial1
  (almost-factorial factorial0))
```

也就是：

``` Scheme
(define factotial1
  (almost-factorial
    (almost-factorial identity)))
```

这个函数可以准确的计算 `0` 和 `1` 的阶乘，你可以验证一下，但是对于 `n > 1` 则不行。
（把原文的验证过程省略了）

我们可以继续下去，定义更多可以计算到某个限制的阶乘函数：

``` Scheme
(define factorial2 (almost-factorial factorial1))
(define factorial3 (almost-factorial factorial2))
(define factorial4 (almost-factorial factorial3))
(define factorial5 (almost-factorial factorial4))
。。。
```

`factorial2` 可以计算 `0` 到 `2` 的阶乘， `factotial3` 可以计算 `0` 到 `3` 的阶乘等等。你可以用上面的模型自己验证，尽管你可能不能再脑子里来验证（至少我做不到）

换一种有趣的角度来看，`almost-factorial` 接受一个不完整的阶乘函数，然后返回一个稍微完整一点的版本，新的函数仅仅可以比原来的函数多计算一个阶乘。

注意，你也可以像这样充血这些阶乘函数的定义:

``` Scheme
(define factorial0 (almost-factorial identity))

(define factorial1
  (almost-factorial
    (almost-factorial identity)))

(define factorial2
  (almost-factorial
    (almost-factorial
      (almost-factorial identity))))

(define factorial3
  (almost-factorial
    (almost-factorial
      (almost-factorial
        (almost-factorial identity)))))
。。。
```

再一次，如果你很聪明的话，你可能会想我们是不是可以这么做：

``` Scheme
(define factorial-infinity
  (almost-factorial
    (almost-factorial
      (almost-factorial
        。。。))))
```

`。。。`在这里代表重复执行 `almost-factorial`无限次。如果你真的很好奇，你可以在脑子里想象一下！很可惜我们不能把它直接写出来，但是我们可以定义和这个等价的形式。注意 `factorial-infinity` 就是我们想要的 `factorial` 函数：它对于所有大于等于零的数都可以正常工作。

我们刚才展示了，如果我们可以定义无限的`almost-factorial`调用，就可以得到阶乘函数。换一种表达方式说，阶乘函数是 `almost-factorial` 的 *固定点（fixpoint）*，我们下面要解释的东西。

## 函数的固定点 （Fixpoints of functions）

任何人手边有一个计算器的话，就可以熟悉固定点这个概念。你可以从 `0` 开始一直重复的按 `cos` 键。你会发现答案很快变成一个固定数，大概是 `0。73908513321516067`，在按 `cos` 键也不会改变这个数字，因为 `cos(0。73908513321516067) = 0。73908513321516067` 。我们说数字 `0。73908513321516067` 是cosine函数的一个 *固定点*。

cosine函数接受一个参数（一个实数），产生一个输出（也是一个实数）。因为输入和输出是同一种类型，所以才能够重复把输出在作为输入。所以如果 `x` 是一个实数，我们就可以计算 `cos(x)`， 因为结果还是一个实数，所以可以继续计算 `cos(cos(x))` ，`cos(cos(cos(x)))` 等等。如果 `cos(x) = x` ， 那么 `x` 就是cosine函数的固定点。

固定点不一定要是实数，实际上它们可是是任意类型，只要返回他们的函数可以接受同样类型的输入然后产生同样类型的输出。对我们的讨论最重要的一点是，固定点可以是函数。如果你有一个像 `almost-factorial` 这样的高阶函数，它接受一个函数作为参数，然后产生一个函数作为输出（输入和输出都是一个接受单一整数作为输入， 产生一个整数作为输出的函数），那么就应该可以计算出它的固定点（自然的，结果也是一个可以接受单一整数作为输入，产生一个整数作为输出的函数）。这个固定点函数也就是下面的函数

<p class="indent">fixpoint-function = (almost-factorial fixpoint-function)</p>

通过重复替换等式的右边的 `fixpoint-funcation` ，我们可以得到：
 
<p class="indent">fixpoint-function =
  (almost-factorial
    (almost-factorial fixpoint-function))

= (almost-factorial
  (almost-factorial
    (almost-factorial fixpoint-function)))

= 。。。

= (almost-factorial
    (almost-factorial
      (almost-factorial
        (almost-factorial
          (almost-factorial 。。。)))))
</p>

就像上面看到的，这就是我们想要的阶乘函数。所以 `almost-factorial` 的固定点就是 `factorial` 函数。

目前为止都还不错，但是只是知道 `factorial` 是 `almost-factorial` 的固定点并不能告诉我们怎么来计算出它。如果有一个高阶函数可以接受像 `almost-factorial`  这种函数作为参数，然后输出它的固定点函数，那不是很好吗。如果可以的话就会返回 `factorial`? 是不是想的太美了？

这个函数的确存在，就是Y combinator。Y combinator又称*固定点组合子（fixpoint combinaotr）* 。 它接受一个函数然后返回它的固定点。

## 消除（大部分）显示递归（惰性求值版本）

好了，现在是时间推导Y combinator了。

我们从指定Y combinator可以做什么开始：

<p class="indent">(Y f) = fixpoiint-of-f</p>

关于 `f` 的固定点，我们已经知道

<p class="indent">(f fixpoint-of-f) = fixpoint-of-f</p>

通过函数固定点的定义，我们有：

<p class="indent">(Y f) = fixpoint-of-f = (f fixpoint-of-f)</p>

我们可以用 `(Y f)` 替换 `fixpoint-of-f`，然后就能得到:

<p class="indent">(Y f) = (f (Y f))</p>

看啊，我们刚才定义了Y combinator，如果你想把它表示成一个Scheme函数，我们可以把它写成这样：

``` Scheme
(define (Y f) (f (Y f)))
```

或者，使用 `lambda` 表达式：

``` Scheme
(define Y
  (lambda (f)
    (f (Y f))))
```

但是，关于这个Y combinator的定有，有两个警告：

1。 它只能在惰性求值的语言下工作（看下面）
2。 它不是一个combinator，因为函数体里面的 `Y` 是自由变量，只有定义结束了才被绑定。换句话说，我们不能在需要的时候只有这个版本的函数体，因为它还需要 `Y` 在某个地方被定义。

尽管如此，如果你使用惰性的Scheme，你确实可以像这样定义Y combinator：

``` Scheme
(define Y
  (lambda (f)
    (f (Y f))))

(define almost-factorial
  (lambda (f)
    (lambda (n)
      (if (= n 0)
          1
          (* n (f (- n 1)))))))
          
(define factorial (Y almost-factorial))
```
而且它也能正常的工作。

我们刚刚完成了什么？我们最初是想定义一个不使用显示递归的阶乘函数。我们*几乎*已经完成了。我们对于 `Y` 的定义仍然是递归的，但是我们跨出了巨大的一步，因为这是我们在语言里定义递归函数唯一需要的现实递归。有了这个版本的 `Y` 我们就可以继续定义其他递归函数(比如，把 `fibonacci` 定义成 `(Y almost-fibonacci)`)。

## 消除（大部分）显示递归（严格求值版本） ##

我刚才说了我们推动出来的 `Y` 不能再严格求值的语言里工作（像标准的Scheme语言）。再一个严格求值的语言里，在把参数传递给函数前，我们要执行所有的参数来得到对应的值，不管这些参数是不是需要被求值。所以如果我们有一个函数 `f`，并且尝试用上面的定义去执行 `(Y f)` ，我们只能得到：

<p class="indent">(Y f)
= (f (Y f)) 
= (f (f (Y f)))
= (f (f (f (Y f))))
。。。
</p>
无休止的执行下去。执行 `(Y f)` 的过程永远不会终止，所以我们永远也得不到一个可以用的函数。惰性求值的Y combinator定义并不适合严格求值的语言。

但是，有一个聪明的技巧我们可以用来解决这个问题并且定义一个可以在严格求值的语言下正常工作的Y combinator。这个技巧就是明白 `(Y f)` 将会是一个函数的参数，所以下面的等式也是成立的：

<p class="indent">(Y f) = (lambda (x) ((Y f) x))</p>

无论 `(Y f)` 得到的是一个什么样的接受一个参数的函数，`(lambda (x) ((Y f) x))` 也将是同样的函数。我们做的只是拿一个输入的值 `x` 然后把它给 `(Y f)` 得到的函数。同样地，下面也是成立的：

<p class="indent">cos = (lambda (x) (cos x))</p>

用 `cos` 或者 `(lambda (x) (cos x))` 作为cosine函数都无所谓，它们做的都是同样的事。

但是，事实证明在严格求值的语言里定义Y combinator `(lambda (x) ((Y f) x))` 有更大的优势。通过上面的解释，我们可以像这样定义Y combinator：

``` Scheme
(define Y
  (lambda (f)
    (f (lambda (x) ((Y f) x)))))
```

因为我们知道 `(lambda (x) ((Y f) x)` 和 `(Y f)` 是一样的，所以这也是一个可以和上一个版本一样工作的，合法的Y combinator版本，尽管它比之前的复杂了一点（而且可能在实际中也慢一点）。我们可以用这个版本的Y combinator在惰性求值的Scheme里面定义 `factorial`。

关于*这个版本*的Y combinator更酷的是，它也能在一个严格求值的语言下工作（像标准的Scheme语言）！因为当你传递一个函数 `f` 给 `Y` 来获取 `f` 的固定点的时候，它会返回

<p class="indent">(Y f) = (f (lambda (x) ((Y f ) x))</p>

这次没有死循环，因为内部的 `(Y f)` 被包围在一个 `lambda` 表达式里面，直到需要的时候才会被执行（因为在Scheme里，在把lambda表达式应用到一个参数之前，函数体里的lambda表达式永远不会执行）。基本上，我们用lambda来推迟了 `(Y f)` 的执行。所以如果 `f` 是 `almost-factorial`，我们就有：

``` Scheme
(define almost-factorial
  (lambda (f)
    (lambda (n)
      (if (= n 0)
          1
          (* n (f (- n 1)))))))
          
(define factorial (Y almost-factorial))
```

把对Y的调用扩展开，可以得到：

``` Scheme
(define factorial
  ((lambda (f) (f (lambda (x) ((Y f) x))))
   almost-factorial))

==>

(define factorial
  (almost-factorial (lambda (x) ((Y almost-factorial) x))))

==>

(define factorial
  (lambda (n)
    (if (= n 0)
        1
        (* n ((lambda (x) ((Y almost-factorial) x)) (- n 1))))))
```

这里还是的，`(lambda (x) ((Y almost-factorial) x)` 和 `(Y almost-factorial)` 是同一个函数， 都是 `almost-factorial` 的固定点，也就是阶乘函数。但是在 `(lambda (x) ((Y almost-factorial) x)` 里面的 `(Y almost-factorial))` 直到整个lambda表达式被应用到它的参数的时候才会被执行，这个只会在后面才发生（或者永远不会被执行，比如计算0的阶乘的时候）。所以这个阶乘函数可以在一个严格求值的语言里工作，而且这个版本的Y也能在一个严格求值的语言里工作。

我意识到前面的讨论和推导的东西不是那么轻松，所以如果你没有马上明白的话也不要灰心。好好考虑一下，在脑子里和值得幸赖的DrScheme解释器里尝试一下，你最终会弄明白的。

目前为止，我们已经完成了我们之前设定的所有事，除了一个微小的细节：我们还没有推导Y combinator它自己。
<hr/>

## 推导Y combinator ##

## 惰性（正则序）Y combinator ##

现在，我们不是想定义函数Y，而是Y combinator。注意一下前一个版本（惰性）的Y：

``` Scheme
(define Y
  (lambda (f)
    (f (Y f))))
```

是一个合法的函数Y，但却不是一个Y combinator，因为函数的定义里用到了它自己。换句话说，这个定义是显示递归的。Combinator是不允许显示递归的，它是一个没有自由变量的lambda表达式，这也就意味着它不能在定义里使用自己的名字（如果它居然有一个名字的话）。如果它用了，那个名字就是一个自由变量，像我们对Y的定义：

``` Scheme
(lambda (f)
  (f (Y f)))
```

Y在这里就是自由变量，它没有被任何lambda表达式绑定。所以这个表达式不是一个combinator。

换一种方式来想的话，就是你应该可以在任何使用combinator的地方，用combinator的定义来替换，而且替换后还可以正常工作（你知道为什么用显示递归定义的Y不能正常工作了吗？你会陷入一个死循环，而且你永远也没办法用定义替换掉所有的Y）。所以不管Y combinator最终会是什么样，它肯定是不是一个显示递归。通过这个非递归的函数，我们能够定义任何我们想要的递归函数。

我想先回到我们最初的问题，然后由下到上推导出Y combinator。等我完成后，我会检查下它确实一个固定点组合子（fixpoint combinator），像我们之前看到的版本一样。下面的部分，我会大量借鉴（抄袭）Eli Barzilay发给我的一个很优雅的Y combinator推导过程（感谢Eli!），他是一个DrScheme开发者也是一个全能的Scheme高手。

回忆下我们最初的 `factorial` 函数：

``` Scheme
(define (factorial n)
  (if (= n 0)
      1
      (* n (factorial (- n 1)))))
```

记得最开始我们是想定义一个没有显示递归的版本。其中一个方法是把阶乘函数自己作为调用阶乘函数时的一个额外的参数：

``` Scheme
;; 这个还不能用:
(define (part-factorial self n)
  (if (= n 0)
      1
      (* n (self (- n 1)))))
```

注意这里的 `part-factorial` 和 上面描述的`almost-factorial` 并不一样。我们可以用一种不同的方式调用 `part-factorial` 好让它可以计算阶乘：

<p class="indent">(part-factorial part-factorial 5) ==> 120</p>

这个不是显示递归，因为我们把 `part-factorial` 的额外拷贝作为 `self` 传递给了函数。不过它还不能工作，除非在递归的地方用完全一样的方式来调用：

``` Scheme
(define (part-factorial self n)
  (if (= n 0)
      1
      (* n (self self (- n 1))))) ;; 注意这里额外的 “self”

（part-factorial part-factorial 5） ==> 120
```

现在可以了，但是我们已经偏离了原来调用阶乘函数的方式。我们可以把它重写成下面这样，后退到更接近原来样子的版本：

``` Scheme
(define (part-factorial self)
  (lambda (n)
    (if (= n 0)
        1
        (* n (* (self self) (- n 1))))))

((part-factorial part-factorial) 5) ==> 120
(define factorial (part-factorial part-factorial))
(factorial 5) ==> 120
```

这里稍微停一下。注意我们*已经*定义了一个没有显示递归的阶乘函数！这是最至关重要的一步。剩下要做的所有事都是把我们已经完成的东西打包起来，这样我们就可以在定义其他函数的时候重用它。

现在我们尝试移除调用 `(self self)` 来得到和 `almost-factorial` 差不多的函数，通过在 `lambda` 外面使用 `let` 表达式：

``` Scheme
(define (part-factorial self)
  (let ((f (self self))
    (lambda (n)
      (if (= n 0)
          1
          (* n (f (- n 1))))))))

(define factorial (part-factorial part-factorial))
(factorial 5) ==> 120
```

这个可以在惰性求值的语言下工作。在一个严格求值的语言里，`let` 表达式中的 `(self self)` 调用会导致死循环，因为为了计算 `(part-factorial part-factorial)`(在 `factorial` 的定义里)，就需要先计算 `(part-factorial part-factorial)`(在 `let` 表达式里面)。(玩一下：找出为什么对于前一个定义这不是一个问题。)。我准备就到这了，因为我想定义的就是一个
惰性的Y combinator，但是下一节我会用之前同样的方式解决这个问题（通过把 `(self self)`的调用包在一个 `lambda` 里面）。注意在惰性求值的语言里面， `(self self)`调用永远不会求值，除非确实需要 `f`(比如，如果 `n = 0`，就不需要 `f`来计算结果，所以 `(self self)`不会被执行)。理解惰性求值的语言怎么执行表达式不是很轻松，所以如果你觉得有点疑惑也不要担心。建议你打开DrRacket（原文是DrScheme，现在已经改名）的惰性Scheme语言选项，来试验这里的代码，这样你就可以更好地理解这里发生了什么。

事实证明使用下面的等式，任一 `let` 表达式都可以被转换成等价的 `lambda` 表达式：

``` Scheme
(let ((x <expr1>)) <expr2>)
==> ((lambda (x) <expr2>) <expr1>)
```

这里的 `<expr1>` 和 `<expr2>` 可以是任意的Scheme表达式。（我在这里只考虑了只有一个绑定的 `let` 表达式和只有一个参数的 `lambda` 表达式，但是这个转换对有多个绑定的 `let` 表达式和多个参数的 `lambda` 表达式都是通用的），所以我们就能得到：

``` Scheme
(define (part-factorial self)
  ((lambda (f)
    (lambda (n)
      (if (= n 0)
          1
          (* n (f (- n 1))))))
   (self self)))

(define factorial (part-factorial part-factorial))
(factorial 5) ==> 120
```

如果你仔细看，就会发现我们的老朋友 `almost-factorial` 被嵌在 `part-factorial` 函数里面，让我们把它拿出来：

``` Scheme
(define almost-factorial
  (lambda (f)
    (lambda (n)
      (if (= n 0)
          1
          (* n (f (- n 1)))))))
          
(define (part-factorial self)
  (almost-factorial
    (self self)))

(define factorial (part-factorial part-factorial))
(factorial 5) ==> 120
```

我不知道你是什么感觉，反正我是受够了 `(part-factorial part-factorial)` 这样的表达式，我不准备在写它了，幸运的是，我也不需要在写了，我可以先把 `part-factorial` 函数重写成这样：

``` Scheme
(define part-factorial
  (lambda (self)
    (almost-factorial
      (self self))))
```
然后就可以把 `factorial` 重写成这样：

``` Scheme
(define factorial
  (let ((part-factorial (lambda (self)
                          (almost-factorial
                            (self self)))))
    (part-factorial part-factorial)))

(factorial 5) ==> 120
```

`factorial` 可以写的更简洁一点，把 `part-factorial` 改成 `x` （因为我们已经不需要再其他地方用这个名字了）：

``` Scheme
(define factorial
  (let ((x (lambda (self)
             (almost-factorial (self self)))))
    (x x)))
```

现在我们把同样的 `let` 转换成 `lambda` 的方法应用到上面：

``` Scheme
(define factorial
  ((lambda (x) (x x))
   (lambda (self)
     (almost-factorial (self self)))))
     
(factorial 5) ==> 120
```

同样地，为了让这个定义更简洁，我们可以用 `x` 代替 `self`:

``` Scheme
(define factorial
    ((lambda (x) (x x))
         (lambda (x)
                (almost-factorial (x x)))))
                
(factorial 5) ==> 120
```

注意这里 `factorial` 定义里的两个 `lambda` 表达式都是关于 `x` 的函数，但是这两个x并不互相冲突。实际上我们可以把 `self` 改成 `y` 或者其他任何名字，不过下面就能看到用x更方便。

我们马上就要完成了！上面的表达式已经可以很好的工作了，但是它只针对于 `factorial` 函数。我们把它改成一个通用的  `make-recursive` 函数，可以从一个非递归函数生成一个递归的函数（听着很熟悉？）：

``` Scheme
(define (make-recursive f)
  ((lambda (x) (x x))
   (lambda (x) (f (x x)))))

(define factorial (make-recursive almost-factorial))

(factorial 5) ==> 120
```

`make-recursive` 函数就是我们一直想要的惰性求值的Y combinator，又称 *应用序Y combinator*，我们把它改成Y：

``` Scheme
(define (Y f)
  ((lambda (x) (x x))
   (lambda (x) (f (x x)))))

(define factorial (Y almost-factorial))
```

把Y的定义在展开一点：

``` Scheme
(define Y
  (lambda (f)
    ((lambda (x) (x x))
     (lambda (x) (f (x x ))))))
```

我们可以把中间那个 `lambda` 表达式应用到它的参数上（最后一个lambda），得到另一个版本的Y：

``` Scheme
(define Y
  (lambda (f)
   ((lambda (x) (f (x x)))
    (lambda (x) (f (x x))))))
```

这就意味着，给定一个函数 `f` （一个像 `almost-factorial` 这样的非递归函数），对应的递归函数可以通过先计算 `(lambda (x) (f (x x)))`，然后把这个 `lambda` 表达式应用给自身来得到。这就是一般的应用序Y combinator的定义。

剩下的事就是检查这个Y combinator是一个固定点组合子（只有是固定点组合子它才能计算出正确的结果）。为了验证这个，我们需要演示下面的等式是正确的：

<p class="indent">(Y f) = (f (Y f))</p>

从上面惰性求值的Y combinator定义可以知道：

<p class="indent">(Y f)

= ((lambda (x) (f (x x )))
   (lambda (x) (f (x x))))
</p>

把第一个lambda表达式应用到它的参数上，也就是第二个lambda表达式，就能得到：

<p class="indent">
= (f ((lambda (x) (f (x x)))
      (lambda (x) (f (x x)))

= (f (Y f))
</p>

和预想的一样。所以它不仅是一个惰性求值的Y combinator，也是一个固定点组合子。它就是最明显的一个固定点组合子，所以刚才的证明一点也不重要。

如果你自己做完了这些推导，你可以好好休息一下了，休息好以后，我们就可以开始推导...

## 严格求值（应用序）Y combinator

我们看一下之前不能再严格求值语言里使用的推导：

``` Scheme
(define (part-factorial self)
  (lamba (n)
    (if (= n 0)
        1
        (* n ((self self) (- n 1))))))
        
((part-factorial part-factorial) 5) ==> 120
(define factorial (part-factorial part-factorial))
(factorial 5) ==> 120
```
到目前为止还都可以在严格求值的语言里工作。如果我们像之前一样把 `(self self)`拿到一个 `let` 表达式里面，就有：

``` Scheme
(define (part-factorial self)
  (let ((f (self self)))
    (lambda (n)
      (if (= n 0)
          1
          (* n (f (- n 1)))))))

(define factorial (part-factorial part-factorial))
(factorial 5) ==> 120
```

上面已经说过，这样就不能再严格求值的语言里工作了，因为当 `factorial` 被调用的时候，它就要执行 `(part-factorial part-factorial)` ，然后就要执行 `let` 表达式里的 `(self self)` 部分，再这种情况下就是 `(part-factorial part-factorial)`，陷入一个 `(part-factorial part-factorial)` 的死循环中。

我们上面已经知道，解决这种问题的办法就是意识到我们尝试执行的是一个接受一个参数的函数。现在这种情况， `(self self)` 就是一个接受一个参数的函数（它和 `(part-factorial part-factorial)` 是一样的，就是 `factorial` 函数）。我们可以用一个lambda表达式把他包装起来，等到一个等价的函数：

``` Scheme
(define (part-factorial self)
  (let ((f (lambda (y) ((self self) y))))
    (lambda (n)
      (if (= n 0)
          1
          (* n (f (- n 1)))))))
```

我们这里做的只是把 `(self self)`，接受一个参数的函数，转换成 `(lambda (y) ((self self) y)`，一个等价的只接受一个参数的函数（我们之前已经看过这个方法）。我用了 `y` 而不是 `x` 来作为新的lambda表达式中的绑定变量，是为了后面要把 `self` 重命名成 `x` 的时候不产生冲突，不过我也可以用其他名字。

在我们完成这一步以后， `part-factorial` 函数现在已经可以在一个严格求值的语言里工作。因为一旦 `(part-factorial part-factorial)` 被执行，执行过程中的 `let` 表达式 `(lambda (y) ((self self) y))` 也会被执行。但是和之前不同的是，这一次我们不会陷入死循环，lambda表达式在应用到它的参数之前并不会被执行。这个lambda包装器并没有改变包装的东西，但是却延迟了被包装函数的执行。这就是我们为了让 `part-factorial` 能够在严格求值的语言里工作需要的一切。

这就是我们用的方法，之后我们继续其他推导步骤，就能得到严格求值的Y combinator的定义：

``` Scheme
(define Y
  (lambda (f)
    ((lambda (x) (f (lambda (y) ((x x) y))))
     (lambda (x) (f (lambda (y) ((x x) y)))))))
```

也可以写成这种等价形式：

``` Scheme
(define Y
  (lambda (f)
    (lambda (x) (x x)
      (lambda (x) (f (lambda (y) ((x x) y)))))))
```

希望你能看出为什么他们是等价的。着两个都是严格求值的Y combinator，或者按照技术文献里的叫法*应用序Y combinator*。在一个严格求值的语言里面（像标准的Scheme）你可以按通常的方式用这个来定义阶乘函数：

``` Scheme
(define factorial (Y almost-factorial))
```

我建议你用DrRacket尝试一遍，感受一下应用序Y combinator令人震撼的力量，可以在递归还没存在的时候创建递归。















