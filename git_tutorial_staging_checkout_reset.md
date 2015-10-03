# Understanding the git staging area

And the related commands, `git checkout` and `git reset`

**Author: Everett**

Based on https://www.atlassian.com/git/tutorials/resetting-checking-out-and-reverting/

## About this tutorial

The git **staging area** is a weird concept for most people. Why do we need it? Honestly, often we don't, but it can be very useful. At minimum, understanding its theoretical use cases can help demistify the `git add/commit/checkout/reset` weirdness.

We're also going to focus on individual files here. `git reset` and `git checkout` can be run on on entire branches (e.g. `git reset HEAD`) or just specific files (e.g. `git reset HEAD foo.py`). These have *very* different behaviors, and here, we are only focused on the latter. More loudly, **we're only concerned with git checkout/reset on individual files.**

One last thing -- the staging area is also called the "index" -- a much worse name, but the git documentation (git help...) uses that term. Just so you know.

### The three-headed git chimera

Let's say you have a file checked into your repo called `foo.py`, and its contents read 'FOO!' like this:
```
 _______________
|foo.py         |
|---------------|
|FOO!           |
|_______________|
```

There are actually **three** versions of this file in git's mind.

1. The version in the last commit
2. The version sitting in your workspace right now
3. The version you currently have staged

We'll walk through these, but right now, you haven't changed anything, so (1) and (2) are the same, and (3) flatly does not exist yet.

Now let's say you change foo.py, in a text editor, to have "BAR!" as its contents instead:
```
(Edit foo.py's contents)
 _______________
|foo.py         |
|---------------|
|BAR!           |
|_______________|
```

If you do this, and run `git status`, you'll see this file with a red `M` next to its name.
```
> git status
## master
 M foo.py
```
What does this mean?

It means the local version is modified from the version in the latest commit. All we've done is change the file itself, so we still do not have a staged version. Git still tracks three versions of the file, just the staging version does not exist yet. Like this:

```
Git's picture of foo.py

 [1.Last commit]  [2.Local workspace]  [3.Staging area]
 _______________    _______________   
|foo.py         |  |foo.py         |
|---------------|  |---------------|    (nothing yet)
|FOO!           |  |BAR!           |
|_______________|  |_______________|
```

If you were to run `git commit` now, nothing would happen. Only staged files can be committed, and there are none here. Git will just tell you to buzz off.

Running `git add foo.py` will copy the local version into the "staging" area, making it ready to commit (but not committed yet):

```
> git add foo.py

 [1.Last commit]  [2.Local workspace]  [3.Staging area]
 _______________    _______________    _______________
|foo.py         |  |foo.py         |  |foo.py         |
|---------------|  |---------------|  |---------------|
|FOO!           |  |BAR!           |  |BAR!           |
|_______________|  |_______________|  |_______________|
```

If you run `git status`, you'll see that the file now has a green 'M' next to it (and is moved one character to the left -- it's subtle but you'll see why it matters further down).

```
> git status
## master
M  foo.py
```

NOW you can `git commit`, and the staging version will be stored in a new commit. The last commit version and local version will be the same, and the staging version will disappear:
```
> git commit

 [1.Last commit]  [2.Local workspace]  [3.Staging area]
 _______________    _______________   
|foo.py         |  |foo.py         |
|---------------|  |---------------|    (nothing yet)
|BAR!           |  |BAR!           |
|_______________|  |_______________|
```

You might ask: why the hell would we do this? Can't we just commit the local changes directly, without this extra "staging" crap?

