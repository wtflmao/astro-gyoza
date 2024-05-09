---
title: Bot in Discord with discord.js (11)
date: 2022-11-23
tags: [Discord bot, JS]
category: Discord.js
summary: 这是一份非常详细和系统化的利用 NodeJS 进行 Discord bot 开发的教程，本文介绍了什么是表单 Modal，可理解为简单的收集表。
---

## Chapter 14 - 交互四大组件之：表单 Modal

建议你先学习上上一章的按钮，再来学习这章的 Modal，因为上一章已经比较详细地介绍了 MessageComponentCollector，虽然上一章的 MessageComponentCollector 在本章的 Modal 用不了，但是仍具有借鉴意义。

### 修改 `events/interactionCreate.js`

在文件的最后那块儿附近，找到最后一个 else，在这个 else 上面插入一种新情况，用于 Modal：
```js
else if (interaction.isChatInputCommand()) {
	console.log("a modal!");
}
```

### Modal

Modal 可以让你的机器人弹出一个表单，用户可以通过这个表单为你提供格式化的输入。

与消息组件（Message Component）不同，Modal 本身并不是严格意义上的组件。它们是用于响应交互的回调结构。

每个 ModalBuilder 最多可以有五个 ActionRowBuilder，一个 ActionRowBuilder 中最多可以有一个 TextInputBuilder。目前，你不能在 Modal row 中使用 SelectMenuBuilders 或 ButtonBuilders。

> 提前说明一点，`ModalSubmitInteraction` 不属于 `MessageComponentInteraction`，而 `ButtonInteraction` 和 `SelectMenuInteraction` 属于 `MessageComponentInteraction`，这一点可能会让你感到奇怪。`ModalSubmitInteraction` 和 `MessageComponentInteraction` 都直接继承自 `BaseInteraction`，而区别是前者（Modal那个）还实现（implement）了 `InteractionResponses`。初学者在搞 Modal 的收集器时，很容易按照之前搞按钮和下拉菜单的经验，把收集器代码搬过来改一下就用，然鹅这是错的。

### 创建一个 Modal

我们至少需要用到 discord.js 的 ModalBuilder 来建立一个 Modal 值。

```js
const { Events, ModalBuilder } = require('discord.js');

const modal = new ModalBuilder()
	.setCustomId('myModal')
	.setTitle('My Modal');

	// TODO: Add components to modal...
};
```

需要指出的是，customId 是一个由程序员指定的长度不超过 100 字符的字符串。尽量不要让它们的 customId 与其他交互组件的 customId 重复，从而让 filter 更轻松地工作。

目前我们至少建立了一个空表单，什么输入域都没有指定。

下面，我们将建立一个完整的 Modal，具备完整的展示 Modal 和收集数据并响应功能。

