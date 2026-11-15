# Final Fight Recomp

> Part of **[CPS-2 Recomp](../../README.md)**: one download and one launcher for every Capcom arcade game this
> project has ported. This page is about Final Fight; the [main page](../../README.md) says how to download and play.

An unofficial native PC port of the arcade game *Final Fight* (Capcom, 1989, CPS-1 board) for macOS, Linux (Steam
Deck included) and Windows, made by static recompilation: a beat-'em-up for two players at once.

> **Initial classic CPS-1 port, with sound.** The game's picture, controls and arcade music, voices and effects run
> on the toolkit's shared classic CPS-1 board, as for [Ghouls'n Ghosts](../ghouls/README.md). There are no board
> options, cheats or features of its own yet.

**No game files are included.** You need your own copy of the game: the arcade ROM set `ffight` (World, set 1), from
a board you own. This project cannot help you get one.

This project is not affiliated with, endorsed by, or sponsored by Capcom. *Final Fight* is a trademark of Capcom Co.,
Ltd.

## What you need

- The ROM set `ffight` (*Final Fight*, World, set 1): `ffight.zip` or a folder with its files. Other regions and
  revisions are refused.
- One of: macOS 13 or later; Linux x86_64 or SteamOS; Windows 10 or 11, 64-bit.

## Building from source

With your own ROM set in `rom/ffight/` (ignored by Git), from the repository's top:

```sh
make GAME=ffight            # twice the first time
make GAME=ffight explore    # once: finds the game's code by made-up play and translates it
make GAME=ffight run
```

## Progress

**Where it stands.** On the classic CPS-1 board, with what MAME's tables give for the set (`game.cfg`):

- **The 68000 at 10 MHz** (`cpu_clock = 10`, MAME's `cps1_10MHz`).
- **Its program**: two pairs of byte-wide files, then `ff-32m.8h`, whose words are low byte first (MAME's
  `ROM_LOAD16_WORD_SWAP`, so no `/be`; with `/be` every word the game reads from 80000 on is swapped and its pictures
  come out wrong).
- **The CPS-B-04**: its registers where that chip has them, and its ID register at 20, which the game's start-up test
  reads (it must read 4, or the game stops).
- **The graphics mapper**, the PAL S224B: one bank; objects 0-43ff, scroll 1 4400-4bff, scroll 3 4c00-5fff, scroll 2
  6000-7fff (`gfx_map`).
- **Sound**: the Z80 with the YM2151 and the OKI MSM6295, its samples in `ff_18.11c` and `ff_19.12c`.
- **The coin lock-outs**, new in the toolkit for every classic CPS-1 board: the game writes bits 10 and 11 of 800030
  to 0 at nine credits, and a coin put in then is not seen (MAME's `cps1_coinctrl_w`). Without them the game took a
  coin at nine credits and its coin routine ran where the board's does not.
- **The video registers' writes** (`video_writes`), from a census of attract mode and the listing, so the picture is
  drawn from what the game hands over.

The DIP switches are MAME's defaults (`dip = FF F4 9F`): 1 coin 1 credit, difficulty 4 and 2 (normal), a bonus life
at 100k, 2 lives, the demonstrations' sound, continues allowed.

**Checked against MAME 0.264**, by the 68000's work RAM (FF0000-FFFFFF, leaving out the stack below FF1000 and the
task state byte FF10B0, which depend only on where the 68000 is inside a frame), with the same input replayed on both:

- `movies/smoke_play.txt`, 12000 frames of made-up play from reset (coins, starts, two players): the same as MAME on
  11980 frames; the other 20 are the start-up's RAM test, a few cycles apart.
- 108000 frames of made-up play (stage 1 to stage 3: Damnd, the subway, Sodom, the West Side), every tenth frame:
  the same as MAME on all but two frames, both in the RAM test.
- The picture at the attract mode's frame 1799 is MAME's.

**Next.** Board options, cheats and features of its own.

## Reproducible checks

```sh
make GAME=ffight check FRAMES=12000 MOVIE=games/ffight/movies/smoke_play.txt
make GAME=ffight check-rollback FRAMES=12000
make GAME=ffight check-online FRAMES=6000
```

All three pass on Linux x86_64. The online check uses two local peers and simulated packet delay and loss; it does
not test Internet play. The classic sound program runs on the Z80, so `check-sound`'s QSound comparison does not apply.
