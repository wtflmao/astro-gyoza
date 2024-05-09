---
title: Bot in Discord with discord.js (10)
date: 2022-11-21
tags: [Discord bot, JS]
category: Discord.js
summary: 这是一份非常详细和系统化的利用 NodeJS 进行 Discord bot 开发的教程，本文介绍了什么是下拉菜单 Select Menu，包括单选和多选。
---

### 注意

本文起，项目已从 Discord.js v14.6.0 更新至 v14.7.1。

你可以在项目目录中执行 `npm install discord.js@v14.7.1` 来更新依赖。

## Chapter 13 - 交互四大组件之：下拉菜单 Select Menu

建议你先学习上一章的按钮，再来学习这章的下拉菜单，因为上一章已经比较详细地介绍了 MessageComponentCollector，这里不会重复介绍。

### 修改 `events/interactionCreate.js`

在文件的最后那块儿附近，找到最后一个 else，在这个 else 上面插入一种新情况，用于下拉菜单：
```js
...
else if (interaction.isStringSelectMenu()) {
	console.log("a string select menu!");
} else if (interaction.isUserSelectMenu()) {
	console.log("a user select menu!");
} else if (interaction.isRoleSelectMenu()) {
	console.log("a role select menu!");
} else if (interaction.isChannelSelectMenu()) {
	console.log("a channel select menu!");
} else if (interaction.isMentionableSelectMenu()) {
	console.log("a mentionable select menu!");
}
...
```

对应的 commit：
https://github.com/wtflmao/discord_bot_example/commit/f306c472de37580acc14fabfe6d09dca07ff59a6#diff-5ee61780af95f78329f68020bc0a8de616c98b4b416554d60606744cffa51af2

### 建立一个下拉菜单

下拉菜单（Select Menu）是交互组件（Component）的一种。

在新建一个下拉菜单之前，你需要先导入这里用到的 ActionRowBuilder 和 StringSelectMenuBuilder。

```js
const { ActionRowBuilder, StringSelectMenuBuilder } = require('discord.js');

const row = new ActionRowBuilder()
    .addComponents(
        new StringSelectMenuBuilder()
            .setCustomId('select_1')
            .setPlaceholder('Nothing selected')
            .addOptions(
                {
                    label: 'Las Vegas, NV',
                    description: 'This is a description',
                    value: 'Las_Vegas_NV_US',
                },
                {
                    label: 'Denver, CO',
                    description: 'This is also a description',
                    value: 'Denver_CO_US',
                },
            ),
    );
```

