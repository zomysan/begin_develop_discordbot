　ここでは、より実践的なBot開発について述べます。実装するコードの全体像は、下記リポジトリにあります。こちらも参考にしてください。
　https://github.com/cod-sushi/discord_dev_sample.git

## Botの機能をファイルに分割する - Cog
### コード整理のためのしくみ - Cog
　**Cog**とは、複数のイベントリスナーやコマンドを1つのクラスにまとめることができる機能です。[@sec:medium]まではすべてのイベントリスナーとコマンドを単一のlauncher.pyというファイルに実装してきました。しかしイベントリスナーとコマンドが増えてくると、単一のファイルにすべてを書いていてはコードが巨大化し、複雑になってしまいます。Cogを利用すれば、イベントリスナー・コマンドごとをファイルに分割し、コードを見通しよく整理できます。Cogは、通常Extensionといわれる機能と併用して利用します。Cogを実際に使ってみる前に、Extensionについて簡単に解説します。

### 動的なモジュール読み込み - Extension
#### Extensionとは
　Extensionは、Botの動作を止めることなくモジュールを再読み込みするための仕組みです。一般的にこのような再読み込みは**hot-reloading**と呼ばれます。テスト中のBotは、好きなときに動作を停止し、好きなときに再開できます。しかし、すでに稼働中のBotはできるなら止めたくありませんよね。Extensionを使えば、Botの稼働を止めることなく変更したコードをBotに反映できます。

#### Extensionの実装方法
　Cogやコマンドの定義を含むモジュールに関数`setup`を定義すると、そのモジュールをExtensionとして扱えるようになります。[@lst:extension_setup_func]をごらんください。

```{#lst:extension_setup_func .python caption="モジュールをExtension化するためのインターフェイス"}
# Sample Cogの定義部分を省略
...
def setup(bot):
    bot.add_cog(SampleCog(bot))
```

　`setup`はコルーチン関数ではない通常の関数で、Bot型の引数を1つとります。この引数は、このExtensionをインストールする対象のBotインスタンスを指します。ここでは例として、`SampleCog`というCogクラス（[@lst:extension_setup_func]の省略部分に定義されているもの）のインスタンスを作成し、`bot`に追加する処理を実装しています。

#### Extensionの使用方法
　このようにして作成したExtensionは、簡単にBotへインストールできます。[@lst:extension_setup_func]のファイル名が「sample_cog.py」だったとすると、同一ディレクトリのコードから、[@lst:easy_to_use_extension]のように書くだけでインストールができます。

```{#lst:easy_to_use_extension .python caption="Extensionの利用"}
bot.load_extension("sample_cog")
```

　Cogの項でも述べましたが、Cogを定義したモジュールはExtensionとしておくと開発中・運用中の両方で大変便利です。コーディングを実践することで、その便利さを実感してみましょう。

#### Extensionのhot-reloadingを実現するコマンドの例
　コマンドによってExtensionの再読み込みをする`reload`コマンドがあると便利です。このコマンドの定義の一例を[@lst:reload_command_example]に示します。このコマンドはBotの動作に大きな影響を与えるので、`@commands.is_owner()`デコレータを付けることで、Botの所有者（つまり開発者であるあなたです）だけが使えるようにしておきましょう。

```{#lst:reload_command_example .python caption="Extensionの再読み込みをするreloadコマンドの実装例"}
    # Cogクラスの中にこのコードを入れる。
    @commands.is_owner()
    @commands.command()
    async def reload(self, ctx, module_name):
        await ctx.send(f"モジュール{module_name}の再読み込みを開始します。")
        try:
            self.bot.reload_extension(module_name)
            await ctx.send(f"モジュール{module_name}の再読み込みを終了しました。")
        except (commands.errors.ExtensionNotLoaded, commands.errors.ExtensionNotFound,
                commands.errors.NoEntryPointError, commands.errors.ExtensionFailed) as e:
            await ctx.send(f"モジュール{module_name}の再読み込みに失敗しました。理由：{e}")
            return
```

