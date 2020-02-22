　さあ、いよいよBotの開発です。実装するコードの全体像は、下記リポジトリにあります。こちらも参考にしてください。
　https://github.com/cod-sushi/discord_dev_sample.git

## Python仮想環境を作ろう - venv
　discord.pyをインストールするまえに、「おたすけBot」を作るための**仮想環境**を作成しましょう。Pythonにおける仮想環境とは、Pythonインタプリタとパッケージのインストール先をまとめたもののことです。
　まずは、Pythonによる**パッケージ管理**についてと、なぜ仮想環境が必要なのかということについて説明します。手順だけを知りたい場合、「仮想環境を使ってみよう」まで読み飛ばしても構いません。

### pip - Pythonのパッケージ管理ツール
　**pip**は、PyPIなどに公開されているPythonパッケージ（拡張機能）の管理（インストール、アンインストール、アップデートなど）を行うユーティリティです。Python 3.4以降にはデフォルトで付属しています。たとえば、[@lst:pip_example]のようにコマンドを実行することで、`<package>`パッケージをインストールできます。

```{#lst:pip_example .shell caption="pip 利用イメージ"}
$ python -m pip install <package>
```

　discord.pyも数あるPythonパッケージのひとつです。では、discord.pyをインストールしたいときは、`$ python -m pip install discord.py`とすればよいのでしょうか。実は、これでは不十分な場合があるのです。

#### pipは複数バージョンのインストールには非対応
　複数のBotを開発していると「これまで使っていたBot1ではdiscord.pyの**v0.16.12**を動かしたいが、新しく作るBot2ではdiscord.pyの**v1.0.1**を動かしたい」という状況があるとします。実はpip単独では、このような状況に対応するのはすこし不便です。
　pipは、デフォルトではグローバル領域（全ユーザー共通のパッケージインストール先）にパッケージをインストールします。pipではバージョンを指定してパッケージをインストールすることはできますが、同じインストール場所にインストールできるのは、ひとつのパッケージにつきひとつのバージョンだけなのです。pipの挙動を確認してみましょう。[@lst:pip_cant_install_mutiple_version]をごらんください（実際に実行する必要はありません）。

```{#lst:pip_cant_install_mutiple_version .shell caption="pipでは複数バージョンをインストールできない"}
$ pip install discord==0.16.12 ←①
$ pip install discord==1.0.1   ←②
$ pip list  ←③
Package       Version
------------- -------
aiohttp       3.6.2
async-timeout 3.0.1
attrs         19.3.0
chardet       3.0.4
discord       1.0.1   ←④
...
```

1. discord v0.16.12のインストールを行う
2. discord v1.0.1のインストールを行う
3. インストール済パッケージ一覧を出力する
4. 1.0.1しか存在しないことが確認できる

#### インストール場所を分ければ解決
　さきほど、「同じインストール場所にインストールできるのは、ひとつのパッケージにつきひとつのバージョンだけ」とお話ししました。これは、裏を返せば、「インストール場所が違えば、同一パッケージの異なるバージョンをインストールすることができる」ともいえます。Bot1にはBot1のためのパッケージのインストール場所、Bot2にはBot2のためのパッケージのインストール場所、といったふうにプロジェクトごとにパッケージのインストール場所を分けられたら、同じパッケージの別バージョンをそれぞれインストールできるのです。ちなみに、pipでは`$ pip install package -t path`のように`-t`オプションを指定することでインストール先を指定できます。しかし、毎回インストール先を指定するのは面倒です。

#### ツール「venv」は仮想環境切り替えツール
　先述した「プロジェクトごとにパッケージのインストール先を分ける」といった操作を簡単に行う仕組みとして提供されているのが**venv**です。venvは、Python3以降に標準ライブラリとして付属しています。
　venvで切り替えられるのは、パッケージのインストール先だけではありません。たとえば、Python3.5とPython3.7といった**Pythonインタプリタ**（Pythonコードを解釈して動かすためのプログラム本体）を切り替えることもできます。**Pythonインタプリタ**と**パッケージのインストール先**といったPythonプログラムの開発および実行に必要なものをまとめて、**仮想環境**と呼びます。venvは、**Python仮想環境を切り替えるためのツール**なのです。

### 仮想環境を使ってみよう
　前項では、venvを使えばPython仮想環境の作成・切り替えが簡単に行えることについて説明しました。それではいよいよ、実際にvenvを使ってPython仮想環境を作成してみましょう。

