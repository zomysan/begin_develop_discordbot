　つづいて、Botをより実用的な形にしていきましょう。繰り返しになりますが、実装するコードの全体像は、下記リポジトリにあります。こちらも参考にしてください。
　https://github.com/cod-sushi/discord_dev_sample.git

## トークンを隠そう
### トークンは漏洩厳禁なので隠す
　[@lst:first_discord_bot]では、`bot.run("ThisIsDummyToken00000000.XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX")`のようにトークンをソースコード中に直接記述していました。すでに述べたとおり、これはよくありません。
　もちろん、Botを動かすだけならまったく問題ありません。問題は、Botを開発していく過程で発生します。NGパターンである理由を一言でいうと、「トークンの流出につながるため」です。
　ソースコード中に直接トークンを記述していると、Gitなどのバージョン管理ツールにトークンが含まれてしまう可能性があります。後々修正したとしても、特別な操作をしない限り、履歴には残ります。また、Botがうまく動かなかった際、解決するためにインターネット上の質問サイトなどにソースコードを転記することがあるかもしれません。そのようなときに、トークンを消し忘れたまま質問を投稿してしまうという事故はよくあることです。

### configモジュールにトークンを隠す
　では、トークンをソースコードに記述しないためには、どのようにするのがよいのでしょうか？いろいろな方法がありますが、今回は**configモジュール**を作り、そこにトークンを記述する方法をとることにしましょう。
　まずは実際にconfigモジュールを作成してみましょう。Visual Studio Codeで、[@sec:easy]にて作成したフォルダー内に「config.py」という名前のファイルを作成します。ファイルを作成すると、フォルダー構成は[@lst:config_folder_structure]のようになります。

```{#lst:config_folder_structure .shell caption="フォルダー構成"}
my_bot
│  config.py
│  launcher.py
│  requirements.txt
└─.venv
```

　config.pyに、[@lst:config_py]の内容を入力してください。"ThisIsDummy..."の内容は、あなたのトークンに置き換えてください。

```{#lst:config_py .python caption="config.py 内容"}
TOKEN = "ThisIsDummyToken00000000.XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX"
```

### configモジュールを使う
　launcher.pyからconfigモジュールを使い、直書きのトークンをソースコードから除去しましょう。まずは書き換えたコードの全景を確認しましょう。[@lst:use_config_py_entire]をごらんください。

```{#lst:use_config_py_entire .python caption="configを使ってトークンを隠す"}
from discord.ext import commands
import config

bot = commands.Bot(command_prefix="!")


@bot.event
async def on_ready():
    print("on_ready")


@bot.event
async def on_message(message):
    if message.author == bot.user:
        # Botからのメッセージには反応しない
        # この判定をしないと無限ループが起きる
        return

    if "Bot" in message.content:
        await message.channel.send("は～い、Botで～す")

bot.run(config.TOKEN)
```

　launcher.pyの2行目に`config`をインポートする処理を追記しましあｔ。

```{#lst:use_config_py_1 .python caption="configモジュールのインポート"}
from discord.ext import commands
import config
...
```

　つづいて、`bot.run`を行っている行の内容を、[@lst:use_config_py_2]のとおり変更しています。

```{#lst:use_config_py_2 .python caption="トークンからconfig.TOKENへの書き換え"}
bot.run("ThisIsDummyToken00000000.XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX")
    ↓
bot.run(config.TOKEN)
```

　これでlauncher.py自体にトークンは含まれなくなりました。ソースコード全文を転記しても問題ありません。

## Gitリポジトリを作ろう
### Gitってなに？
　Gitはバージョン管理ツールです。ある時点でのファイル群の状態を、日時とコメント付きでまるまる保存し、履歴として保持してくれます。その時点でのソースコードが入っているフォルダーを、まるまるコピーして保存してくれるようなイメージです。ひとりで開発するならあまりメリットはありませんが、誰がその状態で保存したかということも記録できます。筆者も、本書の記事データ管理をはじめ、さまざまな用途にGitを使っています。Git自体の詳細な説明は他の書籍等に譲りますが、ここでは最低限の操作について説明します。

