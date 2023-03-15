# NabuMultiPutt

This is a Mini-putt multi-user networked golf game for the ancient Nabu
computer.

The goal of this project is to make a game that works on the circa 1984
[Nabu computer](https://en.wikipedia.org/wiki/NABU_Network), now that a
bunch of new old stock has gone on sale (and there's a MAME emulator for it
too) and interest has risen after
[Adrian Black's episode](https://www.youtube.com/watch?v=HLYjZoShjy0
"Adrian Black's Digital Basement episode about the Nabu") on YouTube.
The wave is currently (March 2023) lead by programmer
[D.J. Sures](https://www.youtube.com/@DJSures "DJSures Channel on YouTube")
who is doing a lot of work getting supporting infrastructure in place for
programming and playing with the Nabu.

## Tips for Programmers

Bitmap of terrain, each pixel (one for each 8x8 screen tile, don't need high
resolution) has a code to show what's there: flat, tilted, bouncing boundaries,
absorptive boundaries, score zone, etc.  Use at most 8 directions for things
like walls else it takes too much work.

Make a tall image of all screen animation frames (like the windmill turning)
and run that through tile generator to get common tile patterns for the whole
level.  Complicated by division of screen into thirds in graphics mode 2.
Hopefully we can adapt an existing graphics tile finding utility.

Sprites of balls, player colour, dimples to show rolling and spin (8 directions
and spin, can cut directions in half by using reverse playback of one
direction).  Shadow under ball for height.

Simple fake physics using 16 bit fixed point integers.  Allow for balls to
bounce off each other, and push balls out that get stuck inside walls or other
balls.  Spin for low friction, until you hit a ramp then it turns into sideways
motion.  Then animation rolls in direction of velocity.

Ball hit user interface done by a bit like billiards, pick a spot touching the
ball, how far offside affects the spin the ball gets, then drag a line (made of
a row of circle sprites) to a further spot to show the direction and strength
of the shot.  Let go.

Animation sequences are preloaded in the level data and include name changes to
screen subarea (run length encoded strips of names), terrain bitmap changes,
sound effect.

Tracker style music, look for "AY-3-8910 Tracker" there are several, Furnace,
ChipnSfx (1K player!), etc, want one which has priorities so sound effects can
override/steal music instruments.

Network transmission per frame, from the current player to everybody else, just
which animation sequences change, position of balls, just name changes in score
area (includes scrolling online chat, scores).  At 10KB/s, 20 frames per second
give us 500 bytes per update.  Receivers play the appropriate animation delta
(writes names to a subarea of the screen, updates terrain bitmap, starts
sound).

Central server to keep track of who current player is, redistributes video
updates from the active player's computer to all others.  Has whole level data
(pattern table, initial screen, animation data, etc) so newcomers can download
it and start watching.  Or maybe have level data in a shared file rather than
uploaded by player who started the round.  Also relays chat text (master player
converts it into scrolling name changes in the video update).  Watches for dead
players and kicks them out.  Nominates new current player if old one dies.

