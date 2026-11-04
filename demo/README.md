# zmm デモ

[English](README.en.md)

Try zmm!

- [3 分で分かる zmm](#3-分で分かる-zmm)
- [zmm のしくみ](#zmm-のしくみ)
- [1. はじめてのインストール](#1-はじめてのインストール)
- [2. dotfiles の merge](#2-dotfiles-の-merge)
- [3. dotfiles の conflict](#3-dotfiles-の-conflict)
- [4. その他の zmm の機能](#4-その他の-zmm-の機能)
- [5. Makefile の活用](#5-makefile-の活用)
- [6. Git の活用](#6-git-の活用)
- [後片付け](#後片付け)
- [自分の dotfiles で始める](#自分の-dotfiles-で始める)

## 3 分で分かる zmm

dotfiles を `$(DESTDIR)` に生成する `Makefile` を用意しよう:

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

zmm なら、直接編集を残したまま、`Makefile` の修正を取り込める:

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

`Makefile` による差分 (draft) と直接編集による差分 (capture) は、Git で別々に管理される:

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

[src1/Makefile](src1/Makefile) はファイルを `$(DESTDIR)` に `install` するだけの簡単な dotfiles src:

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

## 2. dotfiles の merge

`castle-3` だけを直接編集する:

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

同時に `Makefile` の素材も修正する ([src2/config/castle-3](src2/config/castle-3)):

```sh
rm -rf ~/.zmm-demo/src
cp -R demo/src2 ~/.zmm-demo/src
```

インストールする。両方の変更が残る:

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

- `Makefile` を更新しても、直接編集した差分は消えない。
- 直接編集が記録 (capture) されるのは、`Makefile` の出力が変わったファイルだけ。

---

## 3. dotfiles の conflict

すでに手で書いた dotfiles があるとしよう:

```console
$ cp demo/hand/big-ugly ~/.rhodes/
$ cat ~/.rhodes/big-ugly
[pilot]
autopilot = on
```

これも `Makefile` で管理しよう ([src3/](src3/)):

```diff
 	install -m 644 config/lancet-2 $(DESTDIR)/.rhodes/lancet-2
+	install -m 644 config/big-ugly $(DESTDIR)/.rhodes/big-ugly
```

`Makefile` を入れ替えて、インストールする:

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

- 手で書いた `big-ugly` は、上書きされる前に記録 (capture) された。`$HOME` はまだ変わっていない。

conflict は `~/.zmm-demo/proof/` で解消する:

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

- `HEAD` は `$HOME`、`draft` は `Makefile` の出力

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

`zmm proof-git` で add して、続ける:

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

- `zmm proof-git` は、`git -C ~/.zmm-demo/proof` の alias。
- 解消した結果は記録されるので、同じ conflict は二度と出ない。`Makefile` が `autopilot = off` のままでも、`$HOME` は `on` のまま。

---

## 4. その他の zmm の機能

前の章の続きから試そう。

### 4.1 インストール前に出力を確認

素材を編集する:

```sh
vi ~/.zmm-demo/src/config/castle-3     # armor = thick を armor = thicker に
```

`zmm make` なら `$HOME` はそのままに `Makefile` の出力を観察できる:

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

- 出力は `~/.zmm-demo/draft/` に置かれる。
- `zmm draft-git` は `git -C ~/.zmm-demo/draft` の alias。

よければ続ける:

```console
$ zmm install --continue
zmm: committed the draft
Auto-merging .rhodes/castle-3
zmm: merged the draft into proof in /home/doctor/.zmm-demo/proof
 .rhodes/castle-3 | 2 +-
 1 file changed, 1 insertion(+), 1 deletion(-)
zmm: merged proof into master and the home directory
```

### 4.2 conflict からの中断

`Makefile` の側でも、`castle-3` の `mode` を変える:

```sh
vi ~/.zmm-demo/src/config/castle-3     # mode = 4wd を mode = awd に
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
```

やめるなら、`--abort` で戻す:

```console
$ zmm install --abort
$ tail -n 1 ~/.rhodes/castle-3
mode = 2wd
```

- `Makefile` がそのままなら、次のインストールでまた同じ conflict が出る。

素材を戻せば、conflict は出ない:

```sh
vi ~/.zmm-demo/src/config/castle-3     # mode = awd を mode = 4wd に
```

```console
$ zmm install
zmm: made the draft in /home/doctor/.zmm-demo/draft
zmm: committed the draft
zmm: merged the draft into proof in /home/doctor/.zmm-demo/proof
zmm: merged proof into master and the home directory
```

### 4.3 アンインストール

`Makefile` から `lancet-2` の行を消して、インストールする:

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

- `Makefile` が作らなくなったファイルは、`$HOME` からも消える。uninstall のコマンドはない。
- 消したファイルも、履歴に残っている (`zmm git show 85bf8c3:.rhodes/lancet-2`)。

### 4.4 permission を保持

`castle-3` を、自分だけが読めるようにする:

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

- permission は `~/.zmm-demo/perm` に記録される。

素材を編集して、インストールする:

```sh
vi ~/.zmm-demo/src/config/castle-3     # armor = thicker を armor = thickest に
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

- 書き換わっても `600` のまま。素の Git では `644` に戻ってしまう。

### 4.5 make の失敗から復帰

素材のファイルを、うっかり消してしまったとする:

```console
$ mv ~/.zmm-demo/src/config/castle-3 ~/.zmm-demo/src/config/castle-3.txt
$ zmm install
install: cannot stat 'config/castle-3': No such file or directory
make: *** [/home/doctor/.zmm-demo/src/Makefile:3: install] Error 1
$ ls ~/.rhodes
big-ugly
castle-3
```

- make が失敗すると、zmm もそこで止まる。`$HOME` は変わらない。

素材を戻せば、元どおり:

```console
$ mv ~/.zmm-demo/src/config/castle-3.txt ~/.zmm-demo/src/config/castle-3
$ zmm install
zmm: made the draft in /home/doctor/.zmm-demo/draft
zmm: the draft is unchanged
zmm: the draft is already merged into master
```

### 4.6 インストールを取り消す

直前のインストールを取り消す:

```console
$ zmm git revert --no-edit -m 1 HEAD
[master 8322c22] Revert "zmm: merge draft"
 Date: Sat Oct 17 01:45:00 2026 +0900
 1 file changed, 1 insertion(+), 1 deletion(-)
$ grep armor ~/.rhodes/castle-3
armor = thicker
```

取り消しは、次のインストールでも残る:

```console
$ zmm install
zmm: made the draft in /home/doctor/.zmm-demo/draft
zmm: the draft is unchanged
zmm: the draft is already merged into master
$ grep armor ~/.rhodes/castle-3
armor = thicker
```

- 「この変更を取り込まない」という手作業として、master に残る。

取り消しをやめるなら、もう一度 revert する:

```console
$ zmm git revert --no-edit HEAD
[master 1cedebb] Reapply "zmm: merge draft"
 Date: Sat Oct 17 01:45:00 2026 +0900
 1 file changed, 1 insertion(+), 1 deletion(-)
$ grep armor ~/.rhodes/castle-3
armor = thickest
```

---

## 5. Makefile の活用

ここまでの `Makefile` は、素材を `install` するだけだった。`Makefile` なら環境に応じて dotfiles を作成したり、ダウンロードしたりもできる ([src4/](src4/)):

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

- `castle-3` は、`$USER` を埋めてインストールされる。
- `lancet-2` は、OS ごとのファイルを選んでインストールされる。
- `big-ugly` は、zmm の repository からダウンロードしてインストールされる。

`big-ugly` のダウンロードは cache が効く。ダウンロードし直すなら clean install する:

```sh
zmm install --clean
```

- この cache は `make` の機能。`--clean` は、`make clean` してからインストールするだけ。

---

## 6. Git の活用

zmm が管理する 3 つの branch は、それぞれ専用のコマンドで操作できる:

| branch | 内容 | working directory | コマンド |
|---|---|---|---|
| `master` | `$HOME` の履歴 | `$HOME` | `zmm git` |
| `draft` | `Makefile` の出力の履歴 | `~/.zmm-demo/draft/` | `zmm draft-git` |
| `proof` | merge の作業場 | `~/.zmm-demo/proof/` | `zmm proof-git` |

よく使うコマンド:

| やりたいこと | コマンド |
|---|---|
| 取り込み前の差分を見る | `zmm git diff` |
| 直接編集を記録する | `zmm git commit -am <message>` |
| 直接編集を取り消す | `zmm git restore <path>` |
| `$HOME` の履歴を見る | `zmm git log --first-parent` |
| `Makefile` との差分を見る | `zmm git diff draft` |
| commit を取り消す | `zmm git revert <commit>` |
| インストールを取り消す | `zmm git revert -m 1 <merge>` |
| conflict で `Makefile` の側を採る | `zmm proof-git checkout --theirs <path>` |

- `--theirs` は `Makefile` の側、`--ours` は `$HOME` の側。

---

## 後片付け

```sh
rm -rf ~/.zmm-demo ~/.rhodes
rm ~/.local/bin/zmm
unset ZMM_DIR
```

---

## 自分の dotfiles で始める

`zmm init` して、`~/.zmm/src/` に自分の `Makefile` を置く。`Makefile` の約束は [README](../README.md#rules-for-the-makefile) にある。

複数の machine で同じ `Makefile` を使うなら、`src/` を自分の Git repository にすると手軽:

```sh
git clone <your dotfiles repository> ~/.zmm/src
```

- そのときは、`src/` を zmm の管理に入れないこと (Git は、repository の中のファイルを外の repository から `add` されても、黙って何もしない)。
- `build/` のような中間の生成物は、`.gitignore` で除外しておく ([src4/.gitignore](src4/.gitignore))。
