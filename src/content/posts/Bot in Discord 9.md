---
title: Bot in Discord with discord.js (9)
date: 2022-11-19
tags: [Discord bot, JS]
category: Discord.js
summary: 这是一份非常详细和系统化的利用 NodeJS 进行 Discord bot 开发的教程，本文介绍了什么是按钮 Button，Discord 客户端允许哪些按钮，以及如何设置按钮的文字、颜色、超链接以及可用性。
---

## Chapter 12 - 交互四大组件之：按钮

### 修改 `events/interactionCreate.js`

对应的 commit：
https://github.com/wtflmao/discord_bot_example/commit/eef8c0f3fb0ebed69c63eb776128acb28ceb8b3f
https://gitee.com/wtflmao/discord_bot_example/commit/eef8c0f3fb0ebed69c63eb776128acb28ceb8b3f

在文件的最后那块儿附近，找到最后一个 else，在这个 else 上面插入一种新情况，用于按钮：
```js
else if (interaction.isButton()) {
	console.log("a button!");
}
```

### 新建一个操作行 ActionRow

上来先导入需要的 `ActionRowBuilder`，用于建立操作行 ActionRow。

通过 `new ActionRowBuilder()` 新建。

```js
const { ActionRowBuilder } = require('discord.js');

let row = new ActionRowBuilder()
    .addComponents(
        ... // 此处确定你需要的组件
    );
```

### 新建按钮 Button

上来先导入需要的 `ActionRowBuilder` 和 `ButtonBuilder`。前者用于建立操作行，后者用于建立一个按钮。按钮需要被包含到一行操作行上去。

通过 `new ButtonBuilder()` 新建一个按钮。

```js
const { ActionRowBuilder, ButtonBuilder } = require('discord.js');

let row = new ActionRowBuilder()
    .addComponents(
        new ButtonBuilder()
	        // 设置按钮自定义 ID
            .setCustomId('button1')

			// 设置按钮标签，就是按钮上的字
            .setLabel('Click me!')

			// 设置按钮样式，具体在下一小节有讲
            .setStyle(ButtonStyle.Primary)

			// 设置按钮是否被禁用，这里的 false 表示未被禁用
            .setDisabled(false)

			// 设置一个 Emoji 前缀
            .setEmoji('😀'),
    );
```

