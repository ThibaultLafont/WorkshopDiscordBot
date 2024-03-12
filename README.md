# Discord Bot Workshop
Welcome to the Discord Bot Workshop! In this workshop, you will learn the basics of creating a Discord bot using Python and the Discord.py library. Follow the steps below to set up and explore various features of bot development.

## Prerequisites

### Python
This workshop will make use of Python to code a Discord bot. If you're unsure whether or not your ps has the language installed, type ``python3`` in the command line. If it displays an output similar to the picture below:
![example](https://i.imgur.com/cKUj9FX.png)

then you're good. If not, type ``sudo apt instal python3``

### Python libraries
We're going to use two different libraries, Dotenv and Discordpy during this workshop. As a result you need to make sure you have them installed:
- Dotenv: ``python3 -m pip install python-dotenv``
- Discordpy: ``python3 -m pip install -U discord.py``

We will use Dotenv to read .env files in this workshop to store sensitive information. While Discordpy is the library we will use to make API calls to Discord during the crafting of the bot.

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
Here we opt to create a class to define what is our bot. Right now inside it we initialise our bot with the \_\_init__ function, and we also use the "on_ready" asynchronous function to print in the console a certain text when the bot is launched. Later down the line you will see we can add more functions in there if we wish to.

(Already if you want, you can modify ``command_prefix="!"`` to put a prefix of your like. )

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

@bot.command(name="shutdown")
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
class MyView(discord.ui.View):
    @discord.ui.button(label="Click me", custom_id="button_click")
    async def on_button_click(self, interaction, button):
        await interaction.response.send_message("Button clicked!", ephemeral=True)

@bot.tree.command(name="interactive")
async def interactive(ctx):
    view = MyView()
    await ctx.response.send_message("Click on the button below!", view=view)
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
    await ctx.response.send_message(f"{member.mention} has been given the role {role.name}.")

@bot.tree.command(name="remove_role")
async def remove_role(ctx, role: discord.Role, member: discord.Member):
    # fill this line with the function that removes the role to the member
    await ctx.response.send_message(f"{member.mention} has been removed from the role {role.name}.")
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
Now try inviting me or another member of the workshop to see if this works!

## Part 7: Moderation Commands

Moderation commands are crucial for generalist-type bots. They aim to make easier the managnment of Discord servers and the maintenance of a healthy community. So let's implemnent them!

```python
# Moderation commands
@bot.tree.command(name="kick", description="Kick a member")
async def kick(ctx, member: discord.Member, reason: str = "No reason provided"):
    # Implement kick functionality here
    await ctx.response.send_message(f"{member.mention} has been kicked. Reason: {reason}")

@bot.tree.command(name="ban", description="Ban a member")
async def ban(ctx, member: discord.Member, reason: str = "No reason provided"):
    # Implement ban functionality here
    await ctx.response.send_message(f"{member.mention} has been banned. Reason: {reason}")

@bot.tree.command(name="mute", description="Mute a member")
async def mute(ctx, member: discord.Member, time: int, reason: str = "No reason provided"):
    # Implement mute functionality here
    await ctx.response.send_message(f"{member.mention} has been muted.")
```

Once you have implemented the commands and launched the bot once again, invite me to the Discord server (my Discord username is @altys). When I join, try to kick me with the bot!

## Part 8: Limiting access to commands

Sometimes you want your commands to only be accessible to certain people. As an example we will take the limitation "is owner of the Bot" to lock a certain command. The syntax for these type of limitations is simple, see below:

First we need to import those limitations:
```python
from discord.ext.commands import has_permissions, has_role, is_owner
```
Second, remember the "Bot" class we made at the beginning? We need to add additional information to it. Specifically to this line:
```python
super().__init__(command_prefix="!", intents=intents, case_sensitive=True)
```
We need to add another parameter called ``owner_id``. It should look like this:
```python
super().__init__(command_prefix="!", intents=intents, case_sensitive=True, owner_id=REPLACE_WITH_YOUR_ID)
```

Now your bot only recognises the discord account whose ID is the one you gave in the code as its owner. This is important in cases where you wish to limit bot commands to only you. For example, let's look again at our ``shutdown`` command and make it so only you can use it:
```python
@bot.tree.command(name="shutdhown", description="Shuts down the bot")
async def shutdown(ctx):
    if await bot.is_owner(ctx.user):
        await ctx.response.send_message("Shutting down the bot...")
        await bot.close()
    else:
        await ctx.response.send_message("You are not authorized to use this command.")
```
Now try inviting another member of the workshop to see if they can shut down the bot! You'll see, they won't.

Bonus: see how I also told you to import ``has_permissions`` and ``has_role``? Try to fiddle with it!

## Part 9: Embed messages
An embed is a special message format that allows you to create rich and visually appealing messages. The code snippet below demonstrates how to create and send an embed.

```python
@bot.tree.command(name="embed")
async def embed(ctx):
    # Creating an Embed object
    embed = discord.Embed(
        title="This is an embed",               # Set the title of the embed
        description="This is a description",   # Set the description of the embed
        color=discord.Color.blurple()           # Set the color of the embed
    )
    
    # Setting the author information
    embed.set_author(name=ctx.user.display_name, icon_url=ctx.user.avatar)
    
    # Setting the thumbnail image
    embed.set_thumbnail(url=ctx.user.avatar)
    
    # Adding fields to the embed
    embed.add_field(name="Field 1", value="Value 1", inline=False)
    embed.add_field(name="Field 2", value="Value 2", inline=False)
    
    # Setting the footer text
    embed.set_footer(text="This is a footer")
    
    # Sending the embed as a response
    await ctx.response.send_message(embed=embed)
```

Try to use the command and see what it does. If it works, edit the code to try and customise your embed! 


## Part 10: Beyond a simple bot
Sending messages, adding and removing roles, and banning users are fairly simple actions to code. Thus if you want to spice it up, here's a list of ideas you can try to implement:
### Generalist bot discord
Dyno, Tatsu, Carlbot. Those names ring a bell? If not, know they're all generalist bots offering a wide array of features to enhance your server. You can go look at their documentations to take inspiration from them and try to recode some of them:
- [Dyno Documentation](https://dyno.gg/commands)
- [Tatsu Documentation](https://support.tatsu.gg/hc/en-us/articles/900007295883-List-of-Tatsu-Commands)
- [Carlbot Documentation](https://docs.carl.gg/#/)

### Specialist bot discord
There's something specific you want your bot to do? Show newly uploaded YouTube videos, play mini-games with friends, display information retrieved from external API calls to a site you're using, etc. A lot can be done! 

To give you a visible example of a specialist, and in this case niche Discord bot, you can look at my [LemanNS](https://github.com/ThibaultLafont/LemanNS) repository. It is a work-in-progress Discord bot making use of the [NationStates API](https://www.nationstates.net/pages/api.html) as well as [SQLite](https://www.sqlite.org/docs.html) databases, made for the site of the same name. You're looking for the ``instance.py`` file for all the Discord Bot-related code.
### Other documenations
Just for convenience, here's other documentation links you could use:
- [Discord API Documentation](https://discord.com/developers/docs/intro)
- [DiscordPY Documentation](https://discordpy.readthedocs.io/en/stable/index.html) (the "Manuals" section is where everything you need to know is linked)
