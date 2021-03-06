
<!--
# A Centralized Version Control Repository For Isolated Configuration Files, Dot Files, Preferences, Etc.
-->

[[Version Control Systems]] are programs that maintain a history of edits or changes to a collection of files.  

I often want to use versioning for individual isolated files, or collections of files, that live somewhere in my directory tree.  However, because management of many independent repositories is hard, I am reluctant to create and maintain a repository wherever the file lives if __the directory itself__ is not under active development.  

Some examples of files like this are dotfiles in my home directory, and other preference or configuration files throughout my directory tree.

# Instructions

Here's a solution that was based on a suggestion by Casey Dahlin on Doug Warner's [[blog][Doug Warner]].   Comments from Peter Manis, Benjamin Schmidt and Martin Geisler were also helpful [[here][Peter Manis]], [[here][How do I find the largest filesdirectories]], and [[here][Mercurial]].

The idea is to create and manage __a single repository__ for these files that will be easy to manage.  Files can be added manually.  Regular commits will be made on an ongoing basis, and can even be automated.

## > initialization

First, create the repo in your home directory

``` bash
cd $HOME
hg init
```

## > safety precautions

[[Peter Manis]] points out that the `hg purge` command can remove all files in the __working directory__ that are not added to the repo!!  He advises to explicitly disable this command for the repo by adding the following to the project-level `.hgrc` file located in `$HOME/.hg/hgrc`

``` text
[extensions]
hgext.purge = !
```

## > list, add, forget, remove, commit, status

You can list, add, forget, remove, and commit files to the repo with the following commands

``` bash
hg manifest
hg add &lt;files&gt;
hg forget &lt;files&gt;
hg remove &lt;files&gt;
hg commit -m "Added/removed/changed file(s)"
```

The status command reports on all files in the __working directory__ whether they have been added to the repo or not.  This can take a long time.

``` bash
hg status
```

## > the default repo

To access the centralized repo from directories other than your `$HOME` directory, set the __default__ path in your user-level `.hgrc` file located in `$HOME/.hgrc`

``` text
[path]
default = $HOME
```

The centralized repo is accessible only if the __current working directory__ (PWD) is not itself the __working directory__ of another repo.

__Warning__: A danger of using this preference is that you may end up using the centralized repo when you intended to use a local repo.  For example, if you accidentally call `hg` from a non-versioned directory.

To identify which repo you are working on, just type

``` bash
hg showconfig bundle.mainreporoot
```

An easier solution is to add the following to your user-level `.hgrc` file

``` text
[hooks]
pre-status = echo "======\nMAIN REPO ROOT = $PWD\n======"
pre-manifest = echo "======\nMAIN REPO ROOT = $PWD\n======"
```

This way, whenevery you type `hg status` or `hg manifest`, you will be told which repo is active.


## > the .hgignore file

An alternative strategy for managing repo files, is to create an `.hgignore` file listing the files that you do not wish to be tracked, and then add / commit everything else.
 
A simple `.hgignore` file looks like this.

``` text
syntax: glob
*~
.*.swp
*.swp
*.o
.hg

syntax: regexp
^[a-zA-Z]

.file1
^\.file2\/file3
```

This file excludes several standard temporary files, any file named ".file1", and files matching "file2/file3" in the repo's root, __working directory__.

After editing the `.hgignore` file to your liking, you can preview your choices with

``` bash
# 1. show "added" files (will be included in the next commit)
hg status -a
# 2. show "unknown" files (will not be included in the next commit)
hg status -u
```

Then, you can use the following shorthand to: 1) add all unknown files and commit the changes to the repo, and 2) view, the resulting contents

``` bash
hg commit -A -m "Added/removed/changed file(s)"
hg manifest
```

## > .file & .directory sizes

One trick for building `.hgignore` is to detect and exclude __LARGE__ dotfiles and directories.  At first, I tried to these using

``` bash
ls -lSd .* | head -20
```

However, `ls` does not measure the size of directories when reporting relative size.  To see the largest items accounting for the total size of directories,
use the following

``` bash
for X in $(du -s .[a-zA-Z]* | sort -nr | cut -f 2); do du -hs $X ; done | head -20
```

This will produce sorted output, for example

``` text
8.2G	.Trash
 99M	.dropbox
 21M	.m2
 19M	.groovy
6.0M	.macports
3.7M	.vim
1.8M	.fontconfig
976K	.ipython
...
```

## > resetting the repo

If you are not happy with the current manifest, and are willing to start again __from scratch__, use the following commands.  WARNING: This will erase any history!

``` bash
cd $HOME
\rm -rf .hg
hg init
hg commit -A -m "Added/removed/changed file(s)"
hg manifest
```

This can be used to refine the `.hgignore` file in order to initialize the repo


# Glossary

Current Working Directory
: PWD

Working Directory
: "To introduce a little terminology, the .hg directory is the “real” repository, and all of the files and directories that coexist with it are said to live in the working directory. An easy way to remember the distinction is that the repository contains the history of your project, while the working directory contains a snapshot of your project at a particular point in history." [[quote][a-tour-of-mercurial-the-basics]]

Hg Init
: Create a fresh repo.  Fails when an existing repo exists in the working directory.

Hg Manifest
: List files currently in repo

Hg Add
: Add a file to repo

Hg Forget
: Forget files previously added to repo, before committing

Hg Remove
: Remove files previously added to repo, after they have been committed

Hg Commit
: Commit all changes to repo


[Version Control Systems]: http://en.wikipedia.org/wiki/Revision_control
[Doug Warner]: http://doug.warner.fm/d//blog/2008/07/Version-controlling-my-home-dir
[Peter Manis]: http://pyverted.com/version-control/using-mercurial-on-your-home-directory/2009/08/
[How do I find the largest filesdirectories]: http://www.cyberciti.biz/faq/how-do-i-find-the-largest-filesdirectories-on-a-linuxunixbsd-filesystem/
[Mercurial]: http://mercurial.selenic.com/wiki/TipsAndTricks
[a-tour-of-mercurial-the-basics]: http://hgbook.red-bean.com/read/a-tour-of-mercurial-the-basics.html


