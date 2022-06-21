---
layout: post
title: git tips & tricks - the fundamentals
---

Most of the time we only use a very limited set of `git` commands, `git checkout
-b <branch>` to start a new branch, `git pull origin` to update your current
branch with any remote changes, `git add .` & `git commit -m` to save your work,
and `git push origin` to push your changes to a remote repository.

This is perfectly fine, and online tools like Github or Gitlab make working with
a `git` repository even easier. But have you wondered what is actually going on
when we execute those commands?

In this article we are going to explore how a `git` repository works, and how
when you use the commands we mentioned above the branches and commits are
created and tracked inside the repository. We will also understand how to fix
common problems that arise when working with other people over the same
repository

All of this will be explained in a very easy to understand way, without diving
into any deep technical details that are not important to our usage of `git`.

## What is a git repository?

<blockquote>
<em>The following explanation is not 100% technically accurate, but instead a
simplification that will allow us to think about this topic without getting too
distracted with details you won't really need to know.</em>
</blockquote>

We can think of a git repository as a collection of commits organized in a
“[Directed Acyclic Graph][]”. A what? A Directed Acyclic Graph (DAG) is a
collection of nodes with connections between them, these connections have a
direction, and it is not possible to form a closed loop by following these
connections.

The following is an example taken from Wikipedia. Notice that no matter which
node you pick, there's no way to follow the arrows and end up in the same
starting node.

![Example of a directed acyclic graph](/media/images/example-directed-acyclic-graph.svg)

You can think of the git commands as commands to manipulate this DAG. Each node
in the graph represents a commit and each commit has one or more parents. In
turn each commit represents a change (or “patch”) to the contents of your
repository.

## Branches are pointers to commits

We can refer to the commits in a graph by their “hash”, each node in a git
repository have a unique “hash”, no two commits can ever have the same one. But
using hashes would be very inconvenient, so instead we usually use “branches” to
refer to commits.

Let's see this in action with an example. Let's initialize an empty repository
and create a couple of commits.

```
$ git init --initial-branch=main test-repository
Initialized empty Git repository in /Users/foo/test-repository/.git/

$ cd test-repository/
$ echo "This is our first file" > README.txt
$ git add README.txt

$ git commit -m "Initial commit, added README.txt"
[main (root-commit) 75c3507] Initial commit, added README.txt
 1 file changed, 1 insertion(+)
 create mode 100644 README.txt
```

We created an empty repository with the name `test-repository`,
this created a directory with that name, and we also specified the name of our
initial branch to be `main`.

We created a file `README.txt` with one line of text, and we committed our
“work” with our very first commit. Notice that git is telling us what the hash
of this commit is with `[main (root-commit) 75c3507]`, in this case the hash is
`75c3507`.

A visual representation of our graph looks like this:

<<<<<graph>>>>>

So far we have only one node (one commit), and a branch named `main` pointing to
this commit. There's also a special pointer named `HEAD` that is pointing to
`main`.

Now let's create a second commit.

```
$ echo "Welcome to our new repository" >> README.txt
$ git add README.txt

$ git commit -m "Added second line to README.txt"
[main b90865f] Added second line to README.txt
 1 file changed, 1 insertion(+)
```

The hash of the second commit is `b90865f`. Our graph now contains two nodes and
it looks like this:

<<<<<graph>>>>>

The newest commit `b90865f` has an arrow pointing to the first commit `75c3507`.
We say that the parent commit of `b90865f` is `75c3507`. The very first commit
in a repository does not have a parent, and every subsequent commit will have
one or more parents.

Notice that the branch `main` has now moved and it is pointing to
`b90865f`, and also notice that the `HEAD` pointer also moved with `main`.

Let's add a third commit just to make this more interesting.

```
$ touch empty-file.txt
$ git add empty-file.txt
$ git commit -m "Added an empty file"
[main edc3f18] Added an empty file
 1 file changed, 0 insertions(+), 0 deletions(-)
 create mode 100644 empty-file.txt
```

<<<<<graph>>>>>

The special `HEAD` pointer will follow us to wherever we jump in the graph of
commits. By using `git checkout` we can jump to any commit we want, and it will
also take care of updating our working files to match the state they had when
that commit was created.

