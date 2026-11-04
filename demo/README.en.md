# zmm demo

[日本語](README.md)

Try zmm!

- [zmm in 3 minutes](#zmm-in-3-minutes)
- [How zmm works](#how-zmm-works)
- [1. The first install](#1-the-first-install)
- [2. Merging dotfiles](#2-merging-dotfiles)
- [3. Conflicts in dotfiles](#3-conflicts-in-dotfiles)
- [4. Other features of zmm](#4-other-features-of-zmm)
- [5. Using the Makefile](#5-using-the-makefile)
- [6. Using Git](#6-using-git)
- [Clean up](#clean-up)
- [Start with your own dotfiles](#start-with-your-own-dotfiles)

## zmm in 3 minutes

Prepare a `Makefile` that makes your dotfiles in `$(DESTDIR)`:

```makefile
install:
	install -m 644 config/bashrc $(DESTDIR)/.bashrc
	install -m 644 config/tmux.conf $(DESTDIR)/.tmux.conf

# ~/.zmm/src
# ├── Makefile
# └── config
#     ├── bashrc
#     └── tmux.conf
```

zmm installs your dotfiles from the `Makefile`:

```console
$ zmm install
zmm: made the draft in /home/doctor/.zmm/draft
zmm: committed the draft
zmm: merged the draft into proof in /home/doctor/.zmm/proof
 .bashrc    | 2 ++
 .tmux.conf | 1 +
 .zmm/perm  | 2 ++
 3 files changed, 5 insertions(+)
zmm: merged proof into master and the home directory
```

Later, you may edit a dotfile in `$HOME` by hand:

```diff
--- ~/.bashrc
+++ ~/.bashrc
@@ -1,2 +1,2 @@
-export EDITOR=vim
+export EDITOR=nano # edited manually
 export PAGER=less
```

Meanwhile, the `Makefile` changes the dotfile too:

```diff
--- ~/.zmm/src/config/bashrc
+++ ~/.zmm/src/config/bashrc
@@ -1,2 +1,3 @@
 export EDITOR=vim
 export PAGER=less
+alias ll='ls -l' # edited via Makefile
```

With zmm, you take the change from the `Makefile`, keeping the direct edit:

```console
$ zmm install
zmm: made the draft in /home/doctor/.zmm/draft
zmm: committed the draft
zmm: captured the home directory into master
 .bashrc | 2 +-
 1 file changed, 1 insertion(+), 1 deletion(-)
Auto-merging .bashrc
zmm: merged the draft into proof in /home/doctor/.zmm/proof
 .bashrc | 1 +
 1 file changed, 1 insertion(+)
zmm: merged proof into master and the home directory
$ cat ~/.bashrc
export EDITOR=nano # edited manually
export PAGER=less
alias ll='ls -l' # edited via Makefile
```

Git keeps the changes from the `Makefile` (draft) and the direct edits (capture) apart:

```console
$ zmm git log --oneline --graph -n 3
*   25199bf zmm: merge draft
|\  
| * b79fca0 zmm: update draft
* | f59c676 zmm: capture
```

If something goes wrong, you can go back. You can even undo only the change from the `Makefile`:

```console
$ zmm git revert --no-edit -m 1 HEAD
[master 861e9c8] Revert "zmm: merge draft"
 Date: Sat Oct 17 01:45:00 2026 +0900
 1 file changed, 1 deletion(-)
$ cat ~/.bashrc
export EDITOR=nano # edited manually
export PAGER=less
```

## How zmm works

zmm merges the output of the `Makefile` (draft) and `$HOME` (master) with Git:

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

The merge happens on the proof branch, and master and `$HOME` move forward to the result.

---

## 1. The first install

Ground rules of the demo:

- The zmm directory is `~/.zmm-demo` (default: `~/.zmm`)
- The demo touches only the dotfiles in `~/.rhodes` (instead of `~/.bashrc` or `~/.tmux.conf`)

First, put zmm in your PATH, and set the zmm directory:

```sh
install -m 755 bin/zmm ~/.local/bin/zmm
export ZMM_DIR=~/.zmm-demo
```

Initialize zmm:

```console
$ zmm init
zmm: initialized /home/doctor/.zmm-demo/local.git
```

Put the sample dotfiles src in place:

```sh
cp -R demo/src1 ~/.zmm-demo/src
```

[src1/Makefile](src1/Makefile) is a simple dotfiles src that only `install`s files into `$(DESTDIR)`:

```makefile
install:
	install -d $(DESTDIR)/.rhodes
	install -m 644 config/castle-3 $(DESTDIR)/.rhodes/castle-3
	install -m 644 config/lancet-2 $(DESTDIR)/.rhodes/lancet-2

# ~/.zmm-demo/src
# ├── Makefile
# └── config
#     ├── castle-3
#     └── lancet-2
```

Install:

```console
$ zmm install
zmm: made the draft in /home/doctor/.zmm-demo/draft
zmm: committed the draft
zmm: merged the draft into proof in /home/doctor/.zmm-demo/proof
 .rhodes/castle-3 | 5 +++++
 .rhodes/lancet-2 | 5 +++++
 .zmm-demo/perm   | 3 +++
 3 files changed, 13 insertions(+)
zmm: merged proof into master and the home directory
```

The dotfiles are there:

```console
$ ls ~/.rhodes
castle-3
lancet-2
```

Git has the history:

```console
$ zmm git log --oneline --graph
*   85bf8c3 zmm: merge draft
|\  
| * a658c84 zmm: update draft
|/  
* af82994 zmm: init
```

---

## 2. Merging dotfiles

Edit only `castle-3` directly:

```sh
vi ~/.rhodes/castle-3         # mode = 4wd to mode = 2wd
```

```console
$ zmm git diff
diff --git a/.rhodes/castle-3 b/.rhodes/castle-3
index 084ea49..e022c93 100644
--- a/.rhodes/castle-3
+++ b/.rhodes/castle-3
@@ -2,4 +2,4 @@
 assistant = hello.
 
 [drive]
-mode = 4wd
+mode = 2wd
```

At the same time, change the source of the `Makefile` ([src2/config/castle-3](src2/config/castle-3)):

```sh
rm -rf ~/.zmm-demo/src
cp -R demo/src2 ~/.zmm-demo/src
```

Install. Both changes stay:

```console
$ zmm install
zmm: made the draft in /home/doctor/.zmm-demo/draft
zmm: committed the draft
zmm: captured the home directory into master
 .rhodes/castle-3 | 2 +-
 1 file changed, 1 insertion(+), 1 deletion(-)
Auto-merging .rhodes/castle-3
zmm: merged the draft into proof in /home/doctor/.zmm-demo/proof
 .rhodes/castle-3 | 3 +++
 1 file changed, 3 insertions(+)
zmm: merged proof into master and the home directory
$ cat ~/.rhodes/castle-3
[greeting]
assistant = hello.

[shell]
armor = thick

[drive]
mode = 2wd
```

- Updating the `Makefile` does not wipe out your direct edits.
- Only the files whose output from the `Makefile` changed have their direct edits recorded (captured).

---

## 3. Conflicts in dotfiles

Say you already have a dotfile written by hand:

```console
$ cp demo/hand/big-ugly ~/.rhodes/
$ cat ~/.rhodes/big-ugly
[pilot]
autopilot = on
```

Let the `Makefile` manage it too ([src3/](src3/)):

```diff
 	install -m 644 config/lancet-2 $(DESTDIR)/.rhodes/lancet-2
+	install -m 644 config/big-ugly $(DESTDIR)/.rhodes/big-ugly
```

Replace the `Makefile`, and install:

```sh
rm -rf ~/.zmm-demo/src
cp -R demo/src3 ~/.zmm-demo/src
```

```console
$ zmm install
zmm: made the draft in /home/doctor/.zmm-demo/draft
zmm: committed the draft
zmm: captured the home directory into master
 .rhodes/big-ugly | 2 ++
 .zmm-demo/perm   | 1 +
 2 files changed, 3 insertions(+)
Auto-merging .rhodes/big-ugly
CONFLICT (add/add): Merge conflict in .rhodes/big-ugly
Automatic merge failed; fix conflicts and then commit the result.
hint: Resolve the conflicts in /home/doctor/.zmm-demo/proof, and run "zmm install --continue".
hint: To throw it away, run "zmm install --abort".
```

- The `big-ugly` written by hand was recorded (captured) before being overwritten. `$HOME` has not changed yet.

Resolve the conflict in `~/.zmm-demo/proof/`:

```console
$ cat ~/.zmm-demo/proof/.rhodes/big-ugly
[pilot]
<<<<<<< HEAD
autopilot = on
||||||| f69c8c9
=======
autopilot = off

[arms]
body = cannon
>>>>>>> draft
```

- `HEAD` is `$HOME`, and `draft` is the output of the `Makefile`.

Take the best of both in your editor:

```sh
vi ~/.zmm-demo/proof/.rhodes/big-ugly
```

```ini
[pilot]
autopilot = on

[arms]
body = cannon
```

Add it with `zmm proof-git`, and continue:

```console
$ zmm proof-git add .rhodes/big-ugly
$ zmm install --continue
zmm: merged proof into master and the home directory
$ cat ~/.rhodes/big-ugly
[pilot]
autopilot = on

[arms]
body = cannon
```

- `zmm proof-git` is an alias of `git -C ~/.zmm-demo/proof`.
- The resolution is recorded, so the same conflict never comes back. `$HOME` keeps `on`, even though the `Makefile` still says `autopilot = off`.

---

## 4. Other features of zmm

Go on from the previous chapter.

### 4.1 Check the output before an install

Edit the source:

```sh
vi ~/.zmm-demo/src/config/castle-3     # armor = thick to armor = thicker
```

`zmm make` lets you look at the output of the `Makefile`, leaving `$HOME` as it is:

```console
$ zmm make
zmm: made the draft in /home/doctor/.zmm-demo/draft
 .rhodes/castle-3 | 2 +-
 1 file changed, 1 insertion(+), 1 deletion(-)
hint: Check it with "zmm draft-git diff --cached", then run "zmm install --continue".
$ zmm draft-git diff --cached
diff --git a/.rhodes/castle-3 b/.rhodes/castle-3
index df80ce7..91105ee 100644
--- a/.rhodes/castle-3
+++ b/.rhodes/castle-3
@@ -2,7 +2,7 @@
 assistant = hello.
 
 [shell]
-armor = thick
+armor = thicker
 
 [drive]
 mode = 4wd
```

- The output goes to `~/.zmm-demo/draft/`.
- `zmm draft-git` is an alias of `git -C ~/.zmm-demo/draft`.

If it looks good, continue:

```console
$ zmm install --continue
zmm: committed the draft
Auto-merging .rhodes/castle-3
zmm: merged the draft into proof in /home/doctor/.zmm-demo/proof
 .rhodes/castle-3 | 2 +-
 1 file changed, 1 insertion(+), 1 deletion(-)
zmm: merged proof into master and the home directory
```

### 4.2 Abort at a conflict

Change the `mode` of `castle-3` on the `Makefile` side too:

```sh
vi ~/.zmm-demo/src/config/castle-3     # mode = 4wd to mode = awd
```

The same line clashes:

```console
$ zmm install
zmm: made the draft in /home/doctor/.zmm-demo/draft
zmm: committed the draft
Auto-merging .rhodes/castle-3
CONFLICT (content): Merge conflict in .rhodes/castle-3
Automatic merge failed; fix conflicts and then commit the result.
hint: Resolve the conflicts in /home/doctor/.zmm-demo/proof, and run "zmm install --continue".
hint: To throw it away, run "zmm install --abort".
```

To throw it away, use `--abort`:

```console
$ zmm install --abort
$ tail -n 1 ~/.rhodes/castle-3
mode = 2wd
```

- If the `Makefile` stays the same, the next install brings the same conflict again.

Put the source back, and no conflict comes:

```sh
vi ~/.zmm-demo/src/config/castle-3     # mode = awd to mode = 4wd
```

```console
$ zmm install
zmm: made the draft in /home/doctor/.zmm-demo/draft
zmm: committed the draft
zmm: merged the draft into proof in /home/doctor/.zmm-demo/proof
zmm: merged proof into master and the home directory
```

### 4.3 Uninstall

Delete the line of `lancet-2` from the `Makefile`, and install:

```diff
 	install -m 644 config/castle-3 $(DESTDIR)/.rhodes/castle-3
-	install -m 644 config/lancet-2 $(DESTDIR)/.rhodes/lancet-2
 	install -m 644 config/big-ugly $(DESTDIR)/.rhodes/big-ugly
```

```console
$ zmm install
zmm: made the draft in /home/doctor/.zmm-demo/draft
zmm: committed the draft
zmm: merged the draft into proof in /home/doctor/.zmm-demo/proof
 .rhodes/lancet-2 | 5 -----
 .zmm-demo/perm   | 1 -
 2 files changed, 6 deletions(-)
zmm: merged proof into master and the home directory
$ ls ~/.rhodes
big-ugly
castle-3
```

- A file the `Makefile` no longer makes goes away from `$HOME` too. There is no command to uninstall.
- The deleted file stays in the history (`zmm git show 85bf8c3:.rhodes/lancet-2`).

### 4.4 Keep the permissions

Make `castle-3` readable only by you:

```console
$ chmod 600 ~/.rhodes/castle-3
$ zmm git diff
diff --git a/.zmm-demo/perm b/.zmm-demo/perm
index a6ce86a..6e08319 100644
--- a/.zmm-demo/perm
+++ b/.zmm-demo/perm
@@ -1,3 +1,3 @@
 0755 .rhodes/
 0644 .rhodes/big-ugly
-0644 .rhodes/castle-3
+0600 .rhodes/castle-3
$ zmm git commit -qam 'castle-3: keep it private'
```

- The permissions are recorded in `~/.zmm-demo/perm`.

Edit the source, and install:

```sh
vi ~/.zmm-demo/src/config/castle-3     # armor = thicker to armor = thickest
```

```console
$ zmm install
zmm: made the draft in /home/doctor/.zmm-demo/draft
zmm: committed the draft
Auto-merging .rhodes/castle-3
zmm: merged the draft into proof in /home/doctor/.zmm-demo/proof
 .rhodes/castle-3 | 2 +-
 1 file changed, 1 insertion(+), 1 deletion(-)
zmm: merged proof into master and the home directory
$ grep armor ~/.rhodes/castle-3
armor = thickest
$ ls -l ~/.rhodes/castle-3 | cut -c 1-10
-rw-------
```

- It stays `600` after being rewritten. Plain Git would put it back to `644`.

### 4.5 Recover from a failed make

Say you deleted a source file by mistake:

```console
$ mv ~/.zmm-demo/src/config/castle-3 ~/.zmm-demo/src/config/castle-3.txt
$ zmm install
install: cannot stat 'config/castle-3': No such file or directory
make: *** [/home/doctor/.zmm-demo/src/Makefile:3: install] Error 1
$ ls ~/.rhodes
big-ugly
castle-3
```

- When make fails, zmm stops there too. `$HOME` does not change.

Put the source back, and all is as before:

```console
$ mv ~/.zmm-demo/src/config/castle-3.txt ~/.zmm-demo/src/config/castle-3
$ zmm install
zmm: made the draft in /home/doctor/.zmm-demo/draft
zmm: the draft is unchanged
zmm: the draft is already merged into master
```

### 4.6 Undo an install

Undo the last install:

```console
$ zmm git revert --no-edit -m 1 HEAD
[master 8322c22] Revert "zmm: merge draft"
 Date: Sat Oct 17 01:45:00 2026 +0900
 1 file changed, 1 insertion(+), 1 deletion(-)
$ grep armor ~/.rhodes/castle-3
armor = thicker
```

The undo stays after the next install:

```console
$ zmm install
zmm: made the draft in /home/doctor/.zmm-demo/draft
zmm: the draft is unchanged
zmm: the draft is already merged into master
$ grep armor ~/.rhodes/castle-3
armor = thicker
```

- It stays in master as an edit by hand: "do not take this change".

To drop the undo, revert it again:

```console
$ zmm git revert --no-edit HEAD
[master 1cedebb] Reapply "zmm: merge draft"
 Date: Sat Oct 17 01:45:00 2026 +0900
 1 file changed, 1 insertion(+), 1 deletion(-)
$ grep armor ~/.rhodes/castle-3
armor = thickest
```

---

## 5. Using the Makefile

So far, the `Makefile` only `install`s the sources. A `Makefile` can also make dotfiles to fit the environment, or download them ([src4/](src4/)):

```sh
rm -rf ~/.zmm-demo/src
cp -R demo/src4 ~/.zmm-demo/src
```

```makefile
install: build/big-ugly
	install -d $(DESTDIR)/.rhodes
	sed 's/@USER@/$(USER)/' config/castle-3.in >$(DESTDIR)/.rhodes/castle-3
	install -m 644 config/lancet-2.$(shell uname -s) $(DESTDIR)/.rhodes/lancet-2
	install -m 644 build/big-ugly $(DESTDIR)/.rhodes/big-ugly

build/big-ugly:
	mkdir -p build
	curl -fsSL -o build/big-ugly https://raw.githubusercontent.com/kurubushi/zmm/master/demo/acahualla/big-ugly

clean:
	rm -rf build

# ~/.zmm-demo/src
# ├── Makefile
# └── config
#     ├── castle-3.in
#     ├── lancet-2.Darwin
#     └── lancet-2.Linux
```

- `castle-3` is installed with `$USER` filled in.
- `lancet-2` is installed from the file picked for the OS.
- `big-ugly` is downloaded from the zmm repository, and installed.

The download of `big-ugly` is cached. To download it again, do a clean install:

```sh
zmm install --clean
```

- This cache is a feature of `make`. `--clean` just runs `make clean` before the install.

---

## 6. Using Git

zmm manages three branches, each with its own command:

| Branch | Contents | Working directory | Command |
|---|---|---|---|
| `master` | History of `$HOME` | `$HOME` | `zmm git` |
| `draft` | History of the output of the `Makefile` | `~/.zmm-demo/draft/` | `zmm draft-git` |
| `proof` | Workplace for merges | `~/.zmm-demo/proof/` | `zmm proof-git` |

Common commands:

| To do this | Run this |
|---|---|
| See the edits not taken in yet | `zmm git diff` |
| Record a direct edit | `zmm git commit -am <message>` |
| Undo a direct edit | `zmm git restore <path>` |
| See the history of `$HOME` | `zmm git log --first-parent` |
| See the difference from the `Makefile` | `zmm git diff draft` |
| Undo a commit | `zmm git revert <commit>` |
| Undo an install | `zmm git revert -m 1 <merge>` |
| Take the `Makefile` side at a conflict | `zmm proof-git checkout --theirs <path>` |

- `--theirs` is the `Makefile` side, and `--ours` is the `$HOME` side.

---

## Clean up

```sh
rm -rf ~/.zmm-demo ~/.rhodes
rm ~/.local/bin/zmm
unset ZMM_DIR
```

---

## Start with your own dotfiles

Run `zmm init`, and put your own `Makefile` in `~/.zmm/src/`. The rules for the `Makefile` are in the [README](../README.md#rules-for-the-makefile).

To use the same `Makefile` on several machines, make `src/` your own Git repository:

```sh
git clone <your dotfiles repository> ~/.zmm/src
```

- Then don't put `src/` under zmm (Git silently does nothing when a repository outside `add`s files in a repository).
- Leave intermediate output like `build/` out with `.gitignore` ([src4/.gitignore](src4/.gitignore)).
