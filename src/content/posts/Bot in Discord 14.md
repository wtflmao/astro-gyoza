---
title: Bot in Discord with discord.js (14)
date: 2023-01-24
tags: [Discord bot, JS]
category: Discord.js
summary: 这是一份非常详细和系统化的利用 NodeJS 进行 Discord bot 开发的教程，本文介绍了什么是 Discord 的表情反应 Reaction。
---

# Chapter 17 - Reaction 反应

反应（Reactions）是这样的一种东西：

![image](https://img2023.cnblogs.com/blog/2455224/202301/2455224-20230124221854869-52977036.png)

你可以这样为一条消息添加反应：

![image](https://img2023.cnblogs.com/blog/2455224/202301/2455224-20230124221912227-851394996.png)

不是任何消息都可以被添加反应的。比如管理员设置了成员在某些频道不允许添加反应，或者只允许某些身份组的人可以为某些消息添加反应。

这样看来，“反应” 就是一种 emoji 回复。下面我们来看看机器人如何为一条消息识别、添加和移除反应。

### Discord Emoji 的种类

在 Discord 里，emoji 有两类，一种是跨平台、跨软件、跨设备都通用的 Unicode 标准 emoji，还有一种是 Discord 自定义 emoji。前者属于 unicode 标准，一个表情属于一个字符，后者则不属于 unicode 标准，只限在 Discord 平台里流通。

下面我们来举个例子：

微笑 🙂 是一个 Unicode emoji，在 discord 里，它可被记为 `:smile:`。

我们在服务器设置里上传了个自己的表情，叫 `:WumpusMistletoe2:`。

下面就是对表情反应的示例图：

![image](https://img2023.cnblogs.com/blog/2455224/202301/2455224-20230124222209218-1136736470.png)

### 给服务器上传一个表情符号

假设你的服务器没有任何表情符号，如果你有，可以跳过上传这一步，直接跳到获取 id。

打开 Discord，点击一个你拥有管理员或服主身份的服务器，打开服务器设置，点击 “表情符号” 选项卡，上传一个表情符号并命名。比如我们选择了一张 Wumpus 图片上传，并命名为 “WumpusMistletoe2”。

![image](https://img2023.cnblogs.com/blog/2455224/202301/2455224-20230124222242340-743465457.png)

![image](https://img2023.cnblogs.com/blog/2455224/202301/2455224-20230124222229823-1884263309.png)

> 注意，不同服务器里的表情名字可以重复，但是它们的 id 将不会相同。当机器人使用 .find() 通过名字寻找表情时，该方法总会返回它找到的第一项，这可能与你所期望的表情不匹配。

### 获取表情 id

相同的图片在相同的服务器里上传，也会得到不同的 id。这是因为 Discord 的 id 是 Snowflake 值。具体关于 Discord snowflake id，我在系列文章 Chapter 6 的最后介绍过，这里不重复介绍。

#### 方法一

接下来回到服务器聊天区获取表情 id。

延续上一步举例的例子。在聊天框输入 `\:WumpusMistletoe2:` 并回车发送，你的消息会被自动替换成表情 id。：

![image](https://img2023.cnblogs.com/blog/2455224/202301/2455224-20230125124039064-1376380584.png)

这样，我们就获取到本例中关键的 id： `<:WumpusMistletoe2:1067024784057184266>`。

不要复制我的 id，这个 id 只在我的服务器里有效。

#### 方法二

打开 PC 端浏览器，按下 `Ctrl + Shift + C`，将鼠标指向你需要获取 id 的表情图标上，单击。

![image](https://img2023.cnblogs.com/blog/2455224/202301/2455224-20230125124113720-849754551.png)

在开发者工具中，你能看到高亮区域代码里的 alt 值是 `:WumpusMistletoe2:`，data-id 值是 `1067024784057184266`。将其用格式 <:alt:data-id> 组合一下就得到了关键的信息： `<:WumpusMistletoe2:1067024784057184266>`。

### 向 index.js 添加关键 GatewayBitField

# **这很重要！！**

给 client 增加一个 `GatewayIntentBits.GuildMessageReactions`，这样才可以获取消息的反应。

修改后的那一行如下：
```js
...
const client = new Client({ intents: [GatewayIntentBits.Guilds, GatewayIntentBits.GuildMessages, GatewayIntentBits.MessageContent, GatewayIntentBits.GuildMessageReactions] });
...
```

具体的 commit：
Github: https://github.com/wtflmao/discord_bot_example/commit/dfbad3ca953e2ca25ac4673fbcea3a549eff9c77
Gitee: https://gitee.com/wtflmao/discord_bot_example/commit/dfbad3ca953e2ca25ac4673fbcea3a549eff9c77

> P.S. 这里我们在 index.js 只使用一个 Client 实例并赋予所有可能用到的 Intents，后续交互复用这个实例，其实是一种偷懒且具有潜在风险的做法。建议不要学我这么做。但是我们的代码已经都这样了，何况我只是以学习为目的的写这个项目，没打算直接上生产环境，那就让它维持现在这样的屎山状态吧哈哈。

### 回复表情

下面修改文件 `/cmdPaths.js`，在 data 域里加入新的一条 "./commands/reactions"。
```js
module.exports = {
    data: ["./commands", "./commands/utils", "./commands/buttons", "./commands/menus", "./commands/modals", "./commands/contextMenus", "./commands/embeds", "./commands/reactions"],
};
```

新建文件 `/commands/reactions/reaction.js`：
```js
const { SlashCommandBuilder} = require('discord.js');

module.exports = {
    data: new SlashCommandBuilder()
        .setName('react')
        .setDescription('Replies with reaction!'),
    async execute(interaction) {

        const message = await interaction.reply({ content: 'You can react with Unicode emojis!', fetchReply: true });
        await message.react('😄');

        const message2 = await interaction.followUp({ content: "Here's a custom emoji!", fetchReply: true });
        await message2.react('<:WumpusMistletoe2:1067024784057184266>');
    },
...
```

运行 /react，得到结果：

![image](https://img2023.cnblogs.com/blog/2455224/202301/2455224-20230124222348664-314695206.png)

> .react() 还支持多种不同格式的传入值，比如
> meaasge.react("<:WumpusMistletoe2:1067024784057184266>");
> meaasge.react("<a:WumpusMistletoe2:1067024784057184266>");
> meaasge.react("a:WumpusMistletoe2:1067024784057184266");
> meaasge.react("WumpusMistletoe2:1067024784057184266");
> meaasge.react("1067024784057184266");

下面我们使用 `.find()` 来用表情名称寻找一个表情。

```js
...
        const message3 = await interaction.followUp({ content: "Here we grab an emoji by its name", fetchReply: true });
        await message3.react(message3.guild.emojis.cache.find(emoji => emoji.name === 'WumpusMistletoe2'));
...
```

效果图：

![image](https://img2023.cnblogs.com/blog/2455224/202301/2455224-20230124222404785-1141340977.png)

下面我们使用 `.get()` 来用纯数字 id 指定一个表情。
```js
...
        const message4 = await interaction.followUp({ content: "Here we grab an emoji by its id", fetchReply: true });
        // Emoji must be a string or GuildEmoji/ReactionEmoji
        await message4.react(interaction.client.emojis.cache.get('1067024784057184266'));
...
```

效果图：

![image](https://img2023.cnblogs.com/blog/2455224/202301/2455224-20230124222416426-220121391.png)

我们得到了 `commands/reactions/reaction.js` 的完整代码：
```js
const { SlashCommandBuilder} = require('discord.js');

module.exports = {
    data: new SlashCommandBuilder()
        .setName('react')
        .setDescription('Replies with reaction!'),
    async execute(interaction) {

        const message = await interaction.reply({ content: 'You can react with Unicode emojis!', fetchReply: true });
        await message.react('😄');

        const message2 = await interaction.followUp({ content: "Here's a custom emoji!", fetchReply: true });
        await message2.react('<:WumpusMistletoe2:1067024784057184266>');

        const message3 = await interaction.followUp({ content: "Here we grab an emoji by its name", fetchReply: true });
        await message3.react(message3.guild.emojis.cache.find(emoji => emoji.name === 'WumpusMistletoe2'));

        const message4 = await interaction.followUp({ content: "Here we grab an emoji by its id", fetchReply: true });
        // Emoji must be a string or GuildEmoji/ReactionEmoji
        await message4.react(interaction.client.emojis.cache.get('1067024784057184266'));
    },
};
```

### 多表情的按顺序逐个反应

用 `commands/reactions/multiReactions.js` 来举例：
```js
const { SlashCommandBuilder} = require('discord.js');

module.exports = {
    data: new SlashCommandBuilder()
        .setName('multireact')
        .setDescription('Replies with reaction!'),
    async execute(interaction) {
        const message = await interaction.reply({ content: 'You can react with Unicode emojis!', fetchReply: true });
        // "c u m" in order
        // the first way to do this is using serial .then() on the previous .react()
        await message.react('🇨')
            .then(() => message.react('🇺')
                .then(() => message.react('🇲')
                    .then(() => message.react('❗'))));

        const msg = await interaction.channel.send({ content: 'AHHhhhhhhHHHhh', fetchReply: true });
        // the second way to do this, is using paralleled .then() on the msg
        await msg.react('🇸')
            .then(() => msg.react('🇭'))
            .then(() => msg.react('🇮'))
            .then(() => msg.react('🇹'));

        // the third way to do this is using .react() multiple times
        const menu = await interaction.channel.send({ content: 'Library search result:\n\n\tThe Art of War, Sun Tzu, Filiquarian 2017, PDF\n\tMinecraft: The Shipwreck, C. B. Lee, Del Ray 2020, Paperback\n\tand more...\n\nPage 1/4', fetchReply: true });
        await menu.react('⬅️');
        await menu.react('1️⃣');
        await menu.react('2️⃣');
        await menu.react('3️⃣');
        await menu.react('4️⃣');
        await menu.react('➡️');
    },
};
```

表情会按顺序逐个添加到指定的消息上去。

效果图：

![image](https://img2023.cnblogs.com/blog/2455224/202301/2455224-20230124222436755-471884608.png)

### 多表情的无特定顺序反应

用 `commands/reactions/multiReactions2.js` 来举例：
```js
const { SlashCommandBuilder} = require('discord.js');

module.exports = {
    data: new SlashCommandBuilder()
        .setName('multireact2')
        .setDescription('Replies with reaction!'),
    async execute(interaction) {
        const message = await interaction.reply({ content: 'You can react with Unicode emojis!', fetchReply: true });

        Promise.all([
            message.react('🇨'),
            message.react('🇺'),
            message.react('🇲'),
            message.react('❗'),
            message.react('🇸'),
            message.react('🇭'),
            message.react('🇮'),
            message.react('🇹'),
        ])
            .catch(error => console.error('One of the emojis failed to react:', error));
    },
};
```

效果图：

![image](https://img2023.cnblogs.com/blog/2455224/202301/2455224-20230124222449548-985445736.png)

### 移除表情反应

> 所有这些方法都需要机器人在服务器里有 ManageMessages 权限。请确保你的机器人具有权限，否则它将出错。

> **确保不要过多地删除表情符号或用户的反应。如果短时间内添加或删除了很多反应，它可以被认为是 API 滥用。**

#### 按表情移除反应

```js
message.reactions.cache.get('🇫').remove()
	.catch(error => console.error('Failed to remove reactions:', error));
```

下面给出 `commands/reactions/rmReaction.js` :

```js
const { SlashCommandBuilder} = require('discord.js');

module.exports = {
    data: new SlashCommandBuilder()
        .setName('rmreact')
        .setDescription('Replies with reaction!'),
    async execute(interaction) {

        const message = await interaction.reply({ content: 'You can react with Unicode emojis!', fetchReply: true });
        await message.react('😄');
        await message.react('🆒');
        await message.react('😃');
        await message.react('😕');

        message.reactions.cache.get('🆒').remove()
            .catch(error => console.error('Failed to remove reactions:', error));
    },
};
```

效果的话，读代码就行，很好懂的。

#### 按用户移除反应

```js
// 获取用户 id 为 “userId” 的用户对消息 “message” 所有的反应
const userReactions = message.reactions.cache.filter(reaction => reaction.users.cache.has(userId));

try {
	// 遍历这些反应，并从消息底下移除
	for (const reaction of userReactions.values()) {
		await reaction.users.remove(userId);
	}
} catch (error) {
	console.error('Failed to remove reactions.');
}

```

#### 移除消息全部反应

```js
message.reactions.removeAll()
	.catch(error => console.error('Failed to clear reactions:', error));
```

### 等待反应

使用曾经用过的 Collector 可以轻松解决这个问题。在咱们的另一个项目 guess_the_number (https://github.com/wtflmao/guess_the_number) 里，我们已经用过了简单的消息收集器 (https://www.cnblogs.com/hhzm/p/16508453.html) ；在 Chapter 12 按钮的 collector，把它们拿来改成适用于表情反应事件的收集器也是很容易的。

下面给出 `commands/reactions/collectReaction.js` ：

```js
const { SlashCommandBuilder } = require('discord.js');

module.exports = {
    data: new SlashCommandBuilder()
        .setName('collectreact')
        .setDescription('Replies with reaction!'),
    async execute(interaction) {

        const message = await interaction.reply({ content: 'You can react with a thumbs UP or a thumbs DOWN.', fetchReply: true })
        message.react('👍').then(() => message.react('👎').then(() => message.react('😋')));
        const filter = (reaction, user) => {
            return (['👍', '👎','😋'].includes(reaction.emoji.name)) && (user.id === interaction.user.id);
        };

        const collector = message.createReactionCollector({ filter, time: 10000 });

        collector.on('collect', (reaction, user) => {
            console.log(`Collected ${reaction.emoji} from ${user}`);
        });

        collector.on('end', collected => {
            if (collected.size === 0) {
                message.reply("Nothing valid you've reacted.");
                return;
            }

            collected.forEach(reaction => {
                if (reaction.emoji.name === '👍') {
                    interaction.channel.send('You reacted with a thumbs UP.');
                } else if (reaction.emoji.name === '👎') {
                    interaction.channel.send('You reacted with a thumbs DOWN.');
                } else {
                    interaction.channel.send('You reacted with a YUM.');
                }
            })
        });
    },
};
```

效果图：

![image](https://img2023.cnblogs.com/blog/2455224/202301/2455224-20230124222637643-85320352.png)

![image](https://img2023.cnblogs.com/blog/2455224/202301/2455224-20230124222643342-919030606.png)

上面的代码写起来很繁琐。幸好我们还有简单的写法。

下面给出 `commands/reactions/awaitReact.js` 的代码：

```js
const { SlashCommandBuilder } = require('discord.js');

module.exports = {
    data: new SlashCommandBuilder()
        .setName('awaitreact')
        .setDescription('Replies with reaction!'),
    async execute(interaction) {

        const message = await interaction.reply({ content: 'You can react with a thumbs UP or a thumbs DOWN.', fetchReply: true })
        message.react('👍').then(() => message.react('👎').then(() => message.react('😋')));
        const filter = (reaction, user) => {
            return (['👍', '👎','😋'].includes(reaction.emoji.name)) && (user.id === interaction.user.id);
        };

		// 注意这里的 errors: ['time'] 不要忘了
        await message.awaitReactions({ filter, time: 10000, errors: ['time'] })
            .then(collected => {
                collected.forEach(reaction => {
                    if (reaction.emoji.name === '👍') {
                        interaction.channel.send('You reacted with a thumbs UP.');
                    } else if (reaction.emoji.name === '👎') {
                        interaction.channel.send('You reacted with a thumbs DOWN.');
                    } else {
                        interaction.channel.send('You reacted with a YUM.');
                    }
                })
            })
            .catch(() => {
                interaction.channel.send("Nothing valid you've reacted.");
            });
    },
};
```

效果图：

![image](https://img2023.cnblogs.com/blog/2455224/202301/2455224-20230124222701824-1624056038.png)

![image](https://img2023.cnblogs.com/blog/2455224/202301/2455224-20230124222754424-506222329.png)

利用 `message.awaitReactions([options])` 来代替 `message.createReactionCollector([options])` 要更简单明了。