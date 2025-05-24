---
title: "Using DiscordChatExporter to scrape / export Discord servers and DMs"
classes: wide
header:
  teaser: /assets/images/scraping/discord/usage.png
ribbon: DodgerBlue
description: "Easily scraping Discord messages with the help of DiscordChatExporter, an open-source tool"
categories:
  - Scraping
toc: true
---
# Scraping Discord messages easily

In this tutorial, we'll learn how to use [DiscordChatExporter](https://github.com/Tyrrrz/DiscordChatExporter) to scrape entire Discord servers / DMs (whatever you choose) very easily.

To start, go to [DiscordChatExporter's GitHub repository](https://github.com/Tyrrrz/DiscordChatExporter), and download the suitable release for your environment.

In my case, I downloaded the CLI version for x64 Windows, `DiscordChatExporter.Cli.win-x64.zip`:

![release](/assets/images/scraping/discord/release.png)

Now, after downloading, using the tool is extremely simple:

![release](/assets/images/scraping/discord/usage.png)

To export an entire guild, you can use the following command template:
```
DiscordChatExporter.Cli exportguild --guild <value> --token <value> [options]
```

So let's find now find the 2 values we need: the guild ID and our user account token.

First, to get the guild ID, right-click the server icon that you want to scape, and select `Copy Server ID`:

![Finding guild ID](/assets/images/scraping/discord/copyguild.png)

In this case, the value we received is `600262311302922261`

To find your account token, follow the manual below:

![Finding Discord account token](/assets/images/scraping/discord/findtoken.png)

you can export the data using different formats like `.txt`, `.json`, etc using the `-f` option.

So the command we get is `DiscordChatExporter.Cli exportguild --g 600262311302922261 -t <mytoken>`.


Let's run our command:

 ![Running our command](/assets/images/scraping/discord/scraping-cli.png)

And, about 20 minutes later, the scraping finished! (tip: for very large servers, just do the downloading while you sleep!)


 ![Scraping finished successfully](/assets/images/scraping/discord/success.png)
 
As you can see, we've received neat `.html` files for every channel. (Note: you can export with different formats like `.txt`, `.json`, etc using the `-f` option!)

 ![Scraped files](/assets/images/scraping/discord/files.png)

Let's add our scraped data to a `.zip` archive

 ![release](/assets/images/scraping/discord/addtoarchive.png)

 And voila, now we have the fully scraped server within our fingertips.

 To store the scraped data, I used a freshly-created [Telegram](https://web.telegram.org) channel:
![release](/assets/images/scraping/discord/telegramchannel.png)

Telegram offers unlimited cloud storage (yes, you can literally upload as many files as you want).

The only practical limitation is a file size limit per message: 2GB or 4GB depending on your plan (Free or Premium). So to upload large amount of data, just split to multiple uploads of 2GB/4GB.


I uploaded the `.zip` to my Telegram channel:

 ![release](/assets/images/scraping/discord/upload.png)

 You can repeat this process as many times as you'd like!




