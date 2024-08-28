---
title: Bot in Discord with discord.js (17)
date: 2024-05-09
tags: [Discord bot, JS]
category: Discord.js

---

# Chapter 20 - Webhook 钩子

Webhook 是一种将消息发布到 Discord 频道的而无需使用 Discord 的省力方式。它不需要你有一个机器人账号或身份验证即可使用这个东西。和 Discord API 一样，Webhook 对于自动化程序来说是十分友好的。

创建和管理一个频道 Webhook 钩子是很轻松的，但是你不能直接修改或删除由 Webhook 发送的消息。

下面我们来讲如何创建一个 Webhook 钩子。

### 手动创建一个 Webhook

打开 Discord，打开服务器设置，左侧侧栏找到并点开 “整合”（Integeration），点击 Webhook，点击 “新 Webhook”，Discord 会新建一个 Webhook 给你。

然后在下面的列表里找到新建的 Webhook，你可以修改钩子的名称和作用的频道（只能选一个，如果需要作用到多个频道，请考虑使用多个钩子或使用机器人）。然后保存。

点击复制 Webhook URL，你就拿到了直接给作用频道发送消息的钩子地址。任何有了这个 URL 的人，都可以往对应频道里发送消息，不需要身份认证。所以，请妥善保存号这个 URL，若不慎泄露，请删除这个钩子。

修改一个已有 Webhook 的作用频道不会导致其 URL 的改变。

### 钩子的格式？

Webhook URL 形式一般如下：
```
https://discord.com/api/webhooks/112183042530477****/1KlxJ3nqTOb_8xPa5iPulgAUz1cX9XcZ157iJGQmiuP8Uqwr****_aeSj0z2pYp****
```
或
```
https://ptb.discord.com/api/webhooks/112183042530477****/1KlxJ3nqTOb_8xPa5iPulgAUz1cX9XcZ157iJGQmiuP8Uqwr****_aeSj0z2pYp****
```

可以看到固定的格式：API Endpoint + 钩子 ID + 钩子 token

根据 Discord API，Webhook 的结构如下表：

| 域 | 类型 | 说明 |
|---|---|---|
| id | snowflake | Webhook 的 ID |
| type | integer | Webhook 的类型 |
| guild\_id? | ?snowflake | Webhook 所在的服务器 ID（如果有的话） |
| channel\_id | ?snowflake | Webhook 所在的频道 ID（如果有的话） |
| user? | user object | 创建该 Webhook 的用户的 ID \(当使用 token 请求时，不会返回该 ID\) |
| name | ?string | 发送消息时展示的默认名字 |
| avatar | ?string | 发送消息时展示的默认头像 |
| token? | string | Webhook 的安全 token（为 Incoming Webhooks 而生） |
| application\_id | ?snowflake | 创建该 Webhook 的 Bot 或 OAuth2 的 ID |
| source\_guild? \* | partial guild object | Webhook 所在频道的服务器（注意返回类型，由 Channel Follower Webhooks 而生） |
| source\_channel? \* | partial channel object | Webhook 所在频道（注意返回类型，为 Channel Follower Webhooks 而生） |
| url? | string | 执行 Webhook 的 URL \(由 Webhooks OAuth2 流返回\) |

### 拿到这个钩子 URL 后，怎么用？

如果你是用户，可以在已经集成了 Discord Webhook 的平台，比如代码托管平台 Github，团队沟通平台 Slack粘贴钩子地址进去即可使用。

如果你是开发者，你可以选择直接对接 Discord API 来使用钩子，功能最全，更新最快，详见官方 API 文档。当然，咱们是 Discord.js 开发者啊，当然得介绍介绍 Discord.js 关于 Webhook 的那点儿支持啊哈哈。

在此之前，我们先来看看手动调 Discord Webhook API 的方法。

### Discohook.org - 一个调试 Discord Webhook 的网站

打开网站，在 `Webhook URL` 填入你的钩子 URL。我们可以看到 Webhook 可以发送很多东西，就行一个普通用户或一个 Bot 那样。

- Content：发送的消息内容，和我们在斜杠命令里的 `content` 域一致，长度限制 2000 字符。
- 
- Profile：发送消息的自定义 Profile，其中：
	- Username：展示的消息发送者的用户名，可以随便填，若不填，则使用钩子的名字，长度限制 80 字符。
	- Avatar：展示的消息发送者的头像，可以随便填，若不填，则使用 Discord 默认头像。

- Thread：发送到的子区，其中：
	- Forum Thread Name：如果 Webhook 所使用的频道类型是 `Forum Channel`，你必须提供这个名字，以供 Discord 创建这个名字的子区。

- Flags：标志位，其中：
	- Supress Embeds：隐藏链接嵌入。 这不能与丰富的嵌入（使用“添加嵌入”创建）结合使用。勾选此项将。。。。。。。。。。。
	- Suppress Notifications：在 `content` 域中，如果你 @ 了用户或身份组，准确的说是你使用了任何的 mention，则是否向被 @ 的对象发送通知。勾选此项将不会向被 @ 的对象发送通知。

- Files：上传的文件，最大 25 MB。

- Embeds：嵌入式对象。我们在前面的文章中介绍过这个类型。对于一个 Embed，你可以设置的域由如下组成：
	- Author：作者，其中：
		- Author：作者名，最长 256 字符。
		- Author URL：作者链接，当点击作者名时，跳转到的 URL。
		- Author Icon URL：作者头像的 URL。
	- Body：
		- Title：标题，最长 256 字符。
		- Description：内容，最长 4096 字符
		- URL：当点击标题时，跳转到的链接 URL。
		- Color：Embed 侧边彩条的颜色，默认为 HEX `#202225` ，推荐颜色 Blurple `#5865F2` 。
	- Fields：表格域，其中对于每一个 Field，有：
		- Field Name：表格域题头，最长 256 字符。
		- Field Value：表格域值，最长 1024 字符。
		- Inline：勾选则该域是个内联域，允许最少连续两个表格域显示时排列在一行上；不勾选则该域在显示是，将独占一行。
	- Images：图片，其中：
		- Image URLs：图片 URL，显示在 Description 的下方。当且仅当 `Embeds - Body - URL` 存在是，你可以设置 4 个图片 URL。否则，只能设置 1 个图片 URL。
		- Thumbnail URL：缩略图 URL，以小图形式显示在作者名以及标题的右侧。
	- Footer：脚底信息，其中：
		- Footer：脚底文字，最长 2048 字符。
		- Timestamp：显示的时间戳，格式 `YYYY-MM-DD hh:mm`。
		- Footer Icon URL：脚底小图标 URL。

- Message Link：消息 ID。输入同频道内的某条消息的 ID 并点击旁边的 Load，将现成消息作为模板导入编辑器中。**此操作会覆盖编辑器里的所有数据。**

你还可以进一步对要发送的消息 JSON 数据进行高级编辑。点击 `JSON Editor` 按钮，可以看到完整的消息 JSON 数据以及报错提示（比如你点击添加了表格域但是没有设置表格域的题头）。在编辑器里，点击 `Copy to Clipboard` 复制到剪切板，点击 `Apply Changes` 保存更改。

在这一切完毕后，你可以回到网页最上方，点击 Send 使用 Webhook 向频道里发送消息。