　上記の`reload`コマンドを使えば、[@fig:sc_reload_extension]のようにExtensionの再読み込みがコマンドで行えます。Bot開発中にも便利ですね。

![reloadによるExtensionの再読み込み 動作イメージ](resource/image/sc_reload_extension.png){#fig:sc_reload_extension}

### あいさつコマンドをCogに切り出す
　ここでは、[@sec:medium]で実装した「あいさつコマンド」をCogにしてみましょう。また、モジュールはExtensionにしてみましょう。ここで作成するファイルと、コーディングする内容のまとめを[@fig:cog_extension_sample]に示します。この図を参照しつつ実践してください。

![CogとExtensionの実装と利用 概要図](resource/image/cog_extension_sample.png){#fig:cog_extension_sample}

#### cogsフォルダーを作成する
　discord.pyを利用したDiscord Bot開発では、慣例としてCogモジュールを`cogs`というフォルダーに格納します。discord.pyの開発リーダーDanny氏が利用例として公開しているBot「RoboDanny」でもこのとおりのフォルダー構造となっています。今回はこれに倣います。「cogs」という名前のフォルダーを作成しましょう。

#### Greet Cogを作成する
　フォルダーが作成できたら、そのフォルダー中にモジュール（Pythonファイル）を作成します。Visual Studio Codeでファイルを新規作成し、`cogs`フォルダー内に`greet.py`というファイル名を付けて保存します。ここまでのフォルダーの構成は[@lst:cogs_folder_structure]のようになります。

```{#lst:cogs_folder_structure .shell caption="フォルダー構成"}
my_bot
│  .gitignore
│  config.py
│  launcher.py
│  requirements.txt
├─.venv
└─cogs
     greet.py # いま作成したGreet Cog用ファイル
```

　ファイルが作成できたら、Cogとしての枠組みをコーディングしていきましょう。[@lst:greet_cog_frame]をごらんください。

```{#lst:greet_cog_frame .python caption="Greet Cog（処理実装前）"}
from discord.ext import commands

class Greet(commands.Cog):
    def __init__(self, bot):
        self.bot = bot

def setup(bot):
    bot.add_cog(Greet(bot))
```

　これが、CogとExtensionに最低限必要なコードです。このコードについて、順番に内容を確認していきましょう。

```{#lst:greet_cog_frame_classdef .python caption="Greetクラス定義"}
from discord.ext import commands

class Greet(commands.Cog):
```

　Cogはcommandsフレームワークの一部なので、`commands`モジュールをインポートします。つづいて、`commands.Cog`クラスを**継承**して`Greet`クラスを定義しています。継承とは、もともとのクラス（`commands.Cog`）の性質や機能を引き継ぎつつ、一部を拡張・変更したあたらしいクラスを作ることをいいます。

```{#lst:greet_cog_init .python caption="init関数の定義"}
    def __init__(self, bot):
        self.bot = bot
```

　`__init__`は、そのクラスのインスタンスが生成された後に呼び出される関数です。`self`は生成されたGreetインスタンスを指す引数です。Cogクラスを定義する際には2つ目の引数として、Bot型の引数をとるよう定義することが多いです。受け取った`bot`変数は、`self.bot`に格納しておきます。

```{#lst:greet_cog_setup .python caption="Extensionにするためのsetup関数定義"}
def setup(bot):
    bot.add_cog(Greet(bot))
```

　前項で解説した、モジュールをExtensionにするための関数定義です。`setup`関数内で`Greet`インスタンスを作成し、`bot`にcogとして追加しています。このように書いておくことで、BotにこのExtensionをインストールすると、BotにGreet Cogが追加されるようになります。

#### あいさつコマンドの定義をお引越し
　まず、launcher.pyからあいさつコマンドの定義を削除します。[@lst:remove_greet_command_from_main]のようにすべてコメントアウトしてもよいですが、Gitを使えば前のバージョンに戻すことができますので、削除してしまって問題ないでしょう。

```{#lst:remove_greet_command_from_main .python caption="あいさつコマンド定義を削除"}
# @bot.command()
# async def hello(ctx):
#     ...
#     await ctx.send(embed=embed)
```

　つづいて、さきほど作成したgreet.pyにあいさつコマンドの定義を追加します。[@lst:greet_cog_hello]をごらんください。ほとんどがlauncher.pyからのコピー＆ペーストで問題ないのですが、3点だけ修正する必要があります。変更点には★と変更内容を記していますので、参考にして修正してください。

```{#lst:greet_cog_hello .python caption="Greet Cog（処理実装後）"}
from discord.ext import commands
# 以下2点のimportも、launcher.pyから持ってくること
import discord
import asyncio

class Greet(commands.Cog):
    def __init__(self, bot):
        self.bot = bot

    
    @commands.command()  # ★変更点1: bot -> commandsに修正
    async def hello(self, ctx):  # ★変更点2: self引数を追加する
        # 待機するメッセージのチェック関数
        def check_message_author(msg):
            return msg.author is ctx.author
        # あいさつする既存の処理
        await ctx.send(f"こんにちは、{ctx.author.name}さん。")
        await ctx.send("ご気分はいかがでしょうか？")
        try:
            # チェック関数に合格するようなメッセージを待つ
            # ★変更点3: bot -> self.botに修正
            msg = await self.bot.wait_for('message', check=check_message_author, timeout=10)
        except asyncio.TimeoutError:
            await ctx.send("タイムアウトしました。")
            return
        # 受け取ったメッセージの内容を使って返信
        embed = discord.Embed()
        embed.color = discord.Color.blue()
        embed.description = "あなたの気分を把握しました。"
        embed.add_field(name="あなたの気分", value=msg.content)
        await ctx.send(embed=embed)

def setup(bot):
    bot.add_cog(Greet(bot))
```

　これでGreet Cogの実装、およびExtension化は完了です。

#### CogをBotに追加する
　最後に、BotにこのExtensionをインストールするコードを書きましょう。launcher.pyに、[@lst:install_greet_cog]のように追記します。

```{#lst:install_greet_cog .python caption="Greet Extensionをbotに追加する"}
bot.load_extension("cogs.greet")  # この行を追加
bot.run(config.TOKEN)
```

　`load_extension`で読み込むモジュールは、Pythonの`import`文でインポートするときと同じように指定します。つまり、`cogs/greet`のようにスラッシュで区切ったり、`greet.py`のように`.py`を付けることはせず、`cogs.greet`と書きます。
　これであいさつコマンドをCogに分け、Extensionにする作業は完了です。動作確認をしてみましょう。ターミナルを開き、仮想環境を有効化して`$ python launcher.py`を実行し、Botを起動します。テスト用Guildのテキストチャンネルに`!hello`と投稿し、Cogに分ける前と動作が変わっていないことを確認しましょう。もしあなたがすでにBotにさまざまな機能を追加してるなら、それらもCogに分けてみるといいかもしれません。

_Bot開発のタネ　さまざまなCog、Extensionを参考にしてみよう

　ここまでに、Cog、Extensionの作り方について学びました。とはいえ、どんなCogを作ればいいのかイメージがつかないかもしれません。ここでは、discord.pyのCogを扱う公開プロジェクトを紹介します。どのコードも整理されており、参考になるものばかりですので、ぜひ設計のお手本にしてみてください。

#### RoboDanny
　RoboDannyは、discord.pyの開発リーダーであるDanny氏が開発・運営しているBotです。cogsフォルダー内に約20個の多種多様なCogのコードがあります。このBotはCog以外にもBot設計の参考になります。「こういうときどうすればいいんだろう？」という疑問を持ったときにソースコードを見ると、開発の強い味方になってくれます。
　https://github.com/Rapptz/RoboDanny

#### dispander
　dispanderは、Discord Bot Portal JPによって公開されているExtensionです。テキストチャンネルにDiscordのメッセージURLが投稿されたら、その内容をEmbedとして展開し、表示してくれる機能をBotに追加します。CogとExtensionの実装、Embedの使い方について非常に参考になりますので、コードを読んでみるとよいでしょう。もちろん、この便利なExtensionをあなたの「おたすけBot」に追加することもできます。
　https://github.com/DiscordBotPortalJP/dispander

## Cogについて詳しく知ろう
　前節ではCogとExtensionの概要について学び、簡単なCogの実装に挑戦しました。ここでは、Cogについてより深く学びましょう。

### Cogにイベントリスナーを定義する
　前節ではCogにコマンドを定義しました。ここでは、Cogにイベントリスナーを定義する方法をお伝えします。とはいえ、Botインスタンスにイベントリスナーを定義したときとほとんど変わりません。[@lst:cog_event_listener]をごらんください。

```{#lst:cog_event_listener .python caption="Greet Cogにイベントリスナーを追加する"}
...
    @commands.Cog.listener()
    async def on_message(self, message):
        if message.author == self.bot.user:
            return
        if message.content.startswith("こんにちは"):
            await message.channel.send(f"こんにちは、{message.author.name}さん！")
```

　`on_message`関数の内容には新しい内容はありません。注意するべき点は2つあり、デコレーターが`@commands.Cog.listener()`になる点と、`on_message`イベントリスナーを定義しても`process_commands`の呼び出しが不要である点です。

### Cog内で発生したエラーの処理を定義する
　コルーチン関数`cog_command_error`を定義すると、そのCog内のコマンドを実行した際に発生したエラーに対する処理を一括で実装できます。[@lst:cog_command_error_example]は、エラーが発生したらエラー内容をテキストチャンネルに通知する実装例です。

```{#lst:cog_command_error_example .python caption="cog_command_error 実装例"}
    async def cog_command_error(self, ctx, error):
        await ctx.send(f"エラーが発生しました。\n{error}")
```

### Cog単位の実行権限を付ける
　`cog_check`を使うと、そのCogにあるすべてのコマンドの使用可否をまとめて設定できます。[@lst:cog_cog_check]をごらんください。

```{#lst:cog_cog_check .python caption="Greet Cogにcog_checkを追加する"}
...
    async def cog_check(self, ctx):
        return ctx.author.name == "name"
```

　このように記述すると、`name`という名前のユーザーのみ、Cog内のコマンドを実行できるようになります。ただし、Discordではユーザーネームは簡単に変えられるため、このようなチェックに意味はありません。実際にコマンドの実行者を絞り込みたいときは、ユーザーIDや権限で絞り込むようにしましょう。

## Commandをもっと便利に使おう
　ここでは、[@sec:medium]で少し触れたCommandについて、より発展的な使い方について見ていきましょう。

### パラメーターとは

　コマンドは**パラメーター**を持つことができます。[@fig:sc_command_parameter]をごらんください。

![コマンド文字列とパラメーター](resource/image/sc_command_parameter.png){#fig:sc_command_parameter}

　コマンド文字列につづいて、半角スペースの後に「3」という文字列があります。このような文字列をさしてパラメーターといいます。discord.pyには、パラメーターをとるコマンドを定義するための便利な仕組みが用意されています。実際に使ってみましょう。

### パラメーターを持つコマンドを実装する
　Greet Cogに、複数回あいさつするコマンドを実装してみましょう。あいさつの回数はユーザーから指示してもらうことにします。[@lst:greet_multi]の内容を実装してみましょう。

```{#lst:greet_multi .python caption="複数回あいさつするコマンド"}
    @commands.command()
    async def hello_multi(self, ctx, count: int):
        for i in range(count):
            await ctx.send(f"こんにちは、{ctx.author.name}さん。")
```

　hello_multiの引数に注目してください。`self`、`context`までは`hello`コマンドと同じです。3つ目以降の引数はそのコマンドのパラメーターを表します。この場合、`count`がパラメーターとなります。ターミナルを開き、仮想環境を有効化して`$ python launcher.py`を実行し、Botを起動します。つづいて、[@fig:sc_greet_multi]のようにコマンドを実行してみましょう。

![パラメーターを持つコマンド 実行例](resource/image/sc_greet_multi.png){#fig:sc_greet_multi}


### Converterについて知る
#### Pythonのアノテーション
　[@lst:greet_multi]の関数宣言に注目すると、`count :int`のように、変数名のすぐ後ろに`:int`という`:<型名>`の記述があることに気づきます。これはPythonにおいて**アノテーション**と呼ばれるものです。通常、文法上の効力はないものの、「この関数にはこの型の変数を入れてくださいね」とコーディングする人に伝えるための記法です。discord.pyでは、このアノテーションをcommandsフレームワークの仕組みのひとつ、**Converter（コンバーター）**に利用しています。

#### Converterとは
　Converterは、ユーザーから受け取った文字列をコマンド処理に必要な型へと変換するための仕組みです。Converterは、コマンド文字列以降のパラメーターをアノテーションで指定した型に変換します。変換できなかった場合は`commands.errors.BadArgument`例外、パラメーターの数がコマンドの定義と合わない場合は`commands.errors.MissingRequiredArgument`例外が発生します。

#### Converterを使うメリット
　Converterを使わない場合、たとえば`int(count)`のように、自力でstrからintへの変換を行う必要があります。この場合、コマンドの本処理と前処理の判別がつきにくいうえに、変換時にエラーが起きた際の処理も実装する必要があり、コードが複雑になりがちです。Converterを使えば、アノテーションとして型を指定しておくことでCommandsフレームワークが自動で指定した型に変換してくれるのです。変換ができなかった場合や、コマンドの定義とパラメーターの数が一致しなかった場合はエラー処理に回されます。
　まとめるとConverterは、

1. ユーザーの入力からパラメーターを取り出す・型変換する
2. パラメーターに異常がある場合の処理

　という、**コマンド処理に必要だが、やりたい処理そのものには関係ない**部分を切り離してくれます。おかげで、コマンド処理についてコーディングする際に処理の本質に集中できますし、エラー処理についてコーディングする際にはエラー処理に集中できます。加えて、コード自体も見通しよくすっきりとします。

#### コンバーターの種類
　コンバーターには、さきほど使用した`int`などに変換するBasic Converters、関数やクラスとして定義するAdvanced Conveters、テキストチャンネルやメンバーなどに変換するDiscord Convetersなどが存在します。Converterについては奥が深いので、公式ドキュメント「Converter」をごらんください。
　https://discordpy.readthedocs.io/en/latest/ext/commands/commands.html#converters

### コマンドのエラー処理
　`@<コマンド名>.error`というデコレーターを関数に付与すると、そのコマンドを処理中にエラーが発生した際の処理を行う関数を定義できます。

```{#lst:hello_multi_error .python caption="コマンドのエラー処理"}
    @hello_multi.error
    async def hello_multi_error(self, ctx, error):
        if isinstance(error, commands.errors.BadArgument):
            await ctx.send("パラメーターの形式が違います")
        if isinstance(error, commands.errors.MissingRequiredArgument):
            await ctx.send("パラメーターの数が足りません")
```

　コマンドの実行中には、さまざまなエラーが発生します。そこでこの関数では、エラー変数`error`がなんの型であるかを確認し、そのエラーにふさわしいエラーメッセージを送る処理を行っています。
　コマンドの処理中に起きうるエラーの一覧は、下記URLにて確認できます。
　`https://discordpy.readthedocs.io/en/latest/ext/commands/api.html#ext-commands-api-errors`

## Botに定周期で処理をさせよう - Task
### Taskってなに？
　`discord.ext.tasks`は、なんらかの処理を一定間隔で繰り返すという、Bot作成によくある処理の実装を助けるモジュールです。インターネットからの切断やインターバルの秒数の最大値などについて気にすることなく、定周期処理を手軽に実装できます。Taskについて学ぶため、またCogについて復習するため、時報機能を提供する**Notify Cog**を作成してみましょう。Botにインストールすると、10分に1度、テキストチャンネルへ時報メッセージを送信する機能を実装します。また、時報を送信するテキストチャンネルを指定するコマンドも実装しましょう。

### Taskの動作確認
　機能を実装するまえに、まずはCogを作成し、Taskがうまく動くか動作確認してみましょう。

#### Notify Cogの実装
　まずはNotify Cogのモジュールを作成します。Visual Studio Codeでファイルを新規作成し、`cogs`フォルダー内に`notify.py`というファイル名を付けて保存します。ここまでで作成したフォルダーの構成は[@lst:cogs_folder_structure]のようになります。

```{#lst:cogs_folder_structure_notify .shell caption="フォルダー構成"}
my_bot
│  .gitignore
│  config.py
│  launcher.py
│  requirements.txt
├─.venv
└─cogs
     greet.py
     notify.py # いま作成したNotify Cog用ファイル
```

　まずはコード全体を掲載します。[@lst:notify_cog_entire]のとおりに実装してください。

```{#lst:notify_cog_entire .python caption="Notify Cog 動作確認用コード 全体像"}
from discord.ext import commands
from discord.ext import tasks


class Notify(commands.Cog):
    def __init__(self, bot):
        self.bot = bot
        # 時報タスクの開始
        self.notifier.start()

    def cog_unload(self):
        # 時報タスクの中断
        self.notifier.cancel()

    @tasks.loop(seconds=1.0)
    async def notifier(self):
        print("start_notifier")


def setup(bot):
    bot.add_cog(Notify(bot))
```

　Taskに関連する処理について、順番に解説します。

```{#lst:first_task_import .python caption="taskモジュールのインポート"}
from discord.ext import tasks
```

　Taskの利用には、`discord.ext.tasks`モジュールが必要なので、import文を追加します。

```{#lst:first_task_loop_frame .python caption="1秒おきに文字列を出力するTask"}
    @tasks.loop(seconds=1.0)
    async def notifier(self):
        print("start_notifier")
```

　[@lst:first_task_loop_frame]は、1秒おきに`start_notifier`と出力するTaskの実装例です。`@tasks.loop`デコレータは続く関数の処理を実施する`Loop`オブジェクトを作成します。`@tasks.loop`デコレータには、インターバル時間（今回は`seconds=1.0`、つまり1秒）を指定します。

```{#lst:task_start_cancel .python caption="Taskの開始、終了"}
    def __init__(self, bot):
        self.bot = bot
        # 時報Taskの開始
        self.notifier.start()

    def cog_unload(self):
        # 時報Taskの中断
        self.notifier.cancel()
```

　定義したTaskを開始・キャンセルするための処理です。`__init__`での初期化と同時に`start`を呼び出すことでインスタンスの生成時にTaskを開始します。`cog_unload`はBotの終了やCogの除去など、そのCogが動作を停止する際に呼び出される関数です。この中で`cancel`を呼び出すことで、Botの停止前にTaskを中断しています。

#### CogをBotに追加する
　BotにNotify Cogをインストールするコードを書きましょう。launcher.pyに、[@lst:install_notify_cog]のように追記します。

```{#lst:install_notify_cog .python caption="Notify Extensionをbotに追加する"}
bot.load_extension("cogs.greet")
bot.load_extension("cogs.notify") # この行を追加
bot.run(config.TOKEN)
```

#### 動作確認
　ここまで実装できたら、動作確認をしてみましょう。Botを実行してみます。

```{#lst:first_task_execute .shell caption="はじめてのtasks 実行"}
(.venv) $ python launcher.py
start_notifier
start_notifier
start_notifier
...
```

　[@lst:first_task_execute]のように、1秒おきに`start_notifier`の文字列が出力されていれば成功です。

### 時報機能の実装
　Taskが無事動作することを確認できました。つづいて時報通知を実装します。

#### 時報通知先設定コマンドを実装する
　ユーザーが「!set_notify_channel」というコマンドを実行したら、それ移行そのチャンネルに時報を送信するようにします。まずは、「時報通知先チャンネル」の情報の初期値を設定します。[@lst:set_channel_init]を参考に、`__init__`関数へ追記しましょう。

```{#lst:set_channel_init .python caption="初期化関数で時報通知先を「なし」にしておく"}
    def __init__(self, bot):
        self.bot = bot
        # 時報Taskの開始
        self.notifier.start()
        # 時報通知先チャンネル
        self.channel = None
```

　つぎに、コマンドを実装します。[@lst:set_channel_command]のコマンド定義をNotifyクラスに追加します。

```{#lst:set_channel_command .python caption="時報通知先チャンネル設定コマンド"}
    @commands.command()
    async def set_notify_channel(self, ctx):
        # 通知先テキストチャンネルを保持
        self.channel = ctx.channel
        # 登録完了を通知
        await ctx.send(f"「{ctx.channel.name}」に時報を通知します！")
```

　ここまでで一度、動作確認をしましょう。ターミナルを開き、仮想環境を有効化して`$ python launcher.py`を実行し、Botを起動します。Discordのテキストチャンネルに「!set_notify_channel」と投稿し、[@fig:sc_set_channel]のように応答が来たら成功です。

![時報通知先チャンネル設定コマンド 実行例](resource/image/sc_set_channel.png){#fig:sc_set_channel}

#### 時報通知処理を実装する
　つづいて時報を出す処理を実装します。まず[@lst:add_import_datetime]のとおりに時報機能の実装に必要なモジュールのimportを追加します。

```{#lst:add_import_datetime .python caption="必要なimportの追加"}
from discord.ext import commands
from discord.ext import tasks
from datetime import datetime  # 追加
```

　`notifier`の内容とデコレーターの指定を、[@lst:time_notify_task]のとおり変更します。

```{#lst:time_notify_task .python caption="時報通知処理本体 コード"}
    @tasks.loop(minutes=10.0)
    async def notifier(self):
        print("start_notifier")
        now = datetime.now()
        # 通知先が登録されていれば通知
        if self.channel:
            await self.channel.send(
                f"現在、{now.strftime('%Y/%m/%d %H:%M:%S')}です")
```

　`task.loop`デコレーターのパラメーターを`minutes=10.0`、つまり実行周期10分に変更していますが、動作確認をするために毎回10分待つのは大変です。開発の最中は、5秒周期（`seconds=5.0`）など確認しやすい周期にしておくとよいでしょう。
　動作確認してみます。ターミナルを開き、仮想環境を有効化して`$ python launcher.py`を実行し、Botを起動します。[@fig:sc_notify_task]のようにコマンドを実行したあとしばらく待ち、無事時報が表示されたら成功です！

![時報通知機能 実行例](resource/image/sc_notify_task.png){#fig:sc_notify_task}


### Taskについて詳しく知る
#### 時分秒で時間指定　
　時報機能の実装では、[@lst:tasks_ten_minutes]のように「10分間隔」でtaskの実行周期を指定しました。

```{#lst:tasks_ten_minutes .python caption="10分間隔で実行するコード例"}
@tasks.loop(minutes=10.0)
```

　この実行周期は、`hours`, `minutes`, `seconds`という3つの組み合わせで指定可能です。いくつかの例を挙げてみましょう。[@lst:tasks_hms_example]をごらんください。

```{#lst:tasks_hms_example .python caption="さまざまな実行間隔指定"}
@tasks.loop(seconds=1.0) # 1秒おきに実行
@tasks.loop(minutes=5, seconds=30) # 5分30秒おきに実行
@tasks.loop(minutes=5.5) # 5分30秒おきに実行（5分30秒=5.5分として表現）
@tasks.loop(minutes=30) # 30分おきに実行
@tasks.loop(hours=0.5) # 30分おきに実行（30分=0.5時間として表現）
```

　なんとなくイメージが掴めたでしょうか。`minutes`や`seconds`などの引数は併用でき、それぞれの数値を足し合わせた数が答えとなります。また、それぞれの値は`float`型なので、小数を使用できます。

#### ループをはじめる前、終わった後に処理を行う
　処理のループがはじまる前や、ループが終了した後に何か処理をすることもできます。`@<task関数名>.before_loop`というデコレーターを付けたコルーチン関数を定義すると、処理のループがはじまる前に実施する処理を定義できます。これは、Botの準備が完了するまで処理のループへ入りたくない場合などに便利です。同じく、`@<task関数名>.after_loop`というデコレーターを付けたコルーチン関数を定義すると、処理のループが終わった後に実施する処理を定義できます。`before_loop`で確保したリソースの開放が必要な場合などに便利です。
　説明だけではわかりにくいので、実装してみましょう。Notify Cogの`notifier`関数定義のすぐ後ろに、[@lst:before_after_loop]の内容を追加します。

```{#lst:before_after_loop .python caption="before_loop, after_loop コード例"}
@notifier.before_loop
async def before_check(self):
    # Botの準備完了まで待機
    await self.bot.wait_until_ready()
    print("start")


@notifier.after_loop
async def after_check(self):
    print("end")
```

　`bot.wait_until_ready()`関数を呼び出すと、BotがDiscord APIサーバーに接続し、必要なデータ取得を完了するまで処理が止まります。処理のループは`before_check`の処理が完了するまで始まらないため、処理のループの開始タイミングをコントロールできます。

#### 回数を限定して処理を行うこともできる
　ここまでで紹介したのは、Botを停止するまでずっと繰り返すループ処理でしたが、回数を限定してループ処理することもできます。[@lst:limited_loop_task]をごらんください。

```{#lst:limited_loop_task .python caption="ループの回数を限定する"}
@tasks.loop(minutes=10.0, count=5)
async def notifier():
    ...
```

　`@tasks.loop`デコレーターにて`count`引数に5を指定していますので、`notifier`関数が5回実行されます。出力は[@lst:limited_loop_task_result]のようになります。

```{#lst:limited_loop_task_result .python caption="ループ回数制限 出力例"}
(.venv) $ python launcher.py
start
start_notifier
on_ready
start_notifier
start_notifier
start_notifier
start_notifier
end
```

　ちょうど5回の実行でループが止まり、`after_loop`に指定された関数が実行され、`end`が出力されています。

_Bot開発のタネ　定時処理をするBotのアイデア

　Taskをマスターすると、実装できるBotの幅が広がります。ここでは、Taskを使ってどんな機能を実装できるかアイデアを出してみましょう。

#### 天気予報機能
　一日に一度、朝の決まった時間に天気予報を投稿する機能です。OpenWeatherMapなど、無料で天気予報を取得できるAPIサービスを利用して天気予報を取得し、テキストチャンネルに投稿します。予報する対象の地域を登録するコマンドを実装すれば、毎朝天気予報をチェックする手間がなくなりますね。

#### オンラインゲーム用お知らせ機能
　オンラインゲームの中に、毎日やるべきこと・毎週やるべきことがあるタイトルは多いです。「毎日やるべきこと」は毎日のことなので忘れにくいですが、「毎週やるべきこと」は先送りにしているうちについつい忘れてしまうこともあります。なので、毎週日曜日の朝に週の切り替わりがあるゲームであれば、

- 日曜日の朝に「週が変わりました！週課を早めに済ませてしまいましょう！」という投稿をする
- 土曜日の夜に「もうすぐ週が変わります。週課は済ませましたか？」という投稿をする

　こんな投稿をしてくれるBotを作れば、そのゲームのプレイヤーに使ってもらえるかもしれません。ゲームによっては、「1日と15日に更新」といった複雑なスケジュールのコンテンツがあったりもするので、そのような通知があるとさらに便利でしょうね。

#### リマインド機能
　ユーザーが登録した予定を通知してくれる機能も便利でしょう。たとえば、「今日の21時に集合」という予定を登録すると、1時間前、5分前に通知してくれるといった具合です。自分ひとりの予定であればスケジュール管理アプリやスマートフォンの標準ツールで事足ります。ですがBotにこの機能があれば、サーバーに参加しているメンバー全員にまとめてメンションを送ることもできます。Role（役職）機能と併用すれば、「サーバー内のこのRoleを持つ人だけにメンションする」といった応用もできます。
