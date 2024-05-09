---
title: Bot in Discord with discord.js (3)
date: 2022-07-11
tags: [Discord bot, JS]
category: Discord.js
summary: 这是一份非常详细和系统化的利用 NodeJS 进行 Discord bot 开发的教程，本文对系列文章（1）的代码做出了解释。
---

## Chapter 4 - 对 Chapter 2 代码的解释

我们在系列文章（1） 中，完成了简单的机器人的建立与交互。

下面，我们开始针对 discord.js V14.0.3 的特性和方法，通过 Discord 机器人的编写实践，来掌握 discord.js 的使用。

### 首先，从 目录结构 开始看起。

```
discord_bot_2022
├─commands
├─events
├─node_modules
├─.gitignore
├─cmdPaths.js
├─config.json
├─deploy_commands.js
├─index.js
├─package.json
├─package-lock.json
├─pnmp-lock.yaml
└─yarn.lock
```

文件 `index.js` 是我们的机器人启动器。
文件 `deploy_commands.js` 是用来注册部署机器人斜杠命令的。
文件 `cmdPaths.js` 用于存储斜杠命令所在的目录路径。
文件 `config.json` 用于存储机密设置（比如机器人 token 和各个 id）。
文件 `.gitignore` 是让 Git 屏蔽某些文件的上传（如果你不需要使用 Git 上传到 Github 或 bitbucket 等代码托管平台，则这个文件没有作用）。
文件夹 `commands` 用于存储我们需要用到的命令，当然里面都是 js 文件。
文件夹 `events` 用于存储事件的文件夹，目前我们只存了俩，其实这俩就够了。
文件夹 `node_modules` 用于存放项目所依赖的外部模块的缓存，不要动它。
其余没谈到的文件各司其职，也不要动它们。

### 下面来看 `events/interactionCreate.js` 和 `ready.js`

`events/interatcionCreate.js`
```js
// module.exports 提供了暴露接口的方法
module.exports = {  

    // 指明 name
    name: "interactionCreate",

	// module 暴露了名为 execute 的函数，接受一个 discord.js interaction 对象
    execute(interaction) {  
    
        // 控制台输出哪位用户在哪个频道触发了一次交互（斜杠命令）
        console.log(`${interaction.user.tag} in #${interaction.channel.name} triggered an interaction.`);  
   },  
};
```

`events/ready.js`
```js
// module.exports 提供了暴露接口的方法
module.exports = {  

    // 指明 name
    name: "ready",  
    
    // 指明该 module 在程序全程只能调用一次
    once: true,  

	// 函数 execute
    execute(client) { 
     
        // 控制台输出机器人以什么身份登录 discord 的
        console.log(`Ready! Logged in as ${client.user.tag}`);  
    },  
};
```

`name` 属性说明该文件用于哪个事件，而 `once` 属性是一个布尔值，用于指定事件是否应该只运行一次。 `execute` 函数用于你的事件逻辑，只要事件发出，事件处理程序就会调用它。

在我们自己写模块的时候，需要在模块最后写好模块接口，声明这个模块对外暴露什么内容，module.exports 提供了暴露接口的方法。

module.exports 是你在 Node.js 中导出数据的方式，以便你可以在其他文件中 require() 它。

### 下面来看 `cmdPaths.js`

文件的 data 域内只有一个数组，用于存储我们需要注册斜杠命令的 js 源代码的文件夹位置。

```js
module.exports = {  
    data: ["./commands"],  
};
```

如果你需要添加更多文件夹的话，可以修改成类似这样子：

```js
module.exports = {  
    data: ["./commands", "./commands/utils", "./commands/moderations"],  
};
```

这些文件夹必须存在，否则会报错。

### 下面来看 `index.js`

咱们分开来看

```js
const fs = require('fs');  
const { Client, Collection, GatewayIntentBits } = require('discord.js');  
const { token } = require('./config.json');  
const { InteractionType } = require("discord-api-types/v10");  
  
const client = new Client({ intents: [GatewayIntentBits.Guilds, GatewayIntentBits.GuildMessages, GatewayIntentBits.MessageContent] });  
const eventFiles = fs.readdirSync('./events').filter(file => file.endsWith('.js'));  

for (const file of eventFiles) {
    const event = require(`./events/${file}`);
    if (event.once) {
        client.once(event.name, (...args) => event.execute(...args));
    } else {
        client.on(event.name, (...args) => event.execute(...args));
    }
}