#### Bot開発用フォルダーの作成
　Visual Studio Codeを立ち上げ、「Terminal」→「New Terminal」を選択し、ターミナルを立ち上げます。ターミナルを立ち上げたら、[@lst:first_venv]のリスト内容を実行してください。

```{#lst:first_venv .shell caption="ホームディレクトリに移動する"}
$ cd ~
$ pwd
/c/Users/(ユーザー名)  # Windowsの場合
/home/(ユーザー名)    # macOSの場合
```

　`cd ~`は、**ホームディレクトリ**に移動するコマンドです。`pwd`は、現在あなたがいる場所を表示するコマンドです。ここで表示された場所が、あなたのホームディレクトリです。この場所にBotのソースコードやBot開発に利用する仮想環境などを格納する**Bot開発用フォルダー**を作成します。[@lst:first_venv_1]をごらんください。

```{#lst:first_venv_1 .python caption="Botを作成するフォルダーを作成する"}
$ mkdir my_bot
$ cd my_bot
$ pwd
/c/Users/(ユーザー名)/my_bot  # Windowsの場合
/home/(ユーザー名)/my_bot    # macOSの場合
```

　`mkdir`は、指定した名前のフォルダーを作成するコマンドです。つづいて、`cd`コマンドで作成した`my_bot`フォルダー内に移動しています。移動したら、`pwd`コマンドで再度自分の現在位置を確認しておきます。


#### 仮想環境の作成
　作成したBot開発用フォルダー内に仮想環境を作成します。[first_venv2]の内容を実行してください。このコマンドの実行には少し時間がかかります。コマンドを実行したら、ふたたびプロンプトが出るまでしばらく待機してください。

```{#lst:first_venv2 .shell caption="はじめてのvenv"}
$ python(※) -m venv .venv
```

　[first_venv2]内にて（※）印のついた`python`は、環境により`python3`、`py`など、**3.5.3以上のバージョンのPythonを呼び出せるコマンド**に置き換えてください。pythonのバージョンは、`--version`オプションで確認できます。
　 `python -m venv`は、venvモジュールをスクリプトとして実行するという意味の記述です。これにより、`.venv`というフォルダー内に仮想環境を作成できました。

#### 仮想環境の有効化
　ここまでの手順で、venvによる仮想環境を作成しました。つづいて、作成した仮想環境を有効化してみましょう。Visual Studio Code上のターミナルで[@lst:activate_venv]を実行します。WindowsとmacOSで一部手順が異なります。

```{#lst:activate_venv .shell caption="はじめての仮想環境有効化"}
$ source .venv/Scripts/activate　←Windowsの場合
$ source .venv/bin/activate　←macOSの場合
(.venv) $ ←プロンプトに「(.venv)」が追加されたことを確認する
```

　仮想環境の切り替えに成功すると、プロンプトの前に`(.venv)`という文字列が表示されるようになります。これは、「`.venv`というディレクトリの仮想環境を有効化中ですよ」ということを表しています。Git Bashの場合、プロンプトが長いので[@lst:activate_venv_windows]のように`(.venv)`が`$`と離れた場所に表示されることもありますが、これも仮想環境が有効になっています。

```{#lst:activate_venv_windows .python caption="Windowsのプロンプト表示例"}
(.venv)
user@PC-name MINGW64 ~/my_bot/
$
```

#### 仮想環境の動作確認
　仮想環境が有効化できたら、動作確認をしてみましょう。[@lst:venv_check_python]の内容を**そのまま**実行してみてください。`python3`や`py`に置き換える必要はありません。

```{#lst:venv_check_python .shell caption="仮想環境の動作確認"}
$ python --version
Python 3.7.3
```

　いかがでしょうか？これまで`py`や`python3`を付けないと実行できなかった方も、`python`というコマンドで実行できたのではないでしょうか。このように、一度`venv`を有効化してしまえば、どのようなOS・環境・バージョンでも`python`というシンプルなコマンドを使えるようになります。`python`コマンドを実行することで動作するPythonのバージョンは、`$ python -m venv .venv`を実行した際のPythonのバージョンと同じものになります。

#### 仮想環境の無効化
　一度仮想環境を無効化してみましょう。[@lst:disable_venv]をごらんください。

