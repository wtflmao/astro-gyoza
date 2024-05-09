---
title: Bot in Discord with discord.js (4)
date: 2022-07-11
tags: [Discord bot, JS]
category: Discord.js
summary: 这是一份非常详细和系统化的利用 NodeJS 进行 Discord bot 开发的教程，本文对slash command /斜杠命令、注册斜杠命令的方法、斜杠命令的服务端逻辑进行说明，并完成了 /ping 命令及其变体。
---

## Chapter 5 - 交互：对斜杠命令回复的那些事儿（1）

斜杠命令是交互的一种。

我们在 Chapter 2 的 `commands/ping.js` 里已经完成了一项斜杠命令的建立：
```js
const { SlashCommandBuilder } = require('discord.js');

module.exports = {
	data: new SlashCommandBuilder()
		.setName('ping')
		.setDescription('Replies with Pong!'),
	async execute(interaction) {
		await interaction.reply("Pong!");
	},
};
```

代码建立了一项命令： `/ping`，作用是回复：`Pong!`。具体的解释已在 Chapter 4 中完成，不再赘述。

Discord 为开发人员提供了创建客户端集成斜杠命令的选项。 在本章中，我们将介绍如何使用 discord.js 响应这些命令。下面我们来看看如何把回复玩出花来。


### 注册斜杠命令

你至少需要在你的应用程序上注册一个斜杠命令才能继续往下走（如果你是跟着我的文章来的，那么你肯定注册过至少一项命令了）。注册斜杠命令在前文章节已说明过，不再赘述。


### 接收斜杠命令

接收斜杠命令交互在前文章节已说明过，不再赘述。（如果你是跟着我的文章来的，那么你的机器人肯定可以接收斜杠命令了）。


### 回复斜杠命令

discord.js 提供了不止一种回复命令的方法。最常见的方法是 `CommandInteraction#reply()`。

需要说明的是，一次 Discord 交互（interaction）的 token 有效期为 3 秒。因此，你需要在 3 秒内完成对斜杠命令的回复，否则按咱们的代码设计，机器人将报错。至于如何延长这个时间，后面会讲到。

比如 `commands/ping.js`：
```js
const { SlashCommandBuilder } = require('discord.js');

module.exports = {
	data: new SlashCommandBuilder()
		.setName('ping')
		.setDescription('Replies with Pong!'),
	async execute(interaction) {
	    // 这里回复了斜杠命令
		await interaction.reply("Pong!");
	},
};
```

