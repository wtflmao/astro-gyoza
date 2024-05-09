---
title: Bot in Discord with discord.js (15)
date: 2023-04-09
tags: [Discord bot, JS]
category: Discord.js
summary: 这是一份非常详细和系统化的利用 NodeJS 进行 Discord bot 开发的教程，本文介绍了可以用于降低服务器压力的命令冷却时间 Cooldown 小技巧。
---

# Chapter 18 - 命令冷却时间

本文撰写时，项目已升级至 discord.js@v14.9.0。

很多时候，我们不希望频道被单一用户用洪水般的命令消息所淹没，又或者出于对服务器性能的考虑，我们不得不为命令加上一个冷却时间（Cooldown）作为使用限制。下面我们来看看如何在我们已有的代码上进行改造，以获得命令冷却的效果。

### 创建 `commands/longCooldownPing.js`

为了与原有代码兼容，我们将设计成这样：`module.exports` 中，没有 `cooldown` 值的旧代码，将使用我们后续设置的默认冷却时间。而有该值的新代码，直接使用该值作为冷却时间。

我们创建一个拥有 15 秒 冷却时间的命令 `/coolping`。同一个用户在第一次成功使用命令后的 15 秒内将不被再次允许使用这项命令，除非 15 秒后，冷却时间结束。

`commands/longCooldownPing.js`：
```js
const { SlashCommandBuilder } = require('discord.js');

module.exports = {
    // 这里设置一个冷却时长，单位为秒
    cooldown: 15,
    data: new SlashCommandBuilder()
        .setName('coolping')
        .setDescription('Replies with Pong, but with a 15 secs cooldown!'),
    async execute(interaction) {
        await interaction.reply("Pong!");
        // 每次都提示这个用户，他的下一次该命令将在 15 秒后可用
        await interaction.followUp({
            content: `Next /coolping will be available in 15 seconds`,
            ephemeral: true, // 必须为 true， 不然不相干的用户也将看见
        });
    },
};
```

现在别急着执行 `/coolping`，我们还没设计判断冷却时间的逻辑呢。

### 修改 `events/interactionCreate.js`

我们要在执行命令前判断是否过了冷却时间，而不是之后。所以相关的判断逻辑应该在咱们 `if (interaction.isChatInputCommand())` 之前。

但是我们知道，一个交互（Interaction），它不一定就是个命令，它也可能是条普通消息，我们不希望在非命令上整这么个冷却时间。所以，先来个 `if (interaction.isCommand()) { ... }`，我们所有冷却时间逻辑都写在这里面。

`events/interactionCreate.js` 开头节选：
```js
const { Events } = require('discord.js');

module.exports = {
	name: Events.InteractionCreate,
	async execute(interaction) {
		// we only need to check cooldown on a command, not on every single interaction
		if (interaction.isCommand()) {

		// 这里写冷却时间判断逻辑，in constrction
		
		}

		if (interaction.isChatInputCommand()) {

...
```

是这么考虑的：对于第一次执行某命令的用户，我们将现在时间 `now` 加上该命令的冷却时间 `cooldown`，计算得到过期时间 `expirationTime`， 即 `expirationTime = now + cooldown`。

接着，将用户 ID 与过期时间结合起来成为一个条目储存到与命令名相关的数据结构里，以便在这个人下一次执行相同命令时，判断确定是否过了冷却时间。为此，需要创建一个 叫 `cooldowns` 的 Collection，并导包 。（注意区分 `cooldown` 和 `cooldowns`）

`events/interactionCreate.js` 开头节选：
```js
const { Events, Collection } = require('discord.js');
let cooldowns = new Collection();
module.exports = {
	name: Events.InteractionCreate,
	async execute(interaction) {
		// we only need to check cooldown on a command, not on every single interaction
		if (interaction.isCommand()) {
		
...
```

下面考虑实现逻辑。在 `cooldowns` 里存储这样的键值对：（ 命令名, ( 用户ID, 过期时间 ) )。所以为每一个第一次出现的命令，往 cooldowns 里存一个键值对，key 是命令名，value 是空 Collection：
```js
...

if (interaction.isCommand()) {
	// 获取命令本体
	const command = await interaction.client.commands.get(interaction.commandName);

	// 如果该命令在程序生存期内第一次出现，则给他开一个 Collection
	if (!cooldowns.has(command.data.name)) {
		cooldowns.set(command.data.name, new Collection());
	}
	
...
```

