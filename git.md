# Git
リポジトリが `sturdy-engine` の例。リポジトリのURLは以下になる。  
https://github.com/ogusu/sturdy-engine

## ローカルリポジトリの作成（init）

GitHubでリポジトリを作れるが、ローカルか作業を初めて、GitHubにPushするときに作成したい場合は以下の通り。

まず、ローカルにフォルダを作り、gitのリポジトリを作る
~~~
mkdir Repository
cd Repository/
git init
    Initialized empty Git repository in /Users/takanori/Documents/Repository/.git/
~~~

## ローカルリポジトリに変更を反映（add, commit）

適当にファイルを作って、変更点を反映する。インデックスの追加と、コミットをする。  
「作業ツリー」は単純なファイルシステム上のファイルですが「インデックス」はGitが管理するバイナリファイルです。(.gitディレクトリ内にあるindexというファイルです。)	

~~~
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
    Committer: 小楠貴紀 <takanori@mba.local>
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

~~~
git remote add origin https://github.com/ogusu/sturdy-engine.git
git push -u origin master
    Username for 'https://github.com': takanori.ogusu@gmail.com
    Password for 'https://takanori.ogusu@gmail.com@github.com':
    Counting objects: 3, done.
    Writing objects: 100% (3/3), 244 bytes | 0 bytes/s, done.
    Total 3 (delta 0), reused 0 (delta 0)
    To https://github.com/ogusu/sturdy-engine.git
    * [new branch] master -> master
    Branch master set up to track remote branch master from origin.
~~~

最後の修正を見てみる。最後のcommitはHEADと呼ばれ、以下のように表示したり、diffで対象にしたりできる。
~~~
git show HEAD
    commit ce0afbfe8ea9390a62604d4d7eca462ed9a2291f
    Author: 小楠貴紀 <takanori@mba.local>
            ・・・
~~~

## 変更の反映

さらに変更してコミット、pushしてみる。  
修正はコミットだけではなく、インデックスに追加してから、コミットする必要がある。
~~~
git add .
git commit -m "next commit"
~~~

インデックスへの追加は複数回に分けて行える。
コミットはインデックスに追加された修正をまとめて行える。
ある固まった修正を一つのコミットにしたい場合に便利な仕組み。

インデックスへの追加とコミットを同時にすることもできる。
~~~
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
~~~
git diff HEAD
~~~
「指定したコミット」→「作業ツリー」の差分を見る。
~~~
git diff <commit>
~~~
「HEAD」→「インデックス」の差分を見る。
~~~
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
~~~
git diff --name-only
~~~

比較するファイルを限定する。「パス」（ディレクトリ名など）も指定可能。
~~~
git diff <コミット名> <コミット名> ―― <ファイル名>
git diff <コミット名>:<ファイル名> <コミット名>:<ファイル名>
~~~
特定のコミット間を比較する。
~~~
git diff 比較元のコミット 比較先のコミット
~~~

## 直前のコミットの内容を表示
HEADの一個前 から　HEADへの変化を表示。
~~~
git diff HEAD^ HEAD
~~~

HEADのコミット内容を表示
~~~
git diff show
git diff show HEAD
~~~

## リモートブランチとローカルブランチの差分を比較

「特定のコミット同士の比較」の応用編です。リモートブランチからmasterをpull する前に「どのくらい先にすすんじゃったんだろ？」なんて、事前に調べたい時に使います。

git pull は、いうなれば「git fetch & git merge」ですので、いきなり git pull を実行すると、いきなりローカルブランチにマージがかかってしまいます。

これを事前にdiffで差分を確認します。イメージしやすいように、masterブランチの例で書くと・・・、
~~~
// リモートを取ってきて・・（// origin から master ブランチに関する履歴と参照を取得する）
git fetch origin master																							
// [ローカル] → [リモート追跡]の差分を見る
git diff master origin/master
~~~

## コミットの一覧
~~~
git log
~~~
																							