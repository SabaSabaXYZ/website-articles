# Git REPL

## Code

The source code and compilation instructions for this application can be found on my Github page: <https://github.com/SabaSabaXYZ/git-repl>.

The README file contains compilation and run instructions for this utility.

## Summary

After using Git Skipper for a couple of years, I began to notice some issues with it.
Firstly, the command line interface used by the interactive mode did not always perform as expected in different terminals.
Sometimes the user's input would not display, or the user could not enter input properly.
Secondly, the performance of Git Skipper was abysmal when skipping a large number of files.
Finally, the limited number of commands meant that coworkers who were less experienced with Git would often end up making mistakes and asking me for help, although Git Skipper did help reduce the overall frequency of this occurrence.

I decided to take a stab at creating a new and improved utility called Git REPL.
I had heard that Common Lisp would provide improved performance over Racket.
Since Common Lisp also performs literally everything in the REPL, I figured that I would design the whole utility around the REPL.
That is, the utility itself would be a Common Lisp REPL with Git helper commands baked in.
It would also offer a single-execution mode in which Git REPL would take a single S-expression as an argument to execute and terminate.

Taking this as an opportunity to learn Common Lisp, I was able to quickly spin up a utility that was 17 times faster than Git Skipper at skipping and unskipping files.
In addition, the interactive mode was recreated with a GUI provided by Ltk, a library that offers Common Lisp bindings for Tk.
With the addition of many new higher-level commands like `git checkout-config` and `git rebase-config`, it also makes working with configurations a lot easier and fool-proof compared to Git Skipper.

The crazy part is that this utility is only 369 lines of Common Lisp code, which is not that much more than the 103 lines of Racket code used to implement Git Skipper.
What makes Git REPL so concise is the extensive use of macros.
This turned many commands into one-liners, making it trivial to add new commands when necessary.
The 17x performance boost is also due to a change in the algorithm I use to skip and unskip files.
Whereas before I would skip one file at a time, I now skip files in batches of ten at a time.