做完这些准备工作后，我们继续。下面的代码挺好懂得，就是获取当前时间，获取命令里存储的 cooldown 值，加一块儿得到过期时间。如果命令没有设置 cooldown 值，则使用默认值，我们这里设置默认值为 0 秒。cooldownAmount 单位是毫秒，用于条目过期后，程序自动删除该条目的。

```js
...

const now = Date.now(); // get current time
const timestamps = await cooldowns.get(command.data.name);
const defaultCooldownDuration = 0; // we set a DEFAULT value for those legacy command that didn't set a cooldown
const cooldownAmount = (command.cooldown ?? defaultCooldownDuration) * 1000;

...
```

接着考虑用户近期（指的是该用户还有条目）非第一次执行的话，判断是否已过期：

```js
...

if (timestamps.has(interaction.user.id)) {
	const expirationTime = await timestamps.get(interaction.user.id) + cooldownAmount;
	if (now < expirationTime) {
		// 不是现在，用户你继续等吧
		const expiredTimestamp = Math.round(expirationTime / 1000);
		// 在 Discord 里提醒下用户，还得继续等
		return interaction.reply({
			content: `Please wait for <t:${expiredTimestamp}:R> more time before reusing the \`${command.data.name}\` command.`,
			ephemeral: true // 设置 true，不然不相干的用户也能看见这条消息
		});
	} else {
		// 该用户的冷却时间已过去了，放行继续
	}
}

...
```

我们还要考虑近期第一次执行某命令，以及关于命令冷却时间已过去的那些用户，为他们关于该命令设置“解锁时间”：
```js
...

// add a cooldown lock
await timestamps.set(interaction.user.id, now);
// wait and wait
await setTimeout(() => timestamps.delete(interaction.user.id), cooldownAmount);

...
```

这样我们就完成了修改。下面是修改后 `events/interactionCreate.js` 的内容开头节选：
```js
const { Events, Collection } = require('discord.js');
let cooldowns = new Collection();
module.exports = {
	name: Events.InteractionCreate,
	async execute(interaction) {
		// we only need to check cooldown on a command, not on every single interaction
		if (interaction.isCommand()) {
			const command = await interaction.client.commands.get(interaction.commandName);

			if (!cooldowns.has(command.data.name)) {
				cooldowns.set(command.data.name, new Collection());
			}

			const now = Date.now(); // get current time
			const timestamps = await cooldowns.get(command.data.name);
			const defaultCooldownDuration = 0; // we set a DEFAULT value for those legacy command that didn't set a cooldown
			const cooldownAmount = (command.cooldown ?? defaultCooldownDuration) * 1000;

			if (timestamps.has(interaction.user.id)) {
				const expirationTime = await timestamps.get(interaction.user.id) + cooldownAmount;
				if (now < expirationTime) {
					// not now, still wait
					const expiredTimestamp = Math.round(expirationTime / 1000);
					return interaction.reply({
						content: `Please wait for <t:${expiredTimestamp}:R> more time before reusing the \`${command.data.name}\` command.`,
						ephemeral: true
					});
				} else {
					// cooldown expired for that user, proceed

				}
			}

			// add a cooldown lock
			await timestamps.set(interaction.user.id, now);
			// wait and wait
			await setTimeout(() => timestamps.delete(interaction.user.id), cooldownAmount);
		}
		
		if (interaction.isChatInputCommand()) {
		// 后面我们没改过，太长了，略
...
```

相关 commit 可以在这里查看：
Github：https://github.com/wtflmao/discord_bot_example/commit/4ecb8c2cb29240776f1cb6660ea7b810dc2ff963
Gitee：https://gitee.com/wtflmao/discord_bot_example/commit/4ecb8c2cb29240776f1cb6660ea7b810dc2ff963

效果图：
近期第一次 `/coolping`：
![image](https://img2023.cnblogs.com/blog/2455224/202304/2455224-20230409160747347-379304443.png)


4 秒后（未过冷却时间），第二次执行 `/coolping`：
![image](https://img2023.cnblogs.com/blog/2455224/202304/2455224-20230409160751181-1126526170.png)

### 小结

我学习了为命令设置冷却时间。