　Botを動かすのに必要な環境について説明します。また、Botを開発するために必要なアプリケーションなどをインストールする手順を紹介します。

## Botの開発環境について知ろう
### 最小構成 - 最低限の動作環境
　Botは、最低限**インターネットに接続されたPC**、**ソースコード**、**Python**があれば動作します。ですので、あなたがいつも使っているPCにPythonをインストールして、メモ帳などでソースコードを書き上げれば、すぐにでもBotを動かすこと自体は可能です。

![Botの最小構成](resource/image/minimum_bot_environment.png){#fig:minimum_bot_environment}

　ですが、コードを編集するのが「メモ帳」では心もとないです（もちろん、不可能ではありません）。継続的に開発を行っていくためには、もう少し高度な環境を用意したいところです。

### PC環境 - 現実的な最小構成
　実際にBotを開発していくのであれば、ソースコードを編集する**コードエディター**や、ソースコードのバージョン管理を行うための**Git**などがあったほうが便利です。Windowsであれば、Gitに付属している**Git Bash**は継続的なBot開発に必須のツールです。本書ではこの環境を使ってBotを開発します。

![PC環境](resource/image/local_bot_environment.png){#fig:local_bot_environment}

　この環境では、あなたのPCの電源を切るとBotは稼働を止めてしまいます。とはいえ、PC環境は開発には十分な環境で、お金もかかりません。友人同士で共有するBotを運営するぶんには、この環境でも十分でしょう。

### サーバー環境 - 実用的な構成
　何十万人ものユーザーに使ってもらうことを目指して本格的なBot運営を行う際には、あなたのPCではなく別のPC、つまり**サーバー上**でBotを稼働させるとよいでしょう。「Python」および「ソースコード」をサーバー上に配置し、それらを操作するためにはPCから**SSHクライアント**を使ってSSH接続を行います。

![サーバー開発環境 動作イメージ](resource/image/server_bot_environment.png){#fig:server_bot_environment}

　サーバー環境では、Botのデータを保存するための**データベース**や、Botの設定をWebインターフェイスで行うための**Webサーバー**などのミドルウェアを利用することもできます。Botが稼働するのはサーバー上なので、あなたのPCをシャットダウンしてもBotは稼働し続けます。
　また、ここで「サーバー」とひとまとめにして呼んでいますが、サーバーとして使用できるマシンにはいくつかの選択肢があります。お金を払ってレンタルサービスのサーバーを使用することもできますし、Raspberry-piなどの小型コンピューターを自宅で稼働させてサーバーとして利用することもできます。

### どの環境を使えばいいの？
　ここまでに3つの環境（最小構成、PC環境、サーバー環境）を紹介しました。3つの環境を比較してみましょう。

![3つの環境の比較](resource/image/compare_three_environment.png){#fig:compare_three_environment}

　機能だけに注目すると、サーバー環境がもっとも優れていることに間違いはありません。しかし、サーバー環境を構築するにはそれなりの知識と手順が必要であり、セキュリティにも気を配る必要があります。いっぽうで、PC環境は簡単なインストールで構築が完了し、お金もかかりません。
　せっかく開発をするぞ！と意気込んでいるのに、環境構築で力尽きてしまうのはもったいない話です。しかし実はこれ、よくあることなのです。本書のチュートリアルはすべての手順を「PC環境」で実践できますので、Windows、macOSのそれぞれについて、「PC環境」の構築手順を詳細に紹介します。はじめての方は無理せず「PC環境」から構築し、不特定多数に向けた本格的なBot運営をしたくなったときに「サーバー環境」への移行を検討するとよいでしょう。

## 環境を構築しよう
### 基礎用語の紹介
　この先のチュートリアルで登場する用語を説明します。

#### ターミナル
　本書では、開発をするときによく出てくる文字だらけの画面、いわゆる「黒い画面」をターミナルと呼びます。**コマンドプロンプト**(Windows)、**ターミナル**（macOS）、**端末**などと呼ばれます。ターミナルとして、Windowsでは**Git Bash**、macOSでは**ターミナル**を使用します。「ターミナルを開く」「ターミナルで実行する」と指定があった場合、これを開いてください。

#### プロンプト
　ターミナルを開くと、[@lst:terminal_prompt_example]のような「プロンプト」が現れます。

```{#lst:terminal_prompt_example .shell caption="プロンプトの例"}
cod@cods-MacBook-Pro ~ $  # macOSの例　記号が「%」のこともある
cod@cods-Windows MINGW64 ~/Desktop $ # Windows(Git Bash)の例
```

　開発においては「ターミナルで`$ command`を実行する」といった指示がよく登場します。ここで、`$`はプロンプトを表しますので、入力しないでください。この指示の場合、ただ`command`と入力し、`<Enter>`を押せばOKです。

### インストールするものの一覧
　それでは、「PC環境」を構築しましょう。まずはインストールすべきものの一覧を確認します。[@fig:pc_environment_table]をごらんください。

![PC環境 概要](resource/image/pc_environment_table.png){#fig:pc_environment_table}

### 必要なPC性能スペックは？
　Bot開発をはじめるだけなら、あなたの手元でネットを見るのに使っているPCがあれば、たいていはそれを使って開発を始められるでしょう。PythonおよびGitについては昨今のPC（Windows 7以降のOS）であれば問題なく動作します。Visual Studio Codeについては、公式サイトによると「1.6GHz以上のCPU」「1GB以上のメモリ」という要件があります（https://code.visualstudio.com/docs/supporting/requirements）。昨今のPCであればたいていはこの条件も満たしているでしょう。
　まず、「Git」「Python」のインストールを行います。この手順はWindowsとmacOSで異なるので、お使いのOSにあわせて参照してください。次に、Visual Studio Codeと必要なプラグインをインストールします。これはOS共通の手順です。

### うまくいかないときのチェックポイント
　Windowsにおいては、インストーラーやコードエディターを管理者権限で実行するとうまくいくことがあります。管理者権限で実行するには、アイコンを右クリックし、「管理者として実行」を選択します。
　また、macOSにおいて、うまくインストーラーを開けないことがあります。その場合、ダブルクリックではなく、右クリックから「開く」を選択してください。

### PythonとGitのインストール - Windows編
　ここでは、Windows向けのインストール手順を紹介します。macOSをご利用の方は、次項「macOS」をごらんください。

#### Gitのインストール
　Gitをインストールします。Git公式サイト（https://git-scm.com/）にアクセスし、「Download 2.25.0 for Windows」をクリックしましょう（[@fig:sc_installer_git]）。「2.25.0」は2020年2月時点のバージョンです。多少この部分の数字が違っても構いません。

![Gitインストーラー ダウンロード](resource/image/sc_installer_git.png){#fig:sc_installer_git}

　インストーラーがダウンロードできたら、インストーラーを起動します。インストーラーの起動後は、画面の表示にしたがってインストール作業をすすめます。基本的にはデフォルトのままの設定で「Next」を押していけばよいのですが、「Configuring the line ending conversions」という項目（[@fig:sc_git_setting_about_newline]）のみ変更の必要があります。

![改行コードについての設定](resource/image/sc_git_setting_about_newline.png){#fig:sc_git_setting_about_newline}

　これについては、[@fig:sc_git_setting_about_newline]のとおりに、「Checkout as-is, Commit as-is」にしておきましょう。これはGitがテキストファイルの改行コードの変換を自動で行うかどうかについての設定ですが、たいていの場合自動変換は不要です。先述の選択肢にしておけばGitが改行コードの変換を行いません。また途中、使用するエディターについての設定項目もありますが、本チュートリアルではGit経由でエディターを起動することはありませんので、どのエディターを選んでも問題ありません。デフォルトのままでも構いません。

#### Pythonのインストール
　つづいて、Pythonをインストールします。discord.pyは、その動作に3.5.3以上のPythonが必要です。すでにPython3.5.3以上がインストールされている場合、下記手順は不要です。バージョンは好きなものを選んで構いませんが、Stable Releases（安定版）から選ぶことでスムーズに開発を進めやすくなります。

1. Python公式サイト ダウンロードページ（https://www.python.org/downloads/windows/）にアクセスする。
2. 3.5.3以上の任意のバージョンを選択し、「Download Windows x86 executable installer」をクリックする。
3. ダウンロードしたインストーラーを起動し、「Add Python 3.x to PATH」にチェックを入れ、「Install Now」をクリックする。
4. 画面の表示にしたがってインストールする。

### PythonとGitのインストール - macOS編
#### HomeBrewのインストール
　HomeBrewは、主にmacOSで使用されるパッケージマネージャーです。まずはあなたのPCにHomeBrewがインストール済みかを確認しましょう。
　ターミナルで、`$ brew help`を実行してください。ここでHomeBrewのヘルプが表示された場合、HomeBrewはすでにインストール済みです。`brew update`でHomeBrewを最新にしておき、次項「Gitのインストール」に移ってください。`command not found`と表示されるた場合、HomeBrewがインストールされていません。

![HomeBrewの公式サイト(2020年2月現在)](resource/image/sc_install_homebrew.png){#fig:sc_install_homebrew}

　ブラウザでHomeBrew公式サイト（https://brew.sh/）にアクセスします。「Install HomeBrew」という文字列の下に記されているコマンド（[@fig:sc_install_homebrew]）をコピーし、ターミナルに貼り付け、実行します。途中、`Press RETURN to continue or any other key to abort`という表示が出るので、内容を確認してEnterキーを押します。`Installation successful!`という表示が出たらインストール完了です。

#### Gitのインストール
　次に、Gitのインストールです。HomeBrewのときと同じく、まずはインストール状況を確認しましょう。
　ターミナルで`$ git --version`を実行してください。`command not found`と表示された場合、Gitがインストールされていないので、ターミナルで次の手順を実施してください。また、表示されたバージョンより新しいバージョンをインストールしたい場合も、下記手順を実施してください。

1. `$ brew install git`を実行する。
2. `$ brew list`を実行し、出力に`git`が表示されていることを確認する。
3. ターミナルを再起動する。
4. `$ git --version`を実行し、意図したバージョンが実行されていることを確認する。

#### Pythonのインストール
　つづいて、Pythonをインストールします。discord.pyは、その動作に「Python3.5.3」以降が必要です。すでにPython3.5.3以上がインストールされている場合、下記手順は不要です。インストールするには、ターミナルで下記手順を実行してください。

1. `$ brew install python3`を実行する。
2. `$ Pythonをbrew list`を実行し、出力に`python`が表示されていることを確認する。
3. ターミナルを再起動する。
4. `$ python3 --version`を実行し、3.5.3以上のPythonがインストールされていることを確認する。

### Visual Studio Codeのセットアップ
　コードエディターとして、Visual Studio Codeをインストールします。これ以降の手順は、WindowsとmacOSで操作は共通です。

#### インストール
　インストーラーをダウンロードし、インストールを行います。

1. 公式サイト ダウンロードページ（https://code.visualstudio.com/download）を開く。
2. OSにあったインストーラーを選択し、ダウンロードする。
3. ダウンロードしたインストーラーを開き、インストールを行う。

#### Visual Studio Codeの起動
　インストールしたVisual Studio Codeを起動してみましょう。Windowsであれば、画面左下のWindowsボタン（またはWindowsキー）を押し、「Visual Studio Code」と入力し、結果に出てきた「Visual Studio Code」をクリックします。macOSであれば、CommandキーとSpaceキーを同時押しし、「Visual Studio Code」と入力し、「Visual Studio Code.app」をクリックします。
　[@fig:sc_vscode_win]のようにVisual Studio Codeが起動すればOKです。

![Visual Studio Code（Windows版）](resource/image/sc_vscode_win.png){#fig:sc_vscode_win}

#### Python Extensionのインストール
　Pythonのコードを編集しやすくするため、Visual Studio CodeにPython Extensions（拡張機能）をインストールします。

1. Visual Studio Codeのメニューから「View」→「Extensions」を選択し、Extensionsを開く。
2. 画面上部、「Search Extensions in Marketplace」に「Python」と入力する。
3. 「Python」という名前のExtensionの「Install」ボタンを押し、インストールを行う。

## 動作確認をしよう
　ここまで作った環境が正しく動作していることを確認して、環境構築を終わりにしましょう。

### ターミナルの起動
　Visual Studio Codeにはターミナルと連携する機能があり、画面内でターミナルを開けます。複数ウィンドウを反復横跳びする必要がなくなり便利ですので、使ってみてください。画面内でターミナルを開くには、「Terminal」→「New Terminal」を選択します（[@fig:sc_vscode_open_terminal]）。

![ターミナルを開く](resource/image/sc_vscode_open_terminal.png){#fig:sc_vscode_open_terminal}

　画面下部にターミナル画面が開きます（[@fig:sc_vscode_terminal]）。

![Visual Studio Code内で開いたターミナル](resource/image/sc_vscode_terminal.png){#fig:sc_vscode_terminal}

　Windowsでは、コマンドプロンプトやGit Bashなどから、どのシェルをデフォルトとして開くかを設定できます。本書ではGit Bashを用いて開発を行いますので、これを設定しましょう。ターミナル右側のコンボボックスをクリックで開き、「Select Default Shell」をクリックします（[@fig:sc_select_shell]）。

![デフォルトシェル設定を開く](resource/image/sc_select_shell.png){#fig:sc_select_shell}

　画面上部に選択肢が表示されるので、「Git Bash」をクリックします（[@fig:sc_select_shell2]）。

![デフォルトシェルの選択](resource/image/sc_select_shell2.png){#fig:sc_select_shell2}

　これで「New Terminal」を実行したときにGit Bashが起動するようになりました。ターミナル表示部分の右上にあるゴミ箱マークを押して現在開いているターミナルを閉じ、「Terminal」→「New Terminal」で新しいターミナルを開きましょう。

### Pythonコマンドの動作確認
　Pythonの動作確認をします。上記の手順で開いたターミナルにて、[@lst:check_python_is_ok]の1行目の内容を実行します。2行目に示したように、インストールしたPythonのバージョン（Python3.5.3以上）が表示されていればOKです。繰り返しになりますが、`$`はプロンプトを表すのでターミナル上で入力する必要はありません。スペースもこのとおりに入力してください。

```{#lst:check_python_is_ok .shell caption="Pythonの動作確認"}
$ python --version
Python 3.7.3
```

　Windowsの場合、`python`ではなく`py`コマンドで動く場合もあります。また、`python3`や`py -3`のようにバージョンを指定するとうまくいく場合もあります。


### おつかれさまでした！
　これですべての環境構築が完了しました。次はBotを開発するのに必要となる、Botアカウントの作成を行います。準備が多くて退屈かもしれませんが、ひとつひとつ確認しながらゆっくり確実に進んでくださいね。


_Bot開発のタネ　開発者のコミュニティに参加しよう

　**Discord Bot Portal JP**は、discord.pyに限らないDiscordBotの開発者・利用者の情報共有コミュニティです。Bot開発者が多く参加しており、質問しやすい仕組みと雰囲気があります。参加するためのURLは下記のTwitterに掲載されていますので、ぜひ参加されることをおすすめします。
　https://twitter.com/discordbot_jp