### Gitリポジトリの作成
　リポジトリとは、履歴を格納するためのデータの入れ物です。百聞は一見に如かず。まずはGitリポジトリを作ってみましょう。Visual Studio Codeで「Terminal」→「New Terminal」を選択し、ターミナルを開きます。launcher.pyやconfig.pyが存在するフォルダーで、`git init`と実行してみてください。

```{#lst:first_git_init .shell caption="はじめてのGitリポジトリ作成"}
$ git init
Initialized empty Git repository in ~/my_bot/.git/
```

　[@lst:first_git_init]のように、「Initialized empty Git repository～」と表示されたら成功です。このメッセージは、「空っぽのGitリポジトリを作成できましたよ」という内容です。まだ何も履歴はないけれど、履歴を入れる準備が整ったということです。

#### ローカルリポジトリとリモートリポジトリ
　実はGitのリポジトリには、**ローカルリポジトリ**と**リモートリポジトリ**があります。ローカルリポジトリは、直接そのリポジトリに対して履歴を書き込めるリポジトリで、リモートリポジトリは、ローカルリポジトリと同期することで履歴を書き込めるリポジトリです。この章で作成するのはローカルリポジトリです。
　リモートリポジトリを作っておくと、他者との共同作業が可能になるだけでなく、PCが壊れたときのバックアップとしても機能します。リモートリポジトリの作成については本書では詳細は掲載しませんが、GitHubの利用を推奨します。無料のプランでもプライベートリポジトリ（自分以外見ることのできないリポジトリ）を作成できます。GitHubにリモートリポジトリを作成し、ローカルリポジトリの内容を同期する方法はWeb上に多くのチュートリアル記事がありますので、実践してみてください。

### Gitに入れたくないファイルがあるときは？
#### Gitで管理するもの、しないもの
　Gitは特定のフォルダー内のファイルについて変更履歴を保存するツールです。一度Gitに履歴を追加すると、あとあと削除や上書きをしたとしても、履歴を遡れば過去の状態を見ることができてしまいます。Gitを使うからには、いずれ他の人がそのリポジトリを利用する可能性があります。リポジトリをGitHubで公開するかもしれません。そのとき、履歴のどこかに情報があると漏洩につながります。そのため、パスワードやトークンなど他人に知られてはいけない情報の入ったファイルは、Gitの管理対象から外す必要があります。今回でいうと、トークンが含まれるファイルであるconfig.pyはGitの管理対象外とする必要があります。

