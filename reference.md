　この章では、discord.pyのAPIからBotに使える関数をクラスごとに**ピックアップ**し、サンプルコードとともに紹介します。Botの機能を考えるヒントになれば幸いです。

## この章のつかいかた
### 重要: この文書はリファレンスではありません
　冒頭で述べましたが、この章はdiscord.pyのAPIからBot製作によく使うような関数をクラスごとに**ピックアップ**し紹介するものです。よって、discord.pyにできることを網羅**したものではない**ことにご注意ください。discord.pyにできることをすべて確認するには、公式ドキュメントをごらんください。また、この章は公式ドキュメントに基づき必要な権限や注意事項について記述していますが、抜け・漏れがある場合もございます。ご容赦ください。
　https://discordpy.readthedocs.io/

### 応用のしかた
#### コマンドではなくイベントリスナーで動かしたい
　サンプルコードは、各クラスのインスタンスさえ取得すれば使えます。たとえば、Guildクラスであれば、[@lst:get_instance_sample]のようにさまざまな取得方法があります。

```{#lst:get_instance_sample .python caption="Guildインスタンスの取得例"}
# Messageインスタンスから取得: on_messageイベントなどで利用可能
guild = message.guild
# Memberインスタンスから取得: on_member_updateイベントなどで利用可能
guild = member.guild
# Contextインスタンスから取得: コマンド、コマンドエラー処理で利用可能
guild = ctx.guild
```

　どのクラスからインスタンスを取得できるかについては、本節冒頭に記した公式ドキュメントから確認してください。

#### コマンドをそのまま動かしてみたい
　この章のコマンドをそのまま動かしたい場合、下記のようなファイルを作成し、ソースコードのコマンドを追加してください。トークンの取得方法は、[@sec:setup]をごらんください。

```{#lst:reference_usage .python caption="コマンドサンプルコード"}
from discord.ext import commands
import discord

bot = commands.Bot(command_prefix="!")

# ここから --->
@bot.command()
async def guild_info(ctx):
    from datetime import timedelta
    guild = ctx.guild
    await ctx.send(
        f"サーバー名: {guild.name}\n"
        f"サーバーオーナー: {guild.owner.name}\n"
        f"メンバー数: {guild.member_count}\n"
        f"作成日: {guild.created_at + timedelta(hours=9)}\n"
    )
# ここまで <---

# 実際のトークンに置き換える
bot.run("ThisIsDummyToken00000000.XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX")
```

#### エラー処理について
　この章では、処理の見通しがよくなるよう、エラーや例外の処理は最低限にしています。関数によっては、Botに特別な権限がないと処理できません。そのような関数を権限がない状態で実行すると`discord.errors.Forbidden`例外が発生します。自分だけが使うBotであれば権限を与えれば解決しますが、不特定多数に公開するBotの場合、以下を参考に例外発生時の処理を実装してください。
　まず、例外処理のない実装の例を[@lst:forbidden_example]に示します。

```{#lst:forbidden_example .python caption="Forbidden例外処理なし"}
@bot.command()
async def guild_kick(ctx):
    guild = ctx.guild
    await ctx.send(f"{member.name}をKickします。")
    await guild.kick(user=member, reason=reason)  # Forbidden例外を発生しうる関数
```

　[@lst:forbidden_example]に例外処理を入れたコードを[@lst:forbidden_example_with_try_catch]に示します。Forbidden例外が発生した場合、その旨をテキストチャンネルに通知しています。

```{#lst:forbidden_example_with_try_catch .python caption="Forbidden例外処理 実装例"}
@bot.command()
async def guild_kick(ctx):
    guild = ctx.guild
    await ctx.send(f"{member.name}をKickします。")
    try:
        await guild.kick(user=member, reason=reason)
    except discord.errors.Forbidden:
        await ctx.send("権限がありません。")
```

#### ソースコードをコピーしたい
　この章のソースコードは下記リポジトリで閲覧できます。
　https://github.com/cod-sushi/discord_dev_sample.git

## Guild
　Guildに関する機能を持つクラスです。Guildインスタンスは、コマンド処理やメッセージ受信イベント処理で引数から取得できます。

