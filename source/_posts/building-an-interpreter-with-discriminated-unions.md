---
title: 'TypeScript: Building an interpreter with discriminated unions'
url: 591.html
id: 591
categories:
  - TypeScript
date: 2018-01-17 08:00:13
tags:
---

In the [previous post](https://codewithstyle.info/typescript-discriminated-union-types) I've explained what **discriminated unions** are in the TypeScript language. We'll now look into a classic example of how this concept can be applied to a real-world scenario - building an interpreter. ![](/images/2018/01/Precise-domain-modelling-with-discriminated-unions-—-kopia.png)

Abstract Syntax Tree
--------------------

By _interpreter_ I mean a program that can execute source code of another program written in a different programming language. For example, JavaScript, PHP or Python are interpreted languages - you need an interpreter to run them. We will look into building an interpreter for a very simple _language_ \- algebraic expressions, such as:

    -5 * (1 + (3 / 6))
    

When building an interpreter (or a compiler) you need to build some data structures that represent the source code. These data structures form something called **Abstract Syntax Tree** (AST). It's a tree-like structure because the source code usually involves lots of nesting. The AST of our example algebraic expression would look like this: ![](/images/2018/01/AST.png) You can see that the tree structure corresponds to the order in which mathematical operations are applied. For example, `3 / 6` should be calculated first and that's why it's low in the tree. The multiplication will be evaluated at the very end and hence it's the tree's root. Discriminated unions are perfect for representing such a tree. Our AST has different kinds of nodes:

*   binary operators - addition, subtraction, multiplication and division
*   unary operators - negation
*   numerical values

Let's create some types that correspond to these kinds.

    interface BinaryOperatorNode {
        kind: "BinaryOperator";
        left: ExpressionNode;
        right: ExpressionNode;
        operator: "+" | "*" | "-" | "/";
    }
    
    interface UnaryOperatorNode {
        kind: "UnaryOperator",
        operator: "+" | "-";
        inner: ExpressionNode;
    }
    
    interface NumberNode {
        kind: "Number";
        value: number;
    }
    
    type ExpressionNode = BinaryOperatorNode | UnaryOperatorNode | NumberNode;
    

We've created the `ExpressionNode` discriminated union. It's a union of three node kinds:

*   `NumberNode` is the simplest type - it contains a numerical value
*   `BinaryOperatorNode` represents an operation on two expressions; the `operator` field determines what kind of operations it is; `left` and `right` fields contain arguments of the operation
*   `UnaryOperatorNode` is like `BinaryOperatorNode` but only contains a single argument

Interestingly, these types are _recursive_ in a way. For example, `UnaryOperatorNode` has `inner` property which can be any `ExpressionNode` \- indeed, we can negate a number, but also a much more complex expression wrapped in parenthesis.

Evaluating the AST
------------------

Now we have data structures in place there are only two questions left:

*   How to create an AST from the source code? This is actually a task for a **parser**. Parsers are beyond the scope of this article. Interestingly, you can **generate** a parser using [Jison](https://github.com/zaach/jison) or some other tool.
*   How to evaluate the AST? Let's look into this.

We've already seen how to consume discriminated unions with a `switch` statement. This is no different here with the exception that since our types are recursive, the evaluation function will be recursive as well.

    function evaluate(expression: ExpressionNode): number {
        switch (expression.kind) {
            case "Number": 
                return expression.value
    

The `evaluate` function takes an `ExpressionNode` object and evaluates it to a number. First, we check for an instance of `NumberNode` in which case the expression will simply evaluate to the value held by the object.

            case "UnaryOperator":
                const innerValue = evaluate(expression.inner);
                return expression.operator === "+" ? innerValue : -innerValue;
    

For `UnaryOperatorNode` we need to evaluate the value of the nested expression first. That's why we make a recursive call to `evaluate` itself. Once this is done we either negate or simply return it as it is.

            case "BinaryOperator":
                const leftValue = evaluate(expression.left);
                const rightValue = evaluate(expression.right);
                switch (expression.operator) {
                    case "+": return leftValue + rightValue;
                    case "-": return leftValue - rightValue;
                    case "*": return leftValue * rightValue;
                    case "/": return leftValue / rightValue;
                }
        }
    }
    

`BinaryOperatorNode` has two nested expression - `left` and `right` \- so we need to evaluate both of them. Next, we use the relevant mathematical operation on them to calculate the final value. Here is the full source code of the `evaluate` function:

    function evaluate(expression: ExpressionNode): number {
        switch (expression.kind) {
            case "Number": 
                return expression.value
            case "UnaryOperator":
                const innerValue = evaluate(expression.inner);
                return expression.operator === "+" ? innerValue : -innerValue;
            case "BinaryOperator":
                const leftValue = evaluate(expression.left);
                const rightValue = evaluate(expression.right);
                switch (expression.operator) {
                    case "+": return leftValue + rightValue;
                    case "-": return leftValue - rightValue;
                    case "*": return leftValue * rightValue;
                    case "/": return leftValue / rightValue;
                }
        }
    }
    

Finally, we need an object instance for testing. Here is the `(42 + 5) * -12` expression represented as an AST:

    const expr1: ExpressionNode = {
        kind: "BinaryOperator",
        operator: "*",
        left: {
            kind: "BinaryOperator",
            operator: "+",
            left: {
                kind: "Number",
                value: 42
            },
            right: {
                kind: "Number",
                value: 5
            }
        },
        right: {
            kind: "UnaryOperator",
            operator: "-",
            inner: {
                kind: "Number",
                value: 12
            }
        }
    };
    

Now we can test our `evaluate` function:

    console.log(evaluate(expr1));
    

Voila, we've just finished our first interpreter written in TypeScript using discriminated unions.

Summary
-------

In this article, we've looked into taking advantage of discriminated unions in a non-trivial, real-world scenario - implementing a language interpreter. It's easy to imagine how the types used to build an AST can be much more complex. For a real programming language, we would have to create types for statements, method calls, function declarations, etc. However, the principle would be roughly the same.