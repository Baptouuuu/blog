# Why I hate to rebase [GIT]

Before jumping right to the point I want to make, just step back and think about what's the purpose of the VCS and why `git rebase` *seems* a good tool.

## Goal of a VCS

We mainly use such tools to improve the collaborative work. But I see it more as the single point of truth for the projects. After all if those systems register all the things we do on code, it's to keep track on what's exactly happening on the code.

That's why I don't like any tool that have a destructive effect on the repositories. Because it means you're lying to the repo. And I can't see in any way how lying to the system supposed to tell the only truth can benefit you or your teammates.

## Seems practical

Even though I never liked to rebase, I tried to listen to those whom are using it. The most common answer was: it helps you keep your commit history straight without plenty of merge commit.

Well, it's right, with the `rebase` you don't have a merge commit each time you want to update your branch with another one. (But, it's your first lie to the repo, as you say that you started your work from a new commit than the real one)

Another answer was as we rebase, when you want to merge to the source branch you simply have a fast forward, and so a straight line; consequently it's easier when looking back to the history, you don't have to understand the subway map. (But still, you're looking at a bunch of lies)
Each time I ear someone tell me it's good to have just one line of history, I have the feeling he tries to use git like it was SVN. If git has a good branching mechanism, why someone would want to use git as if there were no branches?!

## It will bite you

The points above can be debatable and some will disagree (which is normal). But I recently had good examples of how the rebase came back to bite me; that's what really motivated me to write this little article.

The first example concerns 2 teammates, 2 branches and a common one. These coworkers were working on related features but decided to use different branches. Each one produced some code, and then one (he doesn't know that much about git) asked me how to incorporate part of the code of the other developer into his branch. In a normal case I would have done a simple merge, but as I never came accross the context where we mix rebases and merges, I didn't know how git would behave afterward with other rebases on top of the merge. I was a bit afraid git would throw an error as he couldn't link to the merge parent commit; and I didn't want to mess up with his work. So I played it safe by cherry-picking the code of the other developer into his branch (thankfully it was limited to one commit).

Afterward I decided to do a little test to see how git behave if I would have done a merge to resolve this example. And it appears git is smart enough that rebasing after a merge keeps the commits whithout breaking, but it removes the merge commit. Which, in my opinion, is bad. Remember what's the goal of the vcs? Keep track of what's happening! But here, on top of the rebase lies, we had another one by removing the fact we merged the work of someone else into our branch.


The second example has gone hard on my nerves. I was working on a new feature on my own branch, everything was working fine. To keep up with the common branch, to be sure there was no conflicts, I rebased several times (as it's how the team work). And then I deployed my work on a sandbox server to test all my work, and saw that part of my code that was working fine was now broken (even if I didn't change any line of code).

In a normal case (meaning merges instead of rebases), I would have gone for `git bisect` to find the faulty commit, which would have taken approximatively 5 minutes. But here it's of no help, as I'm sure it was the code made on the common branch that was breaking my code. And as I rebased, the code made by my colleagues are now behind of my work instead of being in parallel in the commit history. So the bisect wouldn't work as know even the point were my code was working it's now failing, and the faulty commit is now in a place were my work is not existing.

Example with merge:
```
 A
 |\
 | \
 B  |
 |  C  <- working commit
 |  |
  \ |
    D  <- faulty commit that could be found via `bisect`
    |
    |
    E  <- failing commit
```

Example with rebase:
```
 A
 |
 |
 B  <- faulty commit
  \
   \
    C  <- working commit before rebase (now failing)
    |
    |
    D  <- failing commit (as created after the rebase)
```

In this last example, you can't determine easily where it has gone wrong as know `C` is the child of a failing commit, but previously it was the child of a working one. Here, it's simple to determine what happened as there's only 4 commits, but imagine if you have hundreds of them.

In result of this second example, I've lost almost a full day to figure out what was going on; before to find a teammate had updated all the dependencies of the application, that introduce a parallel code which was modifying the data after the execution of my code (code that was not even directly related to mine).

## Conclusion

Tools such as `rebase` seems to help you keep things simple on your repo (history more readable, etc...), but once again the goal of the vcs is keep track of what's happening. In most cases you won't notice modifying the commit history is a bad habbit, but the day you'll really need to look back at what happened, you won't be able (at least easily) to figure out what happened and in which order.