```
$ git checkout 75c3507
Note: switching to '75c3507'.

You are in 'detached HEAD' state. You can look around, make experimental
changes and commit them, and you can discard any commits you make in this
state without impacting any branches by switching back to a branch.

If you want to create a new branch to retain commits you create, you may
do so (now or later) by using -c with the switch command. Example:

  git switch -c <new-branch-name>

Or undo this operation with:

  git switch -

Turn off this advice by setting config variable advice.detachedHead to false

HEAD is now at 75c3507 Initial commit, added README.txt
```

git is telling you that you are now in a “detached HEAD” state. This means that
`HEAD` is no longer pointing to a branch. This is important because if you
create new commits while in a detached `HEAD` state and then jump to a different
branch, there will be branch pointing to the commits you created and they could
be lost.

> Note that instead of `git checkout 75c3507` you could also have used `git
> switch --detached 75c3507`.

We can see that the contents of our files and our working directory reflect the
state they had when the commit we are pointing now was created.

```
$ cat README.txt
This is our first file

$ cat empty-file.txt
cat: empty-file.txt: No such file or directory
```

We can experiment while in this “detached HEAD” state, we can create new commits
and they won't affect at all our existing branches. If we want to discard this
experiment we can simply jump to another branch. If we want keep the changes
made in this experiment then we have the option of creating a new branch here.

Let's open your favorite code editor and change the `README.txt` file so that it
contains the following lines, then let's save that change in a new commit.

```
This is our first file

Changelog:
- Added README.txt
```

```
$ git add README.txt
$ git commit -m "Added changelog to README.txt"
[detached HEAD 8bfd1cf] Added changelog to README.txt
 1 file changed, 3 insertions(+)
```

Our graph now looks like this, notice that `HEAD` moved to point to the new
commit, and the new commit has an arrow to `75c3507` meaning that `75c3507` is
the parent of our new commit. We are still in a “detached HEAD” state.

<<<<<graph>>>>>

Now let's create a new branch that will point to this new commit, and in this
way we avoid losing this work if we jump to a different branch.

```
$ git switch -c readme-changelog
Switched to a new branch 'readme-changelog'
```

Our graph now looks like this. Notice that now `HEAD` points to our new branch
`readme-changelog`. If we create new commits now, the new branch pointer
`readme-changelog` will move to point to the new commits, and the `HEAD` pointer
will move as well to continue pointing to the new branch.

We are no longer in a “detached HEAD” state.

<<<<<graph>>>>>


## Remote vs local branches


Branches are pointers to commits We have local branches and remote branches, and both of them point to commits.

- Local branches are created by git checkout -b, or git switch -c (like we did in
the previous example). They move when we create commits, and we send them to a
remote repository with git push.
- Remote branches are created and updated when we fetch new changes from a remote
repository with git fetch or git pull.

For the following example I first created a repository under my personal account
in Github. Just go to https://github.com/new, give it any name you want, and
specify it to be either Public or Private, doesn't matter. But DO NOT check the
option to “Add a README file” because this would cause Github to initialize a
new repository. We already initialized a repository in our local computer.

```
$ git remote add origin git@github.com:gajon/test-repository.git
$ git remote -v
origin  git@github.com:gajon/test-repository.git (fetch)
origin  git@github.com:gajon/test-repository.git (push)
```

With `git remote add` we added a reference to the remote repository, giving it
the name of `origin` although you could have given it any name you want. We
verify the name of the repository and where it points to with `git remote -v`.

Now let's push our branch to it.

```
$ git push origin -u readme-changelog
Enumerating objects: 6, done.
Counting objects: 100% (6/6), done.
Delta compression using up to 4 threads
Compressing objects: 100% (3/3), done.
Writing objects: 100% (6/6), 834 bytes | 834.00 KiB/s, done.
Total 6 (delta 0), reused 0 (delta 0), pack-reused 0
To github.com:gajon/test-repository.git
 * [new branch]      readme-changelog -> readme-changelog
branch 'readme-changelog' set up to track 'origin/readme-changelog'.
```

We'll talk later about the `-u` option given to `git push`. The previous command
basically means “push to the repository named origin, our local branch called
readme-changelog”.

