# zmm - a tiny dotfiles manager with make and git

zmm makes your dotfiles with a `Makefile`, and installs them into `$HOME`.
It uses Git to manage both the changes from the `Makefile` and your local changes, and merges them like this:

```
  Makefile
     |
    make
     |
     v
  [draft] ---.
              +-- merge --> [proof] -- ff-merge --> [master]
  [master] --'                                         |
     ^                                                 |
     |                                                 |
   commit                                              |
     |                                                 |
   $HOME <---------------- checked out ----------------'
     ^
     |
  local changes
```

where the names in brackets are Git branches:

- `[master]`: History of the home directory
- `[draft]`: History of the Makefile output, without local changes
- `[proof]`: The draft branch merged into the master branch, to resolve conflicts outside the home directory

## Requirements

- Bash (3.2 is fine), Git 2.35 or later, and make
- Linux, or macOS 11 or later (no Homebrew needed)

## Install

From a clone of this repository:

```sh
install -m 755 bin/zmm ~/.local/bin/zmm
```

## Get started

```sh
zmm init
git clone <your dotfiles repository> ~/.zmm/src   # or write ~/.zmm/src/Makefile
zmm install
```

The `Makefile` installs into `$(DESTDIR)`, which zmm replaces with `$HOME`:

```makefile
install:
	install -m 644 config/bashrc $(DESTDIR)/.bashrc
```

## Rules for the Makefile

- Write an `install` target that installs ALL your dotfiles
- Install into `$(DESTDIR)`, not `$HOME`
- Write a `clean` target, if you use `--clean`
- Put no newlines in the file names of your dotfiles

## Manage other places

Set `ZMM_HOME` to install into another place than `$HOME`, with the zmm directory at `$ZMM_HOME/.zmm`:

```sh
alias zmm-etc='sudo ZMM_HOME=/etc zmm'
zmm-etc init
zmm-etc install
```

Set `ZMM_DIR` to have more zmm directories in one place, each with its own `Makefile`:

```sh
alias zmm-work='ZMM_DIR=$HOME/.zmm-work zmm'
```

Keep each file in one zmm directory only.
zmm keeps the modes of the files, but not their owners and groups: the files that zmm writes belong to the user running it.

## Learn more

- [demo/](demo/README.en.md): a walk through zmm ([日本語](demo/README.md))
- `zmm help`: the commands and the files
- 枯れた技術の水平思考: zmm puts together make and Git, both mature and well refined, for a new use.
