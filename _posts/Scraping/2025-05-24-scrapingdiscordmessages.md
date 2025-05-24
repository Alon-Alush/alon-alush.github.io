---
title: "How I used DiscordChatExporter to scrape millions of Discord messages"
classes: wide
header:
  teaser: /assets/images/pefileformat/iat/image.png
ribbon: DodgerBlue
description: "How I used an open-source tool to scrape discord messages easily"
categories:
  - Scraping
toc: true
---
# Scraping Discord messages easily

In this tutorial, we'll explore how to use [DiscordChatExporter](https://github.com/Tyrrrz/DiscordChatExporter) to scrape entire Discord servers.

To start, go to their GitHub repository, and download the suitable release for your environment.

I downloaded the CLI version for x64 Windows, `DiscordChatExporter.Cli.win-x64.zip`:

![release](/assets/images/scraping/discord/release.png)

Now, to use the tool, it's very simple.

![release](/assets/images/scraping/discord/usage.png)

To export an entire guild, you can use the following command template:
```
DiscordChatExporter.Cli exportguild --guild <value> --token <value> [options]
```

So let's find now find the values we need.

First, to get the guild number, right-click the server icon and select `Copy Server ID`:

![release](/assets/images/scraping/discord/copyguild.png)

In this case, the value we received is `600262311302922261`
As to how to get the token, I think it's explained here pretty straightforwardly

![release](/assets/images/scraping/discord/findtoken.png)

So the command we get is `DiscordChatExporter.Cli exportguild --guild 600262311302922261 <mytoken>`.

Let's run it:

 ![release](/assets/images/scraping/discord/scraping-cli.png)

And, about 20 minutes later, the scraping finished! (tip: for very large servers, just do the downloading while you sleep!)


 ![release](/assets/images/scraping/discord/success.png)

 ![release](/assets/images/scraping/discord/files.png)


As you can see, we've received neat `.html` files for every channel. (Note: you can export with different formats like `.txt`, `.json`, etc using the `-f` option!)



