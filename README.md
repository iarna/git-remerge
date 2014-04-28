git-remerge
===========

Semi-automated rebuilding of integration branches-- rebase that can preserve
and reorder merges

Synopsis
--------

    git remerge [-i | --interactive] [--onto *newbase*] [*upstream*] [*branch*]
    git remerge --continue | --skip | --abort | --edit-todo

Description
-----------

`git rebase --preserve-merges` is broken bordering on useless.  This
endevors to provide a more functional implementation.  It works better by
making some assumptions that are almost but not quite always true.

If *branch* is specified, git remerge will perform an automatic git checkout
*branch* before doing anything else.  Otherwise it remains on the current
branch.

If *upstream* is not specified, then it defaults to 'master' currently.

All changes made by commits in the current branch but that are not in
*upstream* are saved to a temporary area.  This is a list of merges and
commits not contained in merges, just like with `git rebase -p`

The current branch is reset to *upstream*, or *newbase* if the --onto option
was supplied.  This has the exact same effect as git reset --hard *upstream*
(or *newbase*).  ORIG_HEAD is set to point at the tip of the branch before
the reset.

The commits that were previously saved into the temporary area are then
reapplied to the current branch, one by one, in order.  Merge commits are
reissued as `git merge --no-ff` from whatever branch was specified in the
original commit.  The branch will be pulled before its merged.  If the
branch doesn't exist and can't be created from a remote then the remerge
attempt will abort.

In the case of conflicts that rerere is able to resolve, they will be
accepted and automatically committed.

It is possible that a merge failure will prevent this process from being
completely automatic.  You will have to resolve any such merge failure and
run git remerge --continue.  Another option is to bypass the commit that
caused the merge failure with git remerge --skip.  To check out the original
*branch* and remove the .git/remerge-* working files, use the command git
remerge --abort instead.

In case of conflict, git remerge will stop at the first problematic commit
and leave conflict markers in the tree.  You can use git diff to locate the
markers (\<\<\<\<\<\<) and make edits to resolve the conflict.  For each file you
edit, you need to tell Git that the conflict has been resolved, typically
this would be done with

    git add *filename*

After resolving the conflict manually and updating the index with the
desired resolution, you can continue the remerging process with

    git remerge --continue

Alternatively, you can undo the git remerge with

    git remerge --abort

Acknowledgement
---------------

Due to the similarities between this command and git rebase, the
documentation here is largely taken from there.

Options
-------

* --onto *newbase*

Starting point at which to create the new commits. If the --onto option is
not specified, the starting point is *upstream*.  May be any valid commit,
and not just an existing branch name.

As a special case, you may use "A...B" as a shortcut for the merge base of A
and B if there is exactly one merge base.  You can leave out at most one of
A and B, in which case it defaults to HEAD.

* *upstream*

Upstream branch to compare against. May be any valid commit, not just an
existing branch name.  Defaults to 'master' currently.

* *branch*

Working branch; defaults to HEAD.

* --continue

Restart the remerging process after having resolved a merge conflict.

* --abort

Abort the remerge operation and reset HEAD to the original branch. If
*branch* was provided when the remerge operation was started, then HEAD will
be reset to *branch*.  Otherwise HEAD will be reset to where it was when the
rebase operation was started.

* --skip

Restart the remerging process by skipping the current patch.

* --edit-todo

Edit the todo list during an interactive remerge.

