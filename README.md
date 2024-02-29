# Discord Bot Workshop

Welcome to the Discord Bot Workshop! In this workshop, you will learn the basics of creating a Discord bot using Python and the Discord.py library. Follow the steps below to set up and explore various features of bot development.

## Part 0: Creating the Discord application

First off, create the Discord server where you will test your bot. Then, you need to create a Discord *application* which will become the bot. To do this, head over to the [Discord Developper Portal](https://discord.com/developers/applications) and click "New Application". 

### Storing the bot token
Once you've created an application and you find yourself its page, go to the "Bot" section and copy its token (see picture).
![Token1](https://i.imgur.com/axWWzeS.png)
![Token2](https://i.imgur.com/l4uYYCa.png)(no point trying to use this token, I already deleted the app)

Once you've copied the env string, put it in an ``.env`` file in the following fashion:

```env
# .env
DISCORD_TOKEN="YOUR ENV"
```

The reason we put the bot token in a ``.env`` file is related to the safety of your application. In a timeline where you would post your code online, for example in a GitHub repository like this one, you would not want to put your bot token in the code. Otherwise everyone could log into your bot and do stuff with it! Therefore, we make a special file aimed at storing sensitive information, and we will open this file in our code.

### Invite the bot to your Discord server

Once you're finished with creating the application and storing the token, you need to designate your application as a bot. For that head to the ``OAuth2`` section and do the same as the images below.
![invite_bot1](https://i.imgur.com/qsxLmbY.png)
Once you click on "Bot", a new panel will open below:
![invite_bot2](https://i.imgur.com/0jdEwsL.png)
For simplicity's sake we select the "administrator" permission, giving your Bot every permission on your Discord server (outside of the ability to delete it). Once this is done, open the link shown below and add the bot to the Discord server you previously selected.

Finally, we can start coding now!




## Part 1: Bot Setup

First create a python file and copy and paste the below code box into it. It contains:
- the necessary imports to make your bot work;
- the "Bot" class and "intents" and "bot" variables that are imperative for seting up a bot;
- the 'TOKEN' constant and the command used to load it from the previously set up .env file. 

```python
# Bot setup
import discord, os
from discord.ext import commands
from dotenv import load_dotenv

class Bot(commands.Bot):
    def __init__(self, intents: discord.Intents):
        super().__init__(command_prefix="!", intents=intents, case_sensitive=True)

    async def on_ready(self):
        print(f'Logged in as {self.user.name} ({self.user.id})')

intents = discord.Intents.all()
bot = Bot(intents=intents)

load_dotenv()
TOKEN = os.getenv('DISCORD_TOKEN')

bot.run(TOKEN)
```
If you want, you can modify ``command_prefix="!"`` to put a prefix of your like. 

With this code in, try launching your bot! Type ``python3 [your python file]`` in the command line, and see if your bot account shows up online. If it does, great! You can continue to the next steps.

## Part 2: Prefix and Command Handling

If your bot is still running due to the previous part, simply kill the process by pressing ``Ctrl+C`` in the command line. 

Now, let's add some basic commands to our bot. Commands are functions that your bot can perform in response to messages.


```python
# Prefix and command handling
@bot.command(name="ping")
async def ping(ctx):
    await ctx.send("Pong!")

@bot.command(name="say")
async def say(ctx, *, message):
    await ctx.send(message)

@bot.command(name="shutdhown")
async def shutdown(ctx):
    await ctx.send("Shutting down the bot...")
    await bot.close()
```

Now, launch your bot again and try out these three commands. If they work, on to the next step!


## Part 3: Slash Commands

Slash commands are a new way of implementing commands with Discord.py. They provide a cleaner and more structured approach to command interactions. In short, they're easier for users to use, and probably for you too!

First, to make slash commands work you need to make sure Discord's API recognises them. For that, go back to your Bot class and add this line in the "on_ready" function:

```python
await bot.tree.sync()
```
To put it simply, "tree" is your ensemble of slash commands present in your bot. The sync() function will ensure Discord recognises those commands, and thus allow users to use them.


Now, here's a simple slash command below. Fill it with an instruction you want to carry and try using it on Discord.
```python
# Slash commands
@bot.tree.command(name="my_slash_command", description="my first slash command")
async def my_slash_command(ctx):
    #fill the code
```

Now try to use ``/my_slash_command`` on your Discord server. If it worked, try modifying the previous three commands used in Part 2 into slash commands! 


## Part 4: Interactive Components

Interactive components, such as buttons, allow users to interact with your bot in a more engaging way. You've previously used a command that sends a message back when you invoke it, now what if we tried creating a button that sends a message when you click on it?

```python
# Interactive components
class MyView(View):
    @bot.button(label="Click me", custom_id="button_click")
    async def on_button_click(self, button, interaction):
        await interaction.response.send_message("Button clicked!")

@bot.tree.command(name="interactive")
async def interactive(ctx):
    view = MyView()
    await ctx.send("Click on the button below!", view=view)
```


## Part 5: Role Management

Now let's do a bit more than simply sending messages. Create a role on your Discord server, doesn't matter the name, and then try adding it to yourself using the bot!

```python
# Role management
@bot.tree.command(name="add_role")
async def add_role(ctx, role: discord.Role, member: discord.Member):
    if role not in ctx.guild.roles:
        await ctx.send("The specified role does not exist.")
        return
    # fill this line with the function that adds the role to the member
    await ctx.send(f"{member.mention} has been given the role {role.name}.")

@bot.tree.command(name="remove_role")
async def remove_role(ctx, role: discord.Role, member: discord.Member):
    # fill this line with the function that removes the role to the member
    await ctx.send(f"{member.mention} has been removed from the role {role.name}.")
```

If you've succeeded in this, let's spice it up! Create a command that generates a button. When you click on this button, it will give you the role you created.

## Part 6: Event Handling

Event handling allows your bot to respond to various events that occur on Discord, such as new messages or members joining the server.

```python
# Event handling
@bot.event
async def on_message(message):
    if message.author == bot.user:
        return
    if "hello" in message.content.lower():
        await message.channel.send(f"Hi {message.author.mention}!")

@bot.event
async def on_member_join(member):
    welcome_channel = member.guild.get_channel("REPLACE BY THE ID OF YOUR WELCOME CHANNEL")  
    # Replace with your welcome channel ID
    # Implement the welcome message sender here 
```

## Part 7: Moderation Commands

Moderation commands are crucial for generalist-type bots. They aim to make easier the managnment of Discord servers and the maintenance of a healthy community. So let's implemnent them!

```python
# Moderation commands
@bot.tree.command(name="kick", permissions="kick_members")
async def kick(ctx, member: discord.Member, reason: str = "No reason provided"):
    # Implement kick functionality here
    await ctx.send(f"{member.mention} has been kicked. Reason: {reason}")

@bot.tree.command(name="ban", permissions="ban_members")
async def ban(ctx, member: discord.Member, reason: str = "No reason provided"):
    # Implement ban functionality here
    await ctx.send(f"{member.mention} has been banned. Reason: {reason}")

@bot.tree.command(name="mute", permissions="mute_members")
async def mute(ctx, member: discord.Member):
    # Implement mute functionality here
    await ctx.send(f"{member.mention} has been muted.")
```

Once you have implemented the commands and launched the bot once again, invite me to the Discord server (my Discord username is @altys). When I join, try to kick me with the bot!