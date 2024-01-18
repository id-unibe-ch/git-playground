# git-playground

This repo and its content has only one purpose: **to learn, to test and to
train git**. It does so by featuring pre-built branches with example content
laid out in a way that use cases can be played through before applying them to
the "real life" problems.

Additional content is very welcome. Just fork, crate two new branches:

* One branch that extends this README with the new use case. This is then merge
  via a PR
* Another branch for the example data to work on if needed. This won't get merged
  but protected.

## Use Cases

### Clean Up Feature Branches

Working on a new feature on a feature branch is often a journey with hopefully
lots of commits to be able to go back, with commits that fix typos, with commits
that partially revert previous changes - over all - probably a medium to big
mess. Hey, that's totally ok, that's good. Nobody creates production code right
away.

I hear you yelling: "But this messy branch gets merged using Squash and Merge
anyway! Then why bother at all?"

If all of the changes belong together and therefore are related, then you may be
correct, BUT think about the person that has to dig through you feature branch
doing a review. 20 commits working on 15 files with multiple changes of the same
lines is a real burden on a reviewer and takes a lot of time. At the very least,
make the life of the reviewer easier and be a guidance through your commit
history. Commits like "fix typo", "add missing file" and the like only distract
the work for the reviewer. Clean it up and have a bunch of meaningful commits
that make the reviewer discover your feature, i.e. "a first, this aspect, a and
in the next commit sub-feature X is introduced". Sure, this is more work for
you, the developer, but you know your work in and out, the reviewer doesn't yet.

Find in this repo a branch `learn-rebasing-easy` to play around with interactive
rebasing. This tool allows you to rewrite the git history in every manner you
like. Please be aware, you are rewriting the history for everybody, so do this
only on work that others to not rely on. Typically, you only do this on your own
feature branches or on commits in main **that are not yet pushed to remote aka
made public**. But with branch protection on the main branch, this shouldn't
happen anyway. ;-)

**Task 1**: Go ahead, checkout `learn-rebasing-easy` and try to make the branch look
like this:

    A---B---C---D  learn-rebasing-easy

### I Accidentally Committed to Wrong Branch

#### Symptom

You forgot to create the feature branch or to checkout the correct feature
branch and you now have created one or even several commits on the wrong branch,
probably main? Let's assume main for the cases below.  
How to get these commits over to the correct branch?

#### Discussion and Solution

We have to distinguish two cases here:

**Case 1:** If you forgot to create the new feature branch before working on the
new feature, things are simple to fix.

    A---B---C---D---E  main

Let's assume that commits D and E above should have gone into the feature branch
`my-feature`. So, we have to achieve that D and E go onto the new feature branch
and then main can be rolled back to C, right?

The current state above is actually already what you wanted on the feature
branch, rigth? Well, then let's check out this state as the new feature branch.

    git checkout -b my-feature

The repo now looks like this:

    A---B---C---D---E  main, my-feature

The only thing left to do is to remove D and E from main AKA roll back main by
two commits:

    git checkout main
    git reset --hard HEAD~2

Now, the repo looks like this - exactly what we wanted in the first place:

    A---B---C  main
             \
              D---E  my-feature

**Case 2**
Now, if the feature branch already exists and has work on it, we have to
append those wrong commits to the correct feature branch. It again makes sense
to distinguish two cases:

**a)** if you only have one commit that needs to be moved over, use cherry
picking:

    git log # remember the commit hash of the wrong commit
    git checkout <target branch>
    git cherry-pick <commit hash>
    git checkout main
    git reset --hard HEAD~1    # or HEAD^

**b)** You have several commits to get moved from one branch to another, use
`git rebase`. An example might look like this:

    A---B---C---G---H---I  main
             \
              D---E---E  my-feature

The changes in G, H and I should have been committed to the branch `my-feature`
instead of main. Damn!  
First create a temporary branch for these commits: `git checkout -b my-temp`

The situation now looks like this:

    A---B---C---G---H---I  main, my-temp
             \
              D---E---E  my-feature

Now reset main to where it should be, C in the example:

    git reset --hard HEAD~3

The situation now looks like this:

    A---B---C  main
            |\
            | G---H---I  my-temp
            |
             \
              D---E---F  my-feature

Now we can rebase the branch my-temp onto the branch my-feature:

    # First checkout the branch to move
    git co my-temp

    # Now rebse the current branch from the old the new branch:
    git rebase --onto my-feature main

The situation now looks like this:

    A---B---C  main
             \
              D---E---F  my-feature
                       \
                        G'---H'---I'  my-temp

Final tasks, rebase branch `my-feature` to the branch `my-feature` and delete
the reference `my-temp`:

    git checkout my-feature
    git rebase my-temp
    git branch -d my-temp

The situation now looks like this:

    A---B---C  main
             \
              D---E---F---G'---H'---I'  my-feature

And that's what we wanted.

---

More use cases and recipes can be found
[here](https://michael.rollis.ch/myitjournal/gitandco/gitrecipes/#damn-i-branched-off-wrong-parent-branch)
and at a lot of more places all over the internet.