```{#lst:disable_venv .shell caption="仮想環境の無効化"}
(.venv) $ deactivate　←仮想環境の無効化
$　← 「(.venv)」表示が消える
```

　`(.venv)`の表示が消えたら、仮想環境が無効化されたということです。venvの挙動の仕組みについて興味のある方は、.venv/Scripts/activate（macOSでは、.venv/bin/activate）の内容を確認してみるとよいでしょう。シェルスクリプトについての知識が必要ですが、シンプルな操作で複雑な環境切り替えを実現している工夫を見ることができます。
　これ以降、チュートリアルを実践する際には、[@lst:activate_venv]を実行して、仮想環境を有効化しておいてください。一度ターミナルを閉じると仮想環境は無効化されますので、ターミナルを立ち上げるたびに有効化する必要があります。ただし、Visual Studio Codeから「New Terminal」によってターミナルを開いた場合、Visual Studio Codeのバージョンによっては自動的に仮想環境が有効化される場合もあります。

#### 仮想環境の削除
　仮想環境の内容はすべてフォルダーに入っているので、仮想環境を破棄したいときは`$ rm -r .venv`コマンドでフォルダーごと削除します。このように、仮想環境の作成および破棄はとても簡単に行えます。失敗を恐れず、いろいろ試してみましょう。

## discord.pyをインストールしよう
### 仮想環境にモジュールをインストールする
　さて、さきほど作成した仮想環境を使って開発をはじめましょう。まずはPythonでDiscord Botを開発する際に便利なパッケージ、**discord.py**をインストールします。仮想環境が有効化された状態で、[@lst:install_discord_py]の内容を実行します。

```{#lst:install_discord_py .shell caption="discord.py インストール"}
(.venv) $ pip install discord.py
...
Successfully installed aiohttp-3.6.2 async-timeout-3.0.1 attrs-19.3.0 chardet-3.0.4 discord.py-1.3.1 idna-2.8 idna-ssl-1.1.0 multidict-4.7.4 typing-extensions-3.7.4.1 websockets-8.1 yarl-1.4.2
```

　インストールには少し時間がかかります。最後に「Successfully installed」から始まるパッケージ名の羅列が表示されればインストールは無事成功です。この出力をよく確認すると、直接指定していない「aiohttp」や「async-timeout」などのパッケージもインストールされていることがわかります。これは、discord.pyが**依存**しているパッケージについてもpipが自動的にインストールしているためです。
　上記リストは、2020年2月時点のもので、discord.pyのバージョンは1.3.1となっています。コマンド実行時の結果、この数字や表示されたパッケージが多少異なっていても問題はありません。

### 使用パッケージを設定ファイルに書き出す
　さまざまなパッケージをインストールしていると、Botの動作に何のパッケージが必要であったかがわからなくなってしまいます。仮想環境を作り直したり、開発環境を他のPCに移したくなったりした際に、ひとつひとつ`pip install`を繰り返すのは面倒ですし、漏れが発生します。
　そこで、`pip`の機能のひとつ、`freeze`を使いましょう。`pip freeze`は、現時点でその環境にインストールされているパッケージの一覧をバージョンとともに出力するコマンドです。[@lst:pip_freeze_example]をごらんください。

```{#lst:pip_freeze_example .shell caption="freeze 例"}
(.venv) $ pip freeze > requirements.txt
```

　`pip freeze`コマンドによって出力されるパッケージとバージョンの一覧を「requirements.txt」というファイルに書き込んでいます。ファイル「requirements.txt」の内容は、たとえば[@lst:requirements_txt_example]のようになります。

```{#lst:requirements_txt_example .shell caption="requirements.txt 内容の例"}
aiohttp==3.6.2
async-timeout==3.0.1
attrs==19.3.0
chardet==3.0.4
discord.py==1.3.1
idna==2.8
idna-ssl==1.1.0
multidict==4.7.4
typing-extensions==3.7.4.1
websockets==8.1
yarl==1.4.2
```

　ここで設定ファイルには「requirements.txt」という名前を付けています。機能的にはどのようなファイル名でも構いません。しかし慣習として、プロジェクトに必要なパッケージ一覧を示すファイルの名前にはこの「requirements.txt」がよく使われますので、この名前をそのまま使用することを推奨します。