执行这个斜杠命令，这里的回复应该是：
![image](https://img2022.cnblogs.com/blog/2455224/202207/2455224-20220711125819594-935091856.png)


### 短暂响应 Ephemeral responses

我不知道把 Ephemeral responses 翻译成“短暂回复”是否合适，字面意思的确是“短暂的回复”，不过，翻译成“临时回复”，也许更好。

短暂消息 / 临时消息（Ephemeral messages）是 Discord 机器人无需向用户发送 DM（私聊） 即可向用户发送私人消息的一种新方式。

这是一条只有命令发起者（用户自己）和机器人自己才可以看到的消息。 当用户选择无视（dismiss）它们、等待足够长的时间或重新启动 Discord 时，这些消息会消失。

短暂回复是回复的属性，给回复加上 `ephemeral: true` 属性，即可完成设置。

比如我们来个 `commands/pingEphemeral.js`：
```js
const { SlashCommandBuilder } = require('discord.js');

module.exports = {
	data: new SlashCommandBuilder()
		.setName('ping2')
		.setDescription('Replies with Pong, but ephemerally!'),
	async execute(interaction) {
	    // 这里回复了斜杠命令，是个短暂回复
		await interaction.reply({
		    content: "Pong!",
		    ephemeral: true,
		});
	},
};
```

执行效果是这样的：

![image](https://img2022.cnblogs.com/blog/2455224/202207/2455224-20220711165330459-644492374.png)

这里我们的 `ephemeral` 是 interaction.reply 的属性。interaction.reply 来自自己的一共有三个属性，我们会介绍到其他两个的。 interaction.reply 具有多种类型，具体是 content、nonce、embeds、components、attachments、files、tts 和 allowedMentions。这八个都是可选的，以后我们会介绍的。


### 修改响应 Editing responses

发送初始响应后，您可能出于各种原因想要编辑该响应（前提是交互令牌（interaction token）未过期失效）。

在初始响应之后，交互令牌的有效期为 15 分钟，因此这是您可以编辑响应和发送后续消息的时间范围。

修改响应回复可以通过 `CommandInteraction#editReply()` 方法来实现：

比如来个  `commands/pingEditing.js`：
```js
const { SlashCommandBuilder } = require('discord.js');
const wait = require('node:timers/promises').setTimeout; // 别忘了！！！

module.exports = {
	data: new SlashCommandBuilder()
		.setName('ping3')
		.setDescription('Replies with Pong, but will be edited.'),
    
	async execute(interaction) {

        // 先来个回复，作为初始回复
		await interaction.reply("Pong!");

        // 利用 wait 等待 2000 毫秒，即 2 秒
		await wait(2000);

        // 修改回复成新的字符串
		await interaction.editReply('Pong again!');
		},
};
```

注意我们用到了 `node:times/promises` 记为 wait，作为我们的延迟函数。所以第二行需要导包。

别忘了 wait 是在 async 里的，要达到故意延时的目的，需要 await。

![image](https://img2022.cnblogs.com/blog/2455224/202207/2455224-20220711172138742-1265191722.png)

![image](https://img2022.cnblogs.com/blog/2455224/202207/2455224-20220711172147641-1873380108.png)


### 延迟响应 Deferred responses

如前所述，在交互令牌（interaction token）变为无效之前，你有 3 秒钟的时间来响应交互。 但是，如果您有一个命令执行的任务需要超过 3 秒钟才能回复，该怎么办？

在这种情况下，你可以使用 `CommandInteraction#deferReply()` 方法。该方法触发 “<某某某应用程序> 正在响应......” 消息（由 Discord 指定，且与用户语言设置有关）并充当初始响应。 这使你可以在 15 分钟内完成任务，然后再做出响应。

由于调用延迟响应回复，会有个初始响应回复，所以你必须利用修改响应回复功能，通过修改初始回复，来进行真正的回复。

比如来个 `commands/pingDeferred.js`：
```js
const { SlashCommandBuilder } = require('discord.js');
const wait = require('node:timers/promises').setTimeout; // 注意这里！！！！！！

module.exports = {
	data: new SlashCommandBuilder()
		.setName('ping4')
		.setDescription('Replies with Pong, but deferred!'),
	async execute(interaction) {
	
	    // 延迟回复，此时会有个初始回复“应用程序正在响应”
	    await interaction.deferReply();
	    
	    // 利用导入的 wait，故意等待 4000 毫秒
		await wait(4000);
		
		// 进行真正的回复
		await interaction.editReply('Pong!');
	},
};
```

![image](https://img2022.cnblogs.com/blog/2455224/202207/2455224-20220711172408926-2108786535.png)

![image](https://img2022.cnblogs.com/blog/2455224/202207/2455224-20220711172414298-742066049.png)


### 多重响应 Follow-ups

如果你想发送多个响应而不是一个响应怎么办？ 您可以使用 `CommandInteraction#followUp()` 发送多个响应。

在初始响应之后，交互令牌的有效期为 15 分钟，因此这是您可以编辑响应和发送后续消息的时间范围。

比如我们来个 `commands/pingFollowUps.js`：

```js
const { SlashCommandBuilder } = require('discord.js');

module.exports = {
	data: new SlashCommandBuilder()
		.setName('ping5')
		.setDescription('Replies with Pong, but multiple responses!'),
	async execute(interaction) {
		await interaction.reply("Pong!");
		await interaction.followUp({content: "Pong again!", ephemeral: true});
		await interaction.followUp({content: "Pong again again!", ephemeral: false});
	},
};
```

![image](https://img2022.cnblogs.com/blog/2455224/202207/2455224-20220711173058563-931188594.png)

注意到我们的 Follow-ups 都是对机器人初始响应的回复，而不是对用户斜杠命令的回复。


### 回复带上超链接

我们可以给 `content` 字段使用 `[text](http://site.com)` 这样的形式来设置一个超链接。

比如：
```js
const { SlashCommandBuilder } = require('discord.js');

module.exports = {
	data: new SlashCommandBuilder()
		.setName('ping6')
		.setDescription('Replies with Pong, but with hyperlink!'),
	async execute(interaction) {
		await interaction.reply(
		    "Pong!\n" + "[bilibili](https://www.bilibili.com/video/BV1GJ411x7h7)"
		);
	},
};
```

效果图：
![image](https://img2022.cnblogs.com/blog/2455224/202207/2455224-20220711174526099-690908800.png)
