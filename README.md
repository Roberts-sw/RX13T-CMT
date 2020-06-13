# RX13T-CMT
estimated code execution time
---

The CMT can be used together with a time()-macro like the Tcl time instruction
to estimate execution time of a code fragment, but it will give strange results
when single-stepping through the source code.

To proof the odd behaviour during single-step, I made a large clock divider so
the actual results of time() are useless, but indicate that the CMT is running
for far more than just the instructions being single-stepped.

And of course I only made the macro for a single run of code, not with an optional
repeat count as in Tcl, but enough for estimation.

During run, or run to a breakpont after the time() macro, it always timed
correctly, for the few dozen times I have tested it.

My browser refused to upload code to
[Renesas RX forum](http://renesasrulz.com/rx/f/rx---forum/16351/cmt-timer-in-debug-mode),
so I decided to upload it here and link to it.