#### 設定ファイルの使い方
　`pip install`コマンドに`-r <filename>`オプションを使用することで、ファイル内に記述されたパッケージを一括でインストールできます。今回であれば、`$ pip install -r requirements.txt`のように指定すると、そっくりそのまま同じ環境を再現できます。
　詳細は後述しますが、`requirements.txt`はBotのソースコード本体と一緒にGitリポジトリへ登録します。

#### バージョン指定について
　pipでのインストールの際には、バージョンが未指定であればその時点での最新の**安定版**がインストールされます。requirements.txtにバージョンが指定されていないと、仮想環境を作り直すなどした際にパッケージのバージョンが変わり、それによってBotの機能が壊れてしまうことあります。「知らないうちに壊れた」ということがないよう、バージョンは固定しておきましょう。

#### アップデートも適宜行おう
　さきほど、Botの挙動が不意に変わることがないように、パッケージのバージョンを固定するよう推奨しました。ただし、パッケージのアップデートには機能追加やコード刷新のほか、バグ修正や脆弱性対応などの重要な更新も含まれます。discord.pyの場合、discord APIの仕様変更により旧バージョンでは正常に動作しなくなることもあります。そのため、使用するパッケージのアップデート状況などを追跡し、必要に応じて最新バージョンにアップデートしておきましょう。アップデート後は、Botの機能が壊れていないかしっかり確認しましょう。

## プログラミングの基礎用語
　ここでは、これからのBotプログラミングの説明に出てくる用語について説明します。プログラミング経験者の方や、「とにかくコードを書いてみたい！」という方は、「Botにログインしてみよう」まで読み飛ばして頂いて構いません。

### 関数
　関数とは、処理のかたまりをつくり、いつでも呼び出せるように名前をつけたもののことです。関数に名前をつけ、処理の内容を記述することを関数を**定義する**といいます。Pythonにおける関数定義の例を[@lst:python_function]に示します。

```{#lst:python_function .python caption="関数定義の例"}
def first_func():
    print("hello")
    print("world")
```

　これは、`print("hello")`という処理と`print("world")`という処理を連続して行う関数で、`first_func`という名前がつけられています。この関数を呼び出すときは、[@lst:python_function_call]のように書きます。この関数を呼び出すと、`hello`、`world`が出力されます。

```{#lst:python_function_call .python caption="関数呼び出しの例"}
first_func()
# hello
# world
```

### クラスとインスタンス
　クラスとは、ある機能に関連する処理やデータをひとまとめにするしくみのことです。今回使用するPythonはもちろん、オブジェクト指向の考えに基づくプログラミング言語には、たいていクラスというしくみが存在します。また、インスタンスは、クラスを実体化したもののことです。
　クラスは何のために存在するのでしょうか？ゲームや複雑な仕様のシステムなどを、プログラミングの世界に再現するときのことを考えてみましょう。たくさんのデータや機能が散り散りに存在すると、どこで何の処理をしているのか理解しにくいプログラムとなってしまいます。理解しやすいプログラムにするには、機能ごとにデータと処理を分類し、いくつかのかたまりにまとめ、整理する必要があります。クラスは、その整理を助けてくれます。クラスは、Pythonを含む多くのプログラミング言語に存在します。

