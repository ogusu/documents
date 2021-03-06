# Git
リポジトリが `sturdy-engine` の例。リポジトリのURLは以下になる。  
https://github.com/ogusu/sturdy-engine

## ローカルリポジトリの作成（init）

GitHubでリポジトリを作れるが、ローカルか作業を初めて、GitHubにPushするときに作成したい場合は以下の通り。

まず、ローカルにフォルダを作り、gitのリポジトリを作る
~~~git
mkdir Repository
cd Repository/
git init
    Initialized empty Git repository in /Users/xxx/Documents/Repository/.git/
~~~

## comitterを変更する

コミットに登録されるcomitterなどを変更しておく。
~~~git
git config --local user.name takanori.ogusu
git config --local user.email takanori.ogusu@gmail.com
git config --local --list
git commit --amend
~~~

## authorの変更
~~~git
git commit --amend --author="Takanori Ogusu <takanori.ogusu@gmail.com>"
git rebase --continue
git commit --amend
~~~

## ローカルリポジトリに変更を反映（add, commit）

適当にファイルを作って、変更点を反映する。インデックスの追加と、コミットをする。  
「作業ツリー」は単純なファイルシステム上のファイルですが「インデックス」はGitが管理するバイナリファイルです。(.gitディレクトリ内にあるindexというファイルです。)	

~~~git
echo "# sturdy-engine" >> README.md
ls -a
    .  ..  .git  README.md
    
git add README.md

git status
    On branch master
    Initial commit
    Changes to be committed:
    (use "git rm --cached <file>..." to unstage)
    new file:  README.md

git commit -m "my first commit"
    [master (root-commit) 41cee5f] my first commit
    Committer: xxx <xxx@mba.local>
    Your name and email address were configured automatically based
    on your username and hostname. Please check that they are accurate.
    You can suppress this message by setting them explicitly. Run the
    following command and follow the instructions in your editor to edit
    your configuration file:
    git config --global --edit
    After doing this, you may fix the identity used for this commit with:
    git commit --amend --reset-author
    1 file changed, 1 insertion(+)
    create mode 100644 README.md

git status
    On branch master
    nothing to commit, working tree clean
~~~

## リモートリポジトリに変更を反映（push）

次に、リモートリポジトリへPushする。  
pushの前に、gitへリモートリポジトリの位置を登録。以降はエイリアス `origin` でpushできる。  
まだGitHubにリポジトリはないがpushすると作ってくれる。

~~~sh
git remote add origin https://github.com/ogusu/sturdy-engine.git

# ssh公開鍵認証の場合
git remote add origin git@github.com:ogusu/documents.git

git push -u origin master
    Username for 'https://github.com': xxx.ogusu@gmail.com
    Password for 'https://xxx.ogusu@gmail.com@github.com':
    Counting objects: 3, done.
    Writing objects: 100% (3/3), 244 bytes | 0 bytes/s, done.
    Total 3 (delta 0), reused 0 (delta 0)
    To https://github.com/ogusu/sturdy-engine.git
    * [new branch] master -> master
    Branch master set up to track remote branch master from origin.
~~~

最後の修正を見てみる。最後のcommitはHEADと呼ばれ、以下のように表示したり、diffで対象にしたりできる。
~~~git
git show HEAD
    commit ce0afbfe8ea9390a62604d4d7eca462ed9a2291f
    Author: xxx <xxx@mba.local>
            ・・・
~~~

## 変更の反映

さらに変更してコミット、pushしてみる。  
修正はコミットだけではなく、インデックスに追加してから、コミットする必要がある。
~~~git
git add .
git commit -m "next commit"
~~~

インデックスへの追加は複数回に分けて行える。
コミットはインデックスに追加された修正をまとめて行える。
ある固まった修正を一つのコミットにしたい場合に便利な仕組み。

インデックスへの追加とコミットを同時にすることもできる。
~~~git
git commit -a aaa.html
~~~

## 差分の確認（diff）

git diff 比較元 比較先　とすることで比較が可能。

### 作業ツリーの差分

「インデックス」 → 「作業ツリー」 の差分を見る。  
比較元を指定しない場合は、デフォルトで、「インデックス」を比較元とし、 「インデックス」 → 「作業ツリー」 と比較した際の差分を表示する。  
インデックスに対して作業ツリーで行った変更を見ますので、まだgit addしていない変更内容は、このコマンドで簡単にチェックできます。
~~~git
git diff																							
diff --git a/README.md b/README.md																						
index 75382af..77c0149 100644																						
--- a/README.md																						
+++ b/README.md																						
@@ -1,3 +1,4 @@																						
# sturdy-engine																						
# ToDo																						
- Create Branch																						
+ - Show Diff																						
~~~

### コミットとの差分

「最新コミット」(HEAD)→ 「作業ツリー」 の差分を見る。
~~~git
git diff HEAD
~~~
「指定したコミット」→「作業ツリー」の差分を見る。
~~~git
git diff <commit>
~~~
「HEAD」→「インデックス」の差分を見る。
~~~git
git diff --cached
git diff --cached HEAD
~~~

|比較|コマンド|
|:---|:---|
|インデックス → 作業ツリー|git diff	|
|HEAD → 作業ツリーの比較|git diff HEAD	|
|HEAD → インデックス|git diff --cached|

