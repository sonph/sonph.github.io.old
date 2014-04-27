---
layout: post
title: Exact inference in Bayesian networks
category: code
tags: python ai probability code
---
<br>
{% include fig.html src="/assets/img/seashell.png" alt="xkcd 1236 - Seashell" title="xkcd 1236 - Seashell" caption="Xkcd 1236 - Seashell" %}
<br>

This is one of the programming assignments I did in the Introduction to Artificial Intelligence course (CS4365). It implements two algorithms for performing *exact inference* given a *[Bayesian network](http://en.wikipedia.org/wiki/Bayesian_network)*, namely *variable enumeration* and *variable elimination*. These exact inference algorithms produce, well, exact probability distribution over the *query variable* given the *observed evidences*, as compared to the method of *sampling* which gives an approximate result.

Source code is available <a href="https://github.com/sonph/bayesnetinference" target="_blank">here</a>.

## The program

The program was written in Python 3 and only uses modules in the standard library. The program accepts exactly three command line arguments:

    BayesNet.py <file> <enum|elim> <query>

where:

- `file` is the name of the file containing the Bayesian network in the form of conditional probability tables (see below for the BNF notation of the file).

- `enum|elim` specifies the algorithm to be used, either `enum` for enumeration or `elim` for elimination.

- `query` is the query to be executed, for example `"P(B)"` or `"P(A|E=t,M=f)"` (double quotes are needed to pass the raw string to the program). At the moment the query on takes a single query variable and zero or more evidence variables.

## File format

The format for the file that specifies a Bayesian network is as follow in BNF notation:

    <file> ::= <var-prob> (<newline> <newline> <var-prob>)*
    <var-prob> ::= <prior-prob> | <prob-table>
    <prior-prob> ::= "P(" <var> ") = " <decimal>
    <prob-table> ::= <header> <newline> <horiz-sep> <newline>
    <entry-list>
    <header> ::= <var> (" " <var>)* " | " <var>
    <horiz-sep> ::= "-"+ "|" "-"+
    <entry-list> ::= <table-entry> (<newline> <table-entry>)+
    <table-entry> ::= <truth-symb> (" " <truth-symb>)* " | "
    <decimal>
    <truth-symb> ::= "t" | "f"
    <decimal> ::= "0" | "0"? "." <digit>+
    <digit> ::= "0"|"1"|"2"|"3"|"4"|"5"|"6"|"7"|"8"|"9"
    <var> ::= "A"|"B"|"C"|"D"|"E"|"F"|"G"|"H"|"I"|"J"|"K"|"L"|"M"|"N"|
    "O"|"P"|"Q"|"R"|"S"|"T"|"U"|"V"|"W"|"X"|"Y"|"Z"
    <newline> ::= "\n"

Here's an example (`alarm.bn` in the repository):

    P(B) = .001

    P(E) = .002

    B E | A
    ----|-----
    t t | .95
    t f | .94
    f t | .29
    f f | .001

    A | J
    --|-----
    t | 0.90
    f | 0.05

    A | M
    --|-----
    t | 0.7
    f | 0.01

This file indicates that this particular Bayesian network consists of 5 variables, `B`, `E`, `A`, `J`, and `M`. The first line says that `B` is `true` with probability `.001`. We can infer that `B` is `false` with probability `1 - .001 = .99`. Same thing for `E`. The next table following `E` says that `A` is a child of `B` and `E` in the network. So looking at the table we can say that, if `B` is `true` and `E` is `false`, then `A` is `false` with probability `1 - .94 = .06`.

{% include fig.html src="/assets/img/burglary.png" caption="Visual representation of the network" %}

## The algorithms

Here are the pseudocodes that I used for the two algorithms:

- Variable enumeration

    ```
    function ENUMERATION-ASK(X, e, bn) returns a distribution over X
        inputs: X, the query variable
                e, observed values for variables E
                bn, a Bayesian network with variables {X} ∪ E ∪ Y

        Q(X) <- a distribution over X, initially empty
        for each value of x_i for X             // values = false, true
            extend e with value x_i for X       // add X = x_i to e
            Q(x_i) <- ENUMERATE-ALL(VARS[bn], e)
        return NORMALIZE(Q(X))

    function ENUMERATE-ALL(vars, e) returns a real number
        if EMPTY?(vars) then return 1.0
        Y <- FIRST(vars)
        if Y has value y in e
            then return P(Y=y|e) * ENUMERATE-ALL(REST(vars), e)
            else
                sum <- 0
                for each value y of Y          // values = false, true
                    extend e with value y for Y
                    sum <- sum + (P(Y=y|e) * ENUMERATE-ALL(REST(vars), e))
                return sum
    ```

- Variable elimination
    
    ```
    function ELIMINATION-ASK(X, e, bn) returns a distribution over X
        inputs: X, the query variable
                e, observed values for variables E
                bn, a Bayesian network with variables {X} ∪ E ∪ Y
        factors <- empty list
        eliminated <- empty set
        loop
            vars = [variables in VARS(bn) whose children are all in eliminated]
            var = SORT-BY-FACTOR-SIZE(vars, bn)
            // var is the variable that would create a smallest factor

            factors <- factors + MAKE-FACTOR(var, e)
            if var is in Y then
                relevant-factors <- [all factors that contain var]
                factors.remove(relevant-factors)
                factors <- factors + SUM-OUT(var, POINTWISE-PRODUCT(var, relevant-factors))
            add var to eliminated
        return NORMALIZE(POINTWISE-PRODUCT(factors))
    ```


## Sample output

Here's a sample output of the program. Suppose that for the above sample network, we observed that an earthquake has happened. Now we want to find the probability that the alarm will sound. So the query is `P(A|E=t)`, meaning *find the probability that the alarm `A` will sound in the case of an earthquake*.

    % ./BayesNet.py alarm.bn enum "P(A|E=t)"
    M              | B=t J=t A=f E=t      = 1.00000000
    M              | B=t J=f A=f E=t      = 1.00000000
    J M            | B=t A=f E=t          = 1.00000000
    A J M          | B=t A=f E=t          = 0.05000000
    E A J M        | B=t A=f E=t          = 0.00010000
    M              | B=f J=t A=f E=t      = 1.00000000
    M              | B=f J=f A=f E=t      = 1.00000000
    J M            | B=f A=f E=t          = 1.00000000
    A J M          | B=f A=f E=t          = 0.71000000
    E A J M        | B=f A=f E=t          = 0.00142000
    B E A J M      | A=f E=t              = 0.00141868
    M              | B=t J=t A=t E=t      = 1.00000000
    M              | B=t J=f A=t E=t      = 1.00000000
    J M            | B=t A=t E=t          = 1.00000000
    A J M          | B=t A=t E=t          = 0.95000000
    E A J M        | B=t A=t E=t          = 0.00190000
    M              | B=f J=t A=t E=t      = 1.00000000
    M              | B=f J=f A=t E=t      = 1.00000000
    J M            | B=f A=t E=t          = 1.00000000
    A J M          | B=f A=t E=t          = 0.29000000
    E A J M        | B=f A=t E=t          = 0.00058000
    B E A J M      | A=t E=t              = 0.00058132

    RESULT:
    P(A = f | E = t) = 0.7093
    P(A = t | E = t) = 0.2907

And the program responds that in the case of an earthquake, the alarm sounds with a probability of about `0.71`, or `71%`. The lines above the result show the return values from calls to `ENUMERATE-ALL`.

Here's the output for the same query using variable elimination:

    % ./BayesNet.py alarm.bn elim "P(A|E=t)"
    ----- Variable: J -----
    Factors:
    A=f: 1.0000
    A=t: 1.0000

    ----- Variable: M -----
    Factors:
    A=f: 1.0000
    A=t: 1.0000

    A=f: 1.0000
    A=t: 1.0000

    ----- Variable: A -----
    Factors:
    A=f: 1.0000
    A=t: 1.0000

    A=f: 1.0000
    A=t: 1.0000

    A=f B=f: 0.7100
    A=f B=t: 0.0500
    A=t B=f: 0.2900
    A=t B=t: 0.9500

    ----- Variable: E -----
    Factors:
    A=f: 1.0000
    A=t: 1.0000

    A=f: 1.0000
    A=t: 1.0000

    A=f B=f: 0.7100
    A=f B=t: 0.0500
    A=t B=f: 0.2900
    A=t B=t: 0.9500

    ----- Variable: B -----
    Factors:
    A=f: 1.0000
    A=t: 1.0000

    A=f: 1.0000
    A=t: 1.0000

    A=f: 0.7093
    A=t: 0.2907


    RESULT:
    P(A = f | E = t) = 0.7093
    P(A = t | E = t) = 0.2907

This algorithm produces the same result. The output also shows the order in which variables are eliminated, and all factors that are still active after the elimination of each variable.

SP