The answer is yes, yes we could, but having the staging layer gives us some extra powers (that many users don't need). Instead of making it an optional poweruser feature, git baked staging into the basic workflow. For people making simple changes, it's not useful -- in which case you can just run `git add` and `git commit` one after another all the time. But if you would like to be a wizard, Harry...

What are these magic powers? Well, mostly, we can have some provisional local changes that we're tinkering with (and may not keep), without messing everything up. For people doing lots of stuff, this is useful. More below.

### Using all three beasts

Right now we're back at sea level -- we just committed, and have no local changes. Let's climb back up to where we were. Change the file contents to "QUX!", and run `git add foo.py`. This will look familiar, as before:

```
(Edit and save file contents to QUX!)

 [1.Last commit]  [2.Local workspace]  [3.Staging area]
 _______________    _______________   
|foo.py         |  |foo.py         |
|---------------|  |---------------|    (nothing yet)
|BAR!           |  |QUX!           |
|_______________|  |_______________|

> git add foo.py

 [1.Last commit]  [2.Local workspace]  [3.Staging area]
 _______________    _______________    _______________
|foo.py         |  |foo.py         |  |foo.py         |
|---------------|  |---------------|  |---------------|
|BAR!           |  |QUX!           |  |QUX!           |
|_______________|  |_______________|  |_______________|
```

Now edit foo.py **again** to add another line, "LOL!" to the end and save it. Do **not** git add it. Your files will now look like this, with three distinct versions

```
(Edit and save file contents to LOL!)

 [1.Last commit]  [2.Local workspace]  [3.Staging area]
 _______________    _______________    _______________
|foo.py         |  |foo.py         |  |foo.py         |
|---------------|  |---------------|  |---------------|
|BAR!           |  |QUX!           |  |QUX!           |
|               |  |LOL!           |  |               |
|_______________|  |_______________|  |_______________|
```

This is not a totally uncommon scenario. Usually you're working on many files at once, and it takes some time to get a commit ready. You have a change you're ready to commit to foo.py ('BAR' -> QUX'), but maybe don't commit it yet because you're editing other files to join the same commit. In the process, you decide to add the 'LOL' line, but it's just a little fix you thought of, and you want to save it for a different commit by itself later.

If you run `git status`, you'll see TWO 'M's next to foo.py, one green (for the staged version) and one red (for the changed local version)

```
> git status
## master
MM foo.py
```

Running `git commit` now will take the *staging area* and use it to make a new commit. The local file will remain unchanged, like so:
```
> git commit

 [1.Last commit]  [2.Local workspace]  [3.Staging area]
 _______________    _______________  
|foo.py         |  |foo.py         |
|---------------|  |---------------|     (nothing yet)
|QUX!           |  |QUX!           |
|               |  |LOL!           |
|_______________|  |_______________|

> git status
## master
 M foo.py
```

Now you can go and keep working on this fix, and add/commit if you decide to keep it.

### Reset and Checkout

#### `git reset`

In practice, `git reset HEAD [filename]` is a way to throw away the staging version of a file (leaving the actual local file unchanged). If you have no other local changes to it since staging it, it effectively unstages the file. For example:

```
If you have:

 [1.Last commit]  [2.Local workspace]  [3.Staging area]
 _______________    _______________    _______________
|foo.py         |  |foo.py         |  |foo.py         |
|---------------|  |---------------|  |---------------|
|QUX!           |  |QUX!           |  |QUX!           |
|               |  |LOL!           |  |LOL!           |
|_______________|  |_______________|  |_______________|

> git reset HEAD foo.py

 [1.Last commit]  [2.Local workspace]  [3.Staging area]
 _______________    _______________ 
|foo.py         |  |foo.py         |
|---------------|  |---------------|    (nothing yet)
|QUX!           |  |QUX!           |
|               |  |LOL!           |
|_______________|  |_______________|
```

If you had further local changes, they are unaffected. For example:

```
If you have:

 [1.Last commit]  [2.Local workspace]  [3.Staging area]
 _______________    _______________    _______________
|foo.py         |  |foo.py         |  |foo.py         |
|---------------|  |---------------|  |---------------|
|QUX!           |  |QUX!           |  |QUX!           |
|               |  |LOL!           |  |LOL!           |
|               |  |BBQ!           |  |               |
|_______________|  |_______________|  |_______________|

> git reset HEAD foo.py

 [1.Last commit]  [2.Local workspace]  [3.Staging area]
 _______________    _______________ 
|foo.py         |  |foo.py         |
|---------------|  |---------------|    (nothing yet)
|QUX!           |  |QUX!           |
|               |  |LOL!           |
|               |  |BBQ!           |
|_______________|  |_______________|
```

In theory you can `git reset` to any branch or ref (not just HEAD, the latest commit) -- you could do `git reset a6cf3 foo.py` which reverts the staging version to match the specified commit-ish. It's just not typically useful.

#### `git checkout`

There's two flavors of this guy: `git checkout -- [filename]` and `git checkout HEAD [filename]`.

`git checkout -- [filename]` discards any local changes, leaving the staging version (if any) unchanged.

```
If you have:

 [1.Last commit]  [2.Local workspace]  [3.Staging area]
 _______________    _______________    _______________
|foo.py         |  |foo.py         |  |foo.py         |
|---------------|  |---------------|  |---------------|
|QUX!           |  |QUX!           |  |QUX!           |
|               |  |LOL!           |  |LOL!           |
|               |  |BBQ!           |  |               |
|_______________|  |_______________|  |_______________|

> git checkout -- foo.py

 [1.Last commit]  [2.Local workspace]  [3.Staging area]
 _______________    _______________    _______________
|foo.py         |  |foo.py         |  |foo.py         |
|---------------|  |---------------|  |---------------|
|QUX!           |  |QUX!           |  |QUX!           |
|               |  |               |  |LOL!           |
|               |  |               |  |               |
|_______________|  |_______________|  |_______________|
```


`git checkout HEAD [filename]` discards any local changes **and** throws away the staging version, smashing them both back to the latest commit. Basically, this says "aahhhhh please undo any and all horrors that I've done to this file!"

```
If you have:

 [1.Last commit]  [2.Local workspace]  [3.Staging area]
 _______________    _______________    _______________
|foo.py         |  |foo.py         |  |foo.py         |
|---------------|  |---------------|  |---------------|
|QUX!           |  |QUX!           |  |QUX!           |
|               |  |LOL!           |  |LOL!           |
|               |  |BBQ!           |  |               |
|_______________|  |_______________|  |_______________|

> git checkout HEAD foo.py

 [1.Last commit]  [2.Local workspace]  [3.Staging area]
 _______________    _______________ 
|foo.py         |  |foo.py         |
|---------------|  |---------------|    (nothing yet)
|QUX!           |  |QUX!           |
|               |  |               |
|               |  |               |
|_______________|  |_______________|
```

(You can use a different ref or branch than HEAD, if you want to move these guys to another point in history for some reason)

More questions? Write your own tutorial, or start a conversation on #general

Everett