#### ファイルをGitの管理対象外にする
　Gitには、**.gitignore**というファイルに記載したものは**管理対象外とする**という仕組みがあります。つまり、このファイルを使えば、履歴に残すもの、残さないものを自由に選択できるというわけです。実際に体験してみましょう。
　まず、現状を確認しましょう。`git status -s`を実行してみてください。これは、現在のフォルダーにおけるGitの状態を簡略化された形式で出力するというオプションです。おそらく、[@lst:git_ignore_1]のように出力されるでしょう。`

```{#lst:git_ignore_1 .shell caption="git status -s 実行例"}
$ git status -s
?? .venv/
?? .vscode/
?? config.py
?? launcher.py
?? requirements.txt
```

　各行の頭の`??`は、Gitが追尾していないファイルであることを示しています。つまり、`.venv/`、`launcher.py`、`config.py`、`requirements.txt`の4つのファイルが追尾できていない状態であることを示しています。ただし、先述したように`config.py`はGitの管理対象外とする必要があります。また、Python仮想環境を管理するための`.venv`フォルダーや、動作に必要な情報を保存するためVisual Studio Codeによって自動生成された`.vscode`フォルダーも、ソースコードの管理をするためのリポジトリにてバージョンを管理するにはふさわしくありません。
　現状を確認したところで、.gitignoreファイルを作成します。Visual Studio Codeで、.gitignoreという名称のファイルを作成し、[@lst:git_ignore_2]の内容を記述し、保存しましょう。

```{#lst:git_ignore_2 .shell caption=".gitignore 記入例"}
.venv/
.vscode/
config.py
```

　この時点でのフォルダー構成は[@lst:gitignore_folder_structure]のようになります。リスト中に`.vscode`というフォルダーがありますが、本フォルダーはVisual Studio Codenによって自動生成されたフォルダーであり、開発には直接関連しないため、今後フォルダー構成からは省略します。

```{#lst:gitignore_folder_structure .shell caption="フォルダー構成"}
my_bot
│  .gitignore  # 追加した.gitignore
│  config.py
│  launcher.py
│  requirements.txt
├─.venv
└─.vscode
```

　.gitignoreが用意できたら、再度`git status -s`を実行してみます。

```{#lst:git_ignore_3 .shell caption=".gitignore作成後の git status -s 実行例"}
$ git status -s
?? .gitignore
?? launcher.py
?? requirements.txt
```

　うまくいけば、[@lst:git_ignore_3]のように、.gitignoreファイルが増え、.venv/およびconfig.pyは消えているはずです。

### Gitに変更をコミットする
#### はじめてのステージング
　それではいよいよ、はじめてのコミットをしてみましょう。Gitにおけるコミット作業は「ステージング」と「コミット」の2段階に分けて行います。まずは1段階目、ステージングを行いましょう。まずはlauncher.pyをステージングてみましょう。[@lst:first_git_add]のように実行します。

```{#lst:first_git_add .shell caption="はじめてのgit add"}
$ git add launcher.py
```

　状態を確認してみます。`git status -s`を実行してください。

```{#lst:first_git_add_check .shell caption="はじめてのgit add 結果"}
$ git status -s
A  launcher.py
?? .gitignore
?? requirements.txt
```

　[@lst:first_git_add_check]のように、launcher.pyの状態が「A」になっていたら成功です。「A」は、変更がすべてステージングされていることを示します。残りの2つについてもステージングしましょう。ここで便利な方法を紹介します。[@lst:git_add_current]をごらんください。

```{#lst:git_add_current .shell caption="現フォルダ以下をgit add"}
$ git add .
```

　これは、「現在のフォルダー以下の変更をすべてステージングする」という意味のコマンドです。これを実行してから、再度`git status -s`を実行してみましょう。

```{#lst:git_add_current_check .shell caption="git add . 結果"}
$ git status -s
A  .gitignore
A  launcher.py
A  requirements.txt
```

　[@lst:git_add_current_check]のように、すべてのファイルの状態が「A」になっているはずです。ちなみに、ステージングの状態は`git reset`とすることでリセット可能です。一度リセットして、さまざまなステージング方法を試してみるのもよいでしょう。
　さて、これでステージングが完了しました。次はいよいよコミットです。


#### はじめてのコミット
　さきほど、コミット作業に必要な1段階目、「ステージング」が完了しました。次はいよいよ、コミットを行います。[@lst:first_git_commit]の1行目のコマンドを実行してください。

```{#lst:first_git_commit .shell caption="はじめてのgit commit"}
$ git commit -m "first commit"
[master (root-commit) 79adbfa] first commit
 3 files changed, 82 insertions(+)
 create mode 100644 .gitignore
 create mode 100644 launcher.py
 create mode 100644 requirements.txt
```

　Gitの利用がはじめてである場合、途中、名前とメールアドレスを入力するよう求められることがあります。その場合、公開してもかまわないものを入力するようにしましょう。コミットがうまくいけば、[@lst:first_git_commit]のように、数行のログが出るはずです。この状態で、状態を確認してみましょう。

```{#lst:none_git_status .shell caption="コミット直後のgit status"}
$ git status -s
　
```

　いかがでしょうか？[@lst:none_git_status]のように、何も出力されなかったと思います。これでOKです。`$ git add .`によりすべての変更をステージングし、ステージングされた変更はコミットしたので、「コミットすべき変更はなにもありません」という状態になっています。
　うまくコミットできているのかどうか、履歴を見てみましょう。[@lst:first_git_log]の1行目のコマンドを実行してください。

```{#lst:first_git_log .shell caption="はじめてのgit log"}
$ git log
commit 79adbfa9a39f9fbbe29d8528889540b0bfb61d87 (HEAD -> master)
Author: cod <cod@sample.com>
Date:   Wed Jan 1 00:00:00 2020 +0900

    first commit
```

　さきほど作ったコミットが表示されていますね。これではじめてのコミットは完了です。

### コミットの追加
　ここでは、コミットしたあと、さらに新しいコミットを追加する方法をお伝えします。今の時点ではさらにコミットすべき変更はありませんので、この部分は一度飛ばし、Botの実装をすすめたあとに戻ってきてください。
　まずは状態を確認してみましょう。

```{#lst:git_commit2_status .shell caption="編集後にgit status -sしてみる"}
$ git status -s
 M launcher.py
```

　赤色で`M`という表示がされています。これは、追尾中のファイルが変更されており、ステージングされていない状態であることを表しています。ここで、`git add launcher.py`し、再度状態を確認します。

```{#lst:git_commit2_status2 .shell caption="編集後にgit status -sしてみる"}
$ git add launcher.py
$ git status -s
M  launcher.py
```

　`M`の表示が緑色になりました。位置もすこし左に寄っています。これは、追尾中のファイルが変更済みで、ステージングも済んだ状態であるということを表しています。この状態で[@lst:git_commit2_commit]の内容を実行し、コミットを行いましょう。

```{#lst:git_commit2_commit .shell caption="変更をコミットする"}
$ git commit -m "（ここに変更内容のコメントを入力する）"
```

## Botにコマンドを追加しよう - Commands
### コマンドはBotへの命令
　**Command**は、ユーザーからの命令でBotが動作するためのしくみです。より詳細に述べると、Botに固有の**プレフィックス**と、**コマンド文字列**の組み合わせでBotを操作する仕組みのことを指します。たとえば、有名な音楽再生Bot「MusicBot」は、「!summon」でBotの起動を行い、「!shutdown」でBotを終了します。この場合、プレフィックスは「!」、コマンド文字列は「summon」「shutdown」です。著者の運営する読み上げBot「shovel」では、「!sh shovel」で読み上げを開始し、「!sh end」で読み上げを終了します。「!sh 」がプレフィックス、コマンド文字列は「shovel」「end」です。
　実は「おたすけBot」のプレフィックスはすでに`!`として設定済みです。[@lst:first_discord_bot]の5行目、`command_prefix="!"`で指定しています。プレフィックスを変更したい場合、この部分を変更してください。

### はじめてのコマンド実装
　それでは、実際にBotへコマンドを実装してみましょう。BotのいるGuildで「!hello」のように発言すると、返事を返してくれる「あいさつコマンド」を作ってみます。`launcher.py`に、[@lst:hello_command]のように追記します。

```{#lst:hello_command .python caption="あいさつコマンドを実装しよう"}
from discord.ext import commands
import config

bot = commands.Bot(command_prefix="!")


@bot.event
async def on_ready():
    print("on_ready")


@bot.event
async def on_message(message):
    if message.author == bot.user:
        # Botからのメッセージには反応しない
        # この判定をしないと無限ループが起きる
        return

    if "Bot" in message.content:
        await message.channel.send("は～い、Botで～す")

    # 下記の処理を追加
    await bot.process_commands(message)


@bot.command()
async def hello(ctx):
    # あいさつする既存の処理
    await ctx.send(f"こんにちは、{ctx.author.name}さん。")
    
bot.run(config.TOKEN)
```

　これらの処理について詳細に説明します。

```{#lst:hello_command_process_commands .python caption="on_messageイベントリスナー定義時に必須の記述"}
@bot.event
async def on_message(message):
    if message.author == bot.user:
        # Botからのメッセージには反応しない
        # この判定をしないと無限ループが起きる
        return

    if "Bot" in message.content:
        await message.channel.send("は～い、Botで～す")
        
    # 下記の処理を追加
    await bot.process_commands(message)
```

　`@bot.command()`によるコマンド定義と、`on_message`イベントリスナーの定義を同時に使う際にはかならず`bot.process_commands`の呼び出しが必要です。discord.pyでは、`on_message`イベントのデフォルトの処理として`process_commands`を呼び出しています。そのため、`on_message`の処理を上書きしてしまうと、コマンドのための処理が呼び出されなくなります。「コマンドを正しく定義しているはずなのに、Botがまったく反応しない」という場合はこれが原因となっていることが多いです。

```{#lst:hello_command_bot_command .python caption="コマンドの定義"}
#　コマンドの定義
@bot.command()
```

　`@<変数名>.command()`というデコレーターを関数宣言の前に付けると、続く関数名のコマンドをBotに実装します。今回だと、`hello`という名前の関数宣言が後に続くので、`hello`コマンドが実装されます。

```{#lst:hello_command_define_func .python caption="コマンド処理内容の定義"}
async def hello(ctx):
    await ctx.send(f"こんにちは、{ctx.author.name}さん。")
```

　`hello`関数の引数である`ctx`は、コマンドを定義するうえで必須の引数です。ctxは**Context（コンテキスト）**というクラスで、コマンドの実行に関する情報を保持します。たとえば、コマンドが実行されたテキストチャンネルやGuild、コマンドを実行したメンバーの情報などが詰まっています。ここでは、コマンドを実行した人の名前を`ctx.author.name`として取り出し、呼びかけています。また、`Context`クラスの`send`関数を使えば、コマンドが実行された場所にメッセージを投稿できます。[@lst:hello_command_define_func]では、`Context`クラスの機能を活用してあいさつ機能の処理を実装しています。
　実装が完了したらターミナルを開き、仮想環境を有効化して`$ python launcher.py`を実行し、Botを起動しましょう。つづいて、`!hello`コマンドをDiscordで実行してみましょう。[@fig:sc_hello_command]のようにあいさつが返ってきたら成功です。文言を変更したり、コマンドを実行した人の名前以外の情報を出力したりしてみましょう。

![あいさつコマンド 実行例](resource/image/sc_hello_command.png){#fig:sc_hello_command}

　ここまで実践できたら、変更をGitにコミットしておきましょう。手順は「コミットの追加」を参照してください。

## Botと対話的にやりとりしよう
### 対話的なやりとりの動作例
　ここでは、まずBotの動作サンプルを見て頂きましょう。[@fig:sc_dialog]をごらんください。

![対話的なやりとり](resource/image/sc_dialog.png){#fig:sc_dialog}

　はじめは「あいさつコマンド」と同じですが、途中「ご気分はいかがでしょうか？」とたずね、返答を待ちます。返答があったら、その内容を使って会話をすすめます。このようなやりとりを、**対話的なやりとり**と呼ぶことにしましょう。

### 対話的なやりとりを実装する
　それでは、対話的なやりとりをコーディングしてみましょう。[@lst:first_dialog]をごらんください。

```{#lst:first_dialog .python caption="対話的なやりとり コード"}
@bot.command()
async def hello(ctx):
    # 待機するメッセージのチェック関数
    def check_message_author(msg):
        return msg.author is ctx.author
    # あいさつする既存の処理
    await ctx.send(f"こんにちは、{ctx.author.name}さん。")
    await ctx.send("ご気分はいかがでしょうか？")
    # チェック関数に合格するようなメッセージを待つ
    msg = await bot.wait_for('message', check=check_message_author)
    # 受け取ったメッセージの内容を使って返信
    await ctx.send(f"「{msg.content}」という気分なんですね。")
```

　[@lst:hello_command]で定義した`hello`コマンドに処理を追加しています。追加した記述について、詳細に見ていきましょう。`check_message_author`関数の定義が冒頭にありますが、説明の都合上この関数についての解説は後回しにします。ご了承ください。

```{#lst:first_dialog_wait_for .python caption="wait_forでイベントを待つ"}
    await ctx.send("ご気分はいかがでしょうか？")
    # チェック関数に合格するようなメッセージを待つ
    msg = await bot.wait_for('message', check=check_message_author)
```

　[@lst:first_dialog_wait_for]の1行目では、コマンドのコンテキストに「ご気分はいかがでしょうか？」というメッセージを送信しています。重要なのは3行目です。`bot`の`wait_for`という関数に、1つ目の引数として`'message'`が指定されています。このように記述することで、**messageイベントの発生まで待機する**という処理になります。メッセージを受信するまで処理を止めて、受信したらそのメッセージを返します。
　また、`check`というパラメーターに`check_message_author`を指定しています。これは、`check_message_author`関数によるチェックに適うメッセージを受信するまで待機するという意味の記述です。`check_message_author`の内容を見てみましょう。

```{#lst:first_dialog_check_func .python caption="チェック関数の定義"}
    # 待機するメッセージのチェック関数
    def check_message_author(msg):
        return msg.author is ctx.author
```

　wait_forで指定するチェック関数は、指定したイベント（今回であれば`message`）のイベントリスナーと同一の引数を受け取ります。`message`イベントのイベントリスナー`on_message`は、`Message`型の引数を1つとるので、check関数も同じように`Message`型の引数を1つとる関数として定義します。関数内では、受信したメッセージ`msg`の送信者（`msg.author`）が、コマンドを実行した人（`ctx.author`）と同じであるかをチェックしています。テスト用Guildのように、ユーザーとBotが1:1で会話する場合このようなチェック関数は必要ありません。しかし不特定多数のメンバーがいるGuildでBotを運用する場合、やりとりしている最中のユーザー以外のメンバーがメッセージを送信するかもしれません。あくまで受け取りたいのはコマンドを実行したユーザーの返答なので、このように制限して待機を続けます。

```{#lst:first_dialog_use_reply .python caption="受け取ったメッセージの内容を利用して返信"}
    # 受け取ったメッセージの内容を使って返信
    await ctx.send(f"「{msg.content}」という気分なんですね。")
```

　wait_forが返したメッセージを`msg`という変数に格納していますので、`msg.content`で内容を取り出し、Botからの返答に利用しています。以上で実装は完了です。ターミナルを開き、仮想環境を有効化して`$ python launcher.py`を実行し、Botを起動します。テスト用Guildのテキストチャンネルに`!hello`と投稿してみましょう。

### 対話の打ち切り - タイムアウト処理を入れる
　さて、ここまでで実装した機能にはひとつ問題点があります。「ご気分はいかがでしょうか？」に対していつまでも返答がなかったとき、ずっとそのままBotが待機してしまうのです。待機とはいえ、コルーチン関数の中での待機ですのでBotの応答が止まるわけではありませんが、あまりよい状態ではありません。そこで、一定時間応答が無ければそこで動作を打ち切るよう、**タイムアウト**処理を入れましょう。
　まず、[@lst:dialog_timeout_import]のように、`asyncio`モジュールのインポートを追加します。これは、タイムアウト処理を行ううえで`asyncio.TimeoutError`例外を捉える必要があるためです。

```{#lst:dialog_timeout_import .python caption="asyncioモジュールのインポートを追加"}
from discord.ext import commands
import config
import asyncio  # 追加
```

　次に、`wait_for`関数を呼び出している部分を、[@lst:dialog_timeout]のとおり書き換えます。

```{#lst:dialog_timeout .python caption="タイムアウト処理のためtry-exceptで囲む"}
    try:
        # チェック関数に合格するようなメッセージを待つ
        msg = await bot.wait_for('message', check=check_message_author, timeout=10)
    except asyncio.TimeoutError:
        await ctx.send("タイムアウトしました。")
        return
```

　このコードでは、**Exception（例外）**といわれる仕組みを利用してタイムアウト処理をしています。それぞれの処理について詳細に見ていきましょう。

```{#lst:dialog_timeout_wait_for_timeout .python caption="タイムアウトつきのwait_for"}
        msg = await bot.wait_for('message', check=check_message_author, timeout=10)
```

　`wait_for`関数に`timeout`パラメーターの指定を追加しました。このパラメーターには、どれくらいの時間が経ったらタイムアウトと判断するかを**秒数**で指定します。ここでは`timeout=10`と指定しているので、10秒後にタイムアウトと判断されます。タイムアウトとなると、`asyncio.TimeoutError`例外が発生します。つづいて、この例外を捉える処理について説明します。

```{#lst:dialog_timeout_try_except .python caption="try-except文"}
    try:
        # ここで例外が発生すると、続くexceptで指定したブロックに飛ぶ
        ...
    except asyncio.TimeoutError:
        await ctx.send("タイムアウトしました。")
        return
```

　[@lst:dialog_timeout_wait_for_timeout]で発生した例外を捉える処理です。`try:`で開始したブロックにおいて発生した例外は、続く`except`で捉えられます。`asyncio.TimeoutError`が発生した場合、「タイムアウトしました。」というメッセージを送信します。メッセージが受信できなかったため、以降の処理は継続できないため、`return`で関数の処理を打ち切ります。
　実装できたら、ターミナルを開き仮想環境を有効化して`$ python launcher.py`を実行し、Botを起動します。テスト用Guildのテキストチャンネルに`!hello`と投稿したあと、そのまま10秒待機します。[@fig:sc_dialog_timeout]のようにタイムアウトメッセージが表示されればOKです。

![タイムアウトの動作確認](resource/image/sc_dialog_timeout.png){#fig:sc_dialog_timeout}

_Bot開発のタネ　イベントを待って何かしてみよう

　`wait_for`関数を使えば、Botに多様な動作をさせることができます。さきほど紹介したmessageイベントの待機のほか、**reaction_addイベント**を待機するのも、Botとユーザーのやりとりをするためによく使われる手法です。[@lst:wait_for_reaction_example]は、メッセージに対するリアクションを待ち、リアクションに応じて動作を変える処理です。

```{#lst:wait_for_reaction_example .python caption="wait_forでreaction_addイベントを待機する"}
@bot.command()
async def hello_reaction(ctx):
    # イイネ、良くないネの絵文字
    thumbs_up = '\N{THUMBS UP SIGN}'
    thumbs_down = '\N{THUMBS DOWN SIGN}'

    # wait_forに渡すチェック関数
    def check_reaction(reaction, user):
        # リアクションの送信ユーザーを確認
        user_ok = (user == ctx.author)
        # リアクションの種別を確認
        reaction_ok = (reaction.emoji == thumbs_up or
                       reaction.emoji == thumbs_down)
        return user_ok and reaction_ok

    # メッセージを送信
    await ctx.send(f"こんにちは、{ctx.author.name}さん。")
    msg = await ctx.send("いまの気分を選んでください。")
    # 送信したメッセージにリアクションを付与
    await msg.add_reaction(thumbs_up)
    await msg.add_reaction(thumbs_down)
    # ユーザーからのリアクションを待つ
    reaction, user = await bot.wait_for("reaction_add",
                                        check=check_reaction)
    # ユーザーのリアクションに応じてメッセージを変える
    feel = "良い気分" if reaction.emoji == thumbs_up else "良くない気分"
    await ctx.send(f"{feel}なんですね。")
```

　ユーザーに付けてほしいリアクションをBotがあらかじめメッセージに付けておくことで、ユーザーはすでにあるリアクションボタンを押すだけで意思表示できます。

## メッセージを見やすく表現しよう - Embed
### BotにしかできないEmbed形式の表示
　これまでBotを使っていて、[@fig:sc_embed_example]のような形式のメッセージを見たことはありませんか？

![Embedの例](resource/image/sc_embed_example.png){#fig:sc_embed_example}

　これは、**Discord Embed**（以下Embed）と呼ばれるものです。Embedは主に、見やすく整形されたメッセージを表示するために用いられます。URLを貼った際などにEmbed形式で情報が表示されることはありますが、原則ユーザーが任意のEmbedを投稿することはできません。Botのメッセージとユーザーの投稿を明確に区別できるという点、メッセージごとに任意のColor（色）を付けられ、タイトル・本文・その他の値を見やすく表示できるという点で優れています。

### ただのメッセージをEmbedに書き換える

　ここでは、さきほど実装したあいさつコマンドの内容を少し編集し、最後のメッセージをEmbed形式で表示するように変更してみましょう。まず、[@lst:first_embed_import]のように、`discord`モジュールのインポートを追加します。

```{#lst:first_embed_import .python caption="discordのインポートを追加"}
from discord.ext import commands
import config
import asyncio
import discord  # 追加
```

　次に、`hello`コマンドの最後の一文を[@lst:first_embed]のとおり書き換えます。

```{#lst:first_embed .python caption="Embedを使ってみよう"}
    await ctx.send(f"「{msg.content}」という気分なんですね。")
        ↓
    # Embedインスタンスを作成
    embed = discord.Embed()
    # Embedの表示色を青色に設定
    embed.color = discord.Color.blue()
    # Embedの説明文を設定
    embed.description = "あなたの気分を把握しました。"
    # 気分をFieldとして表示
    embed.add_field(name="あなたの気分", value=msg.content)
    await ctx.send(embed=embed)
```

　書き換えが完了したら、ターミナルを開き、仮想環境を有効化して`$ python launcher.py`を実行し、Botを起動します。つづいて、テスト用Guildのテキストチャンネルに`!hello`と投稿しましょう。[@fig:sc_first_embed]のようにEmbed形式で表示されたら成功です。

![はじめてのEmbed 動作例](resource/image/sc_first_embed.png){#fig:sc_first_embed}


### Embedの属性紹介
　Embedには多様な属性があります。たとえば、[@lst:embed_detail]のようにコードを書くと、[@fig:sc_embed_detail]のように表示されます。

```{#lst:embed_detail .python caption="Embedの多様な属性"}
    from datetime import datetime
    embed = discord.Embed()
    embed.title = "title"
    embed.description = "description"
    embed.url = "https://example.com"
    embed.timestamp = datetime.now()
    embed.color = discord.Color.red()
    embed.provider.name = "provider_name"
    embed.provider.url = "https://example.com/provider"
    embed.add_field(name="field name", value="field value")
    embed.set_footer(
        text="footer",
        icon_url="https://cdn.discordapp.com/embed/avatars/0.png"  # 青色
    ).set_image(
        url="https://cdn.discordapp.com/embed/avatars/1.png"  # 灰色
    ).set_thumbnail(
        url="https://cdn.discordapp.com/embed/avatars/2.png"  # 緑色
    ).set_author(
        name="author",
        icon_url="https://cdn.discordapp.com/embed/avatars/3.png"  # 橙色
    )
    await ctx.send(embed=embed)
```

![Embedの多様な属性 表示例](resource/image/sc_embed_detail.png){#fig:sc_embed_detail}

　これらすべてを常に同時に使うことはそうそうないかもしれませんが、どのような属性があるかを頭の片隅に置いておきましょう。