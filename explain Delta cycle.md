hardware is parallel by nature. If you have a bunch of registers the outputs of all those registers change at the same time (clock edge).
If you have "a AND b" and "a AND c", then changing a causes the outputs of both of those gates to change.

When you simulate a design, you are running it as a set of instructions, which is linear. So you can't handle multiple simultaneous events, you have to do some hackery to deal with that.

If you consider a synchronous block with:
```
a <= b;
b <= a;
```
On a clock edge the two registers swap. But if you tried to simulate that with a linear set of insthructions:
```
a = b;
b = a;
```
or vice versa, you don't get the same behaviour. Instead the tools break down events into delta cycle. They take 0 simulation time. So in the above example we could split this into two deltas, evaluation and assignment. Essentially we run through the block twice first evaluating the right hand sides, and then updating the registers:
```
delta 0:
    a_tmp = b;
    b_tmp = a;
delta 1:
    a = a_tmp;
    b = b_tmp;
```
It's more complicated than that, but that's essentially what they are. There are delta cycles for blocking assignments, non blocking assignments, combinatory blocks, assignments (outside of blocks). Then you have delays (#10 / wait), and verilog supports a #0 which delays an assignment to a different delta cycle.

It's all pretty complicated and confusing, but for the most part you don't have to worry about it, as long as you follow some simple rules for writing your RTL to avoid creating weird race conditions.
