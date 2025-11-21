# nostalgic

A simple dotfile manager using git and symlinks. Only requires bash and git!

When provisioning a new system, you shouldn't need complex environment tools just to manage dotfiles. With bash and git available on every system, that's all you need.

## Requirements

Required tools:
 - `bash`
 - `git`

Standard tools used (available on all systems):
 - `find` - Finding files in repos
 - `dirname` - Path manipulation
 - `readlink` - Reading symlink targets
 - `mkdir` - Creating directories
 - `ln` - Creating symlinks
 - `mv` - Moving files
 - `rm` - Removing files

## How It Works

- Clone git repos containing your dotfiles to `$HOME/dots/` (configurable)
- Files stored in repos **without** dots are symlinked to `$HOME` **with** dots
- Alternatively, use the `-p` flag to store files **with** dots in the repo
- Directory trees are recreated and files underneath are symlinked
- Manage multiple repos (public dotfiles, private configs, etc.)

## Usage

```
 Usage: nostalgic [options] <cmd> [args]

 Options:
   -n         dry run (no changes are performed)
   -s         skip conflicts
   -f         auto fix conflicts (default is interactive)
   -p         include files already starting with a dot
   -r <dir>   repo dir (default = $HOME/dots)
   -d <dir>   symlink destination dir (default = $HOME)

 Commands:
   clone <uri>            clone URI as a repo to be used
   list                   list cloned repos being used
   pull <repo>            git pull for the repo ('ALL' for all repos)
   status <repo>          git status for the repo ('ALL' for all repos)
   symlink <repo>         create symlinks for the repo ('ALL' for all repos)
   track <file> <repo>    copy the file to the repo, git add it, and symlink it
   unlink <path> <repo>   remove dangling symlinks (path can be file or dir)
```

## Quick Start

```bash
# Clone your dotfiles repo (creates ~/dots/dotfiles/)
nostalgic clone https://github.com/insanum/dotfiles

# Create symlinks for all files in the repo
# Files like 'config/nvim/init.lua' in repo -> $HOME/.config/nvim/init.lua
nostalgic symlink dotfiles

# Track a new dotfile (moves file to repo, creates symlink)
nostalgic track ~/.zshrc dotfiles

# Pull latest changes from remote
nostalgic pull dotfiles

# Check git status
nostalgic status dotfiles

# Manage the repo directly for commits/pushes
cd $HOME/dots/dotfiles
git commit -m "added zshrc"
git push

# Work with multiple repos (e.g., separate private configs)
nostalgic clone git@github.com:user/private-configs
nostalgic symlink ALL
nostalgic pull ALL
nostalgic status ALL

# Clean up dangling symlinks after removing files from repo
nostalgic unlink ~/.config dotfiles
```

## Notes

- nostalgic does basic git operations (clone, add, pull, status)
- You must manually commit and push changes to your repos
- Files in repos are stored without the leading dot (unless using `-p`)
- Pull frequently and commit changes immediately to avoid conflicts

