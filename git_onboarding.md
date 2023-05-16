# Onboarding with Git

## Problem

Large software projects can be tricky for newcomers.
They often involve following lengthy articles and making numerous configuration changes to files in the codebase.
As the code evolves, these articles may go out of date, resulting in frustration for anyone joining the project.

After making these configuration changes, the developer's working directory may be littered with modified files that must not be committed to the repository.
This makes every commit and push a tiresome and risky affair, with the developer forced to stage each file and hunk manually by picking them out from a sea of configuration changes.
Luckily, Git offers a solution to both of these problems.

## Solution

### Ignoring single files

Git offers a way to selectively ignore tracked files using the command:

    git update-index --skip-worktree path/to/file

This command lets you modify a tracked file without reporting the changes through `git status`.
Thus, if the only modified files are those marked with `--skip-worktree`, then `git status` reports that there are no local changes.
This behaviour is also consistent with other Git commands such as `git rebase` or `git checkout`; these commands will not affect any files ignored using `--skip-worktree`.

Ignoring tracked files through `--skip-worktree` means that you can now use operations such as `git add *` without accidentally committing local configuration changes.
However, it also means that you cannot commit any changes to a configuration file that _should_ be committed.
To commit changes to a configuration file, you will need to undo the `--skip-worktree` operation using the command:

    git update-index --no-skip-worktree path/to/file

### Ignoring multiple files

Typing `git update-index --skip-worktree path/to/file` for every single configuration file can get pretty exhausting.
Fortunately, this can be resolved by chaining some Bash commands:

    git ls-files -m | xargs -d "\n" git update-index --skip-worktree

The `git ls-files -m` command emits the list of modified files in your working tree.
Each line emitted from `git ls-files -m` is then passed in as an argument by `xargs` to `git update-index --skip-worktree`.
Due to the line endings emitted by `git ls-files -m`, the `xargs` command is configured to look for newlines (`"\n"`) instead of the traditional null byte.

You can undo this operation using the following command:

    git ls-files -v | grep ^S | cut -c3- | xargs -d "\n" git update-index --no-skip-worktree

This `git ls-files -v` command prints every file in the Git repository with a prefix indicating its status.
Any file ignored using `--skip-worktree` is prefixed with _S_.
Thus, `grep ^S` returns only the files marked with `--skip-worktree`.
The `cut -c3-` command removes the _S_ prefix, leaving us with a list of files that are passed into `xargs -d "\n" git update-index --no-skip-worktree`.
This final command iterates over each file, marking each one with the `--no-skip-worktree` flag to allow Git to track local changes to these files.

### Switching branches

When checking out different commits or branches, there is the possibility that one of your configuration files has been updated by some other commit.
If this happens, Git will abort the checkout operation with the following message:

    error: Your local changes to the following files would be overwritten by checkout:
          /path/to/foo
          /path/to/bar
    Please commit your changes or stash them before you switch branches.
    Aborting

If this occurs, use the following strategy to switch to the new branch:
* Ensure your working tree is clean (i.e. no changes exist to any tracked files)
* Mark each file with `--no-skip-worktree`
* Stash the changes
* Check out the new branch
* Pop the stash
* Fix any conflicts
* Mark each file with `--skip-worktree`

Here is the above list translated into Git commands:

    <ensure no local changes are present>
    git ls-files -v | grep ^S | cut -c3- | xargs -d "\n" git update-index --no-skip-worktree
    git stash
    git checkout new-branch
    git stash pop
    <fix conflicts>
    git ls-files -m | xargs -d "\n" git update-index --skip-worktree

### Sharing configurations

The use of `--skip-worktree` makes it easy to generate configuration diffs that can be shared with other team members.
Consider a new team member cloning the repository.
You can generate a configuration for them using the following commands:

    <ensure no local changes are present>
    git fetch origin main
    git checkout origin/main
    git ls-files -v | grep ^S | cut -c3- | xargs -d "\n" git update-index --no-skip-worktree
    git diff > newConfig.diff
    git ls-files -m | xargs -d "\n" git update-index --skip-worktree
    <edit newConfig.diff to change names, etc.>

Once `newConfig.diff` is ready, you can send this file to the new team member.
They may then configure their repository using the following commands:

    git apply newConfig.diff
    git ls-files -m | xargs -d "\n" git update-index --skip-worktree

This technique for generating diffs also comes in handy when generating backup configuration files for yourself.
If you ever lose or corrupt your configuration, you can restore your configuration using `git apply config.diff`.
