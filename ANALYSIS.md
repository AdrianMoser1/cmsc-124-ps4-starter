# The Search and the Stack

Case P07 queries `grandparent(tom, X)`.

```text
== trace P07 ==
Call: grandparent(tom,X)
  Call: parent(tom,_)
  Exit: parent(tom,liz)
  Call: parent(liz,X)
  Fail: parent(liz,X)
  Redo: parent(tom,liz)
  Exit: parent(tom,bob)
  Call: parent(bob,X)
  Exit: parent(bob,ann)
Exit: grandparent(tom,ann)
answer 1: X=ann
Redo: grandparent(tom,ann)
  Redo: parent(bob,ann)
  Exit: parent(bob,pat)
Exit: grandparent(tom,pat)
answer 2: X=pat
Redo: grandparent(tom,pat)
  Redo: parent(bob,pat)
  Fail: parent(bob,X)
  Redo: parent(tom,bob)
  Fail: parent(tom,_)
Fail: grandparent(tom,X)
== 2 answers ==
```

## Q1. Which trace lines push and pop frames?

I'd read each `Call` as a push. `Call: grandparent(tom,X)` starts the main call, then `Call: parent(tom,_)` adds a frame for the first goal in the rule. `Exit: parent(tom,liz)` is where that goal returns with `P=liz`. The next `Call: parent(liz,X)` pushes another frame, and `Fail: parent(liz,X)` ends that call because Liz has no children. The `grandparent` frame is still active at that point. Both `parent` calls are indented one level because they run inside the `grandparent` call.

Later, `Call: parent(bob,X)` pushes a frame for Bob's children. `Exit: parent(bob,ann)` returns `X=ann` to the grandparent rule, then `Exit: grandparent(tom,ann)` gives the first answer. Those `Exit` lines are where the calls return, so this is where Activity 15 would pop their frames. Prolog can still come back to them during backtracking. The last two `parent` failures show both subgoals running out of matches. Then `Fail: grandparent(tom,X)` ends the whole search.

## Q2. What does Redo resume?

`Redo: parent(tom,liz)` is Prolog going back to the first `parent` goal after the Liz branch fails. It can do that because it saved a choice point. That choice point remembers where to look for the next fact and which bindings to undo. So Prolog clears `P=liz`, tries `parent(tom,bob)`, and checks Bob's children next.

The same thing happens when we ask for another answer after `X=ann`. `Redo: parent(bob,ann)` clears the binding to `ann` and tries `parent(bob,pat)`. The earlier binding `P=bob` stays because we're still checking Bob's children. In Activity 15, the return address only told the program where to continue in the caller, and popping the frame meant losing its local state. Prolog also has the choice point, which lets it pick up from the next alternative and undo the bindings made since that choice.
