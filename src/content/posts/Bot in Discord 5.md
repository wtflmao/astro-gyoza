---
title: Bot in Discord with discord.js (4)
date: 2022-07-12
tags: [Discord bot, JS]
category: Discord.js
summary: 这是一份非常详细和系统化的利用 NodeJS 进行 Discord bot 开发的教程，本文介绍了什么是响应 response、bot 如何抓取响应、如何删除响应的方法，以及简单拓展介绍了 Discord 所有消息、用户、频道、服务器的统一 ID 类型：Snowflake 类型。
---


## Chapter 6 - 交互：对斜杠命令回复的那些事儿（2）

本章是 Chapter 5 的续集。

### 抓取响应 Fetching responses

除了对斜杠命令进行回复外，我们还可以抓取这个回复，比如用于给这个回复添加表情回复（reactions）。

靠这个：`CommandInteraction#fetchReply()`。

来个例子，`commands/pingFetch.js`：
```js
const { SlashCommandBuilder } = require('discord.js');

module.exports = {
	data: new SlashCommandBuilder()
		.setName('ping7')
		.setDescription('Replies with Pong, and fetch!'),
	async execute(interaction) {
	    await interaction.reply("Pong!");
		const message = await interaction.fetchReply();
		console.log(message); // 控制台输出抓取的消息对象本身
	},
};
```

控制台输出：
```json
<ref *1> Message {
  channelId: '---隐私打码---',
  guildId: '---隐私打码---',
  id: '---隐私打码---',
  createdTimestamp: xxxxxxxxxx021,
  type: 'APPLICATION_COMMAND',
  system: false,
  content: 'Pong!',
  author: ClientUser {
    id: '---隐私打码---',
    bot: true,
    system: false,
    flags: UserFlags { bitfield: 0 },
    username: 'BlogTest',
    discriminator: '---隐私打码---',
    avatar: null,
    banner: undefined,
    accentColor: undefined,
    verified: true,
    mfaEnabled: false
  },
  pinned: false,
  tts: false,
  nonce: null,
  embeds: [],
  components: [],
  attachments: Collection(0) [Map] {},
  stickers: Collection(0) [Map] {},
  editedTimestamp: null,
  reactions: ReactionManager { message: [Circular *1] },
  mentions: MessageMentions {
    everyone: false,
    users: Collection(0) [Map] {},
    roles: Collection(0) [Map] {},
    _members: null,
    _channels: null,
    crosspostedChannels: Collection(0) [Map] {},
    repliedUser: null
  },
  webhookId: '---隐私打码---',
  groupActivityApplication: null,
  applicationId: '---隐私打码---',
  activity: null,
  flags: MessageFlags { bitfield: 0 },
  reference: null,
  interaction: {
    id: '---隐私打码---',
    type: 'APPLICATION_COMMAND',
    commandName: 'ping7',
    user: User {
      id: '---隐私打码---',
      bot: false,
      system: false,
      flags: [UserFlags],
      username: '---隐私打码---',
      discriminator: '---隐私打码---',
      avatar: '---隐私打码---',
      banner: undefined,
      accentColor: undefined
    }
  }
}
```

这里返回的 json 信息很丰富，从命令发起者到机器人自己的信息，都包括了。

### 删除响应 Deleting responses

靠这个 `CommandInteraction#deleteReply()` 。

注意：你不能删除一个“短暂回复”（Ephemeral response）。

比如：`commands/pingDelete.js`：
```js
const { SlashCommandBuilder } = require('discord.js');
const wait = require('node:timers/promises').setTimeout;

module.exports = {
	data: new SlashCommandBuilder()
		.setName('ping')
		.setDescription('Replies with Pong!'),
	async execute(interaction) {
		await interaction.reply("Pong!");
		await wait(1000);
	    await interaction.deleteReply();
	},
};
```

它会在回复 `Pong!` 1 秒后，删除这条回复。


### 服务器命令 Guild commands

如果您的应用程序具有 applications.commands 的 scope 授权（我们在 Chapter 1 生成机器人时，已经选定过这个 scpoe 了），则服务器应用程序命令仅在创建它们的服务器中可用。

我们在 Chapter 2 中所注册的斜杠命令(ping.js)和斜杠命令部署器(deploy_commands.js)都是围绕服务器命令展开的。

### 全局命令 Global commands

全局应用程序命令将在您的应用程序具有 applications.commands 的 scpoe 授权（我们在 Chapter 1 生成机器人时，已经选定过这个 scpoe 了）的所有服务器以及 DM（私聊） 中可用。

要部署全局命令，您可以使用服务器命令部分中的相同脚本并将脚本中的路由调整为 
```js
...

// 利用 .applicationCommands 注册全局命令
rest.put(Routes.applicationCommands(clientId),{ body: commands },)
    .then(() => console.log('Successfully registered application commands.'))  
    .catch(console.error);
    
...
```


### 补充：snowflake 类型

Snowflake 类型源自推特（Twitter）公司。Snowflake 是 64 位无符号整数，具有全局唯一性，基于时间生成，而不是按顺序生成。

在 Javascript 中，整数最大只有 53 位，因此在 JavaScript 中，我们一般选用字符串来存储 snowflake 值。

在 Discord 中，假设我们有一个 Snowflake l类型的值 ‘266241948824764416’，它会被这样解读：
```
64                                          22     17     12          0
 000000111011000111100001101001000101000000  00001  00000  000000000000
 number of milliseconds since Discord epoch  worker  pid    increment
```

63 代表最高位，0代表最低位。

