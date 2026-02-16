---
title: "Discord Bots – MoveIt & MuteMediator"
sidebar:
  exclude: true
---

## Introduction

This project showcases two playful yet practical Discord bots, **MoveIt** and **MuteMediator**, created to experiment with Discord's API and gain experience building interactive integrations.  
Both bots were designed with a social twist, aimed at bringing humor and moderation tools into voice channels with friends.

- **MoveIt**: Rapidly moves a selected user through multiple voice channels before returning them to their original location.
- **MuteMediator**: Temporarily server-mutes two users to de-escalate heated discussions.

These projects provided an engaging way to learn about bot commands, member management, and asynchronous event handling in Discord.

> Note: These bots were personal learning projects intended for private server fun.

## Objectives

**MoveIt Bot**

- Create a command (`!move @user`) to move a target user through a predefined sequence of voice channels.
- Implement timed delays between moves for comedic effect.
- Return the user to their original channel at the end.

**MuteMediator Bot**

- Create a command (`!mm @user1 @user2`) to temporarily server mute two members.
- Automatically unmute after 30 seconds.
- Provide user feedback via chat messages.

**Shared Goals**

- Learn to use `discord.py` for command creation and member control.
- Work with asynchronous operations for timed actions.
- Build a foundation for future interactive bots.

## Program Architecture

Both bots share a similar structure using the `discord.py` library:

- **Command Definitions**:  
  Each bot defines its own command (`move` for MoveIt, `mm` for MuteMediator) using the `commands` extension.
- **Event Handlers**:  
  `on_ready` events confirm the bot’s startup in the console.
- **Asynchronous Actions**:  
  Use of `asyncio.sleep()` to implement timed delays between actions.
- **Configuration Management**:  
  Sensitive tokens stored in separate configuration files (`configMoveBot.py` / `configConflictResolution.py`).

## Program Flow

### MoveIt

1. User invokes `!move @DiscordUser`.
2. Bot stores the target user’s current voice channel.
3. Bot iterates through a hardcoded list of channel IDs:
   - Moves the user to each channel with a 0.5 second pause in between.
4. After the last channel, the bot returns the user to their original voice channel.

### MuteMediator

1. User invokes `!mm @User1 @User2`.
2. Bot server mutes both users.
3. Bot sends a confirmation message to the text channel.
4. Bot waits 30 seconds.
5. Bot unmutes both users and notifies the channel.

## Procedure

1. **Set Up a Discord Application**
   - Visit <https://discord.com/developers/applications>.
   - Create a new application, add a bot, and copy its token.
2. **Install Required Libraries**

    ```python
    pip install discord.py
    ```

3. Configure Tokens
   - Store bot tokens in a separate file apart from the main source code (e.g., configMoveBot.py and configConflictResolution.py).
4. Run the bots

    ```python {linenos=table,linenostart=1}
    python moveit.py
    python mutemediator.py
    ```

## Python Source Code

Below are the source code implementations for each bot.

MoveIt

```python {linenos=table,linenostart=1,filename="MoveIt.py"}
import discord
from discord.ext import commands
import asyncio
from configMoveBot import TOKEN

channelList = [1160108207117701131, 1196307563956949004, 1001008597049815090, 998433996008591450, 998434717051387994, 998434371524636814, 998434517171839046, 1043359687086714943]


bot = commands.Bot(command_prefix=["!"], intents=discord.Intents.all())

@bot.command()
async def move(ctx, member: discord.Member):
    original_channel = member.voice.channel
    for channel_id in channelList:
        channel = bot.get_channel(channel_id)
        await member.move_to(channel)
        print(f"Moved member to channel {channel.name}")
        await asyncio.sleep(.5)
    await member.move_to(original_channel)
    print(f"Moved member back to original channel {original_channel.name}")
    
@bot.event
async def on_ready():
    print('Bot is ready')

bot.run(TOKEN)
```

MuteMediator

```python {linenos=table,linenostart=1,filename="MuteMediator.py"}
import discord
from discord.ext import commands
import asyncio
from configConflictResolution import TOKEN 

# Set up intents and bot
intents = discord.Intents.default()
intents.members = True
bot = commands.Bot(command_prefix="!", intents=discord.Intents.all())

# Let the user know the bot is online
@bot.event
async def on_ready():
    print(f"{bot.user} is now online!")

# Define the Mute users command
@bot.command()
async def mm(ctx, member1: discord.Member, member2: discord.Member):
    await member1.edit(mute=True)
    await member2.edit(mute=True)
    await ctx.send(f"Muted {member1.display_name} and {member2.display_name} for 30 seconds.")
    await ctx.send(f"Bringing temporary peace through enforced silence.")
    await asyncio.sleep(30)
    await member1.edit(mute=False)
    await member2.edit(mute=False)
    await ctx.send(f"Unmuted {member1.display_name} and {member2.display_name}.")

# Run the bot
bot.run(TOKEN)
```