# zmm - A tiny dotfiles manager with Git

This script is a simple dotfiles manager that uses Git to manage dotfiles.
It installs dotfiles with your Makefile and commits changes to the Git repository.

0. Run `zmm init` to create the bare Git repository at `~/.zmm/home.git`.
1. Create a Makefile to install dotfiles for `<package>`
   at `~/.zmm/dotfiles/<package>/Makefile`.
2. Run `zmm store <package>` to store original files to the Git repository
   and avoid conflicts with the installed files
3. Run `zmm install <package>` to install files and manages file changes.
4. Manage installed files with `zmm git` command.