...
```
`fs` 模块能够以标准 POSIX 函数为模型的方式与文件系统进行交互。

`Client` 来自 discord.js，是一个继承来的 class（类），是与 Discord API 交互的主要枢纽，也是任何机器人的起点。Client 类具有十分多的属性、方法和事件。

`Collection` 来自 discord.js，是一个实用程序类，是一个带有其他实用方法的 Map。它扩展了 JavaScript 的原生 Map 类，因此它具有所有 Map 功能。Map 允许在唯一键和它们的值之间建立关联。 Collection 上的许多方法对应于它们在 Array 中的同名方法。

`GatewayIntentBits` 来自 discord.js。Gateway Intents是由 Discord 引入的，因此机器人开发人员可以根据需要运行的数据来选择他们的机器人接收的事件。Intent 是一组命名的预定义 WebSocket 事件，discord.js 客户端将接收这些事件。如果你没有提供 Intent，discord.js 会抛出一个错误。

接着读取 `config.json` 到对象 token 里。

`InteractionType` 是预先定义好的 Interaction 的类型表。Interaction Type 有 5 种，名称及其对应值分别是 PING(1)、APPLICATION_COMMAND(2)、MESSAGE_COMPONENT(3)、APPLICATION_COMMAND_AUTOCOMPLETE(4)、MODAL_SUBMIT(5)。

接下来我们 new 了一个 client，它的类型是 Client。这是你为 Discord 机器人创建客户端实例并登录 Discord 的方式。 `GatewayIntentBits.Guilds` 这个 Intent 选项是你的 client 正常工作所必需的，其余的是为了后续代码而添加的。

接下来我们从 index.js 所在的目录（即工程目录）下的 `events` 文件夹读取所有以 `.js` 为文件名结尾的文件名到 `eventFiles`。

后面的 for 循环就是遍历 `eventFiles` 里的文件名所对应文件，如果模块具有 once 属性，则在 client 启动时只执行一次就完事，这里其实只有 `events\ready.js` 的模块；反之，则保持执行，退出了也要再次调用起来。这里只有 `events\interactionCreate.js`


```js
...

client.commands = new Collection();  
const cmdPaths = require("./cmdPaths.js").data;  
const commandFiles = [];  
for (let i = 0; i < cmdPaths.length; i++) {  
    commandFiles[i] = fs.readdirSync(cmdPaths[i]).filter(file => file.endsWith(".js")); // fs.readdirSync() 的结果是个数组，所以 commandFiles是个二维数组  
    for (let j = 0; j < commandFiles[i].length; j++) {  
        commandFiles[i][j] = cmdPaths[i] + "/" + commandFiles[i][j];  
    }  
}  
  
for (const fileArray of commandFiles) {  
    for (const file of fileArray) {  
        const command = require(`./${file}`);  
        client.commands.set(command.data.name, command);  
  
        // if any ‘aka' name exists  
        if (command.akaNames != null && command.akaNames !== []) {  
            for (let i = 0; i < command.akaNames.length; i++) {  
                client.commands.set(command.akaNames[i], command);  
            }  
        }  
    }  
}

...
```
上面的代码中，我们 new 了一个 Collection 到 client.commands 里。

后面的读取 js 文件和上面从 events 文件夹读取类似。从 `cmdPaths.js` 的 data 域里，读取预先记录好文件夹路径到 `cmdPaths`。

接着读取 `cmdPaths` 所存储的所有文件夹下面的所有 js 文件的相对路径名，保存到数组 `commandFiles[i][j]` 里。i 是有关于文件夹的迭代下标，j 是关于同一 i 时的文件我迭代下标。

akaNames 是用来给同一个斜杠命令起不同命令名用的，就像磁盘上的同一文件可以用于多个文件名那样。

```js
...

client.once('ready', () => {});  
client.on('interactionCreate', async interaction => {  
    if (interaction.type !== InteractionType.ApplicationCommand) return;  
  
    const command = client.commands.get(interaction.commandName);  
  
    if (!command) return;  
  
    try {  
        await command.execute(interaction);  
    } catch (error) {  
        console.error(error);  
        await interaction.reply({  
            content: 'There was an error while executing this command!',  
            ephemeral: true  
        });  
    }  
});

