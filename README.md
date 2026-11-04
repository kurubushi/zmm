# zmm - a tiny dotfiles manager with make and git

zmm proposes a simple workflow with just make and Git:

- Build your dotfiles with a `Makefile`: no template language to learn.
- Merge them with your local changes in Git, as a vendor branch.

`zmm` runs this workflow:

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
- `[draft]`: History of the Makefile output, without local changes (the vendor branch)
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
# create ~/.zmm/src/Makefile (see below)
zmm install
```

## Rules for the Makefile

- Write an `install` target that installs ALL your dotfiles
- Install into `$(DESTDIR)`, not `$HOME`
- Write a `clean` target, if you use `--clean`
- Put no newlines in the file names of your dotfiles

For example:

```makefile
install: build/git-prompt.sh
	install -m 644 config/gitconfig $(DESTDIR)/.gitconfig
	install -m 644 config/bashrc.$(shell uname -s) $(DESTDIR)/.bashrc
	install -m 644 build/git-prompt.sh $(DESTDIR)/.git-prompt.sh

build/git-prompt.sh:
	mkdir -p build
	curl -fsSL -o build/git-prompt.sh https://raw.githubusercontent.com/git/git/master/contrib/completion/git-prompt.sh

clean:
	rm -rf build

# ~/.zmm/src
# ├── Makefile
# └── config
#     ├── bashrc.Darwin
#     ├── bashrc.Linux
#     └── gitconfig

# ~ (after "zmm install")
# ├── .bashrc
# ├── .git-prompt.sh
# └── .gitconfig
```

## Secrets

`~/.zmm/local.git` records your `$HOME` as it is, so sharing it is not recommended.

To share a secret, commit it encrypted to `~/.zmm/src`, and decrypt it in the `Makefile` with any tool:

```makefile
install:
	gpg -d config/netrc.gpg > $(DESTDIR)/.netrc
	chmod 600 $(DESTDIR)/.netrc
```

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
