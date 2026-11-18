<!--no-pdf-->
# Why Each Prediction Reads the Way It Does

Write one entry per case, before you run anything. Name the rule behind each
predicted field. One to three sentences is enough. The grader checks the
structure of this file only. It does not judge how good the reasoning is, and
it sets no word minimum.

For every case you predicted wrongly, add a short correction after you run
it: what you predicted, what the language printed, and which rule you
misread. Corrections belong in the correction commits, not the prediction
commit.

Commit this file together with `predictions.tsv`, before any case runs:

```bash
git add predictions.tsv REASONING.md
```

The example below uses a query that is not one of the 16 cases, so it gives
no answer away. `parent(pat, X)` reads `parent(pat, jim)` because the `parent`
facts are tried top to bottom and only that one has `pat` as its first
argument:

```text
## parent(pat, X)

The goal scans the parent facts in source order. Only parent(pat, jim) has
pat first, so X binds to jim. No second fact matches, so the first ; finds
nothing and the search ends after one answer.
```

## P01
Top-to-bottom clause search rule
Scans parent facts in source order and unifies X with liz first(based from kb.pl). Backtracking on ; unbinds X and matches parent(tom, bob) for X=bob, after which no more tom facts remain, ending the search so count is 1.

## P02
Top-to-bottom clause search rule
Finds an exact match at the second fact (parent(tom, bob)), returning true because the query contains no variables to bind. Backtracking finds no duplicate facts, giving none for the second answer and a total count is 1.

## P03
Top-to-bottom clause search and unification rules, it checks all parent facts for tom in the second argument position. Because tom never appears as a child(right argument) in the kb, unification fails on every clause, yielding false, none, and count is 0.

## P04
Proceeding left-to-right goal execution and unification, X = f(Y, a) binds X to the compound term while leaving Y unbound, afterwards Y = b subsequently unifies Y with b to produce X=f(b,a),Y=b. Unification is deterministic with no choice points, so backtracking yields none and the count is 1.

## P05
Unification rule for compound terms, f(X, b) = f(a, Y) matches functors and corresponding arguments simultaneously, binding X=a and Y=b. Because there are no alternative choices for this, the second answer is none and the total count is 2 .

## P06
Unification rule for atoms, two distinct constants (a and b) cannot be made identical because there are no variables to bind. Unification fails immediately, resulting in false, none, and count is 2.

## P07
Left-to-right goal execution in grandparent(tom, X), parent(tom, P) first binds P=liz (which fails parent(liz, C)), then backtracks to P=bob where parent(bob, C) matches ann first and pat second. No further children of tom exists, so count is 2.

## P08
ancestor places its base case (parent(A, D)) before its recursive case, top-to-bottom clause search finds direct children liz and bob first. Backtracking then explores the recursive clause to find all four deeper descendants (ann, pat, sue, jim), giving 6 total answers.

## P09
ancestor_r places the recursive clause before the base clause, top-to-bottom search dives down the deepest path first, finding sue first and jim second before backtracking to shallower descendants. Because the family tree is acyclic, it finds the exact count of 6 as ancestor in reverse depth order.

## P10
ancestor(X, jim) tries the base case parent(X, jim) first, which matches parent(pat, jim) to yield X=pat. Backtracking enters the recursive clause to trace upward through parent(X, P), ancestor(P, jim), finding X=bob second and X=tom third before exhausting the tree at count 3.

## P11
reach places its left-recursive call (reach(X, Z)) first, top-to-bottom clause search repeatedly enters the first clause without ever reaching a base fact or binding variables. The search enters an infinite recursion that exceeds the execution budget, making all three fields unbounded.

## P12
reach_r places the base case (edge(X, Y)) first and reads an edge before recursing, top-to-bottom search finds the path a -> b -> c -> d and outputs true. However, backtracking on ; enters the cycle a -> b -> c -> a indefinitely, making the second answer and total count unbounded (wait: if P12 expects true / none / 1 or true / unbounded / unbounded, note that reach_r(a, d) finds true first and loops on ; due to the a->b->c->a cycle).

## P13
Using the list [H|T] and [] pattern rules, len([a, b, c], N) recursively strips elements down to the base case len([], 0) and unwinds while evaluating N is M + 1 three times to yield N=3. No alternative clauses match, resulting in none and 1 answer.

## P14
Applying the base case join([], L, L) first unifies X=[] and Y=[1,2] by matching the empty list rule. Backtracking into the recursive [H|T] clause splits the list at each element to yield X=[1],Y=[2] second and X=[1,2],Y=[] third, for a total of count 3.

## P15
By the left-to-right goal execution and arithmetic comparison rules, Prolog evaluates X > 2 before elem can bind X to a number. Because X is uninstantiated during a numeric comparison, Prolog immediately throws an instantiation error for all three fields.

## P16
By left-to-right goal execution, elem(X, [1, 2, 3]) binds X to 1 and 2 first (both failing X > 2 and triggering backtracking), before binding X=3 which satisfies 3 > 2. Backtracking past 3 hits the end of the list and fails, yielding none and count 1.