# Git Skipper

## Code

The source code and compilation instructions for this application can be found by cloning the following Git repository: <https://sabadev.xyz/git/git-skipper.git>.
Note that this path is not accessible through HTTP, and instead requires the use of a Git client to view the code.

To clone this repository, execute the following commands in your terminal:

    git clone https://sabadev.xyz/git/git-skipper.git
    cd git-skipper
    git checkout -b main -t origin/main

The README file in this repository contains compilation and run instructions for the utility.

[Click here](http://sabadev.xyz:4321/?p=git-skipper.git;a=tree;h=refs/heads/main;hb=refs/heads/main) to view the code in your browser.

## Summary

I was on a call with a colleague when he expressed frustration with having to type long commands such as `git ls-files -v | grep ^S | cut -c3- | xargs -d "\n" git update-index --no-skip-worktree` in order to manage his configuration files.
Since I had just picked up Racket at the time, I figured solving this issue would be a fun weekend project to learn the language.

I built a tool in 103 lines of Racket code that does the following:
* Lists every skipped file and modified file
* Skips a single file
* Unskips a single file
* Skips every modified file
* Unskips every skipped file
* Exposes an interactive mode to skip and unskip files

When I sent the utility to my colleague, he commented:

> git-skipper is the best

Consult the [README file](http://sabadev.xyz:4321/?p=git-skipper.git;a=blob_plain;f=README.md;hb=refs/heads/main) for usage and compilation instructions.

Note that this utility is Windows-specific.
For unix-based systems, I recommend replacing `findstr` with `grep` (or equivalent) in [git-skipper.rkt](http://sabadev.xyz:4321/?p=git-skipper.git;a=blob_plain;f=git-skipper.rkt;hb=refs/heads/main).

## Coding with Racket

Racket ended up feeling a lot like JavaScript, albeit with a more elegant syntax.
Implementing git-skipper involved first defining some utility functions, then using these functions to build up more complex functionality.
The composability of these functions resulted in each function having a tiny implementation.
This made it much easier to add extra features such as the interactive mode, and allowed me to complete the whole project in a single weekend.

Although typed Racket exists, I opted to use vanilla Racket for this project.
The lack of types was a bit frustrating as runtime errors would often arise due to type mismatches.
Luckily, the REPL made it much easier to test each function in isolation so that I could see what types of values were being emitted from a given function.

The other major drawback was the size and performance of the executable.
Executing

	raco exe --embed-dlls git-skipper.rkt

involves embedding Racket-specific libraries directly into the executable for release to those without Racket installed on their systems.
This creates an executable that is over 10 MB large.
The size of the executable also increases as I import more modules using `(require)`.
The `--embed-dlls` flag can be removed to decrease the size of the executable, but this would require the end-user to install Racket on their machine.

The current implementation can also be slow on large repositories.
This may of course be due to issues with the current implementation than with Racket itself, but so far the convenience of the utility has outweighed any performance issues I have come across.
