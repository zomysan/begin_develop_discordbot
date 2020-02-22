　さて、ここまででプログラミングをするのに必要な準備は整いました。ここでは、Botの身体、Botアカウントを作成します。この章の作業はあなたのユーザーアカウントでDiscordにログインしたうえで実施してください。

## Botアカウントを作成しよう
　まずはBotの入る身体、**Bot用アカウント**を作成しましょう。ここでは2020年2月時点での登録手順を紹介しますが、本書の内容とDiscordに大きな齟齬があって手順がわからない場合、下記URLをご確認ください。
　https://cod-sushi.com/discord-py-token/

※ 操作は、都度表示される注意事項などを確認の上行いましょう。

### Applicationの作成
　Botアカウントは、**Application**に紐付いたDiscordアカウントとして作成できます。ですので、Botアカウントを作成するにあたって、まずはApplicationの情報を登録します。

　https://discordapp.com/developers/applications/ にアクセスし、「New Application」をクリックします（[@fig:sc_create_application]）。

![Applicationの作成](resource/image/sc_create_application.png){#fig:sc_create_application}

　任意のApplication名を入力し、「OK」を押します（[@fig:sc_application_name]）。ここで入力した名前はいつでも変更可能です。

![Application名を入力](resource/image/sc_application_name.png){#fig:sc_application_name}

### Botアカウントの作成
　次に、作成したアプリケーションに紐づくBotアカウントを作成します。まず「Bot」をクリックし、つづいて「Add Bot」をクリックします（[@fig:sc_add_bot]）。

![ApplicationにBotを追加](resource/image/sc_add_bot.png){#fig:sc_add_bot}

　Botの作成は取り消すことができない旨の説明が表示されますので、読んだ上で「Yes, do it!」をクリックします（[@fig:sc_yes_add_bot]）。

![Bot作成の確認画面](resource/image/sc_yes_add_bot.png){#fig:sc_yes_add_bot}

　アイコン、ユーザー名を好きなように変更します。また、デフォルトでオンになっている「PUBLIC BOT」はオフにしておきます。こうすることで、作成したBotを非公開にできます。設定を変更したら、「Save Changes」ボタンをクリックします（[@fig:sc_edit_bot_info]）。これらの設定もApplicationの名前と同様いつでも変更できます。

![Botの情報編集](resource/image/sc_edit_bot_info.png){#fig:sc_edit_bot_info}

　アイコンやユーザー名は、他者の権利を侵害しないようにしましょう。これでBotアカウントを作成できました！


### トークンの取得
　Botの編集をしたのと同じ画面で、そのままBotにログインする際必要な**トークン**の取得を行いましょう。「Copy」ボタンをクリックします（[@fig:sc_get_token]）。

![トークンの取得](resource/image/sc_get_token.png){#fig:sc_get_token}

　これでクリップボードにトークンがコピーされました。ちなみに、「Copy」ボタンの右隣にある「Regenerate」ボタンは、現在のトークンの無効化と新規トークンの生成を行うためのボタンです。トークンを誤って流出させてしまった際などに使用します。
　以上でDiscord Developer Portalを使ったセットアップは完了です。ブラウザを閉じて構いません。クリップボートにコピーしたトークンは、安全な場所にペーストしておいてください。トークンを紛失してしまったときは、同じ手順でコピーしてください。

#### トークンとは？
　BotアカウントにPythonプログラムからアクセスするには、**トークン**を取得する必要があります。トークンとは、**ログインIDとパスワードが一体になった文字列**のことです。プログラム中でトークンをDiscord APIサーバーに送信することで、Botアカウントにログインできます。
　トークンを他人に知られるとBotが乗っ取られてしまいますので、厳重に管理する必要があります。また、Botのトークンを複数人で共有するのは、気心の知れた友達や家族であっても**厳禁**です。Botは、そのBotを作成したユーザーが責任をもって管理する必要があります。面倒でも各自でBotアカウントを作成するようにしましょう。


## テスト用Guildを作ろう
　つづいて、いま作成したBotとあなたがやりとりできる場所を用意します。

### なぜテスト用Guildが必要なの？
　Botの開発中は、あなたとBotだけがいる**テスト用Guild**でテストすることが望ましいです。たとえば万が一プログラムの誤りで、そのGuildの会話ログがすべて消えてしまったり、友人をGuildからBANしてしまったりしては一大事です。ごく親しい友人同士で使うBotであれば、多少の失敗をしても許してくれるでしょう。ですが開発者としての責任を忘れてはいけません。まずはミスしても誰にも迷惑をかけない場所でしっかり動作確認するべきです。
　またGuildは権限をはじめとして、さまざまな設定がカスタマイズされていることがあります。このような環境で動作確認をした場合、Botがうまく動かなかったときの原因がGuildの設定なのかBotのコードなのか判断がつきません。開発の初期段階では、Botの開発に集中できるよう、まっさらなGuildで動作確認することが望ましいです。

### テスト用Guildを作成する - Botと対話する準備
　あなたとBotだけが使うテスト用Guildを作成しましょう。
　https://discordapp.com/channels/@me にアクセスし、Guild一覧の下にある「＋」をクリックします（[@fig:sc_add_testing_guild]）。

![テスト用Guild追加](resource/image/sc_add_testing_guild.png){#fig:sc_add_testing_guild}

「サーバーを作成」をクリックします（[@fig:sc_add_testing_guild2]）。

![テスト用Guild追加 2](resource/image/sc_add_testing_guild2.png){#fig:sc_add_testing_guild2}

　任意のGuild名を付け、「新規作成」をクリックします（[@fig:sc_add_testing_guild3]）。

![Guild名の入力](resource/image/sc_add_testing_guild3.png){#fig:sc_add_testing_guild3}

　テスト用Guildの作成が完了しました（[@fig:sc_add_testing_guild4]）。

![作成したGuildの初期画面](resource/image/sc_add_testing_guild4.png){#fig:sc_add_testing_guild4}

## Discordの権限について知ろう
### 権限はGuildの治安を守る仕組み
　**権限**とは、そのGuildにおける「そのユーザーができること」を管理するための仕組みです。たとえば「Kick Members」権限を持つユーザーは、Guildのメンバーをキックできます。また、「Read Message History」を持つユーザーは、そのGuildのメッセージ履歴を読めます。ユーザーの荒らし行為や誤操作による被害を防ぎ、Guildを便利で安全にするため用いられます。
　何も権限を持たないユーザーは何もできないのかというと、そうではありません。Guildによって「everyone（全員）」に許されている行動をすることはできます。デフォルト設定のGuildであれば、メッセージ履歴は読めるし、発言もできますが、メンバーのキックはできません。

### Bot開発に権限の検討はつきもの
　Botを開発する際には、どのような権限をBotに与える必要があるかを検討する必要があります。Botがどのような機能を持つかを把握し、必要最低限の権限のみを与えます。とくに注意すべき権限として、**Administrator（管理者）**があります。AdministratorはすべてのGuild設定を上書きして全機能の利用を許可する非常に強力な権限です。この権限を付与すると、当然、チャンネルの削除やGuild自体の消去などの強力すぎる機能も利用できてしまいます。もちろんリスクを把握したうえで、必要のためBotに付与することはあります。ですが、本権限をやみくもに利用するのは厳禁です。

## BotをGuildに招待しよう
### 招待用URLの作成
　テスト用GuildにBotアカウントを招待するため、**招待用URL**を作成します。まず、あなたのアプリケーションの管理画面を開きましょう。
　https://discordapp.com/developers/applications/ にアクセスし、作ったApplicationの名前をクリックします（[@fig:sc_select_your_app]）。この画面は筆者のDiscord Developer Portal画面なので、複数のApplicationが表示されています。

![Application画面を開く](resource/image/sc_select_your_app.png){#fig:sc_select_your_app}

「OAuth2」をクリックし、「SCOPES」の中から、「bot」をクリックします。下部にすこしスクロールします（[@fig:sc_oauth_url1]）。

![Oauth画面でBot招待用URLを作成する](resource/image/sc_oauth_url1.png){#fig:sc_oauth_url1}

　招待用URLを作成する際には、Botに付与したい**権限**を指定できます。このBotには「テキストチャンネルの内容に反応して発言する機能」を実装しますので、それに必要な最低限の権限を付与してURLを作成しましょう。
　BOT PERMISSIONSの中から、「View Channels」「Send Messages」をクリックし、生成されたURLを「Copy」を押してクリップボートにコピーします（[@fig:sc_oauth_url2]）。

![Botに必要な権限を選択し、URLをコピーする](resource/image/sc_oauth_url2.png){#fig:sc_oauth_url2}

### BotをGuildに招待
　Webブラウザのアドレスバーに前項で生成したURLを貼り付け、開きます。コンボボックスからテスト用Guildを選択し、「認証」ボタンを押します（[@fig:sc_invite_bot]）。

![BotをGuildに招待する](resource/image/sc_invite_bot.png){#fig:sc_invite_bot}

「私はロボットではありません」のチェックボックスが現れるので、クリックします（[@fig:sc_invite_bot2]）。認証用のパズルが表示されたら、表示にしたがって操作します。

![Botでないことを証明する](resource/image/sc_invite_bot2.png){#fig:sc_invite_bot2}

　[@fig:sc_invite_bot3]のような画面が表示されたら成功です。

![認証成功画面](resource/image/sc_invite_bot3.png){#fig:sc_invite_bot3}

　Botを招待できたことを確認するために、テスト用Guildを開きましょう。[@fig:sc_invite_bot4]のようにメッセージが表示されていればOKです。また、四角で囲んだアイコンをクリックし、Guildメンバー一覧を開きましょう。あなたの追加したBotアカウントが現れているはずです。

![Botを招待できたことを確認する](resource/image/sc_invite_bot4.png){#fig:sc_invite_bot4}

　うまくいかないときは、URLを開いているWebブラウザがDiscordにログインできているか確認してください。
　これですべての準備は完了です。次回からは実際にコードを書き、作ったBotを動作させていきましょう。

### Botを公開したいときは？
　Botを公開したいときは、まず[@fig:sc_edit_bot_info]でOFFにした「PUBLIC BOT」の設定をONにします。あとは「BotをGuildに招待」で生成したURLを共有すれば、誰でもあなたのBotを自分のGuildへ招待できるようになります。「PUBLIC BOT」の設定がOFFのままでも、あなた自身はあなたのBotをGuildに招待することは可能です。
