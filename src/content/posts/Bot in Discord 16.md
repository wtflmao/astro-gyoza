---
title: Bot in Discord with discord.js (16)
date: 2023-04-10
tags: [Discord bot, JS]
category: Discord.js
summary: 这是一份非常详细和系统化的利用 NodeJS 进行 Discord bot 开发的教程，本文介绍了什么是命令本地化 Localization，也就是“多语言化”，或者叫“本地翻译”。本地化降低了不同语言导致的语言障碍，提高用户满意度。
---


# Chapter 19 - 命令本地化

本文编写时，项目已升级至 discord.js@v14.9.0。

### 本地化（Localization）是什么

本地化是指将软件或产品适应不同的语言、文化和地区的过程。

本地化可以帮助你的程序在全球范围内更好地接受和使用，从而提高用户体验和满意度。

本地化可以帮助你的程序更好地适应当地的文化、习惯和法律法规，从而避免很多问题，特别是你考虑不到的习惯和文化细节的问题。

引入本地化可以带来以下好处：提高竞争优势、增加业务收入、减少市场进入障碍。

如果你想要进军国际市场，那么本地化是非常重要的。

### 为命令引入本地化

首先我们需要判断用户的语言/区域设置（locale），这个值就是用户使用 Discord 发送命令时，使用的 Discord 客户端（包括网页版、手机 App、桌面 App 等）里的语言设置。这个设置是跨平台使用的，即用户在一处设置语言，Discord 将处处使用该语言作为客户端 UI 展示用语言。

截至本文撰写时，Discord 有效 locale值有：

|Locale|Language Name|语言名称|Native Name|
|:----|:----|:----|:----|
|id|Indonesian|印度尼西亚语|Bahasa Indonesia|
|da|Danish|丹麦语|Dansk|
|de|German|德语|Deutsch|
|en-GB|English, UK|英语（英国）|English, UK|
|en-US|English, US|英语（美国）|English, US|
|es-ES|Spanish|西班牙语|Español|
|fr|French|法语|Français|
|hr|Croatian|克罗地亚语|Hrvatski|
|it|Italian|意大利语|Italiano|
|lt|Lithuanian|立陶宛语|Lietuviškai|
|hu|Hungarian|匈牙利语|Magyar|
|nl|Dutch|荷兰语|Nederlands|
|no|Norwegian|挪威语|Norsk|
|pl|Polish|波兰语|Polski|
|pt-BR|Portuguese, Brazilian|葡萄牙语（巴西）|Português do Brasil|
|ro|Romanian, Romania|罗马尼亚语（罗马尼亚）|Română|
|fi|Finnish|芬兰语|Suomi|
|sv-SE|Swedish|瑞典语|Svenska|
|vi|Vietnamese|越南语|Tiếng Việt|
|tr|Turkish|土耳其语|Türkçe|
|cs|Czech|捷克语|Čeština|
|el|Greek|希腊语|Ελληνικά|
|bg|Bulgarian|保加利亚语|български|
|ru|Russian|俄语|Pусский|
|uk|Ukrainian|乌克兰语|Українська|
|hi|Hindi|印地语|हिन्दी|
|th|Thai|泰语|ไทย|
|zh-CN|Chinese, China|简体中文|中文|
|ja|Japanese|日语|日本語|
|zh-TW|Chinese, Taiwan|繁体中文|繁體中文|
|ko|Korean|韩语|한국어|

我们下面使用一个示例斜杠命令来解释如何为命令实现本地化。

下面我们将实现多语言回复“你好”的命令 `/hello`：
```js
const { SlashCommandBuilder, ButtonBuilder, ActionRowBuilder, ButtonStyle } = require('discord.js');

module.exports = {
    data: new SlashCommandBuilder()
        .setName('hello')
        // you can set localized names in bulk
        .setNameLocalizations({
            'en-GB': 'hello2',
            'zh-CN': '你好',
            'zh-TW': '你好',
            'cs': 'ahoj',
            'ru': 'привет',
            'ko': '안녕하세요',
            'es-ES': 'hola'
        })
        .setDescription('Replies with hello, but in localized response!')
        // you can set localized descriptions in bulk
        .setDescriptionLocalizations({
            'zh-TW': '回复你好，但以本地化回复！',
            'cs': 'Odpovědi ahoj, ale v lokalizované odpovědi!',
            'en-GB': 'Replies with hello, but in localized response2!',
            'es-ES': '¡Responde con hola, pero en respuesta localizada!',
            'ko': '안녕하세요로 회신하지만 현지화된 응답으로!',
            'zh-CN': '回复你好，但以本地化回复！',
            'ru': 'Отвечает приветствием, но локализованным ответом!'
        })
    ,
    async execute(interaction) {
        const HelloWorldLocales = {
            'zh-CN': '你好世界！',
            'en-GB': 'Hello world2!',
            'zh-TW': '你好世界！',
            'cs': 'Ahoj světe',
            'ko': '안녕 세상!',
            'es-ES': '¡Hola Mundo!',
            'ru': 'Привет, мир!',
        };

        const eatLocales = {
            'en-GB': 'Eat2',
            'zh-CN': '吃',
            'es-ES': 'Comer'
        }
        const sleepLocales = {
            'zh-CN': '睡觉',
            'es-ES': 'Dormir'
        }

        // add two buttons that are absolutely useless
        // just a localization example
        const row = new ActionRowBuilder()
            .addComponents(
                new ButtonBuilder()
                    .setCustomId(`b1`)
                    .setLabel(eatLocales[interaction.locale] ?? 'Eat')
                    .setStyle(ButtonStyle.Primary),
                new ButtonBuilder()
                    .setCustomId(`b2`)
                    .setLabel(sleepLocales[interaction.locale] ?? 'Sleep')
                    .setStyle(ButtonStyle.Danger),
            );

        // default is English: Hello world!
        await interaction.reply({
            content: `${HelloWorldLocales[interaction.locale] ?? 'Hello world!'}, locale:(${interaction.locale}})`,
            components: [row]
        });
    },
};
```

### 效果图

#### 语言设置为 zh-CN （已完全翻译）
![image](https://img2023.cnblogs.com/blog/2455224/202304/2455224-20230410004054782-1296409362.png)

![image](https://img2023.cnblogs.com/blog/2455224/202304/2455224-20230410004100726-1586746703.png)

![image](https://img2023.cnblogs.com/blog/2455224/202304/2455224-20230410004107247-1693431343.png)

#### 语言设置为 es-ES （已完全翻译）
![image](https://img2023.cnblogs.com/blog/2455224/202304/2455224-20230410004132736-805387899.png)

![image](https://img2023.cnblogs.com/blog/2455224/202304/2455224-20230410004137916-1397801290.png)

![image](https://img2023.cnblogs.com/blog/2455224/202304/2455224-20230410004148445-414270500.png)

#### 语言设置为 ru （未翻译按钮）
![image](https://img2023.cnblogs.com/blog/2455224/202304/2455224-20230410004455186-775268293.png)

![image](https://img2023.cnblogs.com/blog/2455224/202304/2455224-20230410004503871-1561345516.png)

#### 语言设置为 ja （完全未翻译）
![image](https://img2023.cnblogs.com/blog/2455224/202304/2455224-20230410004613590-2078259659.png)

![image](https://img2023.cnblogs.com/blog/2455224/202304/2455224-20230410004616652-516882406.png)

### 小结

我们学习了在 discord.js 里进行本地化翻译的基本操作

原文作者闪电豹猫，原文链接https://www.cnblogs.com/hhzm/p/17301564.html 转载注明出处