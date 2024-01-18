.. currentmodule:: BillaLink.ext.spotify


Intro
-----
The Spotify extension is a QoL extension that helps in searching for and queueing tracks from Spotify URL's or ID's. To get started create a :class:`~SpotifyClient` and pass in your credentials. You then pass this to your :class:`BillaLink.Node`'s.

An example:

.. code-block:: python3

    import discord
    import BillaLink
    from discord.ext import commands
    from BillaLink.ext import spotify


    class Bot(commands.Bot):

        def __init__(self) -> None:
            intents = discord.Intents.default()
            intents.message_content = True

            super().__init__(intents=intents, command_prefix='?')

        async def on_ready(self) -> None:
            print(f'Logged in {self.user} | {self.user.id}')

        async def setup_hook(self) -> None:
            sc = spotify.SpotifyClient(
                client_id='CLIENT_ID',
                client_secret='CLIENT_SECRET'
            )

            node: BillaLink.Node = BillaLink.Node(uri='http://127.0.0.1:2333', password='youshallnotpass')
            await BillaLink.NodePool.connect(client=self, nodes=[node], spotify=sc)


    bot = Bot()


    @bot.command()
    @commands.is_owner()
    async def play(ctx: commands.Context, *, search: str) -> None:

        try:
            vc: BillaLink.Player = await ctx.author.voice.channel.connect(cls=BillaLink.Player)
        except discord.ClientException:
            vc: BillaLink.Player = ctx.voice_client

        vc.autoplay = True

        track: spotify.SpotifyTrack = await spotify.SpotifyTrack.search(search)

        if not vc.is_playing():
            await vc.play(track, populate=True)
        else:
            await vc.queue.put_wait(track)


Helpers
-------
.. autofunction:: decode_url


Client
------
.. attributetable:: SpotifyClient

.. autoclass:: SpotifyClient
    :members:


Enums
-----
.. attributetable:: SpotifySearchType

.. autoclass:: SpotifySearchType
    :members:


Spotify Tracks
--------------
.. attributetable:: SpotifyTrack

.. autoclass:: SpotifyTrack
    :members:


Exceptions
----------
.. py:exception:: SpotifyRequestError

    Base error for Spotify requests.

    status: :class:`int`
        The status code returned from the request.
    reason: Optional[:class:`str`]
        The reason the request failed. Could be ``None``.
