---
title: 使用 Homebrew 安装特定版本的软件
date: 2015-11-4
tags: ["mac", "homebrew"]
categories: "mac"
---
点击[查看原文](http://stackoverflow.com/questions/3987683/homebrew-install-specific-version-of-formula)：

Let’s start with the simplest case:

#### 1) Check, whether the version is already installed (but not activated)

When homebrew installs a new formula, it puts it in a versioned directory like `/usr/local/Cellar/postgresql/9.3.1`. Only symbolic links to this folder are then installed globally. In principle, this makes it pretty easy to switch between two installed versions. (*)

If you have been using homebrew for longer and never removed older versions (using, for example `brew cleanup`), chances are that some older version of your program may still be around. If you want to simply activate that previous version, `brew switch` is the easiest way to do this.

Check with `brew info postgresql` (or `brew switch postgresql <TAB>`) whether the older version is installed:

```
$ brew info postgresql
postgresql: stable 9.3.2 (bottled)
http://www.postgresql.org/
Conflicts with: postgres-xc
/usr/local/Cellar/postgresql/9.1.5 (2755 files, 37M)
  Built from source
/usr/local/Cellar/postgresql/9.3.2 (2924 files, 39M) *
  Poured from bottle
From: https://github.com/Homebrew/homebrew/commits/master/Library/Formula/postgresql.rb
# … and some more
```

We see that some older version is already installed. We may activate it using `brew switch`:

```
$ brew switch postgresql 9.1.5
Cleaning /usr/local/Cellar/postgresql/9.1.5
Cleaning /usr/local/Cellar/postgresql/9.3.2
384 links created for /usr/local/Cellar/postgresql/9.1.5
```

Let’s double-check what is activated:

```
$ brew which postgresql
postgresql: 9.1.5
```

> Please note that `brew switch` only works as long as all dependencies of the older version are still around. In some cases, a rebuild of the older version may become necessary. Therefore, using `brew switch` is mostly useful when one wants to switch between two versions not too far apart.

#### 2) Check, whether the version is available as a tap

Especially for larger software projects, it is very probably that there is a high enough demand for several (potentially API incompatible) major versions of a certain piece of software. As of March 2012, Homebrew 0.9 provides a mechanism for this: `brew tap` & the homebrew versions repository.

That versions repository may include backports of older versions for several formulae. (Mostly only the large and famous ones, but of course they’ll also have several formulae for postgresql.)

`brew search postgresql` will show you where to look:

```
$ brew search postgresql
postgresql
homebrew/versions/postgresql8    homebrew/versions/postgresql91
homebrew/versions/postgresql9    homebrew/versions/postgresql92
```

We can simply install it by typing

```
$ brew install homebrew/versions/postgresql8
Cloning into '/usr/local/Library/Taps/homebrew-versions'...
remote: Counting objects: 1563, done.
remote: Compressing objects: 100% (943/943), done.
remote: Total 1563 (delta 864), reused 1272 (delta 620)
Receiving objects: 100% (1563/1563), 422.83 KiB | 339.00 KiB/s, done.
Resolving deltas: 100% (864/864), done.
Checking connectivity... done.
Tapped 125 formula
==> Downloading http://ftp.postgresql.org/pub/source/v8.4.19/postgresql-8.4.19.tar.bz2
# …
```
> Note that this has automatically tapped the homebrew/versions tap. (Check with brew tap, remove with brew untap homebrew/versions.) The following would have been equivalent:

```
$ brew tap homebrew/versions
$ brew install postgresql8
```

As long as the backported version formulae stay up-to-date, this approach is probably the best way to deal with older software.

#### 3) Try some formula from the past

The following approaches are listed mostly for completeness. Both try to resurrect some undead formula from the brew repository. Due to changed dependencies, API changes in the formula spec or simply a change in the download URL, things may or may not work.

Since the whole formula directory is a git repository, one can install specific versions using plain git commands. However, we need to find a way to get to a commit where the old version was available.

###### a) historic times

Between August 2011 and October 2014, homebrew had a `brew versions` command, which spat out all available versions with their respective SHA hashes. As of October 2014, you have to do a `brew tap homebrew/boneyard` before you can use it. As the name of the tap suggests, you should probably only do this as a last resort.

E.g.

```
$ brew versions postgresql
Warning: brew-versions is unsupported and may be removed soon.
Please use the homebrew-versions tap instead:
  https://github.com/Homebrew/homebrew-versions
9.3.2    git checkout 3c86d2b Library/Formula/postgresql.rb
9.3.1    git checkout a267a3e Library/Formula/postgresql.rb
9.3.0    git checkout ae59e09 Library/Formula/postgresql.rb
9.2.4    git checkout e3ac215 Library/Formula/postgresql.rb
9.2.3    git checkout c80b37c Library/Formula/postgresql.rb
9.2.2    git checkout 9076baa Library/Formula/postgresql.rb
9.2.1    git checkout 5825f62 Library/Formula/postgresql.rb
9.2.0    git checkout 2f6cbc6 Library/Formula/postgresql.rb
9.1.5    git checkout 6b8d25f Library/Formula/postgresql.rb
9.1.4    git checkout c40c7bf Library/Formula/postgresql.rb
9.1.3    git checkout 05c7954 Library/Formula/postgresql.rb
9.1.2    git checkout dfcc838 Library/Formula/postgresql.rb
9.1.1    git checkout 4ef8fb0 Library/Formula/postgresql.rb
9.0.4    git checkout 2accac4 Library/Formula/postgresql.rb
9.0.3    git checkout b782d9d Library/Formula/postgresql.rb
```

As you can see, it advises against using it. Homebrew spits out all versions it can find with its internal heuristic and shows you a way to retrieve the old formulae. Let’s try it.

```
# First, go to the homebrew base directory
$ cd $( brew --prefix )
# Checkout some old formula
$ git checkout 6b8d25f Library/Formula/postgresql.rb
$ brew install postgresql
# … installing
```

Now that the older postgresql version is installed, we can re-install the latest formula in order to keep our repository clean:

```
$ git checkout -- Library/Formula/postgresql.rb
```

`brew switch` is your friend to change between the old and the new.

###### b) prehistoric times

For special needs, we may also try our own digging through the homebrew repo.

```
$ git log -S'8.4.4' -- Library/Formula/postgresql.rb
```

`git log -S` looks for all commits in which the string '8.4.4' was either added or removed in the file `Library/Formula/postgresql.rb`. We get two commits as a result.

```
commit 7dc7ccef9e1ab7d2fc351d7935c96a0e0b031552
Author: Aku Kotkavuo
Date:   Sun Sep 19 18:03:41 2010 +0300

    Update PostgreSQL to 9.0.0.

    Signed-off-by: Adam Vandenberg

commit fa992c6a82eebdc4cc36a0c0d2837f4c02f3f422
Author: David Höppner
Date:   Sun May 16 12:35:18 2010 +0200

    postgresql: update version to 8.4.4
```

Obviously, `fa992c6a82eebdc4cc36a0c0d2837f4c02f3f422` is the commit we’re interested in. As this commit is pretty old, we’ll try to downgrade the complete homebrew installation (that way, the formula API is more or less guaranteed to be valid):

```
$ git checkout -b postgresql-8.4.4 fa992c6a82eebdc4cc36a0c0d2837f4c02f3f422
$ brew install postgresql
$ git checkout master
$ git branch -d postgresql-8.4.4
```

You may skip the last command to keep the reference in your git repository.

One note: When checking out the older commit, you temporarily downgrade your homebrew installation. So, you should be careful as some commands in homebrew might be different to the most recent version.

#### 4) Manually write a formula

It’s not too hard and you may then upload it to Homebrew-Versions.

###### A.) Bonus: Pinning

If you want to keep a certain version of, say postgresql, around and stop it from being updated when you do the natural brew update; brew upgrade procedure, you can pin a formula:

```
$ brew pin postgresql
```

Pinned formulae are listed in /usr/local/Library/PinnedKegs/ and once you want to bring in the latest changes and updates, you have to call

```
$ brew unpin postgresql
```