`commands/modals/modal.js`
```js
const { ActionRowBuilder, ModalBuilder, SlashCommandBuilder, TextInputBuilder, TextInputStyle } = require('discord.js');

module.exports = {
    data: new SlashCommandBuilder()
        .setName('modal')
        .setDescription('Replies with a pop-up form!'),
    async execute(interaction) {

        const modal = new ModalBuilder()
            .setCustomId('myModal')
            .setTitle('My Modal');

        // 向交互组件里添加一个 Modal
        // 创建文本输入域
        const favoriteColorInput = new TextInputBuilder()
            .setCustomId('favoriteColorInput')
            // Label 是用户看到的有关该域 的提示语，用来告诉用户这里是该填什么的
            .setLabel("What's your favorite color?")
            // "Short" 意味着该域接受一行文本输入
            // TextInputStyle 只有两种值： .Short 和 .Paragraph
            .setStyle(TextInputStyle.Short)
            // .SetRequired(true) 意味着该文本域必填
            .setRequired(true)
            // 为该文本域设置一个占位符，占位符不是预先写好的值
            .setPlaceholder("Blurple");

        const hobbiesInput = new TextInputBuilder()
            .setCustomId('hobbiesInput')
            .setLabel("What's some of your favorite hobbies?")
            // "Paragraph" 意味着该域是个大文本框，接受多行输入
            .setStyle(TextInputStyle.Paragraph)
            // 设置该域必填
            .setRequired(false)
            // 为该域设置一个预先写好的值，不是占位符
            .setValue("Touching grass outside occasionally.")

        const numberInput = new TextInputBuilder()
            .setCustomId('numberInput')
            .setLabel ("Which year did you first use Discord?")
            .setStyle(TextInputStyle.Short)
            .setRequired(true)
            .setPlaceholder("2022")
            // 设置文本最小长度
            .setMinLength(4)
            // 设置文本最大长度
            .setMaxLength(4);

        // 一个 ActionRow 只能拥有一个 TextInput,
        // 所以你需要三个 ActionRow 来维护三个 TextInput
        const firstActionRow = new ActionRowBuilder().addComponents(favoriteColorInput);
        const secondActionRow = new ActionRowBuilder().addComponents(hobbiesInput);
        const thirdActionRow = new ActionRowBuilder().addComponents(numberInput);

        // 为表单 Modal 添加三个 ActionRow 组件
        modal.addComponents(firstActionRow, secondActionRow, thirdActionRow);

        // 向执行斜杠命令的用户展示 Modal
        // interaction.interaction.showModal(modal) 必须是斜杠命令的第一条回复，而不能是 .reply()
        // 你甚至不能 .defer() 或者 .deferUpdate()，第一条必须是 .showModal()。
        await interaction.showModal(modal);


        // 一旦用户提交了填写完了的 Modal，我们就抓取这个 ModalSubmitInteraction
        const submitted = await interaction.awaitModalSubmit({
            // 为有效 Modal 的填写设置超时 60000 毫秒，即 60 秒
            time: 60000,
            // 确保我们的 Modal 填写者是最初发起斜杠命令的那个人
            filter: i => {
                if (i.user.id === interaction.user.id && i.customId === 'myModal') {
                    return true;
                }},
        }).catch(error => {
            // 捕获任何抛出的 error (e.g. awaitModalSubmit 60 秒后 超时了)
            console.error(error)
            return null
        })

        // 只要我们获取到了有效提交的 Modal，我们就能干我们该干的事了
        // 记住，一份 Modal 可以具有多个 ActionRow，而一个 ActionRow 只有一个 TextInputComponent。
        // 你可以使用 ModalSubmitInteraction.fields 再凭借 customId 来获取用户在某个文本域输入的值
        if (submitted) {
            // 提取用户输入的数据
            const favoriteColor = submitted.fields.getTextInputValue('favoriteColorInput');
            const hobbies = submitted.fields.getTextInputValue('hobbiesInput');
            const number = submitted.fields.getTextInputValue('numberInput');
            console.log({ favoriteColor, hobbies, number });
            await submitted.reply({
                content: `Your fav color is ${favoriteColor}, you like ${hobbies}, you claimed that you registered Discord in ${number}.`
            })
        }
    },
};
```

效果图（第一次执行斜杠命令时）：

![image](https://img2022.cnblogs.com/blog/2455224/202211/2455224-20221123184542359-722808485.png)

![image](https://img2022.cnblogs.com/blog/2455224/202211/2455224-20221123184603579-197508538.png)

有一点需要说明的是，在本例中，第三个输入域向用户说明了需要输入长度为 4 的年份，正常人可能会输入“2022”、“2021”、“2020”之类的。事实上，“2077”、“abcd” 都是合法的输入，你需要进一步对该域的用户输入进行处理。

### 对 `ModalSubmitInteraction` 的其他操作

-   `reply()`
-   `editReply()`
-   `deferReply()`
-   `fetchReply()`
-   `deleteReply()`
-   `followUp()`

上面已经提到过，ModalSubmitInteraction 不属于 MessageComponentInetraction，而属于  ChatInputCommandInteraction。

如果如果 Modal 是从 ButtonInteraction 或 SelectMenuInteraction 显示的，它还将还有这些方法：

-   `update()`
-   `deferUpdate()`