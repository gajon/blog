---
layout: post
title: git tips & tricks - the fundamentals
---

Most of the time we only use a very limited set of git commands, `git checkout
-b <branch>` to start a new branch, `git pull origin` to update your current
branch with any remote changes, `git add .` & `git commit -m` to save your work,
and `git push origin` to push your changes to a remote repository.

This is perfectly fine, and online tools like Github or Gitlab make working with
a git repository even easier. But have you wondered what is going on when we
execute those commands?

In this and the following articles we are going to explore how a git repository
works, and how when you use the commands we mentioned above the branches and
commits are created and tracked inside the repository. We will also understand
how to fix common problems that arise when working with other people over the
same repository.

All of this will be explained in a very easy-to-understand manner, without
diving into deep technical details that are not important to our usage of git.

## What is a git repository?

<blockquote>
<em>The following explanation is not 100% technically accurate, but instead a
simplification that will allow us to think about this topic without getting too
distracted with details we don't need.</em>
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
turn, each commit represents a change (or “patch”) to the contents of your
repository.

## Branches are pointers to commits

We can refer to the commits in a graph by their “hash”, each node in a git
repository has a unique “hash”, no two commits can ever have the same one. But
using hashes would be very inconvenient, so we use “branches” to refer to
commits instead.

Let's see this in action with an example. Let's initialize an empty repository
and create a couple of commits.

```
$ git init --initial-branch=main test-repository
Initialized empty Git repository in /Users/foo/test-repository/.git/

$ cd test-repository/
$ echo "This is our first file" > README.txt
$ git add README.txt

$ git commit -m "Initial commit, added README.txt"
[main (root-commit) c8c57bf] Initial commit, added README.txt
 1 file changed, 1 insertion(+)
 create mode 100644 README.txt
```

We created an empty repository with the name `test-repository`, this created a
directory with that name, and we also specified the name of our initial branch
to be `main`.

We created a file `README.txt` with one line of text, and we committed our work
with our very first commit. Notice that git is telling us what the hash of this
commit is with `[main (root-commit) c8c57bf]`, in this case, the hash is
`c8c57bf`.

A visual representation of our graph looks like this:

```
┌────┐    ┌────┐     .─────────.
│HEAD│───▶│main│───▶(  c8c57bf  )
└────┘    └────┘     `─────────'
```

> **NOTE:** If you follow these commands on your computer you'll notice that the
> commit hashes will be entirely different for you, this is normal.

So far we have only one node with commit hash `c8c57bf`, and a branch named
`main` pointing to this commit. There's also a special pointer named `HEAD` that
is pointing to `main`.

Now let's create a second commit.

```
$ echo "Welcome to our new repository" >> README.txt
$ git add README.txt

$ git commit -m "Added second line to README.txt"
[main 3bbdb69] Added second line to README.txt
 1 file changed, 1 insertion(+)
```

The hash of the second commit is `3bbdb69`. Our graph now contains two nodes and
it looks like this:

```
┌────┐    ┌────┐     .─────────.
│HEAD│───▶│main│───▶(  3bbdb69  )
└────┘    └────┘     `─────────'
                          │
                          │
                          ▼
                     .─────────.
                    (  c8c57bf  )
                     `─────────'
```

The newest commit `3bbdb69` has an arrow pointing to the first commit `c8c57bf`.
We say that the parent commit of `3bbdb69` is `c8c57bf`. The very first commit
in a repository does not have a parent, and every subsequent commit will have
one or more parents.

Also notice that the branch `main` has moved and it is pointing to `3bbdb69` and
that the `HEAD` pointer also moved with `main`.

Let's add a third commit just to make this more interesting.

```
$ touch empty-file.txt
$ git add empty-file.txt
$ git commit -m "Added an empty file"
[main bece83c] Added an empty file
 1 file changed, 0 insertions(+), 0 deletions(-)
 create mode 100644 empty-file.txt
```

Our graph now looks like this:

```
┌────┐    ┌────┐     .─────────.
│HEAD│───▶│main│───▶(  bece83c  )
└────┘    └────┘     `─────────'
                          │
                          │
                          ▼
                     .─────────.
                    (  3bbdb69  )
                     `─────────'
                          │
                          │
                          ▼
                     .─────────.
                    (  c8c57bf  )
                     `─────────'
```

