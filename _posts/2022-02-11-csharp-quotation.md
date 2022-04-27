---
title: C# quotations
date: "2022-04-26 19:00:00 +0300"
id: csharp-quotations
excerpt: "Using expression trees to symbolic calculation of derivative functions."
mathjax: true
---

> В этом году весенний DotNext перенесли на неопределённый срок. А я к нему готовлюсь.

This year spring's DotNext was moved to undefined time. Buy I'm preparing the material for it.

> В рамках доклада про разные интересности в C#, разработал небольшую библиотеку для генерации
> производных функций.

While I've been learning different interesting parts of C#, I made a small library to symbolic calculation of derivative functions.

> ### Задача

### The Problem

> Саму задачу я встретил, решая упражнения из великой книги [Структура и Интерпретация Компьютерных Программ](https://ru.wikipedia.org/wiki/Структура_и_интерпретация_компьютерных_программ).
> Обычно её называют SICP (читается *сик-пи*) — это аббревиатура названия на английском языке.

I have found the problem when I've been solving excersices from great book [Structure and Interpretation of Computer Programs](https://en.wikipedia.org/wiki/Structure_and_Interpretation_of_Computer_Programs#:~:text=Structure%20and%20Interpretation%20of%20Computer%20Programs%20(SICP)%20is%20a%20computer,Wizard%20Book%22%20in%20hacker%20culture.).
Usually it called SICP (pronounced as *sick pea*).

> Раздел 2.3 посвящён *цитированию* в LISP и *символическим вычислениям*.

The section 2.3 talks about *the quotation* and *the symbolic calculations*.

> Обычные — несимволические — вычисления сводятся к тому, что мы считаем какие-то величины с
> помощью арифметических операций. Например, если я попрошу вас вычислить производную функции
> $x^2$ в точке $x = 17$, вы можете сделать это по формуле:

Usual (non-symbolic) calculations mean that we calculate some values with a help of arithmetics operations. For example, if I ask you to calculate derivative function $x^2$ in the pont $x = 17$
you can do it with the formula:

$$
(x^2)' = \frac{(x+dx)^2-x^2}{dx}
$$

> при каком-нибудь не очень большом значении $dx$:

with some not so large value of $dx$:

$$
\frac{(17+0.0001)^2-17^2}{0.0001} = 34.0001000001
$$

> Подгоняя $dx$, мы можем вычислить производную с хорошей точностью. Символические же вычисления
> позволяют нам применить правила выведения производных и получить значение абсолютно точно.

Changing $dx$ we can calculte the derivative with a good accuracy. Symbolic calculations let us use the rules of derive of derivatives and get the result with the absolute accuracy.

$$
(x^2)' = 2x
$$

> При $x = 17$ значение производной будет абсолютно точно равно $34$.

The value of the derivative will be exactly $34$ when $x = 17$.

> ### Реализация на LISP

### LISP Implementation

> SICP предлагает решать задачу вычисляения производной с помощью *цитирования*.
> По английски этот механизм называется *quotation*.

SICP proposes to solve the problem of derivative calculation with a help of *quotation*.

> Если мы вводим в интепретатор LISP любое выражение:

If we type any expression in the interpreter of LISP:

```scheme
(+ (/ 1 1) (/ 1 1) (/ 1 2) (/ 1 6) (/ 1 24) (/ 1 120) (/ 1 720) (/ 1 5040))
; => 2.7182539682539684
```

the interpreter calculates immediately. But if we start the expression with a *quote* (')
then LISP stores it as a lisp without calcualtion:

> он его сразу вычисляет. Но если мы предваряем его *кавычкой* (*quote*), LISP сохраняет
> его в виде списка, не вычисляя:

```scheme
'(+ (/ 1 1) (/ 1 1) (/ 1 2) (/ 1 6) (/ 1 24) (/ 1 120) (/ 1 720) (/ 1 5040))
; => (+ (/ 1 1) (/ 1 1) (/ 1 2) (/ 1 6) (/ 1 24) (/ 1 120) (/ 1 720) (/ 1 5040))
```

> Таким образом, мы можем получить корректное выражение на LISP и обработать его,
> как любой другой список, в частности, преобразовать по правилам вычисления производной.

So we can get the valid LISP expression and process it like any other list. Particulary we can transform it using rules of deriv of derivatives.

> Кажется, что такая функция преобразования может быть сложной. Если реализовывать разные
> особоые случае, то да. Но хорошие результаты даёт уже достаточно простая функция:

It seems that such transformation function can be complicated. And yes, it can, if we want implement differenct specific cases. But we can get eough good results with the simple function:

```scheme
(define (variable? x) (symbol? x))
(define (same-variable? v1 v2)
  (and (variable? v1) (variable? v2) (eq? v1 v2)))
(define (make-sum a1 a2) (list '+ a1 a2))
(define (make-product m1 m2) (list '* m1 m2))
(define (sum? x)
  (and (pair? x) (eq? (car x) '+)))
(define (addend s) (cadr s))
(define (augend s) (caddr s))
(define (product? x)
  (and (pair? x) (eq? (car x) '*)))
(define (multiplier p) (cadr p))
(define (multiplicand p) (caddr p))

(define (deriv exp var)
  (cond ((number? exp) 0)
        ((variable? exp)
         (if (same-variable? exp var) 1 0))
        ((sum? exp)
         (make-sum (deriv (addend exp) var)
                   (deriv (augend exp) var)))
        ((product? exp)
         (make-sum
          (make-product (multiplier exp)
                        (deriv (multiplicand exp) var))
          (make-product (deriv (multiplier exp) var)
                        (multiplicand exp))))
        (else
         (error "Unknown expression type"))))
```

> Очевидной недоработкой функции является сложность получаемых выражений.

Obvious problem of the function is that result expressions are too complicated.

```scheme
(deriv '(+ x 3) 'x)
; => (+ 1 0)

(deriv '(* x y) 'x)
; => (+ (* x 0) (* 1 y))

(deriv '(* (* x y) (+ x 3)) 'x)
; => (+ (* (* x y) (+ 1 0)) (* (+ (* x 0) (* 1 y)) (+ x 3)))
```

> Их надо упрощать, для чего может быть написана отдельная функция.

We need to simplify them, so we need different simplification fucntion.

> Упрощение выражений рассматривается в SICP, но не очень подробно.

SICP discuss the topic of simplification but without details.

> ### Реализация на F#

### F# Implementation

> Цитирование на F# всё ещё похоже на цитирование:

Quotation in F# still looks like quotation:

```fsharp
let expSquare = <@ fun x -> x * x @>
// => val expSquare : Quotations.Expr<(int -> int)> = Lambda (x, Call (None, op_Multiply, [x, x]))
```

> Чтобы получить вместо кода его представление в виде сложного объекта, заключим код в
> своеобразные кавычки — **<@** и **@>**.

To get the representation of the code instead of the code, we have to enclose the code in special quotes — **<@** and **@>**.

> Результатом будет значение типа `Expr`, с которым можно работать также, как с
> деревьями выражений в C#. Вот простой код генерации производной функции:

The result will have the value of the type `Expr`, and we can process it like expression trees in C#. Here simple code to generate derivative functions:

```fsharp
open Microsoft.FSharp.Quotations
open Microsoft.FSharp.Quotations.Patterns
open Microsoft.FSharp.Quotations.DerivedPatterns

let  make_sum left right =
    let left = Expr.Cast<float> left
    let right = Expr.Cast<float> right 
    <@ %left + %right @> :> Expr
    
let make_prod left right =
    let left = Expr.Cast<float> left
    let right = Expr.Cast<float> right 
    <@ %left * %right @> :> Expr

let deriv (exp: Expr) =
    match exp with
    | Lambda(arg, body) ->
        let rec d exp =
            match exp with
            | Int32(_) ->
                Expr.Value 0.0
            | Var(var) ->
                if var.Name = arg.Name
                then Expr.Value 1.0
                else Expr.Value 0.0
            | Double(_) ->
                Expr.Value 0.0
            | SpecificCall <@ (+) @> (None, _, [left; right]) ->
                make_sum (d left) (d right)
            | SpecificCall <@ (*) @> (_, _, [left; right]) ->
                let left = Expr.Cast<float> left
                let right = Expr.Cast<float> left
                make_sum (make_prod left (d right)) (make_prod (d left) right)
            | _ -> failwith "Unknown expression type"

        d body
    | _ -> failwith "Expr.Lambda expected"

<@ fun (x: double) -> x * x @>
// => Lambda (x, Call (None, op_Multiply, [x, x]))

deriv <@ fun (x: double) -> x * x @>
// => Call (None op_Addition,
//          [Call (None, op_Multiply, [x, Value (1.0)]),
//           Call (None, op_Multiply, [Value (1.0), x])])
```

> ### Реализация на C#

### C# Implementation

> В C# существует аналог *цитирования* — *деревья выражений*. В отличие от F#, здесь нет
> специальных кавычек для выделения кода. Вместо этого вы указываете тип выражения `Expression`.

C# has analogue of *quotation* that called *expression trees*. Unlike F# there are not special quotes to enclose the code. Instead of quotes you have to set type of expression to `Expression`.

> Обычные выражения вычисляются сразу.

The usual expressions are calculated immediately:

```c#
Func<double, double> square = x => x * x;
sqaure(2) // 4
```

> Выражения, которые приводятся к типу
> [`Expression`](https://docs.microsoft.com/en-us/dotnet/api/system.linq.expressions.expression),
> складываются в древовидную структуру, которую мы можем обрабатывать.

The expressions that cast to the type `Expression` are stored in a tree-structure that we can process.

```c#
Expression<Func<double, double>> expSquare = x => x * x;
expSquare.Compile()(2) // 4
```

> Деревья выражений хорошо знакомы многим программистам на C#, поскольку они активно
> применяются в библиотеке Entity Framework. Однако, с помощью их можно делать и более
> сложную обработку.

Expression trees are well known to many C# programmers, cause they are used in Entity Framework library. However, they can help in more complicated cases too.

> Вот функция, которая получает на вход лямбда-функцию и применяет её к самой себе.

Here the function that get lamda-function and apply it to itself.

```c#
static Expression<Func<double, double>> DoubleFunc(Expression<Func<double, double>> f)
{
    var parameter = Expression.Parameter(typeof(double));
    var inner = Expression.Invoke(f, parameter);
    var outer = Expression.Invoke(f, inner);
    return Expression.Lambda<Func<double, double>>(outer, parameter);
}

var expFourth = DoubleFunc(expSquare);
```

> Если два раза применить функцию возведения в квадрат к какому-то числу, мы получим
> возведение в четвёртую степень:

If apply square function twice, we get the fourth power function.

```c#
expFourth.Compile()(2) // 16
```

> Я разработал небольшой [пакет](https://github.com/markshevchenko/sysharp),
> который умеет генерировать производные функции по деревьям выражений.

I have made the small [package](https://github.com/markshevchenko/sysharp)
that can generate derivative fucntions from expression trees.

```c#
Symbolic.Derivative(x => x * x).ToString()
// => x => ((x * 1) + (1 * x))
```

> Там же реализован код для упращения выражений:

The simplification function is implemented too:

```c#
Symbolic.Derivative(x => x * x).Simplify().ToString()
// => x => (2 * x)
```

> В отличие от F#, в C# очень просто из дерева выражения получить работающий код:

Unlike F#, C# let us get working function from expression tree without difficults:

```c#
var d = (Func<double, double>)Symbolic.Derivative(x => x * x).Compile();
d(17)
// => 34
```