这个下拉菜单长这样：
![image](https://img2022.cnblogs.com/blog/2455224/202211/2455224-20221121130819101-1051976461.png)

需要指出的是，customId 是一个由程序员指定的长度不超过 100 字符的字符串。尽量不要让它们的 customId 与其他交互组件的 customId 重复，从而让 filter 更轻松地工作。

每条消息最多可以有五个 ActionRow，一个 ActionRow 中可以有一个选择菜单。

你可以为一条下拉菜单消息指定 `ephemeral: true` 和 `embeds: []` 来丰富样式。

### 如何回复下拉菜单事件：交互事件收集器

建立收集器前，先建立 filter。具体看上一章。不够你也可以直接看下面例子的代码，应该也能看懂。

上一章按钮里讲过，你可以在频道 channel 上建立一个 MessageComponentCollector， 也可以选择在 message 上建立 MessageComponentCollector，二者都能完成对某一条交互事件的收集，但是二者的收集范围各有不同。

（如果代码上文里有 async 的话，才需要用 await）

通过 `const collector = await interaction.channel.createMessageComponentCollector()` 来在 channel 上建立一个消息事件收集器。

通过 `const message = await interaction.fetchReply(); const collector = await mesage.createMessageComponentCollector()` 在 bot 最后回复的 message 上建立一个交互组件事件的收集器。

具体看代码：`commmands/menus/selectMenu.js`

相关 commit：
- https://github.com/wtflmao/discord_bot_example/commit/a79b1e61d8e3e837a2934577837c8447f6f88777#diff-1184ab849c7784c7807f8c257df3a3190b2a16328a41dcad3754ee0d3ed6d0af
- https://github.com/wtflmao/discord_bot_example/commit/f306c472de37580acc14fabfe6d09dca07ff59a6#diff-1184ab849c7784c7807f8c257df3a3190b2a16328a41dcad3754ee0d3ed6d0af

```js
const { ActionRowBuilder, EmbedBuilder, StringSelectMenuBuilder, SlashCommandBuilder } = require('discord.js');
const wait = require('node:timers/promises').setTimeout;

module.exports = {
    data: new SlashCommandBuilder()
        .setName('selectmenu')
        .setDescription('Replies with a select menu!'),
    async execute(interaction) {

        const row = new ActionRowBuilder()
            .addComponents(
                new StringSelectMenuBuilder()
                    .setCustomId('select_1')
                    .setPlaceholder('Nothing selected')
                    .addOptions(
                        {
                            label: 'Las Vegas, NV',
                            description: 'This is a description',
                            value: 'Las_Vegas_NV_US',
                        },
                        {
                            label: 'Denver, CO',
                            description: 'This is also a description',
                            value: 'Denver_CO_US',
                        },
                    ),
            );

        const embed1 = new EmbedBuilder()
            .setColor(0x0099FF)
            .setTitle('Live Weather Report')
            .setURL('https://weather.com/weather/today/l/Las+Vegas+NV?canonicalCityId=8699c391df74aabce6a01ab22e01fd094d01ff77fcc7ef7e314ea4067fbc1066')
            .setDescription(`Las Vegas, NV\nAs of ${Math.floor(Math.random() * 11) + 1}:${10 + Math.floor(Math.random() * 49)} am PST\n38°F\nClear\nDay ${Math.floor(Math.random() * 10) + 55}°F • Night ${35 - Math.floor(Math.random() * 10)}°F`);
        const embed2 = new EmbedBuilder()
            .setColor(0x0099FF)
            .setTitle('Live Weather Report')
            .setURL('https://weather.com/weather/today/l/3f345b93f02bdea125a122a4798a6b17174a3153bb0f45b4d5238343613d7368')
            .setDescription(`Denver, CO\nAs of ${Math.floor(Math.random() * 11) + 1}:${10 + Math.floor(Math.random() * 49)} am PST\n25°F\nClear\nDay ${Math.floor(Math.random() * 10) + 47}°F • Night ${25 - Math.floor(Math.random() * 10)}°F`);

        await interaction.reply({ content: "Choose a city to see its weather report:", components: [row], embeds: [] });

        const filter = i => {
            return interaction.customId === 'select_1' && i.user.id === interaction.user.id;
        }

        const message = await interaction.fetchReply();
        const collector = message.createMessageComponentCollector(
            filter,
        );

        collector.on('collect', async i=> {
            await i.update({ content: 'Selected! Syncing the weather data...', components: [], embeds:[] });
            await wait(3750); // wait 3.75 secs to emulate the delay of the network
            const city = Array.from(i.values)[0];
            if (city === 'Las_Vegas_NV_US') {
                await i.editReply({content: "Here's your weather report!", embeds: [embed1], components: [row]});
            } else if (city === 'Denver_CO_US') {
                await i.editReply({content: "Here's your weather report!", embeds: [embed2], components: [row]});
            }
        });
    },
};
```

效果图：
![image](https://img2022.cnblogs.com/blog/2455224/202211/2455224-20221121130839162-751537786.png)

![image](https://img2022.cnblogs.com/blog/2455224/202211/2455224-20221121130846070-957290113.png)

![image](https://img2022.cnblogs.com/blog/2455224/202211/2455224-20221121130927974-1295632187.png)

![image](https://img2022.cnblogs.com/blog/2455224/202211/2455224-20221121130934003-1189860505.png)


### 多项选择下拉菜单：

前面的下拉菜单是单选的，如果我们需要多选或者对用户的选择数量进行限制比如1到3项呢？

我们可以在建立 SelectMenu 的时候，设置 `.setMinValues()` 和 `.setMaxValues()` 来指导用户的最少选择量和最大选择量。

比如 `commands/menus/multiSelect.js`：

相关 commit：
- https://github.com/wtflmao/discord_bot_example/commit/a79b1e61d8e3e837a2934577837c8447f6f88777#diff-69116720e7a64621abca0d97ad15c72efe94d5b08c0fcb2c5f895d121b29034c
- https://github.com/wtflmao/discord_bot_example/commit/f306c472de37580acc14fabfe6d09dca07ff59a6#diff-69116720e7a64621abca0d97ad15c72efe94d5b08c0fcb2c5f895d121b29034c

```js
const { ActionRowBuilder, EmbedBuilder, StringSelectMenuBuilder, SlashCommandBuilder } = require('discord.js');
const wait = require('node:timers/promises').setTimeout;

module.exports = {
    data: new SlashCommandBuilder()
        .setName('multiselect')
        .setDescription('Replies with a multi-select menu!'),
    async execute(interaction) {

        const row = new ActionRowBuilder()
            .addComponents(
                new StringSelectMenuBuilder()
                    .setCustomId('select_1')
                    .setPlaceholder('Nothing selected')
                    // here we demand our friendly user to choose more than 2 options while less than 4 options
                    .setMinValues(2)
                    .setMaxValues(4)
                    .addOptions(
                        {
                            label: 'Las Vegas, NV',
                            description: 'This is a description',
                            value: 'Las_Vegas_NV_US',
                        },
                        {
                            label: 'Denver, CO',
                            description: 'This is also a description',
                            value: 'Denver_CO_US',
                        },
                        {
                            label: 'Houston, TX',
                            description: 'This is also a description',
                            value: 'Houston_TX_US',
                        },
                        {
                            label: 'Seattle, WA',
                            description: 'This is also a description',
                            value: 'Seattle_WA_US',
                        },
                        {
                            label: 'Salt Lake City, UT',
                            description: 'This is also a description',
                            value: 'Salt_Lake_City_UT_US',
                        },
                    ),
            );

        const embed1 = new EmbedBuilder()
            .setColor(0x0099FF)
            .setTitle('Las Vegas\nCity in Nevada')
            .setDescription(`Las Vegas, often known simply as Vegas, is the 25th-most populous city in the United States, the most populous city in the state of Nevada, and the county seat of Clark County. The city anchors the Las Vegas Valley metropolitan area and is the largest city within the greater Mojave Desert.\n` +
                `Sales tax: 8.38%\n` +
                `Time zone: Pacific Standard Time, GMT-8`);
        const embed2 = new EmbedBuilder()
            .setColor(0x0099FF)
            .setTitle('Denver\nCity in Colorado')
            .setDescription(`Denver, the capital of Colorado, is an American metropolis dating to the Old West era. Larimer Square, the city’s oldest block, features landmark 19th-century buildings. Museums include the Denver Art Museum, an ultramodern complex known for its collection of indigenous works, and the mansion of famed Titanic survivor Molly Brown. Denver is also a jumping-off point for ski resorts in the nearby Rocky Mountains.\n` +
                `Sales tax: 8.81%\n` +
                `Time Zone: Mountain Standard Time, GMT-7`);
        const embed3 = new EmbedBuilder()
            .setColor(0x0099FF)
            .setTitle('Houston\nCity in Texas')
            .setDescription(`Houston is a large metropolis in Texas, extending to Galveston Bay. It’s closely linked with the Space Center Houston, the coastal visitor center at NASA’s astronaut training and flight control complex. The city’s relatively compact Downtown includes the Theater District, home to the renowned Houston Grand Opera, and the Historic District, with 19th-century architecture and upscale restaurants.\n` +
                `Sales tax: 8.25%\n` +
                `Time zone: Central Standard Time, GMT-6`);
        const embed4 = new EmbedBuilder()
            .setColor(0x0099FF)
            .setTitle('Seattle\nCity in Washington State')
            .setDescription(`Seattle, a city on Puget Sound in the Pacific Northwest, is surrounded by water, mountains and evergreen forests, and contains thousands of acres of parkland. Washington State’s largest city, it’s home to a large tech industry, with Microsoft and Amazon headquartered in its metropolitan area. The futuristic Space Needle, a 1962 World’s Fair legacy, is its most iconic landmark.\n` +
                `Sales tax: 10.25%\n` +
                `Time zone: Pacific Standard Time, GMT-8`);
        const embed5 = new EmbedBuilder()
            .setColor(0x0099FF)
            .setTitle('Salt Lake City\nCity in Utah')
            .setDescription(`Salt Lake City is the capital and most populous city of Utah, as well as the seat of Salt Lake County, the most populous county in Utah. With a population of 200,133 in 2020, the city is the core of the Salt Lake City metropolitan area, which had a population of 1,257,936 at the 2020 census.\n` +
                `Sales tax: 7.75%\n`+
                `Time zone: Mountain Standard Time, GMT-7`);

        await interaction.reply({ content: "Choose 2-4 cities to make your very own vacation destination list:", components: [row], embeds: [] });

        const filter = i => {
            return interaction.customId === 'select_1' && i.user.id === interaction.user.id;
        }

        const collector = interaction.channel.createMessageComponentCollector(
            filter,
        );

        collector.on('collect', async i=> {
            await i.update({ content: 'Selected! Fetching the detailed data relating to your choices...', components: [], embeds:[] });
            await wait(3150); // wait 3.15 secs to emulate the delay of the network
            await i.editReply({content: "Here's your detailed descriptions related to your choices!", embeds: [], components: []});

            const cityArr = Array.from(i.values);
            if (cityArr.includes('Las_Vegas_NV_US')) {
                await i.followUp({content: "", embeds: [embed1], components: []});
            }
            if (cityArr.includes('Denver_CO_US')) {
                await i.followUp({content: "", embeds: [embed2], components: []});
            }
            if (cityArr.includes('Houston_TX_US')) {
                await i.followUp({content: "", embeds: [embed3], components: []});
            }
            if (cityArr.includes('Salt_Lake_City_UT_US')) {
                await i.followUp({content: "", embeds: [embed4], components: []});
            }
            if (cityArr.includes('Seattle_WA_US')) {
                await i.followUp({content: "", embeds: [embed5], components: []});
            }
        });
    },
};
```

效果图：

![image](https://img2022.cnblogs.com/blog/2455224/202211/2455224-20221121130952741-1113771410.png)

![image](https://img2022.cnblogs.com/blog/2455224/202211/2455224-20221121131000278-445865959.png)

![image](https://img2022.cnblogs.com/blog/2455224/202211/2455224-20221121131004833-191889417.png)

![image](https://img2022.cnblogs.com/blog/2455224/202211/2455224-20221121131007863-1019711950.png)

### 其他操作

-   `reply()`
-   `editReply()`
-   `deferReply()`
-   `fetchReply()`
-   `deleteReply()`
-   `followUp()`

### 获取下拉菜单被选中的值

不论你的下拉菜单是单选、多选还是混合的，下拉菜单返回值是个数组。

- 比如单选菜单，i 是收集器里收集到的 interaction 变量：
```js
	const value = Array.from(i.values)[0];
```

- 如果不是单选：
```js
	const valueArr = Array.from(i.values);
```
	此时，你可以用简单的 for 遍历数组里的所有值：
```js
	for (let j=0; j<valueArr.length; j++) {
		console.log(valueArr[j]);
	}
```

如果你想判断该数组是否包含某项：
```js
if (valueArr.includes('Choice_6')) {
    await i.editReply({content: "You choosed the sixth option", embeds: [], components: []});
}
```

### 其他 SelectMenu 的类型

在 Discord.js v14.6.0 级更早版本中，SelectMenu 只有它自己这一个类型。

而自从 v14.7.1 起，原 SelectMenu 改名为 StringSelectMenu，并增加了 UserSelectMenu、ChannelSelectMenu、MentionableSelectMenu、RoleSelectMenu。

对应的 Builder 就是 StringSelectMenuBuilder()、UserSelectMenuBuilder()、ChannelSelectMenuBuilder()、MentionableSelectMenuBuilder()、RoleSelectMenuBuilder()。