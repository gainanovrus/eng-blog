---
published: false
layout: single
title: Git: Squash Multiple Commits in to One Commit
excerpt: >-
  This note shows how to merge an ugly feature branch with multiple dirty WIP
  commits back into the master as one pretty commit.
categories: dev
tags: git linux
toc: true
# header:
#  teaser: /assets/images/custom-oauth2-provider-to-nextcloud.png
#  og_image: /assets/images/custom-oauth2-provider-to-nextcloud.png
last_modified_at: 2019-10-24
---

In Git you can merge several commits into one with the powerful interactive rebase.
It's a handy tool I use quite often;
I usually tidy up my working space by grouping together several small intermediate
commits into a single lump to push upstream.

## choose your starting commit

When you squash commits, you're combining 2 or more commits in to a single commit.
For example, let's say your recent commit history looks something like this:

```
$ git log --pretty=format:"%h %ad | %s [%an]" --graph --date=short

* 639c220 2019-11-21 | #454, CI is fully implemented [Ruslan Gainanov]  --- newer commit
* f1a9a3e 2019-11-21 | fix [Ruslan Gainanov]
* 67f56b9 2019-11-21 | fix [Ruslan Gainanov]
* 2673619 2019-11-21 | fix [Ruslan Gainanov]
* a5398ae 2019-11-21 | fix [Ruslan Gainanov]
* 6f577b6 2019-11-21 | fix [Ruslan Gainanov]
* 3aee03c 2019-11-20 | #454, prepare new CI [Ruslan Gainanov]           --- older commit
* e807180 2019-11-18 | #123, feature Y implemented [romanicus]          --- parent commit
```

I'm working on a CI feature. And this is what I would like to do:
```
* 639c220 2019-11-21 | #454, CI is fully implemented [Ruslan Gainanov]  --- newer commit  --┐
* f1a9a3e 2019-11-21 | fix [Ruslan Gainanov]                                                |
* 67f56b9 2019-11-21 | fix [Ruslan Gainanov]                                                |
* 2673619 2019-11-21 | fix [Ruslan Gainanov]                                                |---- Join these into one
* a5398ae 2019-11-21 | fix [Ruslan Gainanov]                                                |
* 6f577b6 2019-11-21 | fix [Ruslan Gainanov]                                                |
* 3aee03c 2019-11-20 | #454, prepare new CI [Ruslan Gainanov]           --- older commit ---┘
* e807180 2019-11-18 | #123, feature Y implemented [romanicus]
```

I often have tons of commits to squash. And a command `git rebase -i HEAD~[N]` with
number of commits you want to join (`N`), starting from the most recent one, doesn't accept for me.

Also I don't want to count how many commits should be squashed. So there is another way:
```
git rebase --interactive [commit-hash]
```

Where `commit-hash` is the hash of the commit just before the first one you want to rewrite from.
So in my example the command would be:
```
git rebase --interactive e807180
```

Where `e807180` is Feature Y. You can read the whole thing as:
> Merge all my commits on top of commit [commit-hash].

Way easier, isn't it?

## picking and squashing

At this point your editor of choice will pop up, showing the list of commits you want to merge.
Note that it might be confusing at first, since they are displayed in a reverse order,
where the older commit is on top.

I've added `--- older commit` and `--- newer commit` to make it clear, you won't find those notes in the editor.
```
3aee03c #454, prepare new CI               --- older commit
6f577b6 fix                 
a5398ae fix
2673619 fix
67f56b9 fix
f1a9a3e fix
639c220 #454, CI is fully implemented      --- newer commit

[...]
```

Below the commit list there is a short comment (omitted in my example)
which outlines all the operations available.
You can do many smart tricks during an interactive rebase,
let's stick with the basics for now though.
Our task here is to mark all the commits as squashable, except the first/older one:
it will be used as a starting point.

You mark a commit as squashable by changing the work pick into squash next to it (or s for brevity, as stated in the comments). The result would be:
```
pick 3aee03c #454, prepare new CI                --- older commit
s 6f577b6 fix                 
s a5398ae fix
s 2673619 fix
s 67f56b9 fix
s f1a9a3e fix
s 639c220 #454, CI is fully implemented         --- newer commit

[...]
```

Save the file and close the editor.

## create the new commit

You have just told Git to combine all seven commits into the the first commit in the list.
It's now time to give it a name: your editor pops up again with a default message,
made of the names of all the commits you have squashed.

I wipe out the default message and use something more self-explanatory like `Implemented CI`.

## pushing the squashed commit

If the commits have been pushed to the remote:
```
git push origin +name-of-branch
```

The plus sign [forces][force-push] the remote branch to accept your rewritten history,
otherwise you will end up with divergent branches.

If the commits have NOT yet been pushed to the remote:
```
git push origin name-of-branch
```

In other words, just a normal push like any other

## conclusion

Fixing up your commits in this way is good practice before sharing with your team
members or before pushing the changes to a remote repository.
It's much easier to read through a source tree and understand what bugs have been
fixed when a single commit fixes a single bug, for example.

## Additional information

* [git-rebase - Reapply commits on top of another base tip](https://git-scm.com/docs/git-rebase) -
  Full information about git-rebase command.
* [Squash commits into one with Git](https://www.internalpointers.com/post/squash-commits-into-one-git) -
  The base of this article.
* [Git_ Squash Multiple Commits in to One Commit](https://stackabuse.com/git-squash-multiple-commits-in-to-one-commit/) -
  Another good source that I use to get more knowledge about git squash.
* [How can I merge two commits into one if I already started rebase - Stack Overflow](https://stackoverflow.com/questions/2563632/how-can-i-merge-two-commits-into-one-if-i-already-started-rebase) -
  Conversation in Stack Overflow about this topic.


[force-push]: https://git-scm.com/docs/git-push#Documentation/git-push.txt---force