## その他の差分確認

差分が生じたファイルの、ファイル名の一覧を表示します。比較元（引数）は自由に指定できます。
~~~git
git diff --name-only
~~~

比較するファイルを限定する。「パス」（ディレクトリ名など）も指定可能。
~~~git
git diff <コミット名> <コミット名> ―― <ファイル名>
git diff <コミット名>:<ファイル名> <コミット名>:<ファイル名>
~~~
特定のコミット間を比較する。
~~~git
git diff 比較元のコミット 比較先のコミット
~~~

## 直前のコミットの内容を表示
HEADの一個前 から　HEADへの変化を表示。
~~~git
git diff HEAD^ HEAD
~~~

HEADのコミット内容を表示
~~~git
git diff show
git diff show HEAD
~~~

## リモートブランチとローカルブランチの差分を比較

リモート側で修正が入っているとpushが失敗し、pullしろと言われるが、
git pull すると確認もなしにいきなりローカルリポジトリにマージされてしまう。
「git fetch & git merge」の順に行ったほうがいい。

以下のようにすると、リモートリポジトリの現在の状態との差分を見ながらマージができる。
~~~git
# origin から master ブランチの履歴と参照を取得
git fetch origin master																							
# ローカルとリモートの差分を確認
git diff master origin/master
# 問題なさそうならマージ
git merge?
~~~

## コミットの一覧
~~~git
git log
git log --decorate --graph --oneline
git log --all --decorate --graph --oneline
~~~

### 認証
通常はssh公開鍵・認証鍵方式で接続するが、httpsのBASIC認証でも接続可能。
~~~git
git remote set-url origin https://ユーザー名:パスワード@github.com/repogitory-name.git
~~~
ローカルに `.netrc` ファイルを配置しても可能だが推奨されない。
~~~git
machine github.com
login ユーザー名
password パスワード
~~~

### git addの取り消し
resetコマンドで取り消せる。rmでもいいが--cachedを忘れるとファイル自体が消えてしまうのでresetが安全。
~~~
git reset HEAD [ファイルパス]
git reset HEAD
git reset
git rm --cached [ファイル]
~~~

### 変更点の退避
commit, addしてない状態でブランチを切り替えたい時がある。その場合はstashすると良い。saveは省略できる。
~~~
git stash save
git stash
~~~
戻す時。いまどんな変更を退避しているかを確認
~~~
# 2つの変更を保存している状態（2回stashを行った状態）
git stash list
 stash@{0}: WIP on sub: a0d2f1b add fourth line
 stash@{1}: WIP on sub: 1a61919 add second line
 <stash名>: WIP on <stashを行ったブランチ名>: <ハッシュ> <コミットコメント>

# さらに細かい情報が見たい場合。
git stash list -p
git stash show <stash名>
~~~
### stashの復元
~~~
# 復活させたいstash名で取り出す
git stash apply stash@{0}

# 復活したらstashを削除
git stash drop <消したいstash名>

# 復元と削除を同時に
git stash pop stash@{0}

# 復元の取り消し
git stash show <適用したstash名> -p | git apply -R
~~~

### ブランチ
ブランチを作成する時
~~~
# developブランチへ移動
git checkout develop

# ブランチを作成して移動。-bオプションは作成と移動
git checkout -b feature/11


# 現在のブランチはの確認
git branch
~~~

ブランチの削除
~~~
# 消したいブランチとは異なるブランチに移動し、ブランチを消す。
git checkout develop
git checkout -d feature-creaxxx
~~~

ブランチをリモートへpush
~~~
git add .
git commit -am "commit"
git push origin feature/11
~~~

作業ブランチを親ブランチにマージ
~~~
# feature/11 を developにマージ

# まずマージ先に移動。必要ならpullしておくこと。
git checkout develop

# マージしたいブランチを指定して、マージ。
git merge --no-ff feature/11


--no-ffオプション：fast-forwardの関係であっても、必ずマージコミットを作る。機能追加が見やすい。
mergeする時、ローカルのdevelopブランチを最新にすること。
また、feature/11のマージの前にdevelopにすでに修正が入っていると失敗する。feature/11にpullして動作確認しておく。
~~~

図で分かるgit-mergeの--ff, --no-ff, --squashの違い
http://d.hatena.ne.jp/sinsoku/20111025/1319497900

他の人とpushが交互になる場合、以下のようにすればブランチが分かれず1本につながる。
~~~
git fetch origin
git rebase origin/master
~~~

git rebase 失敗した時の対処法
http://qiita.com/shuntaro_tamura/items/c505b76c1021a35ca9ff

### ログ
ログのツリーを見やすくする
~~~
# ~/.gitconfigに以下を追加。git graphで表示。
[alias]
graph = log --graph --date-order -C -M --pretty=format:¥""<%h> %ad [%an] %Cgreen%d%Creset %s¥"" --date=format:¥""%m/%d %R¥""c"
~~~
ログの確認
~~~
# リモートの状況の把握
git log origin/develop

# ローカルのチェックアウトの状況の把握
git remote show origin
~~~

### ブランチ
リモートではすでに削除されたが、ローカルに残っているブランチを削除するには以下の通り。
~~~
# 削除対象の確認
git remote prune origin --dry-run
# 削除実行
git remote prune origin
~~~
