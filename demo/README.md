# zmm デモ

[English](README.en.md)

zmm の使い方をたどる。

- [3 分で分かる zmm](#3-分で分かる-zmm)
- [1. はじめてのインストール](#1-はじめてのインストール)
- [2. すでにある設定ファイルとぶつかる](#2-すでにある設定ファイルとぶつかる)
- [3. 手作業と Makefile の変更を両方生かす](#3-手作業と-makefile-の変更を両方生かす)
- [4. Makefile ならでは](#4-makefile-ならでは)
- [5. いろいろな場面](#5-いろいろな場面)
- [後片付け](#後片付け)

## 3 分で分かる zmm

dotfiles を生成する `Makefile` を用意しよう:

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

zmm は `Makefile` を元に dotfiles をインストールする:

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

その後、手で直接 `$HOME` の dotfiles を修正することもある:

```diff
--- ~/.bashrc
+++ ~/.bashrc
@@ -1,2 +1,2 @@
-export EDITOR=vim
+export EDITOR=nano # edited manually
 export PAGER=less
```

その一方で `Makefile` でも dotfiles が修正される:

```diff
--- ~/.zmm/src/config/bashrc
+++ ~/.zmm/src/config/bashrc
@@ -1,2 +1,3 @@
 export EDITOR=vim
 export PAGER=less
+alias ll='ls -l' # edited via Makefile
```

zmm なら、手による修正を残したまま、`Makefile` の修正を取り込める:

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

`Makefile` による差分 (draft) と手による差分 (capture) は、Git で別々に管理される:

```console
$ zmm git log --oneline --graph -n 3
*   25199bf zmm: merge draft
|\  
| * b79fca0 zmm: update draft
* | f59c676 zmm: capture
```

問題があれば元に戻せる。`Makefile` の修正だけを取り消すこともできる:

```console
$ zmm git revert --no-edit -m 1 HEAD
[master 861e9c8] Revert "zmm: merge draft"
 Date: Sat Oct 17 01:45:00 2026 +0900
 1 file changed, 1 deletion(-)
$ cat ~/.bashrc
export EDITOR=nano # edited manually
export PAGER=less
```

## zmm のしくみ

zmm は、`Makefile` の出力 (draft) と `$HOME` (master) を Git で merge する:

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

merge は proof branch で行い、できたものまで master と `$HOME` を進める。

---

## 1. はじめてのインストール

デモのやくそく:

- zmm 用 directory として `~/.zmm-demo` を利用 (default: `~/.zmm`)
- デモで扱う dotfiles は `~/.rhodes` のみ (`~/.bashrc` や `~/.tmux.conf` の代わり)

まずは zmm を PATH に置いて、zmm の場所を設定:

```sh
install -m 755 bin/zmm ~/.local/bin/zmm
export ZMM_DIR=~/.zmm-demo
```

zmm を初期化:

```console
$ zmm init
zmm: initialized /home/doctor/.zmm-demo/local.git
```

サンプルの dotfiles src を設置:

```sh
cp -R demo/src1 ~/.zmm-demo/src
```

[src1/Makefile](src1/Makefile) はファイルを `install` するだけの簡単な dotfiles src:

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

インストールする:

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

dotfiles ができた:

```console
$ ls ~/.rhodes
castle-3
lancet-2
```

履歴は Git で見られる:

```console
$ zmm git log --oneline --graph
*   85bf8c3 zmm: merge draft
|\  
| * a658c84 zmm: update draft
|/  
* af82994 zmm: init
```

---

## 2. すでにある設定ファイルとぶつかる

すでに手で書いた dotfiles があるとしよう:

```console
$ cp demo/hand/big-ugly ~/.rhodes/
$ cat ~/.rhodes/big-ugly
[pilot]
autopilot = on
```

これも `Makefile` で管理しよう ([src2/](src2/)):

```diff
 	install -m 644 config/lancet-2 $(DESTDIR)/.rhodes/lancet-2
+	install -m 644 config/big-ugly $(DESTDIR)/.rhodes/big-ugly
```

`Makefile` を入れ替えて、インストールする:

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

- 手で書いた `big-ugly` は、上書きされる前に記録 (capture) された。`$HOME` はまだ変わっていない。

conflict は `~/.zmm-demo/proof/` で解消する:

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

- `HEAD` は `$HOME`、`draft` は `Makefile` の出力だ。

エディタで、両方のよいところを取ろう:

```sh
vi ~/.zmm-demo/proof/.rhodes/big-ugly
```

```ini
[pilot]
autopilot = on

[arms]
body = cannon
```

add して、続ける。`zmm proof-git` は、`~/.zmm-demo/proof/` で打つ Git だ:

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

- 解消した結果は記録されるので、同じ conflict は二度と出ない。`Makefile` が `autopilot = off` のままでも、`$HOME` は `on` のままだ。

---

## 3. 手作業と Makefile の変更を両方生かす

この machine だけ、`castle-3` を手で直す:

```sh
vi ~/.rhodes/castle-3         # mode = 4wd を mode = 2wd に
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

一方、`Makefile` の素材にも節を足す ([src3/config/castle-3](src3/config/castle-3)):

```sh
rm -rf ~/.zmm-demo/src
cp -R demo/src3 ~/.zmm-demo/src
```

`zmm make` なら `$HOME` はそのままに `Makefile` の出力を観察できる。出力は `~/.zmm-demo/draft/` に置かれる。`zmm draft-git` は `git -C ~/.zmm-demo/draft` の alias だ:

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

よければ続ける。両方の変更が残る:

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

- `Makefile` を更新しても、machine ごとの手直しは消えない。

---

## 4. Makefile ならでは

ここまでの `Makefile` は、素材を `install` するだけだった。`Makefile` なら環境に応じて設定ファイルを作成できる ([src4/](src4/)):

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

### 4.1 テンプレートを埋める

`castle-3` は、テンプレートにユーザー名を埋めて作る:

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

- `lancet-2` と `big-ugly` は、作り方が変わっても中身が同じなので、差に出ない。

### 4.2 OS で分ける

`lancet-2` は、共通の部分と OS ごとの部分をつないで作る。Linux なら `xdg-open`、macOS なら `open` になる:

```console
$ ls ~/.zmm-demo/src/config/lancet-2
Darwin
Linux
common
$ tail -n 1 ~/.rhodes/lancet-2
open = xdg-open
```

### 4.3 値を渡して試す

`Makefile` には、環境変数で値を渡せる (`AUTOPILOT ?= off`):

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

- 入れたくなければ、何もしなくていい。次の `zmm make` か `zmm install` が作り直す。

### 4.4 作り直さなくてよいものは作り直さない

`lancet-2` は、`build/` に作ってからインストールしている。`zmm make -v` で、make が打つコマンドを見よう:

```console
$ zmm make -v 2>&1 | grep lancet-2
install -m 644 build/lancet-2 /home/doctor/.zmm-demo/draft/.rhodes/lancet-2
```

- 部品が変わっていないので、つなぎ直していない。

部品に触れると、つなぎ直す:

```console
$ touch ~/.zmm-demo/src/config/lancet-2/common
$ zmm make -v 2>&1 | grep lancet-2
cat config/lancet-2/common config/lancet-2/Linux >build/lancet-2
install -m 644 build/lancet-2 /home/doctor/.zmm-demo/draft/.rhodes/lancet-2
```

- `$(DESTDIR)` はインストールのたびに空になるが、`build/` は残る。重い生成は、この形にしておくと速くなる。

全部作り直したいときは、`--clean` を付ける。`make clean` してから作る:

```console
$ zmm make -v --clean 2>&1 | grep build
rm -rf build
mkdir -p build
cat config/lancet-2/common config/lancet-2/Linux >build/lancet-2
install -m 644 build/lancet-2 /home/doctor/.zmm-demo/draft/.rhodes/lancet-2
```

- 外部から素材をダウンロードする場合も、普段は `zmm install` で cache を効かせ、更新させたいときは `zmm install --clean` で更新できる。

---

## 5. いろいろな場面

前の章の続きから、よくある場面を試そう。

### 5.1 手作業を見る、取り消す

手で直したところは、`zmm git` で見える:

```sh
vi ~/.rhodes/lancet-2         # open = xdg-open を open = firefox に
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

やめるなら、`restore` で戻す:

```console
$ zmm git restore .rhodes/lancet-2
$ zmm git status --short
```

### 5.2 手作業を記録する

もう一度同じ手直しをして、インストールする:

```console
$ zmm install
zmm: made the draft in /home/doctor/.zmm-demo/draft
zmm: the draft is unchanged
zmm: the draft is already merged into master
$ zmm git status --short
 M .rhodes/lancet-2
```

- インストールで記録 (capture) されるのは、`Makefile` の出力が変わったファイルだけだ。この手直しは、記録されずに残る。

記録しておきたければ、`zmm git commit` する:

```console
$ zmm git commit -am 'lancet-2: open reports with firefox'
[master 50ae6b1] lancet-2: open reports with firefox
 1 file changed, 1 insertion(+), 1 deletion(-)
```

### 5.3 `$HOME` の履歴を見る

`--first-parent` で、`$HOME` に起きたことだけが並ぶ:

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

- `zmm: merge draft` が install、`zmm: capture` が install の前の手作業の記録だ。

### 5.4 元に戻す

自分の commit は、`zmm git revert` で取り消せる:

```console
$ zmm git revert --no-edit HEAD
[master 1abdc43] Revert "lancet-2: open reports with firefox"
 Date: Sat Oct 17 01:45:00 2026 +0900
 1 file changed, 1 insertion(+), 1 deletion(-)
$ tail -n 1 ~/.rhodes/lancet-2
open = xdg-open
```

インストールも取り消せる。前の章のインストール (ユーザー名を埋めたもの) を取り消そう:

```console
$ zmm git revert --no-edit -m 1 c521384
[master bb565bb] Revert "zmm: merge draft"
 Date: Sat Oct 17 01:45:00 2026 +0900
 1 file changed, 1 insertion(+), 1 deletion(-)
$ head -n 2 ~/.rhodes/castle-3
[greeting]
assistant = hello.
```

この取り消しは、次のインストールでも残る:

```console
$ zmm install
zmm: made the draft in /home/doctor/.zmm-demo/draft
zmm: the draft is unchanged
zmm: the draft is already merged into master
$ head -n 2 ~/.rhodes/castle-3
[greeting]
assistant = hello.
```

- 「この machine では、この変更を受け取らない」という手作業として、master に残る。

取り消しをやめるなら、もう一度 revert する:

```console
$ zmm git revert --no-edit HEAD
[master 8c5ce60] Reapply "zmm: merge draft"
 Date: Sat Oct 17 01:45:00 2026 +0900
 1 file changed, 1 insertion(+), 1 deletion(-)
$ head -n 2 ~/.rhodes/castle-3
[greeting]
assistant = hello, doctor.
```

### 5.5 conflict を片付ける

`Makefile` の側でも、`castle-3` の `mode` を変える:

```sh
vi ~/.zmm-demo/src/config/castle-3.in     # mode = 4wd を mode = awd に
```

同じ行がぶつかる:

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

- `|||||||` の下は、前回の draft だ。

`--abort` で、なかったことにできる:

```console
$ zmm install --abort
$ tail -n 1 ~/.rhodes/castle-3
mode = 2wd
```

- `Makefile` がそのままなら、次のインストールでまた同じ conflict が出る。

今度は解消しよう。zmm がしているのは、ただの `git merge` だ。Git の道具がそのまま使える。ここでは、`Makefile` の側を丸ごと採る:

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

- `--theirs` は `Makefile` の側、`--ours` は `$HOME` の側だ。パスに `.` を渡せば、全部を揃えられる。
- 見比べたいなら、`~/.zmm-demo/proof/` で好きな道具を使えばいい:

  ```sh
  cd ~/.zmm-demo/proof
  git mergetool
  ```

- 捨てた側も、master の履歴に残っている:

  ```console
  $ zmm git show HEAD^:.rhodes/castle-3 | tail -n 1
  mode = 2wd
  ```

### 5.6 Makefile から外す (uninstall)

`Makefile` から `lancet-2` の行を消して、インストールする:

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

- `Makefile` が作らなくなったファイルは、`$HOME` からも消える。uninstall のコマンドはない。
- 消したファイルも、履歴に残っている (`zmm git show 85bf8c3:.rhodes/lancet-2`)。

### 5.7 permission を守る

`big-ugly` を、自分だけが読めるようにする。permission は `~/.zmm-demo/perm` に記録される:

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

素材の `big-ugly.in` で `body = cannon` を `body = large-caliber-cannon` にして、インストールする:

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

- 書き換わっても `600` のままだ。素の Git では `644` に戻ってしまう。

### 5.8 make が失敗したら

素材のファイルを、うっかり消してしまったとする:

```console
$ mv ~/.zmm-demo/src/config/castle-3.in ~/.zmm-demo/src/config/castle-3.txt
$ zmm install
sed: can't read config/castle-3.in: No such file or directory
make: *** [/home/doctor/.zmm-demo/src/Makefile:14: install] Error 2
$ ls ~/.rhodes
big-ugly
castle-3
```

- make が失敗すると、zmm もそこで止まる。`$HOME` は変わらない。

素材を戻せば、元どおりだ:

```console
$ mv ~/.zmm-demo/src/config/castle-3.txt ~/.zmm-demo/src/config/castle-3.in
$ zmm install
zmm: made the draft in /home/doctor/.zmm-demo/draft
zmm: the draft is unchanged
zmm: the draft is already merged into master
```

### 5.9 Makefile と素材を、自分の repository で管理する

複数の machine で同じ `Makefile` を使うなら、`src/` を自分の Git repository にすると手軽だ。`zmm dir` で zmm の場所が分かる:

```sh
rm -rf "$(zmm dir)/src"
git clone <your dotfiles repository> "$(zmm dir)/src"
```

- そのときは、`src/` を zmm の管理に入れないこと (Git は、repository の中のファイルを外の repository から `add` されても、黙って何もしない)。
- `build/` のような中間の生成物は、`.gitignore` で除外しておく ([src4/.gitignore](src4/.gitignore))。

---

## 後片付け

```sh
rm -rf ~/.zmm-demo ~/.rhodes
rm ~/.local/bin/zmm
unset ZMM_DIR
```

- 本番では `ZMM_DIR` を決めずに `zmm init` して、`~/.zmm/src/` に自分の `Makefile` を置く。`Makefile` の約束は [README](../README.md#rules-for-the-makefile) にある。