The special `HEAD` pointer will follow us wherever we jump in the graph of
commits. By using `git checkout` we can jump to any commit we want, and it will
also take care of updating our working files to match the state they had when
that commit was created.

```
$ git checkout c8c57bf
Note: switching to 'c8c57bf'.

You are in 'detached HEAD' state. You can look around, make experimental
changes and commit them, and you can discard any commits you make in this
state without impacting any branches by switching back to a branch.

If you want to create a new branch to retain commits you create, you may
do so (now or later) by using -c with the switch command. Example:

  git switch -c <new-branch-name>

Or undo this operation with:

  git switch -

Turn off this advice by setting config variable advice.detachedHead to false

HEAD is now at c8c57bf Initial commit, added README.txt
```

git is telling us that we are now in a “detached HEAD” state. This means that
`HEAD` is no longer pointing to a branch. This is important because if you
create new commits while in this “detached HEAD” state and then jump back to a
different branch, you could lose those commits.

> The new recommended way to switch to a “detached HEAD” state is to use the
> command `git switch --detach`, this way you are being more explicit about your
> intention. For example:
>
> ```
> $ git switch --detach c8c57bf
> HEAD is now at c8c57bf Initial commit, added README.txt
> ```

We can see that the contents of our files and our working directory reflect the
state they had when the commit we are pointing to now was created.

```
$ cat README.txt
This is our first file

$ cat empty-file.txt
cat: empty-file.txt: No such file or directory
```

We can experiment while in this “detached HEAD” state. For example, we can
create new commits and they won't affect other branches. If we want to discard
this experiment we can simply jump to another branch. If we want to keep the
changes made in this experiment then we have the option of creating a new branch
here.

Let's open your favorite code editor and change the `README.txt` file so that it
contains the following lines:

```
This is our first file

Changelog:
- Added README.txt
```

Then save that change in a new commit.

```
$ git add README.txt
$ git commit -m "Added changelog to README.txt"
[detached HEAD fb75aff] Added changelog to README.txt
 1 file changed, 3 insertions(+)
```

Our graph now looks like the following. Notice that `HEAD` moved to point to the
new commit `fb75aff`, and the new commit has an arrow to `c8c57bf` meaning that
`c8c57bf` is the parent of our new commit. We are still in a “detached HEAD”
state.

```
                 ┌────┐     .─────────.
                 │main│───▶(  bece83c  )
                 └────┘     `─────────'
                                 │
                                 │
                                 ▼
┌────┐     .─────────.      .─────────.
│HEAD│───▶(  fb75aff  )    (  3bbdb69  )
└────┘     `─────────'      `─────────'
                │                │
                └───────────────┐│
                                ▼▼
                            .─────────.
                           (  c8c57bf  )
                            `─────────'
```

Now let's create a new branch that will point to this new commit, and in this
way, we avoid losing this work if we jump to a different branch.

```
$ git switch -c changelog
Switched to a new branch 'changelog'
```

Notice that now `HEAD` points to our new branch `changelog`, we are no longer in
a “detached HEAD” state.

```
                 ┌────┐     .─────────.
                 │main│───▶(  bece83c  )
                 └────┘     `─────────'
                                 │
                                 │
┌────┐  ┌─────────┐              │
│HEAD│─▶│changelog│              │
└────┘  └─────────┘              │
          │                      │
          │                      ▼
          │  .─────────.    .─────────.
          └▶(  fb75aff  )  (  3bbdb69  )
             `─────────'    `─────────'
                  │              │
                  └─────────────┐│
                                ▼▼
                            .─────────.
                           (  c8c57bf  )
                            `─────────'
```

If we create new commits now, the new branch `changelog` will move to point to
the new commits, and the `HEAD` pointer will move as well to continue pointing
to our new branch.

## Going back to our main branch

Let's go back to our `main` branch, this will make our special `HEAD` pointer
move to point to `main` and this will also update the contents of our working
files to match the state they had when we last created a commit in `main`.

```
$ git switch main
Switched to branch 'main'
```

Our graph now looks like this:

