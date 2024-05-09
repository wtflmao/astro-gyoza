---
title: Bot in Discord with discord.js (7)
date: 2022-07-14
tags: [Discord bot, JS]
category: Discord.js
summary: 这是一份非常详细和系统化的利用 NodeJS 进行 Discord bot 开发的教程，本文介绍了什么是斜杠命令的子命令 subcommand 以及为同一套命令逻辑起别名的方法。
---

## Chapter 7 - 斜杠命令的子命令 Subcommands

如果你有一个包含子命令的命令，你可以以使用与解析 Options 和 Choices 的值那样相似的方式解析它们。 以下代码片段详细说明了解析子命令并使用 `CommandInteractionOptionResolver#getSubcommand()` 方法做出相应响应所需的逻辑： 

代码 `commands/subCmd.js`：
```js
const { SlashCommandBuilder } = require('discord.js');

module.exports = {
	data: new SlashCommandBuilder()
		.setName('subcmd')
		.setDescription('Subcommand! Reply with user\'s or server \'s info.')
		.addSubcommand(subcommand => 
			subcommand.setName('user')
			.setDescription('Info about a user')
			.addUserOption(option => option.setName('target').setDescription('The user')))
		.addSubcommand(subcommand =>
			subcommand.setName('server')
			.setDescription('Info about the server')),

	async execute(interaction) {
		if (interaction.options.getSubcommand() === 'user') {
			const theUser = interaction.options.getUser('target');

			if (theUser) {
				await interaction.reply(`Username: ${theUser.username}\nID: ${theUser.id}`);
			} else {
				// 用户没有指定 target user，我们输出命令发起者自己就好
				await interaction.reply(`Your username: ${interaction.user.username}\nYour ID: ${interaction.user.id}`);
			}
		} else if (interaction.options.getSubcommand() === 'server') {
			await interaction.reply(`Server name: ${interaction.guild.name}\nTotal members: ${interaction.guild.memberCount}`);
		}
	},
};
```

注意，只有子命令才是真正可以执行的命令，比如上面的代码中 `/subcmd` 没有真正的函数逻辑实现来支撑这个命令，只有子命令 `/subcmd server` 和 `/subcmd user` 才是真正有函数实现的命令。直接执行 `/subcmd` 指望它能干事是不可能的。

还是先上效果图，再分析代码：

- 输入命令 `/subcmd` 时，提示了两个子命令：
![image](https://img2022.cnblogs.com/blog/2455224/202207/2455224-20220713000603222-1244642579.png)

- 执行 `/subcmd server` ：
![image](https://img2022.cnblogs.com/blog/2455224/202207/2455224-20220713000606307-524172865.png)

- 执行 `/subcmd user @BlogTest`：（注意你需要从提示栏中回车选定 target 用户，可不能直接输入 "@ + 用户名" 就完活了，因为可能存在重名用户）
![image](https://img2022.cnblogs.com/blog/2455224/202207/2455224-20220713014652764-1775725127.png)

- 执行 `/subcmd user`：
![image](https://img2022.cnblogs.com/blog/2455224/202207/2455224-20220713010022170-1269680805.png)

分析代码：

下面的节选代码，它声明了一个叫 user 的子命令，它的输入参数是一个 Option，一个 User 类型的 Option。这个 Option 叫 target。
```js
...

		.addSubcommand(subcommand => 
			subcommand.setName('user')
			.setDescription('Info about a user')
			.addUserOption(option => option.setName('target').setDescription('The user')))

...
```

下面的节选代码，它声明了一个叫 server 的子命令，没有参数。
```js
...

		.addSubcommand(subcommand =>
			subcommand.setName('server')
			.setDescription('Info about the server')),

...
```

下面的代码，首先上来，判断子命令是 user 还是 server。

- 如果子命令是 user，则通过 `interaction.options.getUser(target)` 获取 target 字段的值，并赋值给 theUser 变量。
	- 如果 theUser 为 null，则用户没有给定值，则会回复用户的信息（用户名和用户的 Snowflake ID）。
	- 如果 theUser 非 null，则输出这个用户的信息（用户名和用户的 Snowflake ID）。注意这里的 user 将是一个有效 Discord 用户，不会是无效的 Discord 用户作为输入。
- 如果子命令是 server，则回复服务器的信息（服务器名称和服务器人数）。

```js
...

	async execute(interaction) {
		if (interaction.options.getSubcommand() === 'user') {
			const theUser = interaction.options.getUser('target');

			if (theUser) {
				await interaction.reply(`Username: ${theUser.username}\nID: ${theUser.id}`);
			} else {
				// 用户没有指定 target user，我们输出命令发起者自己就好
				await interaction.reply(`Your username: ${interaction.user.username}\nYour ID: ${interaction.user.id}`);
			}
		} else if (interaction.options.getSubcommand() === 'server') {
			await interaction.reply(`Server name: ${interaction.guild.name}\nTotal members: ${interaction.guild.memberCount}`);
		}
	},

...
```

## Chapter 8 - 斜杠命令的多重命令名

比如代码 `commands/utils/sameCmd.js`，改造自 `commands/subCmd.js`：
```js
const { SlashCommandBuilder } = require('discord.js');  
  
module.exports = {  
   data: new SlashCommandBuilder()  
      .setName('samecmd')  
      .setDescription('Subcommand! Reply with user\'s or server \'s info.')  
      .addSubcommand(subcommand =>   
         subcommand.setName('user')  
         .setDescription('Info about a user')  
         .addUserOption(option => option.setName('target').setDescription('The user')))  
      .addSubcommand(subcommand =>  
         subcommand.setName('server')  
         .setDescription('Info about the server')),  
   akaNames: ['samecmd2', 'samecmd3', "samecmd4"],  
  
   async execute(interaction) {  
      if (interaction.options.getSubcommand() === 'user') {  
         const theUser = interaction.options.getUser('target');  
  
         if (theUser) {  
            await interaction.reply(`Username: ${theUser.username}\nID: ${theUser.id}`);  
         } else {  
            // 用户没有指定 target user，我们输出命令发起者自己就好  
            await interaction.reply(`Your username: ${interaction.user.username}\nYour ID: ${interaction.user.id}`);  
         }  
      } else if (interaction.options.getSubcommand() === 'server') {  
         await interaction.reply(`Server name: ${interaction.guild.name}\nTotal members: ${interaction.guild.memberCount}`);  
      }  
   },  
};
```

关键就是中间的 `akaNames: ['samecmd2', 'samecmd3', "samecmd4"], `。

我们为该命令起了 4 个名字：一开始的 `samecmd`，和 akaNames 描述的 `sameCmd2`、 `sameCmd3`、 `sameCmd4` 。

对了，我们把代码保存到了 `commands/utils` 这个新文件夹，为了使得 `sameCmd.js` 注册，需要修改 `cmdPaths.js`：
```js
module.exports = {  
    data: ["./commands", "./commands/utils"],  
};
```

效果图：
![image](https://img2022.cnblogs.com/blog/2455224/202207/2455224-20220723060051400-163673119.png)

![image](https://img2022.cnblogs.com/blog/2455224/202207/2455224-20220723060119552-533298490.png)

