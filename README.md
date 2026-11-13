# SR1Recomp

Smuggler's Run (PS2, 2000) as a native Windows program, built with
[ps2-native-recomp](https://github.com/exoteric33/ps2-native-recomp) from your own disc, with a launcher and a set of
optional improvements: widescreen, 60 FPS, higher resolutions, upscaling, and more. A sister project of
[MC1Recomp](https://github.com/jcoker85/MC1Recomp) (Midnight Club: Street Racing, the same studio's engine a few months
later), whose launcher and toolkit changes it reuses.

| Game | Disc | Serial |
|---|---|---|
| Smuggler's Run | USA | SLUS-20065 (program CRC 0E7F91DA) |

This is an unofficial fan project. It is not affiliated with or endorsed by Rockstar Games, Take-Two Interactive or Sony
Interactive Entertainment.

## Play

You need your own disc image of Smuggler's Run (USA) as an `.iso`. The game is on a CD, so a `.chd` is unpacked with
MAME's chdman: `chdman extractcd -i "Smuggler's Run.chd" -o "Smuggler's Run.cue" -ob "Smuggler's Run.iso"`. If the `.cue`
it writes says `MODE1/2048`, the `.iso` is ready. If it says `MODE1/2352` or `MODE2/2352` (a raw dump), convert it with the
toolkit: `python ps2-native-recomp/scripts/disc.py iso "Smuggler's Run.cue" "Smuggler's Run.iso"`.

Windows 10 or 11 (64-bit), a CPU with AVX2 and a Direct3D 12 graphics card.

1. Run `SR1Recomp.exe` (build it first, below).
2. Press **Play**. The first time, it asks for your disc image; the launcher remembers it
   (**Settings > Disc Image** changes it).
3. **Settings** has three tabs: **Graphics**, with the options below, **P1 Controls** and **P2 Controls**. Every option
   starts off, so a fresh install plays the game as it is on a PS2.

### Options

- **Fullscreen**, **Resolution** (1x to 4x the PS2's), **Vsync**, **Frame Pacing Fix** (frames on an even 59.94 Hz clock)
- **Picture Mode**: how the picture is enlarged to the window: Crisp, Soft or Sharp, each with optional edge smoothing,
  or **FSR** (AMD FidelityFX Super Resolution 1)
- **Texture Filtering**: the game's own (Bilinear PS2), Nearest, or Bilinear (Forced)
- **60 FPS** (after the PCSX2 community patch by Souzooka): driving runs at one vsync a frame instead of two (the menus
  already do)
- **Widescreen** 16:9 (after the PCSX2 community patch by Devina, sergx12 and Aced14), **Fixed Widescreen HUD** (the
  driving HUD keeps its 4:3 shapes at the screen's edges: speedometer, radar, timer, money, score, the countdown and
  messages in the middle), **Menu/Movie Aspect Ratio** (menus, loading screens and videos in 4:3 while driving is 16:9)
- **Color Settings**: the PS2's 16-bit picture, or the game's full 32-bit picture; **Dithering** (the PS2's dither on the
  16-bit picture)
- **Smooth Distant Textures** (less shimmer on far terrain; not something a PS2 does)
- **Fix Distant Z-Fighting** (not something a PS2 does): far away, the game's 16-bit depth buffer can't tell apart two
  surfaces drawn on top of each other, such as a mountain's rock and the snow layer on it, so they take turns as the view
  moves (the snow on the snow level's mountains changes shape while you drive). With this on, the depth test also keeps the
  fraction of each depth, so the nearer surface wins every frame. Where the game's snow sits just behind the rock, that
  means steady rock where the PS2 shows flickering patches of snow
- **Skip Intro**: straight to the title screen, with the game's own developer switch (`-novideo`, read by the movie
  player's constructor at `0x1D7374`)

**P1 Controls** and **P2 Controls** work as in MC1Recomp: choose Controller or Keyboard (with the mouse) for player 1 and
rebind every PS2 button and stick direction; the bindings are kept in `ps2-native-recomp/games/sr1/keys.ini`. Saves go
to the memory card file in `ps2-native-recomp/games/sr1` (Smuggler's Mission asks for a card).

## Build

On Windows, with Git for Windows (Git Bash), CMake, Ninja, Python 3 and llvm-mingw (UCRT). The toolkit's
`scripts\bootstrap.ps1` installs them with winget; no Visual Studio is needed.

```bash
git clone --recursive https://github.com/jcoker85/SR1Recomp.git
cd SR1Recomp
bash tools/build.sh "/c/Games/Smuggler's Run (USA).iso"
```

`tools/build.sh`:

1. fetches ps2-native-recomp v0.0.4 (the submodule) and applies `patches/ps2-native-recomp.patch`;
2. sets up the toolkit's build tools;
3. translates the game's code from your disc into C++ and builds the game program (`ps2run.exe`), with the Widescreen and
   60 FPS code built in as switchable instructions (`sr1-options/toggles.txt`) and the Fixed Widescreen HUD's hooks
   (`sr1-options/hud43.pnach`), all off until the launcher turns them on;
4. builds the launcher, `SR1Recomp.exe`. It has no icon built in: on its first start it downloads its icon (credited
   below) into `covers/icon.ico`, beside the box art, and its window and the game window show it from there (a square PNG
   put at `covers/icon.png` is used instead). It also writes the icon into its own `SR1Recomp.exe` (it renames itself to
   `SR1Recomp.exe.old` and puts a copy with the icon in its place), so Explorer and shortcuts show it from the next start.

The first build takes a while. Code the first build did not see is translated while you play and kept for next time.

## What is and is not in this repository

The repository holds the launcher, patches, build files and documentation. It does **not** contain disc images, the game's
program, game assets, cover art or recompiled game code. The recompiled C++ and the game program are generated on your
machine from your own disc and stay in `ps2-native-recomp/generated` and `ps2-native-recomp/games`, which git ignores. When
the game runs, it reads its data from your disc image.

- `hub/`: the launcher (`hub.c`, MC1Recomp's launcher with this game's options) and its build files.
- `sr1-options/`:
  - `toggles.txt`: the Widescreen and 60 FPS options as eerecomp `--toggles` lines. Both PCSX2 patches change the game's
    code, so they are translated into the game program in both versions and switched at run time by a word below the
    program (`0xFFF00` widescreen, `0xFFF08` 60 FPS):
    - Widescreen: the community patch jumps into a code cave to multiply the view's width by 4/3 before the game divides it
      by the height (`0x23CA7C`). Toggles switch only plain instructions, so here the same is done in three of the
      function's empty load-delay slots: `lui at,0x3F40` / `mtc1 at,f31` / `div.s f2,f2,f31` (the aspect divided by
      0.75, exactly 4/3 wider);
    - 60 FPS: the community patch has the game set its frame interval (`0x2C4F14`) to 1 vblank instead of 2 when driving
      starts. Here the one instruction that reads it (`0x234C5C`) reads 1 instead: the same effect, and the word keeps
      telling driving (2) from the menus (1).
  - `widescreen.txt` and `60fps.txt`: the `ps2run --patches` lines that set those words; the launcher copies the lines of
    the options that are on into `boot.txt` at each start.
  - `menus43.txt` / `widescreen43.txt`: Menu/Movie Aspect Ratio. The picture is 16:9 only while the frame interval at
    `0x2C4F14` is 2 (from the first frame of driving until the player quits, the pause menu included), and the runtime
    mirrors that into the widescreen word, so the game's view follows the picture.
  - `hud43.py` / `hud43.pnach` / `hud43.txt`: Fixed Widescreen HUD, the same method as MC1Recomp's with this game's
    addresses. This older engine keeps its HUD parts in an array on a HUD object and draws them all from one loop
    (`0x15F728`), so the one call there (`0x15F76C`) is hooked: while the HUD option word (`0xFFF04`) and the widescreen
    word are on, each part's drawing is marked in the game's VIF1 packets (pointer `0x70000428`, end `0x7000042C`, full
    buffer `0x23A378`), and the runtime squeezes what a marked part draws toward the edge it is nearest. `hud43.txt` tells
    the launcher the hooks are built in.
- `patches/ps2-native-recomp.patch`: this project's changes to the toolkit: MC1Recomp's (MC2Recomp's fixes for this engine,
  dithering, texture filtering, smooth distant textures, FSR, keyboard and controller bindings, two players, conditional
  patch lines, the sound chip's ENDX, faster drawing at higher resolutions, fair interrupt turns and more; see MC1Recomp's
  README for the list), plus one for Smuggler's Run:
  - graphics speed: while driving the game reads some 550 small pieces of a 512-pixel-wide texture a frame from memory
    that its 640-pixel-wide frame also takes up. The runtime's memory test ran both from their buffers' start, so every one
    of those reads ended the drawing run: ~2,760 runs a frame, each waiting for the one before, 20 to 45 ms a frame even at
    1x on an RTX 4090. Now such a read is tested page by page against what the run has drawn, which leaves about 110 runs
    a frame; driving at 4x holds 60 FPS (16.7 ms a frame), and the pictures are bit-for-bit the same (`PS2_PAGE_TEST=0`
    turns it off, for comparison);
  - Fix Distant Z-Fighting (`PS2_DEPTH_FRAC=1`): 16-bit depth keeps 8 bits of its fraction in a second region after the
    emulated memory (the buffer is twice as large when it is on), tagged with the low 8 bits of the depth it belongs to, so a
    depth written another way (an upload, a copy) counts as having none, as on the PS2. Found on the snow level: a mountain's
    rock and its snow layer about 0.6 of a depth step apart at depth ~25 of 65,535, where rounding decided which showed.
- `ps2-native-recomp/`: the toolkit, as a git submodule.
- `tools/`: the build script, the release packaging script and `mips.py` (a small R5900 disassembler and
  address-reference finder used to trace the game's code).

## Credits

- [ps2-native-recomp](https://github.com/exoteric33/ps2-native-recomp) by Samir Nuri and exoteric33 (MIT): the
  recompiler and runtime this project is built on.
- The PCSX2 community patches (from [PCSX2/pcsx2_patches](https://github.com/PCSX2/pcsx2_patches)): Widescreen 16:9 by
  Devina, sergx12 and Aced14, 60 FPS by Souzooka.
- AMD FidelityFX Super Resolution 1.0 (MIT, Copyright (c) 2021 Advanced Micro Devices, Inc.): the FSR picture mode is
  written after AMD's published shaders.
- Box art in the launcher: [libretro-thumbnails](https://github.com/libretro-thumbnails/libretro-thumbnails).
- The launcher and game window icon: [Smuggler's Run icon by Lock n' Stock](https://www.steamgriddb.com/icon/29061) on
  SteamGridDB, downloaded by the launcher on its first start.

## License

MIT; see `LICENSE`. ps2-native-recomp is MIT as well.

Smuggler's Run is a trademark of Take-Two Interactive Software, Inc., and the game is copyright its owners. This
repository contains none of its code, data or artwork, nor the icon and box art the launcher downloads.

## AI disclosure

This project was built with Claude (Anthropic) working as a coding assistant under the direction of the author.

- **Claude did:** ported MC1Recomp's launcher, build and packaging scripts to this game, rewrote the PCSX2 patches as
  run-time switchable instructions, traced the game's code for the addresses involved, and drafted this README.
- **Claude did not:** write ps2-native-recomp, which is another project used here as a submodule, the PCSX2 community
  patches, or the game itself, whose code the recompiler translates from your own disc.