| 项目 | 含义 | 位 | 位的个数 | 说明 | 提取方法 |
|---|---|---|---|---|---|
| Timestamp | 时间戳，单位毫秒 | 63 to 22 | 42 bits | 自 Discord Epoch 以来的毫秒数，即 2015 年的第一毫秒（即 UNIX 时间戳 1420070400000 所对应的那一毫秒） | (snowflake >> 22) + 1420070400000 |
| Internal worker ID | Discord 集群编号 | 21 to 17 | 5 bits | 无需理会 | (snowflake & 0x3E0000) >> 17 |
| Internal process ID | 集群内部进程编号 | 16 to 12 | 5 bits | 无需理会 | (snowflake & 0x1F000) >> 12 |
| Increment | 这 1 毫秒内，条目的增量 | 11 to 0 | 12 bits | 对于在该进程上生成的每个 ID，此数字都会递增 | snowflake & 0xFFF |


### CommandInteraction 类

表示命令交互。

来自文档 https://discord.js.org/#/docs/discord.js/main/class/CommandInteraction

#### 类方法

我们在上面谈到的许多方法，都是来自于 `CommandInteraction` 类，这个类拓展自 `BaseInteraction` 类。

我们已经见过了 `.deferReply()` 、`.deleteReply()` 、`.editReply()` 、`.fetchReply()` 、`.followUp()` 、`.reply()`，不再赘述。

下面来看 `CommandInteraction` 类自己非继承来的方法：
| 方法名 | 有参数吗 | 返回值类型 | 说明 |
|---|---|---|---|
| .awaitModalSubmit() | Y | Promise < ModalSubmitInteraction > | 收集通过过滤器的单个模态提交交互。 如果时间到期，Promise 将拒绝 |
| .deferReply() | Y | Promise < (Message or InteractionResponse) > | 推迟对此交互的回复 |
| .deleteReply() | N | Promise < void > | 删除对此交互的初始回复 |
| .editReply() | Y | Promise < Message > | 编辑对此交互的初始回复 |
| .fetchReply() | N | Promise < Message > | 获取对此交互的初始回复 |
| .followUp() | Y | Promise < Message > | 向此交互发送后续消息 |
| .reply() | Y | Promise < (Message or InteractionResponse) > | 创建对此交互的回复 |
| .showModel() | Y | void | 显示模态组件 |

以下方法来自父类 `BaseInteraction`：
| 方法名 | 有参数吗 | 返回值类型 | 说明 |
|---|---|---|---|
| .inCachedGuild() | N | Boolean | 指示此交互是否同时被缓存并从服务器接收 |
| .inGuild() | N | Boolean | 指示此交互是否来自服务器 |
| .inRawGuild() | N | Boolean | 指示此交互是否来自未缓存的服务器 |
| .isButton() | N | Boolean | 指示此交互是否为 ButtonInteraction |
| .isChatInputCommand() | N | Boolean | 指示此交互是否为 ChatInputCommandInteraction |
| .isContextMenuCommand() | N | Boolean | 指示此交互是否为 ContextMenuCommandInteraction |
| .isMessageContextMenuCommand() | N | Boolean | 指示此交互是否为 MessageContextMenuCommandInteraction |
| .isRepliable() | N | Boolean | 指示是否可以回复此交互 |
| .isSelectMenu() | N | Boolean | 指示此交互是否为 SelectMenuInteraction |
| .isUserContextMenuCommand() | N | Boolean | 指示此交互是否为 UserContextMenuCommandInteraction |

#### 类属性

| 属性名 | 类型 | 描述 |
|---|---|---|
| .applicationId | Snowflake | 应用程序的 ID |
| .appPermissions | < PermissionsBitField > | 应用程序或机器人在发送交互的频道内拥有的权限集 |
| .channel | TextBasedChannels | 发送此交互的频道 |
| .channelId | Snowflake | 发送此交互的频道的 ID |
| .client | Client | 实例化这个交互的客户端 |
| .command | ApplicationCommand | 调用的应用程序命令（如果之前已获取） |
| .commandGuildId | Snowflake | 调用的应用程序命令注册到的服务器 id |
| .commandId | Snowflake | 调用的应用程序命令的 id |
| .commandName | string | 调用的应用程序命令的名称 |
| .commandType | ApplicationCommandType | 调用的应用程序命令的类型 |
| .createdAt | Date | 创建交互的时间 |
| .createdTimestamp | number | 创建交互的时间戳 |
| .deferred | boolean | 是否已推迟对此交互的回复 |
| .ephemeral | boolean | 对此交互的回复是否是短暂的 |
| .guild | Guild | 发送此交互的服务器 |
| .guildId | Snowflake | 发送此交互的服务器 ID |
| .guildLocale | Locale | 发送此交互的服务器的首选语言环境 |
| .id | Snowflake | 交互的 ID |
| .locale | Locale | 调用此交互的用户的语言环境 |
| .member | GuildMember or APIGuildMember | 如果此交互是在公会中发送的，发送它的成员 |
| .memberPermissions | PermissionsBitField | 执行此交互的通道中成员的权限（如果存在） |
| .replied | Guild | 发送此交互的服务器 |
| .token | string | 交互的令牌 |
| .type | InteractionType | 交互的类型 |
| .user | User | 发送此交互的用户 |
| .version | number | 版本 |
| .webhook | InteractionWebhook | 关联的交互 webhook，可用于进一步与此交互进行交互 |


很多我们都用不上，当然很多能节约我们的敲代码的时间。要学会在不断地尝试中学习，学会在错误中吸取经验。