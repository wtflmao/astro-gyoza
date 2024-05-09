---
title: Bot in Discord with discord.js (8)
date: 2022-07-15
tags: [Discord bot, JS]
category: Discord.js
summary: 这是一份非常详细和系统化的利用 NodeJS 进行 Discord bot 开发的教程，本文介绍了什么是事件处理 Event handling。NodeJS 本身就是一个事件驱动的框架，DiscordJS 也完全按照事件驱动的思路进行构建的，因此，学习事件处理的方法是很重要的。除此之外，还完成了自动完成 Autocomplete，可类比于搜索引擎的自动填充。
---
## Chapter 10 - 事件处理 Event handling

这一章只是根据新的 discord.js v14.8.0，对已有文件进行小修小补。

> 如果你是跟着本教程前几章来的，不要跳过本章！

根据官方的说法，Node.js 使用事件驱动的架构，使得在特定事件发生时执行特定代码成为可能。discord.js 库就充分利用了这一点。

#### 修改后的 `events/interactionCreate.js`
```js
const { Events } = require('discord.js');

module.exports = {
	name: Events.InteractionCreate,
	execute(interaction) {
		console.log(`${interaction.user.tag} in #${interaction.channel.name} triggered an interaction.`);
	},
};
```

#### 修改后的 `events/ready.js`
```js
const { Events } = require('discord.js');

module.exports = {  
   name: Events.ClientReady,  
   once: true,  
   execute(client) {  
      console.log(`Ready! Logged in as ${client.user.tag}`);  
   },  
};
```

#### 修改后的 `index.js`

这个文件改动的地方不算多，但是把完整源码粘贴过来又显得冗杂，因此，我把这次的 git commit 地址放在这里，大家可以自己点开去看。

https://github.com/wtflmao/discord_bot_example/commit/f6eeddd2d975cb43803075e884bbc4d7174f6dfe

如果上面的 Github 链接在国内打不开，你可以使用下面的 Gitee 链接：

https://gitee.com/wtflmao/discord_bot_example/commit/f6eeddd2d975cb43803075e884bbc4d7174f6dfe

## Chapter 11 自动完成 Autocomplete

自动完成允许根据用户的输入动态地向用户提供一系列值，而不是依赖于静态选项。你需要学习 Chapter 6 的 Option 才能继续学习本章。换句话说，Autocomplete 只是服务于 Option 的。

用户被自动完成提供了一些 choices，但是用户可以键入执行除了给定的 choices 以外的其他任意合法值。想像一个新闻搜索机器人，`query` 项用于接收关键词。用户可以键入任何字符串，但是为了方便用户使用，机器人维护了一个“24小时热搜前一百”的列表，用户只需输入某个字或词，就能把列表里所有相关的新闻标题作为选项展示给用户，以减少用户输入量。当然用户也可以输入自己任何想搜的，不受热搜的限制。

在使用自动完成之前，一定要根据 Chapter 9 的内容，修改你的机器人代码，因为 Chapter 9 对 index.js 和 events/interactionCreate.js 做了大改。本章开始的代码直接跑在前八章里是会报错的。

自动完成举例：`commands/autocomplete.js`
https://github.com/wtflmao/discord_bot_example/commit/625dcad5fc03f28a0faf14bf66e35400c4ad767b
https://gitee.com/wtflmao/discord_bot_example/commit/625dcad5fc03f28a0faf14bf66e35400c4ad767b
```js
const { SlashCommandBuilder } = require('discord.js');

