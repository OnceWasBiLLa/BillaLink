Recipes and Examples
=============================
Below are common short examples and recipes for use with BillaLink 2.
This is not an exhaustive list, for more detailed examples, see: `GitHub Examples <https://github.com/PythonistaGuild/BillaLink/tree/main/examples>`_


Listening to Events
-------------------
BillaLink 2 makes use of the built in event dispatcher of Discord.py.
This means you can listen to BillaLink events the same way you listen to discord.py events.

*All BillaLink events are prefixed with ``on_BillaLink_``*


**Outside of a Cog:**

.. code:: python3

    @bot.event
    async def on_BillaLink_node_ready(node: Node) -> None:
        print(f"Node {node.id} is ready!")


**Inside a Cog:**

.. code:: python3

    @commands.Cog.listener()
    async def on_BillaLink_node_ready(self, node: Node) -> None:
        print(f"Node {node.id} is ready!")



Creating and using Nodes
------------------------
BillaLink 2 has a more intuitive way of creating and storing Nodes.
Nodes are now stored at class level in a `NodePool`. Once a node has been created, you can access that node anywhere that
BillaLink can be imported.


**Creating a Node:**

.. code:: python3

    # Creating a node is as simple as this...
    # The node will be automatically stored to the global NodePool...
    # You can create as many nodes as you like, most people only need 1...
    node: BillaLink.Node = BillaLink.Node(uri='http://localhost:2333', password='youshallnotpass')
    await BillaLink.NodePool.connect(client=bot, nodes=[node])


**Accessing the best Node from the NodePool:**

.. code:: python3

    # Accessing a Node is easy...
    node = BillaLink.NodePool.get_node()


**Accessing a node by identifier from the NodePool:**

.. code:: python3

    node = BillaLink.NodePool.get_node(id="MY_NODE_ID")


**Accessing a list of Players a Node contains:**

.. code:: python3

    # A mapping of Guild ID to Player.
    node = BillaLink.NodePool.get_node()
    print(node.players)


**Attaching Spotify support to a Node:**

.. code:: python3

    from BillaLink.ext import spotify


    sc = spotify.SpotifyClient(
        client_id='CLIENT_ID',
        client_secret='SECRET'
    )
    node: BillaLink.Node = BillaLink.Node(uri='http://localhost:2333', password='youshallnotpass')
    await BillaLink.NodePool.connect(client=self, nodes=[node], spotify=sc)


Searching Tracks
----------------
Below are some common recipes for searching tracks.


**A Simple YouTube search:**

.. code:: python3

    track = await BillaLink.YouTubeTrack.search("Ocean Drive", return_first=True)


**Returning more than one result:**

.. code:: python3

    tracks = await BillaLink.YouTubeTrack.search("Ocean Drive")


**As a Discord.py converter:**

.. code:: python3

    @commands.command()
    async def play(self, ctx: commands.Context, *, track: BillaLink.YouTubeTrack):
        # The track will be the first result from what you searched when invoking the command...
        ...


Creating Players and VoiceProtocol
----------------------------------
Below are some common examples of how to use the new VoiceProtocol with BillaLink.


**A Simple Player:**

.. code:: python3

    import discord
    import BillaLink

    from discord.ext import commands


    @commands.command()
    async def connect(self, ctx: commands.Context, *, channel: discord.VoiceChannel = None):
        try:
            channel = channel or ctx.author.channel.voice
        except AttributeError:
            return await ctx.send('No voice channel to connect to. Please either provide one or join one.')

        # vc is short for voice client...
        # Our "vc" will be our BillaLink.Player as typehinted below...
        # BillaLink.Player is also a VoiceProtocol...

        vc: BillaLink.Player = await channel.connect(cls=BillaLink.Player)
        return vc


**A custom Player setup:**

.. code:: python3

    import discord
    import BillaLink

    from discord.ext import commands


    class Player(BillaLink.Player):
        """A Player with a DJ attribute."""

        def __init__(self, dj: discord.Member):
            self.dj = dj


    @commands.command()
    async def connect(self, ctx: commands.Context, *, channel: discord.VoiceChannel = None):
        try:
            channel = channel or ctx.author.channel.voice
        except AttributeError:
            return await ctx.send('No voice channel to connect to. Please either provide one or join one.')

        # vc is short for voice client...
        # Our "vc" will be our Player as type hinted below...
        # Player is also a VoiceProtocol...

        player = Player(dj=ctx.author)
        vc: Player = await channel.connect(cls=player)

        return vc


**Accessing the Player(VoiceProtocol) (with ctx or guild):**

.. code:: python3

    @commands.command()
    async def play(self, ctx: commands.Context, *, track: BillaLink.YouTubeTrack):
        vc: BillaLink.Player = ctx.voice_client

        if not vc:
            # Call a connect command or similar that returns a vc...
            vc = ...

        # You can also access player from anywhere you have guild...
        vc = ctx.guild.voice_client


**Accessing a Player from your Node:**

.. code:: python3

    # Could return None, if the Player was not found...

    node = BillaLink.NodePool.get_node()
    player = node.get_player(ctx.guild.id)


Common Operations
-----------------
Below are some common operations used with BillaLink.
See the documentation for more info.

.. code:: python3

    # Play a track...
    await player.play(track)

    # Turn AutoPlay on...
    player.autoplay = True

    # Similarly turn AutoPlay off...
    player.autoplay = False

    # Pause the current song...
    await player.pause()

    # Resume the current song from pause state...
    await player.resume()

    # Stop the current song from playing...
    await player.stop()

    # Stop the current song from playing and disconnect and cleanup the player...
    await player.disconnect()

    # Move the player to another channel...
    await player.move_to(channel)

    # Set the player volume...
    await player.set_volume(30)

    # Seek the currently playing song (position is an integer of milliseconds)...
    await player.seek(position)

    # Check if the player is playing...
    player.is_playing()

    # Check if the player is paused...
    player.is_paused()

    # Get the best connected node...
    node = BillaLink.NodePool.get_connected_node()

    # Common node properties...
    node.uri
    node.id
    node.players
    node.status

    # Common player properties...
    player.queue  # The players inbuilt queue...
    player.guild  # The guild associated with the player...
    player.current  # The currently playing song...
    player.position  # The currently playing songs position in milliseconds...
    player.ping  # The ping of this current player...