Our graph now looks like this. Notice that we have a new pointer named
`origin/readme-changelog`, we call this a remote reference or remote branch.
Both our local `readme-changelog` and the remote `origin/readme-changelog` point
to the same commit.

<<<<<graph>>>>>

If you reload the Github page with our repository you'll notice that only the
`readme-changelog` branch is listed there, and we don't see any `main` branch
nor the changes to the `README.txt` file that the `main` branch contains, nor
the empty `empty-file.txt`.

This is because we have not pushed our `main` branch to Github yet. We'll do
that in a bit, but first let's talk about how we can see our graph from the
command line.


```
$ git log --graph --all
* commit 8bfd1cfbaa5a69e6b8f0f0380d1e8e29de1d1b5f (HEAD -> readme-changelog, origin/readme-changelog)
| Author: Jorge Gajon <gajon@gajon.org>
| Date:   Mon Jun 20 15:22:56 2022 -0500
|
|     Added changelog to README.txt
|
| * commit edc3f186532e38af99ef27b6b372954f5ca52426 (main)
| | Author: Jorge Gajon <gajon@gajon.org>
| | Date:   Mon Jun 20 14:29:25 2022 -0500
| |
| |     Added an empty file
| |
| * commit b90865f0103e52ba7b5e1084b5852bdc3b2e2b35
|/  Author: Jorge Gajon <gajon@gajon.org>
|   Date:   Fri Jun 17 21:01:53 2022 -0500
|
|       Added second line to README.txt
|
* commit 75c3507b2ee661a869c53ba491886bf3a8ebe66f
  Author: Jorge Gajon <gajon@gajon.org>
  Date:   Fri Jun 17 20:55:37 2022 -0500

      Initial commit, added README.txt
```




If you remove the `--all` option, then `git log` will only show you the commits
that are visible from the current branch you are on. In this example we are
on the `readme-changelog` branch and we cannot see the commits on the `main`
branch. Remember this is a “Directed” graph, meaning the arrows only go in one
direction.

```
$ git log --graph
* commit 8bfd1cfbaa5a69e6b8f0f0380d1e8e29de1d1b5f (HEAD -> readme-changelog, origin/readme-changelog)
| Author: Jorge Gajon <gajon@gajon.org>
| Date:   Mon Jun 20 15:22:56 2022 -0500
|
|     Added changelog to README.txt
|
* commit 75c3507b2ee661a869c53ba491886bf3a8ebe66f
  Author: Jorge Gajon <gajon@gajon.org>
  Date:   Fri Jun 17 20:55:37 2022 -0500

      Initial commit, added README.txt
```

Making it prettier with abbreviated commit hashes and dates, and collapsing each commit to
a single line.

```
$ git log --format=oneline --graph --all --abbrev-commit --date=short
* 8bfd1cf (HEAD -> readme-changelog, origin/readme-changelog) Added changelog to README.txt
| * edc3f18 (main) Added an empty file
| * b90865f Added second line to README.txt
|/
* 75c3507 Initial commit, added README.txt
```

```
$ git log --pretty=format:'[%ad] %h %d %s (%an)' --graph --all --abbrev-commit --date=short

* [2022-06-20] 8bfd1cf  (HEAD -> readme-changelog, origin/readme-changelog) Added changelog to README.txt (Jorge Gajon)
| * [2022-06-20] edc3f18  (main) Added an empty file (Jorge Gajon)
| * [2022-06-17] b90865f  Added second line to README.txt (Jorge Gajon)
|/
* [2022-06-17] 75c3507  Initial commit, added README.txt (Jorge Gajon)
```

After we applied a format with `--pretty` we lost some colors. We can add them
back like this:

```
$ git log --pretty=format:'%Creset%C(red bold)[%ad] %C(blue bold)%h %Creset%C(magenta bold)%d %Creset%s %C(green bold)(%an)%Creset' --graph --all --abbrev-commit --date=short

* [2022-06-20] 8bfd1cf  (HEAD -> readme-changelog, origin/readme-changelog) Added changelog to README.txt (Jorge Gajon)
| * [2022-06-20] edc3f18  (main) Added an empty file (Jorge Gajon)
| * [2022-06-17] b90865f  Added second line to README.txt (Jorge Gajon)
|/
* [2022-06-17] 75c3507  Initial commit, added README.txt (Jorge Gajon)
```

