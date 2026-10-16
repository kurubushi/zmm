# zmm demo

[日本語](README.md)

A walk through zmm.

- [zmm in 3 minutes](#zmm-in-3-minutes)
- [1. The first install](#1-the-first-install)
- [2. Clash with existing dotfiles](#2-clash-with-existing-dotfiles)
- [3. Keep both hand edits and Makefile changes](#3-keep-both-hand-edits-and-makefile-changes)
- [4. What a Makefile can do](#4-what-a-makefile-can-do)
- [5. Everyday scenes](#5-everyday-scenes)
- [Clean up](#clean-up)

## zmm in 3 minutes

Prepare a `Makefile` that makes your dotfiles:

```makefile
# zmm replaces $DESTDIR with $HOME.

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

With zmm, you take the change from the `Makefile`, keeping the edit by hand:

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

Git keeps the changes from the `Makefile` (draft) and those by hand (capture) apart:

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

[src1/Makefile](src1/Makefile) is a simple dotfiles src that only `install`s files:

```makefile
# zmm replaces $DESTDIR with $HOME.

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

## 2. Clash with existing dotfiles

Say you already have a dotfile written by hand:

```console
$ cp demo/hand/big-ugly ~/.rhodes/
$ cat ~/.rhodes/big-ugly
[pilot]
autopilot = on
```

Let the `Makefile` manage it too ([src2/](src2/)):

```diff
 	install -m 644 config/lancet-2 $(DESTDIR)/.rhodes/lancet-2
+	install -m 644 config/big-ugly $(DESTDIR)/.rhodes/big-ugly
```

Replace the `Makefile`, and install:

```sh
rm -rf ~/.zmm-demo/src
cp -R demo/src2 ~/.zmm-demo/src
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
||||||| a658c84
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

Add it, and continue. `zmm proof-git` is Git in `~/.zmm-demo/proof/`:

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

- The resolution is recorded, so the same conflict never comes back. `$HOME` keeps `on`, even though the `Makefile` still says `autopilot = off`.

---

## 3. Keep both hand edits and Makefile changes

On this machine only, edit `castle-3` by hand:

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

Meanwhile, add a section to the source of the `Makefile` ([src3/config/castle-3](src3/config/castle-3)):

```sh
rm -rf ~/.zmm-demo/src
cp -R demo/src3 ~/.zmm-demo/src
```

`zmm make` lets you look at the output of the `Makefile`, leaving `$HOME` as it is. The output goes to `~/.zmm-demo/draft/`. `zmm draft-git` is an alias of `git -C ~/.zmm-demo/draft`:

```console
$ zmm make
zmm: made the draft in /home/doctor/.zmm-demo/draft
 .rhodes/castle-3 | 3 +++
 1 file changed, 3 insertions(+)
hint: Check it with "zmm draft-git diff --cached", then run "zmm install --continue".
$ zmm draft-git diff --cached
diff --git a/.rhodes/castle-3 b/.rhodes/castle-3
index 084ea49..df80ce7 100644
--- a/.rhodes/castle-3
+++ b/.rhodes/castle-3
@@ -1,5 +1,8 @@
 [greeting]
 assistant = hello.
 
+[shell]
+armor = thick
+
 [drive]
 mode = 4wd
```

If it looks good, continue. Both changes stay:

```console
$ zmm install --continue
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

- Updating the `Makefile` does not wipe out the edits for each machine.

---

## 4. What a Makefile can do

So far, the `Makefile` only `install`s the sources. A `Makefile` can also make dotfiles to fit the environment ([src4/](src4/)):

```sh
rm -rf ~/.zmm-demo/src
cp -R demo/src4 ~/.zmm-demo/src
```

```makefile
# zmm replaces $DESTDIR with $HOME.

# Linux, Darwin, ...
OS := $(shell uname -s)

# User name
# $ echo $USER # doctor

# From the environment, if any ("AUTOPILOT=on zmm install")
AUTOPILOT ?= off

install: build/lancet-2
	install -d $(DESTDIR)/.rhodes
	sed 's/@USER@/$(USER)/' config/castle-3.in >$(DESTDIR)/.rhodes/castle-3
	install -m 644 build/lancet-2 $(DESTDIR)/.rhodes/lancet-2
	sed 's/@AUTOPILOT@/$(AUTOPILOT)/' config/big-ugly.in >$(DESTDIR)/.rhodes/big-ugly

# Join the common part and the part for the OS
# (make joins them again only when a part changes)
build/lancet-2: config/lancet-2/common config/lancet-2/$(OS)
	mkdir -p build
	cat config/lancet-2/common config/lancet-2/$(OS) >build/lancet-2

# For "zmm install --clean"
clean:
	rm -rf build

# ~/.zmm-demo/src
# ├── Makefile
# └── config
#     ├── big-ugly.in
#     ├── castle-3.in
#     └── lancet-2
#         ├── Darwin
#         ├── Linux
#         └── common
```

### 4.1 Fill in a template

`castle-3` is made from a template, with your user name filled in:

```console
$ zmm make
zmm: made the draft in /home/doctor/.zmm-demo/draft
 .rhodes/castle-3 | 2 +-
 1 file changed, 1 insertion(+), 1 deletion(-)
hint: Check it with "zmm draft-git diff --cached", then run "zmm install --continue".
$ zmm draft-git diff --cached
diff --git a/.rhodes/castle-3 b/.rhodes/castle-3
index df80ce7..bacddce 100644
--- a/.rhodes/castle-3
+++ b/.rhodes/castle-3
@@ -1,5 +1,5 @@
 [greeting]
-assistant = hello.
+assistant = hello, doctor.
 
 [shell]
 armor = thick
$ zmm install --continue
zmm: committed the draft
Auto-merging .rhodes/castle-3
zmm: merged the draft into proof in /home/doctor/.zmm-demo/proof
 .rhodes/castle-3 | 2 +-
 1 file changed, 1 insertion(+), 1 deletion(-)
zmm: merged proof into master and the home directory
```

- `lancet-2` and `big-ugly` are made in a new way, but their contents are the same, so they make no difference.

### 4.2 Split by OS

`lancet-2` is made by joining the common part and the part for the OS. It says `xdg-open` on Linux, and `open` on macOS:

```console
$ ls ~/.zmm-demo/src/config/lancet-2
Darwin
Linux
common
$ tail -n 1 ~/.rhodes/lancet-2
open = xdg-open
```

### 4.3 Pass a value to try

You can pass a value to the `Makefile` in an environment variable (`AUTOPILOT ?= off`):

```console
$ AUTOPILOT=on zmm make
zmm: made the draft in /home/doctor/.zmm-demo/draft
 .rhodes/big-ugly | 2 +-
 1 file changed, 1 insertion(+), 1 deletion(-)
hint: Check it with "zmm draft-git diff --cached", then run "zmm install --continue".
$ zmm draft-git diff --cached
diff --git a/.rhodes/big-ugly b/.rhodes/big-ugly
index e7b8477..8c03da7 100644
--- a/.rhodes/big-ugly
+++ b/.rhodes/big-ugly
@@ -1,5 +1,5 @@
 [pilot]
-autopilot = off
+autopilot = on
 
 [arms]
 body = cannon
```

- If you don't want it, do nothing. The next `zmm make` or `zmm install` makes it again.

### 4.4 Don't make again what needs no remaking

`lancet-2` is made in `build/` first, and then installed. Look at the commands that make runs, with `zmm make -v`:

```console
$ zmm make -v 2>&1 | grep lancet-2
install -m 644 build/lancet-2 /home/doctor/.zmm-demo/draft/.rhodes/lancet-2
```

- The parts have not changed, so make does not join them again.

Touch a part, and make joins them again:

```console
$ touch ~/.zmm-demo/src/config/lancet-2/common
$ zmm make -v 2>&1 | grep lancet-2
cat config/lancet-2/common config/lancet-2/Linux >build/lancet-2
install -m 644 build/lancet-2 /home/doctor/.zmm-demo/draft/.rhodes/lancet-2
```

- `$(DESTDIR)` is emptied at every install, but `build/` stays. Heavy work gets faster in this form.

To make everything again, add `--clean`. It makes after `make clean`:

```console
$ zmm make -v --clean 2>&1 | grep build
rm -rf build
mkdir -p build
cat config/lancet-2/common config/lancet-2/Linux >build/lancet-2
install -m 644 build/lancet-2 /home/doctor/.zmm-demo/draft/.rhodes/lancet-2
```

- The same goes for sources downloaded from outside: `zmm install` uses the cache as usual, and `zmm install --clean` gets them fresh when you want.

---

## 5. Everyday scenes

Go on from the previous chapter, and try common scenes.

### 5.1 See and undo edits by hand

`zmm git` shows what you edited by hand:

```sh
vi ~/.rhodes/lancet-2         # open = xdg-open to open = firefox
```

```console
$ zmm git status --short
 M .rhodes/lancet-2
$ zmm git diff
diff --git a/.rhodes/lancet-2 b/.rhodes/lancet-2
index da91ef7..b757f36 100644
--- a/.rhodes/lancet-2
+++ b/.rhodes/lancet-2
@@ -2,4 +2,4 @@
 assistant = hi.
 
 [report]
-open = xdg-open
+open = firefox
```

To drop it, `restore` it:

```console
$ zmm git restore .rhodes/lancet-2
$ zmm git status --short
```

### 5.2 Record edits by hand

Make the same edit again, and install:

```console
$ zmm install
zmm: made the draft in /home/doctor/.zmm-demo/draft
zmm: the draft is unchanged
zmm: the draft is already merged into master
$ zmm git status --short
 M .rhodes/lancet-2
```

- An install records (captures) only the files whose output from the `Makefile` changed. This edit stays unrecorded.

To record it, `zmm git commit` it:

```console
$ zmm git commit -am 'lancet-2: open reports with firefox'
[master 50ae6b1] lancet-2: open reports with firefox
 1 file changed, 1 insertion(+), 1 deletion(-)
```

### 5.3 See the history of `$HOME`

`--first-parent` lists only what happened to `$HOME`:

```console
$ zmm git log --oneline --first-parent
50ae6b1 lancet-2: open reports with firefox
c521384 zmm: merge draft
090f59e zmm: merge draft
43b8874 zmm: capture
06ef7b4 zmm: merge draft
c0073c7 zmm: capture
85bf8c3 zmm: merge draft
af82994 zmm: init
```

- `zmm: merge draft` is an install, and `zmm: capture` is the record of edits by hand before an install.

### 5.4 Go back

`zmm git revert` undoes your own commit:

```console
$ zmm git revert --no-edit HEAD
[master 1abdc43] Revert "lancet-2: open reports with firefox"
 Date: Sat Oct 17 01:45:00 2026 +0900
 1 file changed, 1 insertion(+), 1 deletion(-)
$ tail -n 1 ~/.rhodes/lancet-2
open = xdg-open
```

It undoes an install too. Undo the install in the previous chapter (the one that filled in the user name):

```console
$ zmm git revert --no-edit -m 1 c521384
[master bb565bb] Revert "zmm: merge draft"
 Date: Sat Oct 17 01:45:00 2026 +0900
 1 file changed, 1 insertion(+), 1 deletion(-)
$ head -n 2 ~/.rhodes/castle-3
[greeting]
assistant = hello.
```

The undo stays through the next install:

```console
$ zmm install
zmm: made the draft in /home/doctor/.zmm-demo/draft
zmm: the draft is unchanged
zmm: the draft is already merged into master
$ head -n 2 ~/.rhodes/castle-3
[greeting]
assistant = hello.
```

- It stays in master as an edit by hand: "this machine does not take this change".

To drop the undo, revert again:

```console
$ zmm git revert --no-edit HEAD
[master 8c5ce60] Reapply "zmm: merge draft"
 Date: Sat Oct 17 01:45:00 2026 +0900
 1 file changed, 1 insertion(+), 1 deletion(-)
$ head -n 2 ~/.rhodes/castle-3
[greeting]
assistant = hello, doctor.
```

### 5.5 Settle a conflict

Change the `mode` of `castle-3` on the `Makefile` side too:

```sh
vi ~/.zmm-demo/src/config/castle-3.in     # mode = 4wd to mode = awd
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
$ tail -n 8 ~/.zmm-demo/proof/.rhodes/castle-3
[drive]
<<<<<<< HEAD
mode = 2wd
||||||| 56019c2
mode = 4wd
=======
mode = awd
>>>>>>> draft
```

- Under `|||||||` is the previous draft.

`--abort` makes it as if nothing happened:

```console
$ zmm install --abort
$ tail -n 1 ~/.rhodes/castle-3
mode = 2wd
```

- If the `Makefile` stays the same, the next install brings the same conflict again.

This time, resolve it. All zmm does is a plain `git merge`, so the tools of Git work as they are. Here, take the `Makefile` side as a whole:

```console
$ zmm install
zmm: made the draft in /home/doctor/.zmm-demo/draft
zmm: the draft is unchanged
Auto-merging .rhodes/castle-3
CONFLICT (content): Merge conflict in .rhodes/castle-3
Automatic merge failed; fix conflicts and then commit the result.
hint: Resolve the conflicts in /home/doctor/.zmm-demo/proof, and run "zmm install --continue".
hint: To throw it away, run "zmm install --abort".
$ zmm proof-git checkout --theirs .rhodes/castle-3
Updated 1 path from the index
$ zmm proof-git add .rhodes/castle-3
$ zmm install --continue
zmm: merged proof into master and the home directory
$ tail -n 1 ~/.rhodes/castle-3
mode = awd
```

- `--theirs` is the `Makefile` side, and `--ours` is the `$HOME` side. Pass `.` as the path to settle them all.
- To compare them, use any tool you like in `~/.zmm-demo/proof/`:

  ```sh
  cd ~/.zmm-demo/proof
  git mergetool
  ```

- The side you threw away stays in the history of master:

  ```console
  $ zmm git show HEAD^:.rhodes/castle-3 | tail -n 1
  mode = 2wd
  ```

### 5.6 Remove from the Makefile (uninstall)

Delete the lines of `lancet-2` from the `Makefile`, and install:

```diff
-install: build/lancet-2
+install:
 	install -d $(DESTDIR)/.rhodes
 	sed 's/@USER@/$(USER)/' config/castle-3.in >$(DESTDIR)/.rhodes/castle-3
-	install -m 644 build/lancet-2 $(DESTDIR)/.rhodes/lancet-2
 	sed 's/@AUTOPILOT@/$(AUTOPILOT)/' config/big-ugly.in >$(DESTDIR)/.rhodes/big-ugly
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

### 5.7 Keep the permissions

Make `big-ugly` readable only by you. The permissions are recorded in `~/.zmm-demo/perm`:

```console
$ chmod 600 ~/.rhodes/big-ugly
$ zmm git diff
diff --git a/.zmm-demo/perm b/.zmm-demo/perm
index a6ce86a..c419e6b 100644
--- a/.zmm-demo/perm
+++ b/.zmm-demo/perm
@@ -1,3 +1,3 @@
 0755 .rhodes/
-0644 .rhodes/big-ugly
+0600 .rhodes/big-ugly
 0644 .rhodes/castle-3
$ zmm git commit -qam 'big-ugly: keep it private'
```

Change `body = cannon` to `body = large-caliber-cannon` in the source `big-ugly.in`, and install:

```console
$ zmm install
zmm: made the draft in /home/doctor/.zmm-demo/draft
zmm: committed the draft
Auto-merging .rhodes/big-ugly
zmm: merged the draft into proof in /home/doctor/.zmm-demo/proof
 .rhodes/big-ugly | 2 +-
 1 file changed, 1 insertion(+), 1 deletion(-)
zmm: merged proof into master and the home directory
$ tail -n 1 ~/.rhodes/big-ugly
body = large-caliber-cannon
$ ls -l ~/.rhodes/big-ugly | cut -c 1-10
-rw-------
```

- It stays `600` after being rewritten. Plain Git would put it back to `644`.

### 5.8 When make fails

Say you deleted a source file by mistake:

```console
$ mv ~/.zmm-demo/src/config/castle-3.in ~/.zmm-demo/src/config/castle-3.txt
$ zmm install
sed: can't read config/castle-3.in: No such file or directory
make: *** [/home/doctor/.zmm-demo/src/Makefile:14: install] Error 2
$ ls ~/.rhodes
big-ugly
castle-3
```

- When make fails, zmm stops there. `$HOME` does not change.

Put the source back, and all is as before:

```console
$ mv ~/.zmm-demo/src/config/castle-3.txt ~/.zmm-demo/src/config/castle-3.in
$ zmm install
zmm: made the draft in /home/doctor/.zmm-demo/draft
zmm: the draft is unchanged
zmm: the draft is already merged into master
```

### 5.9 Keep the Makefile and the sources in your own repository

To use the same `Makefile` on several machines, make `src/` your own Git repository. `zmm dir` tells where zmm is:

```sh
rm -rf "$(zmm dir)/src"
git clone <your dotfiles repository> "$(zmm dir)/src"
```

- Then don't put `src/` under zmm (Git silently does nothing when a repository outside `add`s files in a repository).
- Leave intermediate output like `build/` out with `.gitignore` ([src4/.gitignore](src4/.gitignore)).

---

## Clean up

```sh
rm -rf ~/.zmm-demo ~/.rhodes
rm ~/.local/bin/zmm
unset ZMM_DIR
```

- For real use, run `zmm init` without setting `ZMM_DIR`, and put your own `Makefile` in `~/.zmm/src/`. The rules for the `Makefile` are in the [README](../README.md#rules-for-the-makefile).
