---
title: "How to use DiscordChatExporter to scrape / export Discord servers and DMs"
classes: wide
header:
  teaser: /assets/images/scraping/discord/usage.png
ribbon: DodgerBlue
description: "How to use  used an open-source tool to scrape discord messages easily"
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

![release](/assets/images/scraping/discord/copyguild.png)

In this case, the value we received is `600262311302922261`
Now, to find our account token, I think DiscordChatExporter's manual explained it here pretty straightforwardly:

![release](/assets/images/scraping/discord/findtoken.png)

you can export with different formats like `.txt`, `.json`, etc using the `-f` option.

So the command we get is `DiscordChatExporter.Cli exportguild --g 600262311302922261 -t <mytoken>`.


Let's run our command:

 ![release](/assets/images/scraping/discord/scraping-cli.png)

And, about 20 minutes later, the scraping finished! (tip: for very large servers, just do the downloading while you sleep!)


 ![release](/assets/images/scraping/discord/success.png)

 ![release](/assets/images/scraping/discord/files.png)


As you can see, we've received neat `.html` files for every channel. (Note: you can export with different formats like `.txt`, `.json`, etc using the `-f` option!)