...
```
上面第一行，client 对象准备就绪后，运行所有只执行一遍的 client.once 代码。

后面，运行需要保持运行的 cilent.on 代码，这里我们使用的是一个 async 修饰的箭头函数。

函数内， `interaction.type !== InteractionType.ApplicationCommand` 用于确认这个 interaction 交互的类型是斜杠命令。如果不是斜杠命令，则返回。要记住，并不是每个交互都是在触发斜杠命令。

后面通过 `client.commands.get()` 方法，使用名称获取到具体是我们的哪个斜杠命令，并将其分配给变量 `command`。 如果该命令不存在，它将返回 undefined，然后 return。如果确实存在这个命令，则调用命令的 `.execute()` 方法，并将变量 `interaction` 作为其参数传入。 

如果 `.execute()` 出现了错误，则将抛出一个异常，并在 Discord 上回复 `There was an error while executing this command!`，且只有命令发起者和机器人自己能看到。

```node
client.login(token);
```

这行代码就是使用你机器人的令牌登录 Discord。

### 下面来看 `deploy_commands.js`

一样，分部分来看。

```js
const fs = require("fs");  
const { REST } = require('@discordjs/rest');  
const { Routes } = require('discord.js');  
const { clientId, guildId, token } = require('./config.json');

...
```
Discord 允许开发人员注册斜杠命令，这为用户提供了与你的应用程序直接交互的"一等"方式。 在能够回复命令之前，你必须先注册它。

斜杠命令分为服务器命令（guild commands）、全局命令（global commands）、选项（options）、选项类型（option types）、选择（choices）和子命令（subcommands）。我们刚开始不需要了解这么多。

其实上面这四行代码就是导包和导配置。

```js
...

const commands = [];   
const cmdPaths = require("./cmdPaths.js").data;  
const commandFiles = [];  
for (let i = 0; i < cmdPaths.length; i++) {  
	commandFiles[i] = fs.readdirSync(cmdPaths[i]).filter(file => file.endsWith(".js")); // fs.readdirSync() 的结果是个数组，所以 commandFiles是个二维数组  
	for (let j = 0; j < commandFiles[i].length; j++) {  
		commandFiles[i][j] = cmdPaths[i] + "/" + commandFiles[i][j];  
	}  
}

//console.log(commandFiles);  
for (const fileArray of commandFiles) {  
	for (const file of fileArray) {
		console.log(file);  
		let command = require(`./${file}`);  
		commands.push(command.data.toJSON());  
  
		// if any ‘aka' name exists  
		if (command.akaNames != null && command.akaNames !== []) {  
			for (let i = 0; i < command.akaNames.length; i++) {  
				let akaData = command.data;  
				akaData.name = command.akaNames[i];  
				commands.push(akaData.toJSON());  
			}
		}
	}
}

...
```
上面这就是读取所有 `cmdPaths.js` 里记录的文件夹下的 js 文件。

```js
...

const rest = new REST({ version: '10' }).setToken(token);  
  
rest.put(Routes.applicationGuildCommands(clientId, guildId), { body: commands })  
   .then(() => console.log('Successfully registered application commands.'))  
   .catch(console.error);

...
```
这里就是根据你的 `config.json` 的配置来向特定服务器注册命令。成功时，控制台输出 `Successfully registered application commands.`

你只需要运行一次 `node deploy_commands.js` 来向单一服务器注册命令。 只有在添加或编辑现有命令时才应再次运行它。

### 下面来看 `commands/ping.js`

我们的第一个斜杠命令就来自 `ping.js`。

事实上，决定命令名的不是 js 文件名，而是由各文件 `.setName()` 方法决定。，因此事实上，这些文件名可以随便起。

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

第一行导包。事实上我们的每个斜杠命令都将需要包 `discord.js`。

后面暴露数据，new 了个斜杠命令。

通过 `.setName()` 设置斜杠命令的名称。

通过 `.setDescription()` 来设置命令的简介描述。

通过 async 修饰的 `execute()` 来执行命令该执行的。这里我们就回复了 `Pong!`，注意加上 await 关键字。

后面我们还会学到什么是 ephemeral response、如何执行多条回复、如何修改回复、如何删除回复、如何延长回复时间、如何让回复更花哨等。

### 结语

这样，我们就完成了对 Chapter 2 的代码的解释。