这个按钮的样式如图：
![image](https://img2022.cnblogs.com/blog/2455224/202211/2455224-20221119140537206-293552005.png)

### 五种按钮的基本样式

按钮（Button）是组件（Component）的一种，通过 `components: []` 指定哪些按钮将发送出去。

一条消息里最多加入五个按钮。准确来说，是最多五个组件（components），按钮是组件的一种。

按钮有五种样式：Primary、Secondary、Success、Danger 和 Link，下面用 `commands/buttons/stylishButton.js` 来说明：

对应的 commit：
https://github.com/wtflmao/discord_bot_example/commit/21c5f65d81cedbce987a59dfe4c9db4251817d41#diff-81e417f8348606916071c665b42372b6e3a496eec5d6f934c129483d412b941b
https://gitee.com/wtflmao/discord_bot_example/commit/21c5f65d81cedbce987a59dfe4c9db4251817d41#diff-81e417f8348606916071c665b42372b6e3a496eec5d6f934c129483d412b941b

```js
const { ActionRowBuilder, ButtonBuilder, ButtonStyle, SlashCommandBuilder } = require('discord.js');

module.exports = {
    data: new SlashCommandBuilder()
        .setName('stylishbutton')
        .setDescription('Replies with a button, but stylish!'),

    async execute(interaction) {

        let button1 = [], button1d = [];

        // // 批量生成了五个按钮，保存到数组里，根据下标来确定消息内唯一的 customId
        for (let i = 1; i <= 5; i++) {
            button1.push(new ButtonBuilder()
                // 在同一条回复消息中，你需要确保所有 customId 是不重复的。
                .setCustomId(`b1_${i}`)
                .setLabel(`Click me(${i})!`)
                .setStyle(ButtonStyle.Primary),);
        }

        // 批量生成了四个按钮，保存到数组里，根据下标来确定消息内唯一的 customId
        for (let i = 1; i <= 4; i++) {
            button1d.push(new ButtonBuilder()
                // 在同一条回复消息中，你需要确保所有 customId 是不重复的。
                .setCustomId(`b1d_${i}`)
                .setLabel(`Click me(${i})!`)
                .setStyle(ButtonStyle.Primary)
                .setDisabled(true),);
        }

        // 在一行里塞了五个按钮
        const rowPrimary = new ActionRowBuilder()
            .addComponents(button1);

        const rowSecondary = new ActionRowBuilder()
            .addComponents(
                new ButtonBuilder()
                    .setCustomId('b2')
                    .setLabel("Click me!")
                    .setStyle(ButtonStyle.Secondary),
            );

        const rowSuccess = new ActionRowBuilder()
            .addComponents(
                new ButtonBuilder()
                    .setCustomId('b3')
                    .setLabel("Click me!")
                    .setStyle(ButtonStyle.Success),
                new ButtonBuilder()
                    .setCustomId('b3d')
                    .setLabel("Click me!")
                    .setStyle(ButtonStyle.Success)
                    .setDisabled(true),
            );

        const rowDanger = new ActionRowBuilder()
            .addComponents(
                new ButtonBuilder()
                    // 在同一条回复消息中，你需要确保所有 customId 是不重复的。
                    // 这里我们用来随机数生成函数来生成 ID，降低 ID 碰撞的概率。
                    .setCustomId(`b4_${Math.floor(Math.random() * 100000)}`)
                    .setLabel("Click me!")
                    .setStyle(ButtonStyle.Danger),
            );

        const rowLink = new ActionRowBuilder()
            .addComponents(
                new ButtonBuilder()
                    // you can't .setCustomId() to a Link button
                    .setLabel("Click me!")
                    .setStyle(ButtonStyle.Link)
                    .setURL("https://support.discord.com/hc/en-us"),
            );

        // 在一行里塞了四个按钮
        const rowPrimaryD = new ActionRowBuilder()
            .addComponents(button1d);

        const rowPrimary2D = new ActionRowBuilder()
            .addComponents(
                new ButtonBuilder()
                    .setCustomId(`owo`)
                    .setLabel("owo")
                    .setStyle(ButtonStyle.Primary)
                    .setDisabled(true),
            );

        const rowSecondaryD = new ActionRowBuilder()
            .addComponents(
                new ButtonBuilder()
                    .setCustomId('b2d')
                    .setLabel("Click me!")
                    .setStyle(ButtonStyle.Secondary)
                    .setDisabled(true),
            );

        const rowDangerD = new ActionRowBuilder()
            .addComponents(
                new ButtonBuilder()
                    .setCustomId('b4d')
                    .setLabel("Click me!")
                    .setStyle(ButtonStyle.Danger)
                    .setDisabled(true),
            );

        const rowLinkD = new ActionRowBuilder()
            .addComponents(
                new ButtonBuilder()
                    // 你不能对一个链接按钮设置 customId 属性
                    .setLabel("Click me!")
                    .setStyle(ButtonStyle.Link)
                    .setURL("https://support.discord.com/hc/en-us")
                    .setDisabled(true),
            );

        // 这条消息里我们插了三行 ActionRow 进去。
        await interaction.reply({ content: 'These are some Primary buttons:', components: [rowPrimary, rowPrimaryD, rowPrimary2D], ephemeral: false });

        // 这条消息里我们插了两行 ActionRow 进去。
        await interaction.followUp({ content: 'These are two Secondary buttons. They are NOT on the same row:', components: [rowSecondary, rowSecondaryD], ephemeral: false });

        // 这条消息里我们插了一行 ActionRow 进去。
        await interaction.followUp({ content: 'These are two Success buttons. They are on the same row:', components: [rowSuccess], ephemeral: false });

        // 这条消息里我们插了两行 ActionRow 进去。
        await interaction.followUp({ content: 'These are two Danger buttons. They are NOT on the same row:', components: [rowDanger, rowDangerD], ephemeral: false });

        // 这条消息里我们插了两行 ActionRow 进去。
        await interaction.followUp({ content: 'These are two Link buttons. They are NOT on the same row:', components: [rowLink, rowLinkD], ephemeral: false });
    },
};
```

使用按钮样式时，要注意第一行要导入 ButtonStyle。

我们可以看到，一个回复消息，其实有（至少）四个域：`content`、`components` 、 `embeds` 和 `ephemeral`。其中，`content` 是必选的，其余是可选的。当只有 `content` 时，"content" 标签可以省略，如：`await interaction.reply("1+1=2");`。

效果图：
![image](https://img2022.cnblogs.com/blog/2455224/202211/2455224-20221119143142765-1071464031.png)

只有链接按钮可以有一个 url。链接按钮不能有 customId，并且在单击时不发送交互事件。

通过 `.setDisabled(true)` 为一个按钮设置不可点击的属性，这样用户就不能点击这个按钮了。但是你不能向一个已经建成的（即`new ActionRowBuilder()`过的）按钮变量设置新的属性，你必须重新 new 一个按钮出来。

### 带有前缀 Emoji 的按钮

下面用 `commands/buttons/emojiButton.js` 来说明：

对应的 commit：
https://github.com/wtflmao/discord_bot_example/commit/21c5f65d81cedbce987a59dfe4c9db4251817d41#diff-b33acfdadb6c12912fe644066f8ef7e2852f86016539498309b0e2c08b3f5933
https://gitee.com/wtflmao/discord_bot_example/commit/21c5f65d81cedbce987a59dfe4c9db4251817d41#diff-b33acfdadb6c12912fe644066f8ef7e2852f86016539498309b0e2c08b3f5933

```js
const { ActionRowBuilder, ButtonBuilder, ButtonStyle, SlashCommandBuilder } = require('discord.js');

module.exports = {
    data: new SlashCommandBuilder()
        .setName('emojibutton')
        .setDescription('Replies with a button, bu stylish!'),

    async execute(interaction) {

        const row = new ActionRowBuilder()
            .addComponents(
                new ButtonBuilder()
                    .setCustomId('b1')
                    .setLabel('Click me!')
                    .setStyle(ButtonStyle.Primary)
                    .setEmoji('😀'),
                new ButtonBuilder()
                    .setCustomId('b2')
                    .setLabel("Click me!")
                    .setStyle(ButtonStyle.Secondary)
                    .setEmoji('😁'),
                new ButtonBuilder()
                    .setCustomId('b3')
                    .setLabel("Click me!")
                    .setStyle(ButtonStyle.Success)
                    .setEmoji('🤣'),
                new ButtonBuilder()
                    .setCustomId(`b4`)
                    .setLabel("Click me!")
                    .setStyle(ButtonStyle.Danger)
                    .setEmoji('😇'),
                new ButtonBuilder()
                    // you cant .setCustomId() to a Link button
                    .setLabel("Click me!")
                    .setStyle(ButtonStyle.Link)
                    .setURL("https://support.discord.com/hc/en-us")
                    .setEmoji('🤪'),
            );

        // 一条消息里最多五行 ActionRow，每一行 ActionRow 里最多插入五个包括按钮在内的组件
        await interaction.reply({ content: 'There are some buttons:', components: [row], ephemeral: false });},
};
```

效果图：
![image](https://img2022.cnblogs.com/blog/2455224/202211/2455224-20221119140812249-1843561049.png)

注意，这里 `.setEmoji()` 只能设置一个 emoji，且只会出现在按钮 Label 的前面。

还有，一条消息里最多五行 ActionRow，每一行 ActionRow 里最多插入五个按钮，准确来说，是最多五个组件（components），按钮是组件的一种。如需五个以上的按钮，可以尝试回复多次，比如 `.followUp()`。

### 带有嵌套元素和按钮的回复

通过 `new EmbedBuilder()` 新建一个嵌套元素。注意第一行要导入 EmbedBuilder。

通过 `embeds: []` 指定哪些嵌套元素将发送出去。

嵌套元素不是按钮专属，嵌套元素其实和按钮等组件是平级的。

下面的源码注释我个人认为该有的都有了，清晰明了。就不再在这里费笔墨再写一遍了。

下面用 `commands/buttons/embeddedbutton.js` 来说明：

对应的 commit：
https://github.com/wtflmao/discord_bot_example/commit/21c5f65d81cedbce987a59dfe4c9db4251817d41#diff-f0a9d6890a4e8d5de14146749628d46fc9d5126a38f635f266993e4d35c4dcfa
https://gitee.com/wtflmao/discord_bot_example/commit/21c5f65d81cedbce987a59dfe4c9db4251817d41#diff-f0a9d6890a4e8d5de14146749628d46fc9d5126a38f635f266993e4d35c4dcfa

```js
const { ActionRowBuilder, ButtonBuilder, ButtonStyle, EmbedBuilder, SlashCommandBuilder } = require('discord.js');

let updatedBtnMsg = new Map();

module.exports = {
    data: new SlashCommandBuilder()
        .setName('embeddedbutton')
        .setDescription('Replies with a button, but embedded!'),

    async execute(interaction) {
        updatedBtnMsg.set("0", false);

        const row = new ActionRowBuilder()
            .addComponents(
                new ButtonBuilder()
                    // 随机化 Id 以求好运
                    .setCustomId(`b3_${Math.floor(Math.random() * 100000)}`)
                    .setLabel("Got it! Dismiss.")
                    .setStyle(ButtonStyle.Success),
            );

        const rowD = new ActionRowBuilder()
            .addComponents(
                new ButtonBuilder()
                    .setCustomId(`b3d`)
                    .setLabel("Got it! Dismiss.")
                    .setStyle(ButtonStyle.Success)
                    .setDisabled(true),
            );

        const hc = new EmbedBuilder()
            .setColor(0x0099FF)
            .setTitle(">>>Discord Help Center<<<")
            .setURL('https://support.discord.com/hc/en-us')
            .setDescription("Need help? We've got your back.");

        await interaction.reply({ content: 'R U seeking for assistance using Discord?', components: [row], embeds: [hc], ephemeral: true });

        const filter = i => {
            // 这里我们用到了下面会将的“收集器”，准确来说是“基本消息组件收集器”
            // filter 是收集器 collector 的一个过滤器。
            // 如果该消息早已被其他收集器实例收集走了，那就返回 false，表示本收集器拒绝收集该消息。这在并发场景下很好用。
            if (updatedBtnMsg.has(i.message.id)) {
                return false;
            } else { // 运行到这条 else 分支，意味着该消息未被曾收集过。
                if (i.customId.startsWith('b3')) {
                    // 我们不需要验证按钮消息的发起者和按按钮的人是否是同一个人，
	                // 因为这俩人肯定是同一个人，因为按钮消息已被设置为 ephemeral，
	                // 甚至不会有外人能看到这条消息，更别说按这个按钮了
                    updatedBtnMsg.set(i.message.id, true);
                    return true;
                } else {
                    // 这条分支意味着不满足我们的要求 i.customId.startsWith('b3') === true
                    // 即按下的按钮的 Id 不是 b3 打头的。我们不要。
                    return false;
                }
            }
        };

        const collector = interaction.channel.createMessageComponentCollector({
            filter,
            max: 1, // 我们只想让收集器收集一个满足 filter 条件的 Interaction
		            // 如果你想只针对特定组件类型进行收集的话，使用 componentType 标签
		            // 如果你想设置收集的最大组件数，使用 maxComponents
		            // 最大交互用户数，使用 maxUsers
            time: 20 * 1000, // 20.000 secs，以毫秒为单位运行收集器多长时间，不设置则一直收集
						    // 如果要设置不活动后停止收集器多长时间（以毫秒为单位），使用 idle
        });

        collector.on('collect', async i => {
            // 明确地令 component 数组为空数组，可以在 .update() 时删除该消息的任何组件。
            // 明确地令 embeds 数组为空数组，可以在 .update() 时删除该消息的任何嵌套元素。
            await i.update({ content: `Nice.`, components: [rowD], embeds: []});
        });

        collector.on('end', collected => {
            console.log(`Collected ${collected.size} items`);

            // 为了确保数组下标访问[0]不越界，我们这里要求数组不为空
            if (collected.size > 0) {
                console.log((Array.from(collected.values()))[0].customId);
            }
        });

    },
};
```

效果图：
![image](https://img2022.cnblogs.com/blog/2455224/202211/2455224-20221119141119800-67771465.png)
按下按钮后：
![image](https://img2022.cnblogs.com/blog/2455224/202211/2455224-20221119141123741-159560186.png)

### 其他对按钮的操作

-   `reply()`
-   `editReply()`
-   `deferReply()`
-   `fetchReply()`
-   `deleteReply()`
-   `followUp()`

我们这里展示一下 `deferReply()` 和 `deleteReply()`。

下面的源码注释我个人认为该有的都有了，清晰明了。就不再在这里费笔墨再写一遍了。

下面用 `commands/buttons/deferAndDelete.js` 来说明：

对应的 commit：
https://github.com/wtflmao/discord_bot_example/commit/21c5f65d81cedbce987a59dfe4c9db4251817d41#diff-16f8992fc6c047410730afb876fa5a8a9c8ed7b312c9f991f52bdb9ca8f85b8f
https://gitee.com/wtflmao/discord_bot_example/commit/21c5f65d81cedbce987a59dfe4c9db4251817d41#diff-16f8992fc6c047410730afb876fa5a8a9c8ed7b312c9f991f52bdb9ca8f85b8f

```js
const { ActionRowBuilder, ButtonBuilder, ButtonStyle, SlashCommandBuilder } = require('discord.js');
const wait = require('node:timers/promises').setTimeout;

let updatedBtnMsg = new Map();

module.exports = {
    data: new SlashCommandBuilder()
        .setName('ddbutton')
        .setDescription('Replies with a button, but deferred, also self-delete!'),

    async execute(interaction) {
        updatedBtnMsg.set("0", false);

        const row = new ActionRowBuilder()
            .addComponents(
                new ButtonBuilder()
                    // Randomize the button's customId for good luck, like "b1_90372" or "b1_1827".
                    .setCustomId(`b1_${Math.floor(Math.random() * 100000)}`)
                    .setLabel("Update weather data")
                    .setStyle(ButtonStyle.Primary),
            );

        await interaction.reply({ content: 'This is a button:', components: [row], embeds: [], ephemeral: false });

        const filter = i => {
            // If the target message has been collected and edited by the other collector instance,
            // then return false, telling the current collector that "do not collect this msg, or else, DIE!!", LOL.
            if (updatedBtnMsg.has(i.message.id)) {
                return false;
            } else {
                // Here, this msg hasn't been collected yet.
                if (i.customId.startsWith('b1') && (i.user.id === interaction.user.id)) {
                    // Here, it fulfills our requirements: a.is a DANGEROUS button(customId starts with "b4_"), and b. the "button msg"'s author is the person that presses the button
                    // Put this <i.message.id, true> record into the Map, so that we can no longer collect it
                    // (like when many active collectors all see the same button interaction fulfills its requirements, then they all wanna edit the same msg, but to only find the interaction has already been acknowledged and the bot be halted.)
                    updatedBtnMsg.set(i.message.id, true);
                } else {
                    // Here, it doesn't meet all the requirements we need, so do not collect it.
                    return false;
                }
            }
            updatedBtnMsg.delete(i.message.id);
            return true;
        };

        const collector = interaction.channel.createMessageComponentCollector({
            filter,
            time: 18000, // 18 secs
        });

        collector.on('collect', async i => {
            // use .deferUpdate() to simulate an API response process from a weather channel.
            await i.deferUpdate();
            await i.editReply({ content: `Updating...`, components:[] });
            await wait(3350); //wait 3.350 secs
            await i.editReply({ content: `Las Vegas, NV\nAs of ${Math.floor(Math.random() * 11) + 1}:${Math.floor(Math.random() * 59)} am PST\n44°F\nFair\nDay ${Math.floor(Math.random() * 10) + 66}°F • Night ${39 - Math.floor(Math.random() * 10)}°F`, embeds: [], components: [row] });
        });

        collector.on('end', collected => {
            console.log(`Collected ${collected.size} items`);

            // To make sure there's at least one item in the array,
            // to prevent accessing the first element of an empty array, which leads to out-of-bounds memory access
            if (collected.size > 0) {
                // To delete the whole reply, do .deleteReply() for the last collected item, not all the items.
                (Array.from(collected.values()))[collected.size - 1].deleteReply();
                console.log((Array.from(collected.values()))[0].customId);
            }
        });

    },
};
```

效果图：
刚发出斜杠命令交互后：![image](https://img2022.cnblogs.com/blog/2455224/202211/2455224-20221119141436425-1654461385.png)
点击 Update weather button 后：
![image](https://img2022.cnblogs.com/blog/2455224/202211/2455224-20221119141504850-2016161105.png)
结果：![image](https://img2022.cnblogs.com/blog/2455224/202211/2455224-20221119141525899-129872057.png)

### 消息组件收集器：处理按钮被按下后的交互操作

我们需要建立一个 Discord.js 为我们提供的一种“收集器”（Collector）来监听一种交互：按钮被按下这种交互。

其实消息组件收集器可以收集任何消息组件，但是这里我们只在乎按钮的交互。

消息组件收集器，这种收集器有个特点，就是 Discord 期望你的机器人在 3 秒内对所有交互做出响应，即使是您不想收集的交互也是如此。出于这个原因，实践中的许多情况下，在 filter 中上来就来个 `.deferUpdate()`，甚至放弃使用 filter，直接让 collector 来者不拒，上来同样先 `.deferUpdate()`。

我们这里没有这么做。

下面的源码注释我个人认为该有的都有了，清晰明了。就不再在这里费笔墨再写一遍了。

下面用 `commands/buttons/deferAndDelete.js` 来说明：

对应的 commit：
https://github.com/wtflmao/discord_bot_example/commit/21c5f65d81cedbce987a59dfe4c9db4251817d41#diff-da8b6abce1aa8943cff70045dec60a8d70461e8f6cb0e5c9327e5235f5ead4c5
https://gitee.com/wtflmao/discord_bot_example/commit/21c5f65d81cedbce987a59dfe4c9db4251817d41#diff-da8b6abce1aa8943cff70045dec60a8d70461e8f6cb0e5c9327e5235f5ead4c5

```js
const { ActionRowBuilder, ButtonBuilder, ButtonStyle, EmbedBuilder, SlashCommandBuilder } = require('discord.js');
const wait = require('node:timers/promises').setTimeout;

let updatedBtnMsg = new Map();

module.exports = {
	data: new SlashCommandBuilder()
       .setName('button')
       .setDescription('Replies with a button!'),

	async execute(interaction) {
		updatedBtnMsg.set("0", false);

		const rowDanger = new ActionRowBuilder()
			.addComponents(
				new ButtonBuilder()
					// Randomize the button's customId for good luck, like "b4_90372" or "b4_1827".
					.setCustomId(`b4_${Math.floor(Math.random() * 100000)}`)
					.setLabel("Click me!")
					.setStyle(ButtonStyle.Danger),
			);

		const rowDangerD = new ActionRowBuilder()
			.addComponents(
				new ButtonBuilder()
					// Because the button is disabled, no one can actually click that.
					// So there's no avail to randomize its customId.
					.setCustomId('b4d')
					.setLabel("YOU CLICKED ME!")
					.setStyle(ButtonStyle.Danger)
					.setDisabled(true),
			);

		const rowSecD = new ActionRowBuilder()
			.addComponents(
				new ButtonBuilder()
					// Because the button is disabled, no one can actually click that.
					// So there's no avail to randomize its customId.
					.setCustomId('b2d')
					.setLabel("It's fine.")
					.setStyle(ButtonStyle.Secondary)
					.setDisabled(true),
			);

		const embed = new EmbedBuilder()
			.setColor(0x0099FF)
			.setTitle("hhzm's blog")
			.setURL('https://www.cnblogs.com/hhzm/')
			.setDescription("hhzm's cnblog site");

		await interaction.reply({ content: 'This is a Danger button:', components: [rowDanger], embeds: [embed], ephemeral: false });

		const filter = i => {
			// If the target message has been collected and edited by the other collector instance,
			// then return false, telling the current collector that "do not collect this msg, or else, DIE!!", LOL.
			if (updatedBtnMsg.has(i.message.id)) {
				return false;
			} else {
				// Here, this msg hasn't been collected yet.
				if (i.customId.startsWith('b4') && (i.user.id === interaction.user.id)) {
					// Here, it fulfills our requirements: a.is a DANGEROUS button(customId starts with "b4_"), and b. the "button msg"'s author is the person that presses the button
					// Put this <i.message.id, true> record into the Map, so that we can no longer collect it
					// (like when many active collectors all see the same button interaction fulfills its requirements, then they all wanna edit the same msg, but to only find the interaction has already been acknowledged and the bot be halted.)
					updatedBtnMsg.set(i.message.id, true);
					return true;
				} else {
					// Here, it doesn't meet all the requirements we need, so do not collect it.
					return false;
				}
			}
		};

		// here we used interaction.channel.createMessageComponentCollector()
		// rather than message.createMessageComponentCollector()
		// to show you guys how a collector that focuses on channel works.
		const collector = interaction.channel.createMessageComponentCollector({
			filter,
			max: 1, // We only want this collector instance collects 1 eligible message component
			time: 15 * 1000, // 15.000 secs
		});

		collector.on('collect', async i => {
			await i.update({ content: `A DANGEROUS button was clicked! ${i.customId}`, embeds: [], components: [rowDangerD] });
			await wait(6969);

			// use .editReply() rather than another .update() here
			// Passing an empty array to the components option will remove any buttons after one has been clicked.
			// Passing an empty array to the embeds option will remove any embeds after one has been clicked.
			await i.editReply({ content: `Danger mitigated.`, components: [rowSecD], embeds: [] });
		});

		collector.on('end', collected => {
			console.log(`Collected ${collected.size} items`);

			// To make sure there's at least one item in the array,
			// to prevent accessing the first element of an empty array, which leads to out-of-bounds memory access
			if (collected.size > 0) {
				console.log((Array.from(collected.values()))[0].customId);
			}
		});

	},
};
```

效果图：
![image](https://img2022.cnblogs.com/blog/2455224/202211/2455224-20221119141809776-952418961.png)
然后
![image](https://img2022.cnblogs.com/blog/2455224/202211/2455224-20221119144938255-2082627602.png)
然后
![image](https://img2022.cnblogs.com/blog/2455224/202211/2455224-20221119141819927-1945822726.png)

需要注明的是，我们这里的收集器“监听”的是频道（Channel）里的组件事件。消息组件收集器还有另一种监听对象：消息（Message）。

监听某个 Channel：
```js
const collector = interaction.channel.createMessageComponentCollector();
```
此时，只要收集器仍在工作，则该收集器的目标频道里的所有 MessageComponent 事件都可以成为收集器的收集对象。比如本例中，哪怕之前机器人已重启过好几次了，只要有人执行了斜杠命令 /button，那么哪怕是（在这个频道里）上个月的按钮，只要满足了 filter 的条件，也能够被收集器收集到，进而干该干的事。

监听某个 Message：
```js
message = await interaction.fetchReply();
const collector = message.createMessageComponentCollector();
```
此时，只要监听某条特定消息的收集器停止了工作，那么在本例中，没有其他的收集器可以在收集到该按钮的组件事件了，哪怕满足 filter 的条件了也不行。本例使用的是对象是频道的收集器。

你可以自由选择是使用对象是频道的消息组件收集器，还是使用对象是消息的消息组件收集器，来简化你的任务。