![クラスが存在する意味](resource/image/class_is_important.png){#fig:class_is_important}

　discord.pyも例にもれずクラスを使ってプログラミングされています。前項で使ったクラス`discord.ext.commands.Bot`は、Botがdiscordに接続する機能についてのデータと処理が集まったクラスです。他にもdiscord.pyには多くのクラスが存在します。たとえば、ボイスチャットへの接続についてのクラス`discord.VoiceClient`や、Guildについてのクラス`discord.Guild`などがあります。discord.pyでは、主にクラスを実体化した**インスタンス**を操作することでクラスの機能を使います。


## Botにログインしてみよう
### はじめてのBotプログラミング
　それでは、実際にBotを動かすコードを書いていきましょう。「Bot」という単語に反応して返事を返す動作をさせてみます。

#### ファイルの作成
　まずはファイルを作成しましょう。

1. Visual Studio Codeのメニューから、「File」→「Open Folder」を選択する
2. 仮想環境`.venv`のあるフォルダーを選択し、決定する
   `.venv`フォルダーではなく、`.venv`フォルダーを含むフォルダーを選択することに注意。
3. Visual Studio Codeのメニューから、「File」→「New File」を選択する
4. 作成したファイルに、「launcher.py」と名前をつけて保存する

　この時点でのフォルダー構成は[@lst:launcher_folder_structure]のようになります。

```{#lst:launcher_folder_structure .shell caption="フォルダー構成"}
my_bot
│  launcher.py
│  requirements.txt
└─.venv
```

#### launcher.pyのコード

　ファイルを作成したら、[@lst:first_discord_bot]の内容を書いていきましょう。`ThisIsDummyToken00000000.XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX`の部分は、[@sec:environment]「トークンの取得」で取得したあなたのBotのトークンに置き換えてください。

```{#lst:first_discord_bot .python caption="はじめてのDiscord Bot"}
from discord.ext import commands

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

bot.run("ThisIsDummyToken00000000.XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX")
```

#### コーディングのポイント
　コードを書いていると、Visual Studio Codeの画面上に「Linter pylint is not installed」と表示されることがあります。これは、Pythonの文法チェックをするツールがインストールされていないことを知らせるものです。開発効率が向上しますので、「Install」ボタンを押してインストールしておきましょう。
　また、`#`から始まる行は**コメント**といい、プログラムの動作には影響しない注釈ですので、書き写さなくても構いません。ただし、**インデント（空白）は正確に**書き写すようにしてください。

#### コードの内容を解説

　このコードについて、詳細に解説していきましょう。

```{#lst:detail_first_bot_1 .python caption="discord.pyをインポート"}
from discord.ext import commands
```

　`from <package> import <module>`で、`<package>`内にある`<module>`を**インポート**します。デフォルトのライブラリや外部ライブラリについて、このプログラム内で使う旨を宣言する構文です。単に`import <module>`と書く場合もあります。
　今回は`discord.ext`パッケージ内の`commands`モジュールをインポートしたいので、このように書きます。

```{#lst:detail_first_bot_2 .python caption="Botインスタンスの作成"}
bot = commands.Bot(command_prefix="!")
```

　`discord`パッケージ内に存在する、`Bot`というクラスのインスタンスを生成しています。生成したインスタンスを、`bot`という名前の変数に格納しています。クラスとインスタンスについては後述します。`commands.Bot`というのは、「`commands`モジュールの中にある`Bot`クラス」という意味です。カッコ内に`command_prefix="!"`という記述がありますが、いまは使いません。

```{#lst:detail_first_bot_3 .python caption="readyイベントが発生したときの動作を定義"}
@bot.event
async def on_ready():
    print("on_ready")
```

　`@bot.event`は**デコレーター**と呼ばれるもので、botに`ready`**イベント**が発生したとき、どのような処理をするか**定義**しています。ここで定義した処理は、BotがDiscord APIサーバーに接続し、Botの動作に必要なデータ取得が完了したときに実行されます。イベントについての詳細は後述します。処理の内容は、`on_ready`と標準出力（通常であればターミナル上です）に出力する、というものです。

```{#lst:detail_first_bot_message .python caption="messageイベントが発生したときの動作を定義"}
@bot.event
async def on_message(message):
    if message.author == bot.user:
        # Botからのメッセージには反応しない
        # この判定をしないと無限ループが起きる
        return

    if "Bot" in message.content:
        await message.channel.send("は～い、Botで～す")
```

　`on_message(message)`は、メッセージ（テキストチャンネルへの投稿）が発生したことを通知するイベントです。無限ループ（呼び出しが繰り返され、処理が終わらなくなること）を避けるため、Bot自身が投稿したメッセージには反応しないようにしています。メッセージの内容`message.content`に「Bot」という文字列が含まれると、そのメッセージが投稿されたチャンネルに「は～い、Botで～す」という返信を投稿します。

```{#lst:detail_first_bot_4 .python caption="指定したトークンでBotの稼働を開始"}
bot.run("ThisIsDummyToken00000000.XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX")
```

　`bot`の`run`メソッドにトークンを引数として渡すと、Discord APIサーバーへの接続を開始し、Botアカウントにログインします。本来であればこのようにソースコードに直接トークンを記載するのは避けるべきですが、まずはBotを動かすため、このようにしています。トークンをソースコードに直書きしてはいけない理由やトークンを隠す方法については[@sec:medium]で紹介します。

### はじめてのBot実行
　それでは、作成したBotを実行してみましょう。

1. Visual Studio Codeのメニューから、「Terminal」→「New Terminal」を選択する
2. 仮想環境を有効化する（Visual Studio Codeのバージョンによっては自動的に有効化される）
3. `(.venv) $ python launcher.py`を実行し、しばらく待つ。

　いかがでしょうか？うまく動きましたか？うまくいけば、[@lst:first_bot_example]のように、on_readyという文字列がターミナルに表示されます。

```{#lst:first_bot_example .shell caption="はじめてのBot 実行例"}
(.venv) $ python launcher.py
on_ready
```

　on_readyという文字列が表示されたことを確認できたら、「Bot」という投稿をしてみましょう。[@fig:sc_on_message]のように、Botが反応したら成功です。

![Botという単語に反応させてみる](resource/image/sc_on_message.png){#fig:sc_on_message}

　**Ctrl-C（コントロールキーと「C」キーの同時押し）**を入力し、しばらく待つとBotがDiscordから切断し、Pythonプロセスが終了します。動作確認ができたらCtrl-CでBotを終了しましょう。

### エラーが出たら実力アップのチャンス
　プログラミングにはエラーがつきものです。上記のとおりに動かなくても、がっかりする必要はありません。むしろうまく動かないときは、自分の**勘違い**や**間違いのクセ**を見つけるチャンス。焦らず原因を究明しましょう。
　たとえば、[@lst:error_example_notfound]のようにエラーが出力され、すぐにBotの実行が終了してしまうことがあります。

```{#lst:error_example_notfound .shell caption="エラーの例"}
(.venv) $ python launcher.py
Traceback (most recent call last):
  File "launcher.py", line 1, in <module>
    from discrd.ext import commands
ModuleNotFoundError: No module named 'discrd'
```

　このようなときは、「Error」とついている単語や、その文章（ModuleNotFoundError: No module named 'discrd'）の内容を読むことがなにより大切です。英語がわからなければ、翻訳サービスで翻訳してみるのもいいでしょう。それでもわからないときは、「ModuleNotFoundError」というエラーをGoogleなどの検索エンジンで検索してみましょう。このエラーがどのような理由で発生するかの手がかりが見つかるかもしれません。ちなみにこのエラーは、「`discrd`という名前のモジュールがないよ」という内容です。`discord`を`discrd`と誤記していることが原因ですね。
　バグの埋め込みと修正を繰り返すうちに、エラー名称とエラー内容、その原因の組み合わせがなんとなく頭に入ってきます。ひとつひとつのエラーにしっかり向き合うことで、デバッグのスピードは徐々に上がります。最初は辛いかもしれませんが、ゆっくり着実に取り組みましょう。まとめると、以下のようになります。

1. エラーメッセージを読む！これがもっとも大事
2. エラー内容をGoogleで検索する
3. どのような原因でどのようなエラーが出たかのパターンを覚える

### よくあるまちがい
　ここでは、discord.pyを使っているとよく遭遇するエラーや誤り、注意すべきことについて記します。

#### awaitの書き忘れ
　よくあるのが、この誤りです。discord.pyのAPIのうち、公式ドキュメントに「This function is a coroutine.」と記載のあるものは、呼び出し時に`await`を付ける必要があります。とはいえ、[@lst:error_never_awaited]のようにわかりやすいエラーが出るため、デバッグに時間がかかることは少ないでしょう。

```{#lst:error_never_awaited .plain caption="await忘れのエラーサンプル"}
launcher.py:67: RuntimeWarning: coroutine 'Messageable.send' was never awaited
```

#### on_messageの反応対象を制限しよう
　[@lst:detail_first_bot_message]の説明でも述べましたが、`on_message`イベントはBot自体の投稿に対しても発行されます。つまり、`on_message`イベントに反応したBotが投稿を行い、その投稿により、また`on_message`が発行されると、永遠に投稿が続く無限ループ状態となる可能性があります。`on_message`イベントを使って投稿に反応する際には、[@lst:on_message_sample]のように処理のはじめにBot自身の反応であれば何もしないようにしておくとよいでしょう。

#### 処理をブロッキングしないようにしよう
　discord.pyによるBotは、**async/await**構文を使った非同期プログラミングの仕組みで動作しています。そのため、ある部分で**ブロック**（時間のかかる処理などを行い、その部分で処理を止めること）してしまうと、常に動作する必要のあるイベント処理や通信処理も動かなくなってしまいます。つまり、ユーザーの操作に反応できなくなってしまうのです。
　たとえばファイルへの書き込みやDBの検索など、時間のかかる外部入出力には要注意です。意図しないブロックをしてしまわないよう、注意しましょう。また、**sleep**でプログラムの処理を止めたいときは、`time.sleep`ではなく`await asyncio.sleep`を使いましょう。


## イベントについて知ろう
### イベントはBotに起きる出来事
　discord.pyの**イベント**とは、Botを動作させる中で発生する**できごと**のことです。イベントに対応した処理を定義することで、Botに好きな動作をさせることができます（[@fig:discord_py_event]）。イベントをマスターしたら、discord.pyの半分をマスターしたと言って良いかもしれません。

![discord.pyはイベントに紐付けた処理を定義することでBotを動作させる](resource/image/discord_py_event.png){#fig:discord_py_event}

　イベントに紐付いた処理を定義する方法はいくつかありますが、まずはもっとも手軽な方法について紹介します。[@fig:discord_py_event_code_description]をごらんください。

![イベントに対応する処理を定義するコード](resource/image/discord_py_event_code_description.png){#fig:discord_py_event_code_description}

　処理を定義したいイベントと同じ名前の関数を定義し、`@bot.event`デコレータを付けるだけです。このように、イベントに対する動作を定義した関数を**イベントリスナー**といいます。通常の関数定義は`def func():`のように行いますが、`async def on_ready():`のように**async**というワードを付ける必要があります。これは、関数を**コルーチン関数**として定義するためです。コルーチン関数について本書では詳細な説明は行いませんが、待ち時間の発生するような処理を実施する際にその関数の実行を中断し、その完了を待つ間に別の処理を実施することで、効率的にプログラムを動作させるための仕組みであると理解していればよいでしょう。イベントリスナーは、必ずコルーチン関数として定義する必要があります。

_Bot開発のタネ イベントで何ができるか考える

　discord.pyにはさまざまなイベントが存在します。ここでは代表的なイベントを紹介します。また、Botのコード（[@lst:first_discord_bot]）に追加すればすぐ動作するコード例も紹介します。ただし、`bot.run`以降に書いた処理はBotの稼動には反映されません。コードを追記する場合は**かならずbot.runの呼び出しの前に追記**してください。

#### on_message
　`on_message(message)`は、メッセージ（テキストチャンネルへの投稿）が発生したことを通知するイベントです。引数`message`には、投稿されたメッセージの情報（投稿されたテキストチャンネル、投稿者の情報、メッセージ内容など）が含まれます。本イベントは、Botから見えるテキストチャンネルに投稿があるたびに発生します。たとえば以下のようなことができます。

- テキストチャンネルに投稿があったら……
  - 事前に定義したNGワードが含まれたら、「それはNGワードですよ」と警告する投稿をする（[@lst:on_message_sample]）。
  - ユーザーごとにその日の投稿数をカウントして、一定数に達したら「本日50投稿目です！」と通知する。
  - 投稿の内容を`mecab-python3`で形態素解析（日本語文法に則って品詞分類すること）して、「〇〇って何？」と投稿する。
  
```{#lst:on_message_sample .python caption="NGワード通知 サンプル"}
@bot.event
async def on_message(message):
    # 事前に定義したNGワードが含まれたら、「NGワードですよ！」と警告する
    if message.author == bot.user:
        # Botからのメッセージには反応しない
        # この判定をしないと無限ループが起きる
        return

    # NGワードの定義
    NG_WORDS = ['NGワード', '禁止単語']

    # 全NGワードについて存在確認
    for ng_word in NG_WORDS:
        if ng_word in message.content:
            # NGワードを発見したらテキストチャンネルに通知
            await message.channel.send(f"**{ng_word}**はNGワードです。")
            break
```


#### on_message_delete
　`on_message_delete(message)`は、メッセージの削除を通知するイベントです。引数`message`には、削除されたメッセージの情報が含まれます。ただし、**キャッシュ**にないメッセージが削除された場合、このイベントは発行されません。キャッシュとはBotの起動中、メモリ上に保存される一時的なデータのことです。キャッシュにないメッセージとして、Botが起動するより前に投稿されたメッセージがあげられます。Botが起動した後に投稿されたメッセージでも、キャッシュのサイズには限りがありますので、古いものはキャッシュから消えていることもあります。キャッシュにないメッセージの削除も検知したい場合、`on_raw_message_delete`イベントを使いましょう。ただし、このイベントは`on_message_delete`とは引数の数や内容が異なりますので、使用する際にはドキュメントを確認してください。
　`on_message_delete`では、たとえば以下のようなことができます。

- テキストチャンネルからメッセージが削除されたら……
  - メッセージが削除された旨を投稿する（[@lst:on_message_delete_sample]）。
  - メッセージを削除したユーザーに、削除したメッセージの内容をDMする。

```{#lst:on_message_delete_sample .python caption="メッセージが削除された旨を投稿する"}
@bot.event
async def on_message_delete(message):
    # キャッシュされていないメッセージの削除は検知できないことに注意！
    # １行が80字を超える場合、このように改行して80字におさめるとよい
    await message.channel.send(
        f"{message.author.name}さんのメッセージが削除されました。")
```

#### on_member_join
　`on_member_join(member)`は、Guildにあたらしいメンバーが入ったことを通知するイベントです。引数`member`には、入ったメンバーについての情報（名前、ユーザーIDなど）が含まれます。

- Guildにメンバーが新しく入ったら……
  - Guildのルールについて、メンションやDMでお知らせする。（[@lst:on_member_join_sample]）
  - サーバーの管理者にDMで知らせる。

```{#lst:on_member_join_sample .python caption="新規メンバーにあいさつを送る"}
@bot.event
async def on_member_join(member):
    # Guildメンバーがふえたら挨拶DMを送信する
    # memberのDM受け取り設定によっては失敗する
    await member.send(
        f"{member.name}さん、サーバー「{member.guild.name}」にようこそ！\n"
        f"わたしは「{bot.user.name}」といいます。\n"
        "このサーバーでは、特にきまりごとはありません。ごゆっくりどうぞ。"
    )
```


#### on_guild_join
　`on_guild_join(guild)`は、BotがあたらしいGuildに参加したことを通知するイベントです。引数`guild`には、入ったGuildについての情報（Guild名、Guild ID、メンバー数など）が含まれます。

- あたらしいGuildに参加したら……
  - 参加したGuildのテキストチャンネルに、あいさつメッセージを投稿する。
  - Botのログに、Guild名、Guild ID、メンバー数などの情報を記録する（[@lst:on_guild_join_sample]）。

```{#lst:on_guild_join_sample .python caption="新規サーバーの情報をログに残す"}
@bot.event
async def on_guild_join(guild):
    # 新しくGuildに入ったらログを残す
    print("新しくGuildに入った！")
    print(f"名前: {guild.name} / ID: {guild.id} / メンバー数: {guild.member_count}")
```

#### on_voice_state_update
　`on_voice_state_update(member, before, after)`は、メンバーのボイスチャット状態が変わったことを通知するイベントです。引数`member`には対象のメンバーの情報、`before`、`after`のは変化前・変化後それぞれの情報が含まれます。これらの情報を総合すると、「誰かがボイスチャンネルに入ったら」「ボイスチャンネルの参加者が0になったら」などの条件をつけてBotを動作させることができます。

- 誰かがボイスチャンネルに入ったら……
  - 誰かがボイスチャットに入ったことをテキストチャンネルに通知する（[@lst:on_voice_state_update_sample]）。
- ボイスチャンネルの参加者が0になったら……
  - ボイスチャットが終了したことをテキストチャンネルに通知する。

```{#lst:on_voice_state_update_sample .python caption="ボイスチャンネル入室を通知する"}
@bot.event
async def on_voice_state_update(member, before, after):
    # 誰かがボイスチャンネルに入ったら、テキストチャンネルに通知する。
    if after.channel is not None:
        # 適当なテキストチャンネルを投稿先として使う。
        # テキストチャンネルが存在しないGuildでは失敗する
        text_channel = member.guild.text_channels[0]
        # テキストチャンネルに通知を送信
        await text_channel.send(
            f"{member.name}さんがボイスチャンネル「{after.channel.name}」に入室しました。")
```


#### その他のイベント
　詳しくは、discord.py公式ドキュメントを読みましょう。すべてのイベントが掲載されています。
　https://discordpy.readthedocs.io/en/latest/api.html#event-reference