```
┌────┐       ┌────┐     .─────────.
│HEAD│──────▶│main│───▶(  bece83c  )
└────┘       └────┘     `─────────'
                             │
                             │
    ┌─────────┐              │
    │changelog│              │
    └─────────┘              │
      │                      │
      │                      ▼
      │  .─────────.    .─────────.
      └▶(  fb75aff  )  (  3bbdb69  )
         `─────────'    `─────────'
              │              │
              └─────────────┐│
                            ▼▼
                        .─────────.
                       (  c8c57bf  )
                        `─────────'
```

And if we inspect the contents of our `README.txt` file we see that the changes
made in the `changelog` branch are not there anymore.

```
$ cat README.txt
This is our first file
Welcome to our new repository
```

## Display the repository graph with `git log`

So far I've shown you a simplified “visual” representation of the graph that
contains our commits inside our repository. You can ask git to display this
graph in your terminal with the following command.

```
$ git log --graph --all
* commit fb75affebd2d44fd7100bfe932a06004623825cf (changelog)
| Author: Jorge Gajon <gajon@gajon.org>
| Date:   Sun Jun 26 23:56:14 2022 -0500
|
|     Added changelog to README.txt
|
| * commit bece83c61b9d0bd58f5d7d90cb9ca10af8771802 (HEAD -> main)
| | Author: Jorge Gajon <gajon@gajon.org>
| | Date:   Sun Jun 26 23:42:06 2022 -0500
| |
| |     Added an empty file
| |
| * commit 3bbdb697b2fe653de8f73c9071bcacb3aaa3b79b
|/  Author: Jorge Gajon <gajon@gajon.org>
|   Date:   Sun Jun 26 23:35:26 2022 -0500
|
|       Added second line to README.txt
|
* commit c8c57bfe49e8d5d1918571f4cbdc708cfd82501e
  Author: Jorge Gajon <gajon@gajon.org>
  Date:   Sun Jun 26 23:15:13 2022 -0500

      Initial commit, added README.txt
```

Notice a few things. First, the commit hashes are being displayed in their
long-form, rather than the short form we've seen so far. For example the commit
hash we've previously seen as `bece83c` is being shown as its full value of
`bece83c61b9d0bd58f5d7d90cb9ca10af8771802`. Both the short and long-form are
interchangeable.

The second thing to notice is that the commit being shown at the very top is the
latest commit we created, in this case, the latest commit in the `changelog`
branch.

If we re-arrange a little bit the visual graph we were seeing previously:

```
 .─────────.     ┌─────────┐
(  fb75aff  )◀───│changelog│
 `─────────'     └─────────┘
     │
     │   .─────────.    ┌────┐   ┌────┐
     │  (  bece83c  )◀──│main│◀──│HEAD│
     │   `─────────'    └────┘   └────┘
     │        │
     │        ▼
     │   .─────────.
     │  (  3bbdb69  )
     │   `─────────'
     │        │
     │┌───────┘
     ▼▼
 .─────────.
(  c8c57bf  )
 `─────────'
```

Compare this to what `git log --graph --all` is showing you. Can you identify
the arrow pointing to each commit's parent? Can you see where the branches and
the `HEAD` are pointing to?

You can ask git to display this graph in an extremely compact form:

```
$ git log --format=oneline --graph --all
* fb75affebd2d44fd7100bfe932a06004623825cf (changelog) Added changelog to README.txt
| * bece83c61b9d0bd58f5d7d90cb9ca10af8771802 (HEAD -> main) Added an empty file
| * 3bbdb697b2fe653de8f73c9071bcacb3aaa3b79b Added second line to README.txt
|/
* c8c57bfe49e8d5d1918571f4cbdc708cfd82501e Initial commit, added README.txt
```

Even though there's no “arrow” shown between `bece83c` and `3bbdb69`, you can
still see that `3bbdb69` is the parent of `bece83c`.

We'll talk more about `git log` in coming articles. But for now, try this
exercise:

- Remove the `--all` option from the command. What happens?
- Switch back to the `changelog` branch, and then display the graph with and
  without the `--all` option.
- Switch to a commit in a “detached HEAD” state, and inspect the graph again.
- What happens if you don't include the `--graph` option?


[Directed Acyclic Graph]: https://en.wikipedia.org/wiki/Directed_acyclic_graph

<!-- vim: set tw=80 sw=2 sts=4 et spell: -->