## A git log alias to view a pretty graph

It would be really bad if we had to type that `git log` command with all those
options all the time. To avoid that we can create an alias in a `.gitconfig`
file. You can create this file inside your current repository, or you can create
one that will be used globally across all your repositories.

You most likely already have one in your `$HOME` (`/Users/<yourusername>` on
macOS, or `/home/<yourusername>` on Linux).

Create it or open it and look if you already have a section starting with an
`[alias]` line. If not you can add it yourself, and inside this section you
will add two aliases, one with the name `h` and the other with the name `ha`.

My `.gitconfig` file looks like this:

```
[user]
  name = Jorge Gajon
  email = gajon@gajon.org

[alias]
  h  = log --pretty=format:'%Creset%C(red bold)[%ad] %C(blue bold)%h %Creset%C(magenta bold)%d %Creset%s %C(green bold)(%an)%Creset' --graph --abbrev-commit --date=short
  ha = log --pretty=format:'%Creset%C(red bold)[%ad] %C(blue bold)%h %Creset%C(magenta bold)%d %Creset%s %C(green bold)(%an)%Creset' --graph --all --abbrev-commit --date=short
```

The only difference between `h` and `ha` is that the first does not use `--all`
option and will only show you the commits visible from your current branch,
while `ha` uses `--all` and thus will show you all the commits visible from all
branches existing in your repository.


git h <ref>


Ok, let's switch to our `main` branch and push it too the remote repository too.

```
$ git switch main
Switched to branch 'main'
$ git push origin -u main
Enumerating objects: 8, done.
Counting objects: 100% (8/8), done.
Delta compression using up to 4 threads
Compressing objects: 100% (4/4), done.
Writing objects: 100% (6/6), 865 bytes | 865.00 KiB/s, done.
Total 6 (delta 0), reused 0 (delta 0), pack-reused 0
To github.com:gajon/test-repository.git
 * [new branch]      main -> main
branch 'main' set up to track 'origin/main'.
```

Now we also have the remote branch `origin/main`.

```
$ git ha
* [2022-06-20] 8bfd1cf  (origin/readme-changelog, readme-changelog) Added changelog to README.txt (Jorge Gajon)
| * [2022-06-20] edc3f18  (HEAD -> main, origin/main) Added an empty file (Jorge Gajon)
| * [2022-06-17] b90865f  Added second line to README.txt (Jorge Gajon)
|/
* [2022-06-17] 75c3507  Initial commit, added README.txt (Jorge Gajon)
```

## two people working on the same repo

Alice
Bob

## Pushing changes to a remote repository

## Pulling changes from a remote repository

git fetch
git pull  == git fetch + git merge


`git merge` when on a feature branch


Merges can be good because
- They show how two independent branches were developed on their own
- and when they were merged together
- can provide insight into how the code evolved.

But merges can also be bad because
- They pollute the graph of commits
- In extreme cases the majority of the commit are just “merge” commits that do
  not represent new code.
- It is difficult to follow when something was merged into the main branch.


For these reason Github & Gitlab allows you to “squash” a pull request.

## Fast-forward merge

## Rebasing your commits

git rebase origin/main

take the commits that are in my current branch but not in `main`, and apply them
one by one on top of `main`.

The commit hashes will change because these are new commits. A commit is never
really modified, what we are doing here is creating new commits with the same
code changes. These commits also have different ancestors and this alone makes
it impossible for these new commits to have the original hash.

The original commits are not modified nor deleted, they stay in the repository
even though there's no longer a branch pointing to them. Eventually they will be
“garbage collected” by git.

## git rebase --onto <ref> <parent> <branch>

## git cherry-pick

## The working directory vs the stage

## git reset --hard HEAD^

## git log -p <ref> -- file

## delete remote branches with git push origin :branch

## push a local branch as a different branch on the remote

## The reflog - a very powerful tool

## Syntax to refer to the history

## Quickly switch between two branches

```
git switch -
```


[Directed Acyclic Graph]: https://en.wikipedia.org/wiki/Directed_acyclic_graph

<!-- vim: set tw=80 sw=2 sts=4 et spell: -->
