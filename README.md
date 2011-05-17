
<!--
# A Centralized Version Control Repository For Isolated Configuration Files, Dot Files, Preferences, Etc.
-->

Version control systems are programs that maintain versioning/development/editing history for a collection of files.  

I often want to use versioning for individual isolated files, or collections of files, that live somewhere in my directory tree.  However, because management of many independent repositories is hard, I am reluctant to create and maintain a repository wherever the file lives if __the directory itself__ is not under active development.  

Some examples of files like this are dotfiles in my home directory, and other preference or configuration files throughout my directory tree.

# Instructions

Here's a solution that was based on a suggestion by Casey Dahlin on Doug Warner's [[blog][Doug Warner]].   Comments from Peter Manis, Benjamin Schmidt and Martin Geisler were also helpful [[here][Peter Manis]], [[here][How do I find the largest filesdirectories]], and [[here][Mercurial]].

## initialization

The idea is to create and manage __a single repository__ for these files that will be easy to manage.  Files can be added manually.  Regular commits will be made on an ongoing basis, and can even be automated.

First, create the repository in your home directory

<pre class="brush: bash; gutter: true; toolbar: false;">
{{{ bash
cd $HOME
hg init
}}}
</pre>

## safety precautions

[[Peter Manis]] points out that the `hg purge` command can remove all files in the _working directory_ that are not added to the repo!!  He advises to explicitly disable this command by adding the following to `$HOME/.hg/hgrc`

<pre class="brush: text; gutter: true; toolbar: false;">
[extensions]
hgext.purge = !
</pre>

## list, add, remove, commit

You can list, add, remove, and commit to the repository with the following commands

<pre class="brush: bash; gutter: true; toolbar: false;">
{{{ bash
hg manifest
hg add &lt;files&gt;
hg forget &lt;files&gt;
hg commit -m "Added/removed/changed file(s)"
}}}
</pre>

## the default repo

To have `hg` use this repository by default, add the following to your user-level preferences `.hgrc` file

<pre class="brush: text; gutter: true; toolbar: false;">
[path]
default = $HOME
</pre>




## the .hgignore file

An alternative strategy for managing repo files, is to create an `.hgignore` file listing the files that you do not wish to be tracked, and then add / commit everything else.
 
A simple `.hgignore` file looks like this.

<pre class="brush: text; gutter: true; toolbar: false;">
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

</pre>

This file excludes several standard temporary files, any file named ".file1", and files matching "file2/file3" in the repo's root, the _working directory_.

After editing the `.hgignore` file to your liking, you can preview your choices with

<pre class="brush: bash; gutter: true; toolbar: false;">
{{{ bash
# 1. show "added" files (will be included in the next commit)
hg status -a
# 2. show "unknown" files (will not be included in the next commit)
hg status -u
}}}
</pre>

Then, you can use the following shorthand to: 1) add all unknown files and commit the changes to the repo, and 2) view, the resulting contents

<pre class="brush: bash; gutter: true; toolbar: false;">
{{{ bash
hg commit -A -m "Added/removed/changed file(s)"
hg manifest
}}}
</pre>

## .file & .directory sizes

One trick for building `.hgignore` is to detect and exclude __LARGE__ dotfiles and directories.  At first, I tried to these using

<pre class="brush: bash; gutter: true; toolbar: false;">
{{{ bash
ls -lSd .* | head -20
}}}
</pre>

However, `ls` does not measure the size of directories when reporting relative size.  To see the largest items accounting for the total size of directories,
use the following

<pre class="brush: bash; gutter: true; toolbar: false;">
{{{ bash
for X in $(du -s .[a-zA-Z]* | sort -nr | cut -f 2); do du -hs $X ; done | head -20
}}}
</pre>

## resetting the repo

If you are not happy with the current manifest, and are willing to start again __from scratch__, use the following commands.  WARNING: This will erase any history!

<pre class="brush: bash; gutter: true; toolbar: false;">
{{{ bash
cd $HOME
\rm -rf .hg
hg init
hg commit -A -m "Added/removed/changed file(s)"
hg manifest
}}}
</pre>

This can be used to refine the `.hgignore` file in order to initialize the repo



[Doug Warner]: http://doug.warner.fm/d//blog/2008/07/Version-controlling-my-home-dir
[Peter Manis]: http://pyverted.com/version-control/using-mercurial-on-your-home-directory/2009/08/
[How do I find the largest filesdirectories]: http://www.cyberciti.biz/faq/how-do-i-find-the-largest-filesdirectories-on-a-linuxunixbsd-filesystem/
[Mercurial]: http://mercurial.selenic.com/wiki/TipsAndTricks
[a-tour-of-mercurial-the-basics]: http://hgbook.red-bean.com/read/a-tour-of-mercurial-the-basics.html
 


# Glossary

Working Directory
: "To introduce a little terminology, the .hg directory is the “real” repository, and all of the files and directories that coexist with it are said to live in the working directory. An easy way to remember the distinction is that the repository contains the history of your project, while the working directory contains a snapshot of your project at a particular point in history." [[quote][a-tour-of-mercurial-the-basics]]

Hg Init
: Create a fresh repository.  Fails when an existing repository exists in the working directory.

Hg Manifest
: List files currently in repo

Hg Add
: Add a file to repo

Hg forget
: Forget files previously added to repo, before committing

Hg Commit
: Commit all changes to repo