module.exports = {
	data: new SlashCommandBuilder()
		.setName('autocomplete')
		.setDescription('Autocomplete test.')
		.addStringOption(option =>
			option.setName("category")
				.setDescription("The category you'd like to dive in")
				.setAutocomplete(true)),
	async autocomplete(interaction) {
		
		const focusedValue = interaction.options.getFocused();
		const choices = ['beer', 'coffee', 'milk', 'apple', 'banana', 'tea', 'zebra'];
		const filtered = choices.filter(choice => choice.includes(focusedValue));
		await interaction.respond(
			filtered.map(choice =>({ name: choice, value: 'v_' + choice })),
		).then(() => console.log('Successfully responded to the autocomplete interaction'))
 		.catch(console.error);
	},
	async execute(interaction) {
		const category = interaction.options.getString("category");
		await interaction.reply(`You acquired some ${category}.`);
	},
};
```

解析：通过 `.setAutoComplete(true)` 来开启一个 Option 的自动完成。

在 `autocomplete()` 函数里：

通过 `interaction.options.getFocused()` 获取用户在该域内实时键入的值。

我们用 choices 数组维护了一些用户可能会输入的字符串，通过 `choices.filter(choice => choice.includes(focusedValue)` （实时键入的字符串是否是 choices 数组某个元素的字串与否）来判断是否要将该元素加入到针对目前实时键入的值的自动完成响应里去。比如，用户输入 "ee"，能匹配 "beer" 和 "coffee"。

`interaction.respond()` 用于向用户响应自动完成列表。

程序执行到现在，用户应还未回车执行命令。

在 `execute()` 函数里：

这里是我们最常见的斜杠命令的函数主题了，执行到这里意味着用户已经敲击了回车，执行了斜杠命令。

这里是实时输入"ee"时，根据我们 filter 设定的规则，自动完成匹配到的：
![image](https://img2022.cnblogs.com/blog/2455224/202211/2455224-20221117172041533-543123094.png)

当我们执行 `/autocomplete category: news`时：
![image](https://img2022.cnblogs.com/blog/2455224/202211/2455224-20221117172055915-1546936651.png)

用户的最终输入可以与自动完成的无关，就像没有自动完成的普通 Option 那样对待：
执行 `/autocomplete category: news`
![image](https://img2022.cnblogs.com/blog/2455224/202211/2455224-20221117172117377-303406663.png)

我们没有对 category 域设置必填，等价于 .setRequired(false)。因此不理会这个域执行命令也是可以的：
执行 `/autocomplete`
![image](https://img2022.cnblogs.com/blog/2455224/202211/2455224-20221117172123011-1830908788.png)

### 多个 Option 的 Autocomplete

上代码：`commands/autocomplete2.js`
https://github.com/wtflmao/discord_bot_example/commit/625dcad5fc03f28a0faf14bf66e35400c4ad767b
https://gitee.com/wtflmao/discord_bot_example/commit/625dcad5fc03f28a0faf14bf66e35400c4ad767b
```js
const { SlashCommandBuilder } = require('discord.js');

module.exports = {
	data: new SlashCommandBuilder()
		.setName('autocomplete2')
		.setDescription('Multi-Autocomplete test.')
		.addStringOption(option =>
			option.setName("category")
				.setDescription("The category you'd like to dive in")
				.setAutocomplete(true)
				.setRequired(true))
		.addNumberOption(option =>
			option.setName("amount")
				.setDescription("How many you want")
				.setAutocomplete(true)
				.setRequired(true)),
	async autocomplete(interaction) {
		
		const focusedOption = interaction.options.getFocused(true);

		if(focusedOption.name === "category") {

			const choices = ['beer', 'coffee', 'milk', 'apple', 'banana', 'tea', 'zebra'];
			const filtered = choices.filter(choice => choice.includes(focusedOption.value));
			await interaction.respond(
				filtered.map(choice =>({ name: choice, value: choice })),
			).then(() => console.log('Successfully responded to the autocomplete interaction'))
	 		.catch(console.error);

		}

		if(focusedOption.name === "amount") {

			const choices = [12, 24, 48, 81, 9];
			const filtered = choices.filter(choice => `${choice}`.includes(`${focusedOption.value}`));
			await interaction.respond(
				filtered.map(choice =>({ name: `${choice}`, value: choice })),
			).then(() => console.log('Successfully responded to the autocomplete interaction'))
	 		.catch(console.error);
 		
		}
	},
	async execute(interaction) {
		const category = interaction.options.getString("category");
		const amount = interaction.options.getNumber("amount");
		// P.S. .getUser(), .getMember(), .getRole(), .getChannel(), .getMentionable() and .getAttachment() methods are not available to autocomplete interactions.
		// but .getBoolean(), .getInteger() are available
		await interaction.reply(`You acquired ${amount} of ${category}.`);
	},
};
```

我们设置了两个 Option，一个叫 category，类型是字符串型；另一个叫 amount，类型是 Number 型。

针对字符串和 Number 的值处理是不同的，这也使得我们用 `if(focusedOption.name === "category")` `if(focusedOption.name === "amount")` 来分别处理他们。

注意，对 `autocomplete()` 的调用是实时关于用户键入的。这意味着，用户（以一个不太快的速度）每敲一个字符，Discord 客户端都会向机器人后台请求一遍新的 Autocomplete 结果列表。

执行 `/autocomplete2 category: tea amount: 24`：
![image](https://img2022.cnblogs.com/blog/2455224/202211/2455224-20221117174254952-147312660.png)

执行 `/autocomplete2 category: zombie amount: 7`：
![image](https://img2022.cnblogs.com/blog/2455224/202211/2455224-20221117174259390-53278945.png)

允许设置 Autocomplete 的只有 string、integer、boolean 和 number 四类 Option。
```js
const string = interaction.options.getString('input');
const integer = interaction.options.getInteger('int');
const boolean = interaction.options.getBoolean('choice');
const number = interaction.options.getNumber('num');
```

像 .getUser(), .getMember(), .getRole(), .getChannel(), .getMentionable() and .getAttachment()，都不能设置自动完成。

#### 其他注意的地方

-   与其他应用程序命令交互一样，自动完成交互必须在 3 秒内收到响应。
-   你不能推迟对自动完成交互的响应。 如果你正在处理异步建议，例如等待来自 API 的数据，请考虑保留本地缓存。
-   用户选择一个值并发送命令后，将作为常规接收 `ChatInputCommandInteraction` 与 Option 的值。
-   你一次最多只能回复 25 项，但如果超过这个数，则可能意味着你应该修改过滤器以进一步缩小选择范围。