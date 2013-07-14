nostalgic
=========

#### Managing your 'dotfiles' with Git

Why is it so complicated? There are many projects out there that help you
manage your dotfiles via an SCM. I've played with a few but not one of them
felt right for me. Why is something so simple written in Ruby or Python?
Why is something so simply have so many SLOC? A tool managing dotfiles is
begging to be written in shell script. If I'm provisioning a new system (and
I do this a lot with Solaris and Linux) I don't want to install a bunch of
tools needed to pull in my perfect and "nostalgic" home environment. I already
have access to bash in a default OS installation and I only want to worry about
installing Git.

Requirements
------------

 * [Bash](http://en.wikipedia.org/wiki/Bash_(Unix_shell)
 * [Git](http://git-scm.com/)

Features
--------

 * files and directories are symlinked directly from inside git repos to $HOME
 * any number of remote git repos can be cloned and symlinked
 * conflicts can be skipped, auto fixed, or interactively fixed
 * add new files to a specified git repo
 * git pull remote changes for git repos
 * Simple!

NOTE
----

One thing to know about this script is every file and directory symlinked from
a git repo is prefixed WITH a dot(.) and each of these files is stored in the
git repo WITHOUT a dot(.). Please submit a pull request if you like the script
but not this behaviour. And make sure it's configurable via a command line
option.

Another thing to know is this script does not fully manage your git repos for
you. It only goes as far to "git clone" new repos, "git add" new files, "git
pull" remote updates, and "git status" for repo status. Any other stuff you
need to do for managing your repos (i.e. submit!) you'll have to do manually.

It's best to get in the habit of pulling frequently and submitting any changes
immediately after they're made. Adhering to this procedure will prevent
conflicts.

HowTo
-----

#### Usage
```
 Usage: nostalgic [options] <cmd> [args]

 Options:
   -n         dry run (no changes are performed)
   -s         skip conflicts
   -f         auto fix conflicts (default is interactive)
   -p         Include files already starting with a dot
   -r <dir>   repo dir (default = $HOME/dots)
   -d <dir>   symlink destination dir (default = $HOME)

 Commands:
   clone <uri>            clone URI as a repo to be used
   list                   list cloned repos being used
   pull <repo>            git pull for the repo ('ALL' for all repos)
   status <repo>          git status for the repo ('ALL' for all repos)
   symlink <repo>         create symlinks for the repo ('ALL' for all repos)
   track <file> <repo>    add a file to a repo, git add and symlink the file
```

#### Example

Cloned git repos live under $HOME/dots (overridden with -r).

Symlinked files go under $HOME (overridden with -d).

```
~ % mkdir dots

~ % mkdir test

~ % nostalgic clone https://github.com/insanum/dotfiles.git
--> cloning [/home/insanum/dots/dotfiles]
Cloning into '/home/insanum/dots/dotfiles'...
remote: Counting objects: 3, done.
remote: Total 3 (delta 0), reused 0 (delta 0)
Unpacking objects: 100% (3/3), done.

~ % ls dots
dotfiles/

~ % mkdir priv

~ % cd priv

~/priv % git init --bare
Initialized empty Git repository in /home/insanum/priv/

~/priv % cd ..

~ % nostalgic clone $HOME/priv
--> cloning [/home/insanum/dots/priv]
Cloning into '/home/insanum/dots/priv'...
warning: You appear to have cloned an empty repository.
done.

~ % nostalgic list
--> repo [/home/insanum/dots/priv] cloned from [/home/insanum/priv]
--> repo [/home/insanum/dots/dotfiles] cloned from [https://github.com/insanum/dotfiles.git]

~ % nostalgic status ALL
--> git status for repo [/home/insanum/dots/priv]
# On branch master
#
# Initial commit
#
nothing to commit (create/copy files and use "git add" to track)
--> git status for repo [/home/insanum/dots/dotfiles]
# On branch master
nothing to commit (working directory clean)

~ % cd test

~/test % ls -a
./  ../

~/test % touch .vimrc

~/test % touch .bashrc

~/test % touch .passwd

~/test % ls -a
./  ../  .bashrc  .passwd  .vimrc

~/test % nostalgic track .bashrc dotfiles
--> adding [/home/insanum/dots/dotfiles/bashrc] to repo [/home/insanum/dots/dotfiles]
--> symlinked [/home/insanum/dots/dotfiles/bashrc] to [.bashrc]
--> git status for repo [/home/insanum/dots/dotfiles]
# On branch master
# Changes to be committed:
#   (use "git reset HEAD <file>..." to unstage)
#
#       new file:   bashrc
#

~/test % ls -a
./  ../  .bashrc@  .passwd  .vimrc

~/test % nostalgic track .vimrc dotfiles
--> adding [/home/insanum/dots/dotfiles/vimrc] to repo [/home/insanum/dots/dotfiles]
--> symlinked [/home/insanum/dots/dotfiles/vimrc] to [.vimrc]
--> git status for repo [/home/insanum/dots/dotfiles]
# On branch master
# Changes to be committed:
#   (use "git reset HEAD <file>..." to unstage)
#
#       new file:   bashrc
#       new file:   vimrc
#

~/test % nostalgic track .passwd priv
--> adding [/home/insanum/dots/priv/passwd] to repo [/home/insanum/dots/priv]
--> symlinked [/home/insanum/dots/priv/passwd] to [.passwd]
--> git status for repo [/home/insanum/dots/priv]
# On branch master
#
# Initial commit
#
# Changes to be committed:
#   (use "git rm --cached <file>..." to unstage)
#
#       new file:   passwd
#

~/test % nostalgic status ALL
--> git status for repo [/home/insanum/dots/priv]
# On branch master
#
# Initial commit
#
# Changes to be committed:
#   (use "git rm --cached <file>..." to unstage)
#
#       new file:   passwd
#
--> git status for repo [/home/insanum/dots/dotfiles]
# On branch master
# Changes to be committed:
#   (use "git reset HEAD <file>..." to unstage)
#
#       new file:   bashrc
#       new file:   vimrc
#

~/test % ls -a
./  ../  .bashrc@  .passwd@  .vimrc@

~/test % rm -i .*
rm: cannot remove ‘.’: Is a directory
rm: cannot remove ‘..’: Is a directory
rm: remove symbolic link ‘.bashrc’? y
rm: remove symbolic link ‘.passwd’? y
rm: remove symbolic link ‘.vimrc’? y

~/test % ls -a
./  ../

~/test % nostalgic -d ~/test symlink ALL
--> symlinking repo [/home/insanum/dots/priv]
--> symlinked [passwd]
--> symlinking repo [/home/insanum/dots/dotfiles]
--> symlinked [bashrc]
--> symlinked [vimrc]

~/test % ls -al
total 8
drwxr-xr-x  2 insanum users 4096 Jun 23 17:32 ./
drwx------ 45 insanum users 4096 Jun 23 17:32 ../
lrwxrwxrwx  1 insanum users   34 Jun 23 17:32 .bashrc -> /home/insanum/dots/dotfiles/bashrc
lrwxrwxrwx  1 insanum users   34 Jun 23 17:32 .passwd -> /home/insanum/dots/priv/passwd
lrwxrwxrwx  1 insanum users   33 Jun 23 17:32 .vimrc -> /home/insanum/dots/dotfiles/vimrc

~/test % cd

~ % ls -a dots/*
dots/priv:
./  ../  .git/  passwd

dots/dotfiles:
./  ../  bashrc  .git/  vimrc
```

