---
title: Bot in Discord with discord.js (6)
date: 2022-07-13
tags: [Discord bot, JS]
category: Discord.js
summary: 这是一份非常详细和系统化的利用 NodeJS 进行 Discord bot 开发的教程，本文介绍了选项 Option 命令的构建方法，包括单选和多选。
---

## Chapter 9 - 斜杠命令的参数：Options 和 Choices

### 注册斜杠命令是前提

Discord 为开发人员提供了创建客户端集成斜杠命令的选项。 在本节中，我们将介绍如何使用 discord.js 注册这些命令！任何斜杠命令在可以被使用前，都需要先向 Discord 服务器注册。命令在没做出改动的情况下，不需要每次启动机器人之前注册一遍命令。

### 选项 Options

应用程序命令可以有选项。你可以将这些选项视为函数的参数并指定它们，如下所示：

比如 `commands/echo.js`：
```js
const { SlashCommandBuilder } = require('discord.js');

module.exports = {
	data: new SlashCommandBuilder()
		.setName('echo')
		.setDescription('Replies with your input!')
		.addStringOption(option =>
			option.setName('input')
				.setDescription('The input to echo back')
				.setRequired(true)),
	async execute(interaction) {
		const input = interaction.options.getString("input");
		await interaction.reply(input);
	},
};
```

我们为这个斜杠命令设置了命令名：`echo`，设置了简介说明：`Replies with your input!`。

除此之外，我们为这个命令添加了 option 用于接受命令参数，就像 c 语言函数接收参数那样。

这里只有一个 option，类型是字符串类型，只接收字符串（由 `.addStringOption` 看出）。我们为这个 option 设置了名称 `input`，设置了说明 `The input to echo back`，并设置其为必填（由 `.setRequired(true)` 指定）。