```{#lst:guild_get_instance .python caption="Guildインスタンスを取得する方法"}
guild = ctx.guild  # コマンドの場合
guild = message.guild  # on_messageイベントの場合
```

### 情報を表示する
　GuildインスタンスはGuildに関するさまざまな属性を保持しています。[@lst:guild_info]はGuildの情報をテキストチャンネルに送信するサンプルです。その他の属性は公式リファレンスをごらんください。

```{#lst:guild_info .python caption="Guildの情報を表示する"}
@bot.command()
async def guild_info(ctx):
    from datetime import timedelta
    guild = ctx.guild
    await ctx.send(
        f"サーバー名: {guild.name}\n"
        f"サーバーID: {guild.id}\n"
        f"サーバーオーナー: {guild.owner.name}\n"
        f"メンバー数: {guild.member_count}\n"
        f"作成日: {guild.created_at + timedelta(hours=9)}\n"
    )
```

![Guildの情報を表示する](resource/image/sc_guild_info.png){#fig:sc_guild_info}

### 名前を変える
　`Guild.edit`関数を使えば、Guildのさまざまな情報を編集できます。[@lst:guild_edit]は、Guildの名前を変更するコマンドの実装例です。権限`manage_guild`が必要です。

```{#lst:guild_edit .python caption="Guildの名前変更"}
@bot.command()
async def guild_edit(ctx, name):
    guild = ctx.guild
    old_name = guild.name
    await guild.edit(name=name)
    await ctx.send(
        "Guildの名前を変更しました。\n"
        f"{old_name}→{name}"
    )
```

### テキストチャンネルを作成する
　`Guild.create_text_channel`関数を使えば、Guildにテキストチャンネルを作成できます。類似の関数として、`Guild.create_voice_channel`があります。権限`manage_channels`が必要です。

```{#lst:guild_create_channel .python caption="テキストチャンネルの作成"}
@bot.command()
async def guild_create_channel(ctx, name):
    guild = ctx.guild
    await guild.create_text_channel(name=name)
    await ctx.send(f"テキストチャンネル{name}を作成しました。")
```

![テキストチャンネルを作成する](resource/image/sc_guild_create_channel.png){#fig:sc_guild_create_channel}

### メンバーをKick、Banする
　`Guild.kick()`関数を使えば、GuildからメンバーをKickできます。KickされたメンバーはGuildから退出させられますが、また招待を受ければGuildにふたたび参加できます。権限`kick_members`が必要です。BotがメンバーをKickしたことは、`reason`で指定した理由とともにGuildのAudit Log（監査ログ）に記録として残ります。類似の関数に`Guild.ban()`があります。

```{#lst:guild_kick .python caption="メンバーをKickする"}
@bot.command()
async def guild_kick(ctx):
    guild = ctx.guild
    await ctx.send(f"{member.name}をKickします。")
    await guild.kick(user=member, reason=reason)
```

### Guildを退出する
　`Guild.leave()`関数を使えば、任意のGuildから退出できます。再度招待してもらうまで同じGuildには参加できません。このコマンドを試したあとは、ふたたびGuildにBotを招待してください。

```{#lst:guild_leave .python caption="Guildからの退出"}
@bot.command()
async def guild_leave(ctx):
    guild = ctx.guild
    await ctx.send("このGuildから退出します。")
    await guild.leave()
```

![Guildからの退出](resource/image/sc_guild_leave.png){#fig:sc_guild_leave}

## Member
　MemberはGuildに所属するメンバーをあらわすクラスです。

```{#lst:member_get_instance .python caption="Memberインスタンスを取得する方法"}
member = ctx.author  # コマンドの場合
member = message.author  # on_messageイベントの場合
```

### 情報を表示する
　MemberインスタンスはMemberに関するさまざまな属性を保持しています。[@lst:member_info]はMemberの情報をテキストチャンネルに送信するサンプルです。その他の属性は公式リファレンスをごらんください。

```{#lst:member_info .python caption="Member情報の表示"}

@bot.command()
async def member_info(ctx):
    from datetime import timedelta
    member = ctx.author
    await ctx.send(
        f"ユーザー名: {member.name}\n"
        f"ユーザーID: {member.id}\n"
        f"Discordへの参加日: {member.created_at + timedelta(hours=9)}\n"
        f"Guildへの参加日: {member.joined_at + timedelta(hours=9)}\n"
        f"ステータス: {str(member.status)}\n"
        f"モバイルからのログイン？: {member.is_on_mobile()}"
    )
```

![Member情報の表示](resource/image/sc_member_info.png){#fig:sc_member_info}

### メンションで通知を送る
　`Member.mention`属性で、メンバーにメンションを送付するための文字列を取得できます。この文字列をメッセージに含めると、Memberに通知を送信できます。Memberが通知をオフにしている場合、プッシュ通知は送信されません。

```{#lst:member_mention .python caption="メンバーにメンションを送信"}
@bot.command()
async def member_mention(ctx):
    member = ctx.author
    await ctx.send(f"{member.mention} メンションのテストです。")
```

![メンバーにメンションを送信](resource/image/sc_member_mention.png){#fig:sc_member_mention}

### メンバーがBotかどうか確認する
　`Member.bot`属性を見れば、対象のメンバーがBotか、通常のユーザーか確認できます。

```{#lst:member_bot .python caption="memberがBotかどうか確認"}
@bot.command()
async def member_bot(ctx, member: discord.Member):
    if member.bot:
        await ctx.send(f"{member.name}はBotです。")
    else:
        await ctx.send(f"{member.name}はBotではありません。")
```

![menberがBotかどうか確認](resource/image/sc_member_bot.png){#fig:sc_member_bot}

### メンバーにDMを送信する
　`Member.send()`関数を使うと、メンバーにDMを送信できます。

```{#lst:member_send .python caption="メンバーにDMを送信"}
@bot.command()
async def member_send(ctx, member: discord.Member, content):
    await ctx.send(f"{member.name}にDMを送信します。")
    await member.send(content=content)
```

![メンバーにDMを送信 コマンド実行例](resource/image/sc_member_send1.png){#fig:sc_member_send1}

![メンバーにDMを送信 DM受信画面](resource/image/sc_member_send2.png){#fig:sc_member_send2}

### ボイスチャットの参加状況を取得する
　`Member.voice`属性には、メンバーのボイス状態（`VoiceState`）が入っています。

```{#lst:member_voice .python caption="メンバーのボイス状態を確認"}
@bot.command()
async def member_voice(ctx):
    member = ctx.author
    if member.voice and member.voice.channel:
        await ctx.send(f"あなたは「{member.voice.channel.name}」にいます。")
    else:
        await ctx.send(f"あなたはボイスチャンネルに参加していません。")
```

![メンバーのボイス状態を確認](resource/image/sc_member_voice.png){#fig:sc_member_voice}

### メンバーを別のボイスチャンネルに移動する
　すでにボイスチャンネルに参加しているメンバーに対して`Member.move_to()`関数を使うと、メンバーを別のボイスチャンネルに移動できます。権限`move_members`が必要です。対戦ゲームのチーム分け時などに便利です。ボイスチャンネルに参加していないメンバーを新しくボイスチャンネルに参加させることはできません。

```{#lst:member_move .python caption="メンバーのボイスチャンネルを移動"}
@bot.command()
async def member_move(ctx, member: discord.Member,
                      voice_channel: discord.VoiceChannel):
    if not member.voice or not member.voice.channel:
        await ctx.send("ボイスチャンネルに参加していません。")
        return
    await member.move_to(voice_channel)
    await ctx.send(
        f"{member.name}をボイスチャンネル{voice_channel.name}に移動しました。")
```

![メンバーを別のボイスチャンネルに移動](resource/image/sc_member_move.png){#fig:sc_member_move}

## Client
　ClientはDiscordサーバーとの接続やBot自身に関する機能を持つクラスです。`Bot`クラスは`Client`クラスを継承するため、`Client`として扱えます。

```{#lst:client_get_instance .python caption="BotクラスとClientクラス"}
client = discord.Client()  # Clientインスタンスの生成
bot = discord.ext.commands.Bot(command_prefix="!")  # Botインスタンスの生成
```

### Botアカウントに「～をプレイ中」を表示する
　Botアカウント名の下に、「～をプレイ中（英語では「Playing ～」）」のようにBotの状態が表示されているのを見たことがあるかもしれません。`Client.change_presense()`関数を使うと、「～」の部分に任意の文字列を表示できます。

```{#lst:client_change_presense .python caption="「～をプレイ中」の表示"}
@bot.command()
async def client_change_presense(ctx, title):
    client = bot
    game = discord.Game(name=title)
    await client.change_presence(activity=game)
```

![「～をプレイ中」の表示](resource/image/sc_client_change_presense.png){#fig:sc_client_change_presense}

### Botを終了する（Botアカウントからのログアウト）
　`Client.close()`関数を呼び出すと、Botアカウントからログアウトします。ログアウトした後はPythonプロセスが終了します。このコマンドをBotに実装する場合、開発者以外のユーザーは使用できないようにしておきましょう。

```{#lst:client_close .python caption="Botの終了"}
@bot.command()
async def client_close(ctx):
    client = bot
    await ctx.send("Botアカウントからログアウトします。")
    await client.close()
```

![Botの終了](resource/image/sc_client_close.png){#fig:sc_client_close}

### Botアカウントの情報を表示する
　Clientクラスの属性には、Botアカウントに関する情報が多く存在します。Botがどれくらい利用されているかの情報を確認するのに便利です。他人にこれらの情報を知られたくない場合、開発者以外のユーザーは使用できないようにしておきましょう。

```{#lst:client_info .python caption="Client情報の表示"}
@bot.command()
async def client_info(ctx):
    client = bot
    await ctx.send(
        f"Botユーザー名: {client.user.name}\n"
        f"BotユーザーID: {client.user.id}\n"
        f"Guild数: {len(client.guilds)}\n"
        f"ボイス接続数: {len(client.voice_clients)}\n"
        f"ユニークユーザー数: {len(client.users)}\n"
        f"延べユーザー数: {sum([g.member_count for g in client.guilds])}\n"
    )
```

![Clientの情報を出力](resource/image/sc_client_info.png){#fig:sc_client_info}

### アプリケーションの情報を表示する
　`Client.application_info()`関数を使うと、Botに紐づくアプリケーションの情報を取得できます。アプリケーション情報には、アプリケーションの名前やアイコン、アプリケーションのオーナーが含まれます。

```{#lst:client_application_info .python caption="アプリケーション情報の表示"}
@bot.command()
async def client_application_info(ctx):
    client = bot
    app_info = await client.application_info()
    await ctx.send(
        f"アプリケーションID: {app_info.id}\n"
        f"Botオーナー: {app_info.owner.name}\n"
        f"Public Bot?: {app_info.bot_public}\n"
    )
```

![アプリケーション情報の表示](resource/image/sc_client_application_info.png){#fig:sc_client_application_info}

## TextChannel
　TextChannelは、Guildのテキストチャンネルについて管理するクラスです。主に`Message.channel`属性からインスタンスを取得して使います。ユーザー同士のDMを管理するDMChannelも、一部を除き類似の関数を備えています。

```{#lst:channel_get_instance .python caption="channelインスタンスを取得する方法"}
channel = ctx.channel  # コマンドの場合
channel = message.channel  # on_messageイベントの場合
```

### 時間のかかる処理中、入力中の表示をする
　`TextChannel.typing()`を使うと、「～が入力中…（英語では「～ is typing…」）」という、ユーザーがチャットを入力している際に出る表示と同じものを表示できます。

```{#lst:tc_typing .python caption="入力中と表示"}
@bot.command()
async def tc_typing(ctx):
    import asyncio
    await ctx.send("処理を開始します。")
    async with ctx.channel.typing():
        # 長い処理の代わりにsleepする
        await asyncio.sleep(3)
    await ctx.send("処理が完了しました！")
```

![入力中と表示](resource/image/sc_tc_typing.png){#fig:sc_tc_typing}

### 条件にあうメッセージをまとめて削除する
　`TextChannel.purge()`関数を使うと、条件にあうメッセージをまとめて削除できます。`manage_message`権限と、`read_message_history`権限が必要です。

```{#lst:tc_purge .python caption="条件にあうメッセージをまとめて削除"}
@bot.command()
async def tc_purge(ctx, purge_word, limit: int):
    def contains_purge_word(message):
        return purge_word in message.content
    channel = ctx.channel
    await ctx.send(f"「{purge_word}」を含むメッセージを最大{limit}件削除します。")
    deleted = await channel.purge(limit=limit, check=contains_purge_word,
                                  before=ctx.message.created_at)
    await ctx.send(f'{len(deleted)}件のメッセージを削除しました。')
```

　最新のメッセージから`limit`で指定した数だけ遡り、`check`で指定した条件にあうメッセージを削除します。

![purge 実行前](resource/image/sc_tc_purge1.png){#fig:sc_tc_purge1}

![purge 実行後](resource/image/sc_tc_purge2.png){#fig:sc_tc_purge2}

### テキストチャンネル（Guild）への招待を送信する
　`TextChannel.invite()`関数を使うと、テキストチャンネル（テキストチャンネルが属するGuild）への招待を作成できます。`create_instant_invite`権限が必要です。以下の例ではあまり意味がないように見えますが、複数Guildをまたいで使用することで効果を発揮します。

```{#lst:tc_create_invite .python caption="招待の作成"}
@bot.command()
async def tc_create_invite(ctx):
    # 24時間有効、10人まで招待可能な招待を作成
    channel = ctx.channel
    invite = await channel.create_invite(max_age=3600 * 24, max_uses=10)
    await ctx.send(invite.url)
```

![招待の作成](resource/image/sc_tc_create_invite.png){#fig:sc_tc_create_invite}

### テキストチャンネルに画像を投稿する
　`TextChannel.send()`関数にファイルを指定すると、メッセージに画像を添付できます。また、画像だけを送信することもできます。

```{#lst:tc_send_file .python caption="テキストチャンネルに画像を送信"}
# attach_files
@bot.command()
async def tc_send_file(ctx):
    await ctx.send(file=discord.File(fp="shovel.png"))
```

![テキストチャンネルに画像を送信](resource/image/sc_tc_send_file.png){#fig:sc_tc_send_file}

## Message
　Messageは、テキストチャンネルに投稿されるメッセージの機能を持つクラスです。`on_message`イベントではMessageのインスタンスそのものが引数として渡されます。

```{#lst:message_get_instance .python caption="messageインスタンスを取得する方法"}
message = ctx.message  # コマンドの場合
# on_messageイベントの場合、引数としてmessageが渡される
```

### メッセージにリアクションを追加する
　`Message.add_reaction()`関数を使うと、メッセージにリアクションを追加できます。そのメッセージにすでに存在するリアクションを追加する場合、権限は不要です。あたらしいリアクションを追加する場合、権限`add_reactions`が必要です。

```{#lst:message_add_reaction .python caption="メッセージにリアクションを追加"}
@bot.command()
async def message_add_reaction(ctx, emoji):
    msg = await ctx.send("このメッセージにリアクションを付ける")
    await msg.add_reaction(emoji)
```

![メッセージにリアクションを追加](resource/image/sc_message_add_reaction.png){#fig:sc_message_add_reaction}

### メッセージに添付されたファイルを保存する
　`Message.attachments`属性で、メッセージに添付されたファイルのリストにアクセスできます。これを利用して、Botが動いているマシンの記憶領域にファイルを保存できます。

```{#lst:message_save_attachment .python caption="添付ファイルの保存"}
@bot.command()
async def message_save_attachment(ctx):
    message = ctx.message
    if not message.attachments:
        await ctx.send("ファイルを添付してください")
        return
    attachment = message.attachments[0]
    await attachment.save(fp=attachment.filename)
    await ctx.send("1番目の添付ファイルを保存しました。")
```

![添付ファイルの保存](resource/image/sc_message_save_attachment.png){#fig:sc_message_save_attachment}