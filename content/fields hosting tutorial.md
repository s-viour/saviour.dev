+++
tags = ["programming", "tutorial"]
title = "fields hosting tutorial"
index = "0002"
slug = "0002"
layout = "default"
+++
fields is a discord bot designed to play a single mp3 file on loop forever, and it can be hosted for completely free

**Setting Up the Discord Application and Bot User:**\
Before we set up the bot itself, you'll need to register a bot account on discord, so navigate to [discord.com/developers/applications](https://discord.com/developers/applications) and login. Click the "new application" button in the top right, and name it whatever you want. I'll be calling my bot "tutorial". After clicking "create" you should see this:


![](/img/fields/fields_tutorial1.png)

After creating the application, you'll need to create the **bot user**, this is what will actually sit in the channel and play the audio. So click the "Bot" button on the left (highlighted in screenshot above). Click the "Add Bot" button, and confirm that you do indeed want to create a bot user. Change its name and profile picture if you want, and keep this tab up for later.

While we're here, we'll go ahead and join the bot to any and all servers you want this bot to work in. Next to where you accessed the "Bot" tab, click on the "OAuth2" tab. Scroll down to the big bunch of checkboxes, and select "bot":

![](/img/fields/fields_tutorial4.png)

The highlighted link is what you'll use to join the bot to any servers you want the bot in. Simply click on it and discord will ask you to select the server. If you don't have permission to add bots in the server you want it in, just give the link to someone who does and let them add it.

**Setting up the Host:**\
Now we'll need to create a replit account to actually host the bot. So navigate to [replit.com](https://replit.com) and login with whatever method you want. After you've set up your account, we'll need to import an instance of the bot. Click the plus button in the top right and select the "import from github" tab. into that box, paste `s-viour/fields` and click the "Import from Github" button.

**Configuring the Bot:**\
All that's left is to configure the bot! So, remember that tab we left open after creating the discord bot? Navigate to it and **copy your token**:
 
![](/img/fields/fields_tutorial2.png)

Keep your bot token secret! don't give anyone else your token, or they could use it to access your bot. That's why the discord webpage hides your token by default. Anyway, copy your token and go to your replit page. Click the button that looks like a lock (it should be on the left side of your screen, and say "Secrets (Environment Variables)")
enter `BOT_TOKEN` in the `key` section of the secret, and then paste **your token** into the `value` section. It should look like this when you're done: 

![](/img/fields/fields_tutorial3.png)

Click "Add new secret", and don't worry about that clearly visible token, I've deleted this bot, so it isn't valid anyway. Now, we should upload an mp3 file to the bot (or, you can use the default `ominous wind` or `smoke detector` audio files). Click on the button that looks like a file-icon on replit, and open the audio folder. You should just be able to drag-and-drop any mp3 file into the folder. I don't think replit will let you upload anything too large though, so if the website freezes, you may have to go smaller. (as an anecdote, the largest file size I've had success with is like a 30-minute mp3 file. any larger and everything just kinda freezes up) anyway, now you need to tell the bot which audio-file to play. So open the `config.json` file. It should look something like this: 
```
{
    "audioPath": "audio/wind.mp3",
    "channelIDs": []
}
```
Replace `audio/wind.mp3` with the name of your audio file you uploaded, or leave it as the default for the ominous wind. Notice the `channelIDs` section of the config, we'll need to list **every** channel id the bot needs to work in. So copy all of them, and enter them comma-separated into those brackets. To copy a channel id, right-click on it and click "Copy ID". If you don't see "Copy ID", make sure you have developer mode turned on in your settings. When you're done setting up the ids list, it should look like this:
```
{
    "audioPath": "audio/wind.mp3",
    "channelIDs": [796817361545986062]
}
```
**You cannot have the bot in more-than-one channel per server!**

After that, just click the big green "Run" button on replit! The bot should begin starting up. after the bot starts, it should immediately join all channels in the channelIDs list. **If you ever want to have it join more channels, you will have to restart it.** If something went wrong, and it didn't join, see the program log on replit for any errors that occurred. If you look at the error log and can't figure out what went wrong, create an issue [here](https://github.com/s-viour/fields/issues) and we'll try to help. 


**Setting up Uptimerobot**\
If you've successfully executed every step up until now, you're almost done! There's only one thing left to do, and that's to make the bot run forever. Replit kills the bot after one hour of inactivity, but we can use a service called [uptimerobot](https://uptimerobot.com/) to cheat a little. Create an account on the service and login. You'll be greeted with a big dashboard. Simply click the "Add New Monitor" button near the top-left and set the "Monitor Type" as "HTTP(s)". Now, navigate back to replit and copy the URL in the top-right box. Paste the URL into the uptimerobot "URL (or IP)" box. Set the "Friendly Name" as whatever you want. If you want to be notified when the bot goes down (that is, if repl goes offline for some reason), check the box on the right side with your email in it. When you're done, the page should look like this:

![](/img/fields/fields_tutorial5.png)

Click "Create Monitor" and you're done! The bot is hosted on replit, with Uptimerobot pinging it every 5 minutes to keep it up. Once again, if you experience any problems with the bot:
1. First, pay attention to the error log, it may tell you what went wrong!
2. If that doesn't help you, create an issue on the [github page](https://github.com/s-viour/fields/issues) 