命令效果图：
![image](https://img2022.cnblogs.com/blog/2455224/202207/2455224-20220712100547224-825130046.png)
![image](https://img2022.cnblogs.com/blog/2455224/202207/2455224-20220712100330532-1982660812.png)

除了字符串选项外，discord.js 提供了很多 Option 类型：
| **类型** | **设置方法名** | **说明** |
| -- | -- | -- |
| STRING | .setStringOption() | 将选项设置为需要字符串值 |
| INTEGET | .setIntegetOption() | 将选项设置为需要整数值 |
| BOOLEAN | .setBooleanOption() | 将选项设置为需要布尔值 |
| USER | .setUserOption() | 设置选项以要求 Discord 用户或 snowflake 作为值 |
| CHANNEL | .setChannelOption() | 将选项设置为需要 Discord 服务器频道(Server Channel) 或 snowflake 作为值 |
| ROLE | .setRoleOption() | 将选项设置为需要 Discord 身份组或 snowflake 作为值 |
| MENTIONABLE | .setMentionableOption() | 设置选项以要求用户、身份组或 snowflake 作为值 |
| NUMBER | .setNumberOption() | 将选项设置为需要小数（也称为浮点数）值 |
| ATTACHMENT | .setAttachmentOption() | 将选项设置为需要附件 |
| SUB_COMMAND | - |将选项设置为子命令 |
| SUB_COMMAND_GROUP | - | 将选项设置为子命令组 |

Option 经常和 Choice 搭配，因此下面我们先介绍 Choice，再给出更多的例子。

### 选择 Choices

“选项” 和 “选择” 我感觉都翻译的不咋样，容易弄混，所以我还是用回英文 Option 和 Choice 吧。

Choice 只能作用于 Option。

Choice 只能是 STRING（字符串） 或 INTEGER（整型）类型的 Option 才可以拥有的。

Option 很自由，用户想输入什么合法的内容就接收什么内容。

Choice 就像 C 语言的 `enum`，用户只能从一些程序给定的选择里挑一个出来。

如果一个 Option 被 Choice “绑定”了，那么 Option 的输入值，有效值只能是 Choices。

我们通过 `.addChoice()` 方法来构造 Choice。

给个例子：`commands/subOption.js`：
```js
const { SlashCommandBuilder } = require('discord.js');

module.exports = {
	data: new SlashCommandBuilder()
		.setName('giftest')
		.setDescription('MultiOption test.')
		.addStringOption(option =>
			option.setName("category")
				.setDescription("The choices category")
				.setRequired(true)
				.addChoices(
					{ name: 'Funny', value: 'gif_funny'},
					{ name: 'Meme', value: 'gif_meme'},
					{ name: 'Movie', value: 'gif_movie'}))
		.addIntegerOption(option =>
			option.setName("amount")
				.setDescription("The amount of gifs you want")
				.setRequired(true)
				.setMinValue(1)
				.setMaxValue(5)),
	async execute(interaction) {
		const category = interaction.options.getString("category");
		await interaction.reply(category);
		for (var i=2; i <= interaction.options.getInteger("amount"); i++) {
			await interaction.followUp(category);
		}
	},
};
```

分析一下这个例子之前，我们先看效果图。

错误用法之一：amount 的给定值小于允许的最小值(1)，或大于了允许的最大值(5)
`/giftest category:Funny amount:7`
![image](https://img2022.cnblogs.com/blog/2455224/202207/2455224-20220712125309697-519004000.png)

错误用法之二：amount 类型为整数，却输入了浮点数
`/giftest category:Movie amount:4.9`
![image](https://img2022.cnblogs.com/blog/2455224/202207/2455224-20220712130257095-983666074.png)

错误用法之三：未填所有必填项
`/giftest category: amount:1`
![image](https://img2022.cnblogs.com/blog/2455224/202207/2455224-20220712130812688-1865799060.png)

错误用法之四：category 字段 作为 Option 类型，用户填入了非有效选择
`/giftest category:ddd amount:2`
![image](https://img2022.cnblogs.com/blog/2455224/202207/2455224-20220712125311439-904337394.png)

正确用法：`/giftest category:Funny amount:3`
![image](https://img2022.cnblogs.com/blog/2455224/202207/2455224-20220712125441257-1308943218.png)
好，我们来解释代码。

这是一个同时使用了 Options 和 Choices 的代码例子。

这段节选代码，它指示了斜杠命令的命令名：`giftest`，和命令描述：`MultiOption test.`。
```js
...

data: new SlashCommandBuilder()
		.setName('giftest')
		.setDescription('MultiOption test.')

...
```


这段节选代码，它为 `/giftest` 构造了第一个 Option 参数，而且是 string 类型的。

这个 Option 它的 “参数名” 是 `category`，必填，且有效值只可以是由紧跟其后的 Choices 的一项（Funny、Meme 或 Movie）。

这个 Choices 组有三条项目，每条都可以看成是 “键值对”。

同一个 Option 的 Choices 中，name 字段必须唯一，value字段倒是可以相同。
```js
...

		.addStringOption(option =>
			option.setName("category")
				.setDescription("The choices category")
				.setRequired(true)
				.addChoices(
					{ name: 'Funny', value: 'gif_funny'},
					{ name: 'Meme', value: 'gif_meme'},
					{ name: 'Movie', value: 'gif_movie'}))
					
...
```


这段节选代码，为斜杠命令构造了第二个 Option 参数，名为 amount，类型是 Integer，必填。

我们为 amount 这个值设定了输入时允许的最小值：1，和最大值：5。
```js
...

		.addIntegerOption(option =>
			option.setName("amount")
				.setDescription("The amount of gifs you want")
				.setRequired(true)
				.setMinValue(1)
				.setMaxValue(5)),

...
```


下面的 execute() 函数，功能是回复 amount 遍 category 字段对应的键值对的值（gif_funny、git_meme 或 gif_movie）。

我们通过方法 `interaction.options.getString("category")` 来获取 category 的值，即选定 choice 的键值对的值（value）。

我们通过方法 `interaction.options.getInteger("amount")` 来获取 amount 的值，这是个用户输入的整数。
```js
...

	async execute(interaction) {
		const category = interaction.options.getString("category");
		await interaction.reply(category);
		for (var i=2; i <= interaction.options.getInteger("amount"); i++) {
			await interaction.followUp(category);
		}
	},

...
```

我们发现，discord.js 提供的方法丰富多样，对于 bot 开发来说，是够用的。

我们在上面的 `commands/subOption.js` 中看到，我们通过 `.setName("giftest")` 设置了 name 字段的值，通过 `.setDescription("MultiOption test.")` 设置了 description 字段的值，通过 `.setMinValue(1)` 设置了 minValue 的值。这些都是利用 discord.js 定义好的方法来设置属性的值，它们有个共同特点，就是都是 “set” 开头的方法名称，我们叫它们 set 方法。

类似地，我们通过 `.getString("category")`、`.getInteger("amount")` 来获取相应字段的值，它们有个共同特点，就是都是 “get” 开头的方法名称，我们叫它们 get 方法。

set和get一般一起出现，如果只定义了一个会有特殊意义：

-   如果只有get，表示该属性只可读，不可写
    
-   如果只有set，表示该属性只可写，不可读


### ApplicationCommandOptionData 类

应用程序命令或子命令的选项。来自 https://discord.js.org/#/docs/discord.js/main/typedef/ApplicationCommandOptionData

请注意，为任何 snake_case (下划线风格）属性提供 camelCase (驼峰风格) 对应项的值将丢弃提供的 snake_case (下划线风格) 属性。

| 属性名 | 类型 | 可选？ | 描述 |
|---|---|---|---|
| .type | ApplicationCommandOptionType |  | Option 的类型 |
| .name | string |  | Option 的名称 |
| .nameLocalizations | Object < Locale, string > | Y | Option 名称的本地化 |
| .description | string |  | Option 说明 |
| .descriptionLocalizations | Object < Locale, string > | Y | Option 描述的本地化 |
| .required | boolean | Y | 是否必须需要该项  Option |
| .autocomplete | boolean | Y | 是否为 ApplicationCommandOptionType.String 、 ApplicationCommandOptionType.Integer 或 ApplicationCommandOptionType.Number 类型的 Option 启用自动完成 |
| .choices | Array < ApplicationCommandOptionChoice > | Y | 供用户选择的 Choices |
| .options | Array < ApplicationCommandOption > | Y | 如果此选项是子命令（组），则附加 Option |
| .channelTypes | Array < ChannelType > | Y | 当Option 类型为频道 (channel) 时，可以选择的允许的频道 (channel) 类型 |
| .minValue | number | Y | ApplicationCommandOptionType.Integer 或 ApplicationCommandOptionType.Number 类型的 Option 的最小值 |
| .maxValue | number | Y | ApplicationCommandOptionType.Integer 或 ApplicationCommandOptionType.Number 类型的 Option 的最大值 |
| .minLength | number | Y | ApplicationCommandOptionType.String 类型的 Option 的最小长度（最大 6000） |
| .maxLength | number | Y | ApplicationCommandOptionType.String 类型的 Option 的最大长度（最大 6000） |

对于上面提到的 ApplicationCommandOptionData 类，有效的 set 方法有：

.setName(name)

.setNameLocalizations(nameLocalizations)

.setDescription(description)

.setDescriptionLocalizations(descriptionLocalizations)

.setOptions(options)

而 get option 的值时，方法会返回`CommandInteractionOptionResolver`。

### ApplicationCommandOptionChoiceData 类
| 属性名 | 类型 | 可选？ | 描述 |
|---|---|---|---|
| .name | string | N | Choice 的名称 |
| .nameLocalizations | Object< Locale,string> | Y | 此 Choice 的本地化名称 |
| .value | string 或 number | N | Choice 的值 |


### CommandInteractionOptionResolver 类

命令交互选项的解析器。

方法（Methods）：
| 方法名 | 第一个参数 | 第二个参数 | 返回值类型 | 说明 |
|---|---|---|---|---|
| .get() | name | [required] | CommandInteractionOption | 按名称获取选项 |
| .getAttachment() | name | [required] | Attachment | 获取附件选项 |
| .getBoolean() | name | [required] | boolean | 获取一个布尔选项 |
| .getChannel() | name | [required] | GuildChannelThreadChannelAPIChannel | 获取频道选项 |
| .getFocused() | [getFull] | 无 | string 或 AutocompleteFocusedOption | 获取关注点选项 |
| .getInteger() | name | [required] | number | 获取整数选项 |
| .getMember() | name | 无 | GuildMemberAPIGuildMember | 获取服务器成员选项 |
| .getMentionable() | name | [required] | UserGuildMemberAPIGuildMember 或 RoleAPIRole | 获得一个服务器成员或身份组的选项 |
| .getMessage() | name | [required] | Message | 获取消息选项 |
| .getNumber() | name | [required] | number | 获取数字选项 |
| .getRole() | name | [required] | RoleAPIRole | 获取身份组选项 |
| .getString() | name | [required] | string | 获取字符串选项 |
| .getSubcommand() | [required] | 无 | string | 获取选定的子命令 |
| .getSubcommandGroup() | [required] | 无 | string | 获取选定的子命令组 |
| .getUser() | name | [required] | User | 获取用户选项 |

属性（Properties）：
| 属性名 | 类型 | 描述 |
|---|---|---|
| .client | Client | 实例化这个的 client |
| .data | Array< CommandInteractionOption > | 交互的 options 数组 |
| .resolved | < CommandInteractionResolvedData > | 交互解析数据 |

介绍了这么多个类，用代码举个例子 `commands/multiOption.js`：
```js
const { SlashCommandBuilder } = require('discord.js');

module.exports = {
	data: new SlashCommandBuilder()
	.setName('multioption')
	.setDescription('Multiple options.')
	.addStringOption(option => option.setName('input').setDescription('Enter a string'))
	.addIntegerOption(option => option.setName('int').setDescription('Enter an integer'))
	.addBooleanOption(option => option.setName('choice').setDescription('Select a boolean'))
	.addUserOption(option => option.setName('target').setDescription('Select a user'))
	.addChannelOption(option => option.setName('destination').setDescription('Select a channel'))
	.addRoleOption(option => option.setName('muted').setDescription('Select a role'))
	.addMentionableOption(option => option.setName('mentionable').setDescription('Mention something'))
	.addNumberOption(option => option.setName('num').setDescription('Enter a number'))
	.addAttachmentOption(option => option.setName('attachment').setDescription('Attach something')),

	async execute(interaction) {
		const string = interaction.options.getString('input');
		const integer = interaction.options.getInteger('int');
		const boolean = interaction.options.getBoolean('choice');
		const user = interaction.options.getUser('target');
		const member = interaction.options.getMember('target');
		const channel = interaction.options.getChannel('destination');
		const role = interaction.options.getRole('muted');
		const mentionable = interaction.options.getMentionable('mentionable');
		const number = interaction.options.getNumber('num');
		const attachment = interaction.options.getAttachment('attachment');

		await console.log(string, integer, boolean, user, member, channel, role, mentionable, number, attachment)
		await interaction.reply("done");
	},
};

```

我们输入如下图所示，注意有些项没填，所以输出将是 null：
![image](https://img2022.cnblogs.com/blog/2455224/202207/2455224-20220712150435869-1215124464.png)

控制台输出：
```json
222 null false null null null Role {
  guild: <ref *1> Guild {
    id: 'xxxxxxxxxxxxxx0988',
    name: '---隐私打码---',
    icon: null,
    features: [],
    commands: GuildApplicationCommandManager {
      permissions: [ApplicationCommandPermissionsManager],
      guild: [Circular *1]
    },
    members: GuildMemberManager { guild: [Circular *1] },
    channels: GuildChannelManager { guild: [Circular *1] },
    bans: GuildBanManager { guild: [Circular *1] },
    roles: RoleManager { guild: [Circular *1] },
    presences: PresenceManager {},
    voiceStates: VoiceStateManager { guild: [Circular *1] },
    stageInstances: StageInstanceManager { guild: [Circular *1] },
    invites: GuildInviteManager { guild: [Circular *1] },
    scheduledEvents: GuildScheduledEventManager { guild: [Circular *1] },
    available: true,
    shardId: 0,
    splash: null,
    banner: null,
    description: null,
    verificationLevel: 'NONE',
    vanityURLCode: null,
    nsfwLevel: 'DEFAULT',
    premiumSubscriptionCount: 0,
    discoverySplash: null,
    memberCount: 3,
    large: false,
    premiumProgressBarEnabled: false,
    applicationId: null,
    afkTimeout: 300,
    afkChannelId: null,
    systemChannelId: 'xxxxxxxxxxxxxx2250',
    premiumTier: 'NONE',
    explicitContentFilter: 'DISABLED',
    mfaLevel: 'NONE',
    joinedTimestamp: xxxxxxxxxx691,
    defaultMessageNotifications: 'ALL_MESSAGES',
    systemChannelFlags: SystemChannelFlags { bitfield: 0 },
    maximumMembers: 500000,
    maximumPresences: null,
    approximateMemberCount: null,
    approximatePresenceCount: null,
    vanityURLUses: null,
    rulesChannelId: null,
    publicUpdatesChannelId: null,
    preferredLocale: 'en-US',
    ownerId: 'xxxxxxxxxxxxxx4590',
    emojis: GuildEmojiManager { guild: [Circular *1] },
    stickers: GuildStickerManager { guild: [Circular *1] }
  },
  icon: null,
  unicodeEmoji: null,
  id: 'xxxxxxxxxxxxxx2137',
  name: 'BlogTest',
  color: 0,
  hoist: false,
  rawPosition: 1,
  permissions: Permissions { bitfield: 8n },
  managed: true,
  mentionable: false,
  tags: { botId: 'xxxxxxxxxxxxxx4939' }
} null 7.5 MessageAttachment {
  attachment: 'https://cdn.discordapp.com/ephemeral-attachments/xxxxxxxxxxxxxx0592/xxxxxxxxxxxxxx9424/1_BPSx-c--z6r7tY29L19ukQ.png',
  name: '1_BPSx-c--z6r7tY29L19ukQ.png',
  id: 'xxxxxxxxxxxxxx9424',
  size: 50120,
  url: 'https://cdn.discordapp.com/ephemeral-attachments/xxxxxxxxxxxxxx0592/xxxxxxxxxxxxxx9424/1_BPSx-c--z6r7tY29L19ukQ.png',
  proxyURL: 'https://media.discordapp.net/ephemeral-attachments/xxxxxxxxxxxxxx0592/xxxxxxxxxxxxxx9424/1_BPSx-c--z6r7tY29L19ukQ.png',
  height: 768,
  width: 1366,
  contentType: 'image/png',
  description: null,
  ephemeral: true
}
```

还是 `commands/multiOption.js`：

我们输入：
![image](https://img2022.cnblogs.com/blog/2455224/202207/2455224-20220712151051919-1157319136.png)

控制台输出：
```json
a1b2c3d4 null true null null null null null 9 null
```

### 结语

Options 和 Choices 作为斜杠命令参数很强大。