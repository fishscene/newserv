# newserv <img align="right" src="s-newserv.png" />

newserv is a game server, proxy, and reverse-engineering tool for Phantasy Star Online (PSO).

This project includes code that was reverse-engineered by the community in ages long past, and has been included in many projects since then. It also includes some game data from Phantasy Star Online itself, which was originally created by Sega.

Feel free to submit GitHub issues if you find bugs or have feature requests. I'd like to make the server as stable and complete as possible, but I can't promise that I'll respond to issues in a timely manner, because this is a personal project undertaken primarily for the fun of reverse-engineering. If you want to contribute to newserv yourself, pull requests are welcome as well.

See TODO.md for a list of known issues and future work I've curated, or go to the GitHub issue tracker for issues and requests submitted by the community.

**Table of contents**
* [Compatibility](#compatibility)
* Setup
    * [Configuration](#configuration)
    * [Installing quests](#installing-quests)
    * [Episode 3 features](#episode-3-features)
    * [Client patch directories for PC and BB](#client-patch-directories)
    * [Memory patches and DOL files for GC](#memory-patches-and-dol-files)
    * [Using newserv as a proxy](#using-newserv-as-a-proxy)
    * [Chat commands](#chat-commands)
* How to connect
    * Connecting local clients
        * [PSO DC](#pso-dc)
        * [PSO DC on Flycast](#pso-dc-on-flycast)
        * [PSO PC](#pso-pc)
        * [PSO GC on a real GameCube](#pso-gc-on-a-real-gamecube)
        * [PSO GC on Dolphin](#pso-gc-on-dolphin)
        * [PSO BB](#pso-bb)
    * [Connecting external clients](#connecting-external-clients)
* [Non-server features](#non-server-features)

## Compatibility

newserv supports several versions of PSO. Specifically:
| Version        | Login        | Lobbies      | Games        | Proxy        |
|----------------|--------------|--------------|--------------|--------------|
| DC Trial       | Yes          | Yes          | Yes          | No           |
| DC 11/2000     | Yes          | Yes          | Yes          | No           |
| DC 12/2000     | Yes          | Yes          | Yes          | Yes          |
| DC 01/2001     | Yes          | Yes          | Yes          | Yes          |
| DC V1          | Yes          | Yes          | Yes          | Yes          |
| DC 08/2001     | Untested (1) | Untested (1) | Untested (1) | Untested (1) |
| DC V2          | Yes          | Yes          | Yes          | Yes          |
| PC             | Yes          | Yes          | Yes          | Yes          |
| GC Ep1&2 Trial | Untested (1) | Untested (1) | Untested (1) | Untested (1) |
| GC Ep1&2       | Yes          | Yes          | Yes          | Yes          |
| GC Ep1&2 Plus  | Yes          | Yes          | Yes          | Yes          |
| GC Ep3 Trial   | Yes          | Yes          | Partial (3)  | Yes          |
| GC Ep3         | Yes          | Yes          | Yes          | Yes          |
| Xbox Ep1&2     | Yes          | Yes          | Yes          | Yes          |
| BB (vanilla)   | Yes          | Yes          | Yes (2)      | Yes          |
| BB (Tethealla) | Yes          | Yes          | Yes (2)      | Yes          |

*Notes:*
1. *newserv's implementations of these versions are based on disassembly of the client executables and have never been tested.*
2. *BB games are mostly playable, but there are still some unimplemented features (for example, some quests that use rare commands may not work). Please submit a GitHub issue if you find something that doesn't work.*
3. *Creating a game works and battle setup behaves mostly normally, but starting a battle doesn't work.*

## Setup

### Configuration

Currently newserv works on macOS, Windows, and Ubuntu Linux. It will likely work on other Linux flavors too.

There is a fairly recent macOS ARM64 release on the newserv GitHub repository. You may need to install libevent manually even if you use this release (run `brew install libevent`).

There is a fairly recent Windows release on the newserv GitHub repository also. It's built with Cygwin, and all the necessary DLL files should be included. That said, I've only tested it on my own machine and there is no CI for Windows builds like there is for macOS and Linux, so if it doesn't work for you, please open a GitHub issue to let me know.

If you're not using a release from the GitHub repository, do this to build newserv:
1. If you're on Windows, install Cygwin. While doing so, install the `cmake`, `gcc-core`, `gcc-g++`, `git`, `libevent2.1_7`, `make`, `libiconv-devel`, and `zlib` packages. Do the rest of these steps inside a Cygwin shell (not a Windows cmd shell or PowerShell).
2. Make sure you have CMake, libevent, and libiconv installed. (On macOS, `brew install cmake libevent libiconv`; on most Linuxes, `sudo apt-get install cmake libevent-dev`; on Windows, you already did this in step 1.)
3. Build and install phosg (https://github.com/fuzziqersoftware/phosg).
4. Optionally, install resource_dasm (https://github.com/fuzziqersoftware/resource_dasm). This will enable newserv to send memory patches and load DOL files on PSO GC clients. PSO GC clients can play PSO normally on newserv without this.
5. Run `cmake . && make` in the newserv directory.

After building newserv or downloading a release, do this to set it up and use it:
1. In the system/ directory, make a copy of config.example.json named config.json, and edit it appropriately.
2. If you plan to play PSO Blue Burst on newserv, set up the patch directory. See the "Client patch directories" section below.
3. Run `./newserv` in the newserv directory. This will start the game server and run the interactive shell. You may need `sudo` if newserv's built-in DNS server is enabled.
4. If you set AllowUnregisteredUsers to false in config.json, use the interactive shell to add your license. Run `help` in the shell to see how to do this.
5. Set your client's network settings appropriately and start an online game. See the "Connecting local clients" or "Connecting remote clients" section to see how to get your game client to connect.

To use newserv in other ways (e.g. for translating data), see the end of this document.

### Installing quests

newserv automatically finds quests in the subdirectories of the system/quests/ directory. To install your own quests, or to use quests you've saved using the proxy's Save Files option, just put them in one of the subdirectories there and name them appropriately. The subdirectories and their behaviors (e.g. in which game modes they should appear and for which PSO versions) is defined in the QuestCategories field in config.json.

Within the category directories, quest files should be named like `q###-VERSION-LANGUAGE.EXT` (although the `q` is ignored, and can be any letter). The fields in each filename are:
- `###`: quest number (this doesn't really matter; it should just be unique across the PSO version)
- `VERSION`: dn = Dreamcast NTE, dp = Dreamcast 11/2000 prototype, d1 = Dreamcast v1, dc = Dreamcast v2, pc = PC, gcn = GameCube Trial Edition, gc = GameCube Episodes 1 & 2, gc3 = Episode 3 (see below), xb = Xbox, bb = Blue Burst
- `LANGUAGE`: j = Japanese, e = English, g = German, f = French, s = Spanish
- `EXT`: file extension (see table below)

For .dat files, the `LANGUAGE` token may be omitted. If it's present, then that .dat file will only be used for that language of the quest; if omitted, then that .dat file will be used for all languages of the quest.

Some quests (mostly battle and challenge mode quests) have additional JSON metadata files that describe how the server should handle them. This includes flags that can be used to hide the quest unless a preceding quest has been cleared, or to hide the quest unless purchased as a team reward. These metadata files are generally named similarly to their .bin and .dat counterparts, except the `VERSION` token may also be omitted if the metadata applies to all versions of the quest on all PSO versions. See system/quests/battle/b88001.json for documentation on the exact format of the JSON file.

Some quests may also include a .pvr file, which contains an image used in the quest. These files are named similarly to their .bin and .dat counterparts.

For example, the GameCube version of Lost HEAT SWORD is in two files named `q058-gc-e.bin` and `q058-gc.dat`. newserv knows these files are quests because they're in the system/quests/ directory, it knows they're for PSO GC because the filenames contain `-gc`, it knows this is the English version of the quest because the .bin filename ends with `-e` (even though the .dat filename does not), and it puts them in the Retrieval category because the files are within the retrieval/ directory within system/quests/.

The GameCube and Xbox quest formats are very similar, but newserv treats them as different. If you want to use the same quest file for GameCube and Xbox clients, you can make one a symbolic link to the other.

There are multiple PSO quest formats out there; newserv supports all of them. It can also decode any known format to standard .bin/.dat format. Specifically:

| Format           | Extension             | Supported  | Decode action    |
|------------------|-----------------------|------------|------------------|
| Compressed       | .bin and .dat         | Yes        | None (1)         |
| Compressed Ep3   | .bin or .mnm          | Yes (4)    | None (1)         |
| Uncompressed     | .bind and .datd       | Yes        | compress-prs (2) |
| Uncompressed Ep3 | .bind or .mnmd        | Yes (4)    | compress-prs (2) |
| VMS (DCv1)       | .bin.vms and .dat.vms | Yes        | decode-vms       |
| VMS (DCv2)       | .bin.vms and .dat.vms | Decode (3) | decode-vms (3)   |
| GCI (decrypted)  | .bin.gci and .dat.gci | Yes        | decode-gci       |
| GCI (with key)   | .bin.gci and .dat.gci | Yes        | decode-gci       |
| GCI (no key)     | .bin.gci and .dat.gci | Decode (3) | decode-gci (3)   |
| GCI (Ep3)        | .bin.gci or .mnm.gci  | Yes        | decode-gci       |
| GCI (Ep3 Trial)  | .bin.gci or .mnm.gci  | Decode (3) | decode-gci (3)   |
| DLQ              | .bin.dlq and .dat.dlq | Yes        | decode-dlq       |
| DLQ (Ep3)        | .bin.dlq or .mnm.dlq  | Yes        | decode-dlq       |
| QST (online)     | .qst                  | Yes        | decode-qst       |
| QST (download)   | .qst                  | Yes        | decode-qst       |

*Notes:*
1. *This is the default format. You can convert these to uncompressed format by running `newserv decompress-prs FILENAME.bin FILENAME.bind` (and similarly for .dat -> .datd)*
2. *Similar to (1), to compress an uncompressed quest file: `newserv compress-prs FILENAME.bind FILENAME.bin` (and likewise for .datd -> .dat)*
3. *Use the decode action to convert these quests to .bin/.dat format before putting them into the server's quests directory. If you know the encryption seed (serial number), pass it in as a hex string with the `--seed=` option. If you don't know the encryption seed, newserv will find it for you, which will likely take a long time.*
4. *Episode 3 quests don't go in the system/quests directory. See the Episode 3 section below.*

Episode 3 download quests consist only of a .bin file - there is no corresponding .dat file. Episode 3 download quest files may be named with the .mnm extension instead of .bin, since the format is the same as the standard map files (in system/ep3/). These files can be encoded in any of the formats described above, except .qst.

When newserv indexes the quests during startup, it will warn (but not fail) if any quests are corrupt or in unrecognized formats.

Quest contents are cached in memory, but if you've changed the contents of the quests directory, you can re-index the quests without restarting the server by running `reload quests` in the interactive shell. The new quests will be available immediately, but any games with quests already in progress will continue using the old versions of the quests until those quests end.

All quests, including those originally in GCI or DLQ format, are treated as online quests unless their filenames specify the dl category. newserv allows players to download all quests, even those in non-download categories.

### Episode 3 features

newserv supports many features unique to Episode 3:
* CARD battles. Not every combination of abilities has been tested yet, so if you find a feature or card ability that doesn't work like it's supposed to, please make a GitHub issue and describe the situation (the attacking card(s), defending card(s), and ability or condition that didn't work).
* Spectator teams.
* Tournaments. (But they work differently than Sega's tournaments did - see below)
* Downloading quests.
* Trading cards.
* Participating in card auctions. (The auction contents must be configured in config.json.)
* Decorations in lobbies. Currently only images are supported; the game also supports loading custom 3D models in lobbies, but newserv does not implement this (yet).

#### Battle records

After playing a battle, you can save the record of the battle with the $saverec command. You can then replay the battle later by using the $playrec command in a lobby - this will create a spectator team and play the recording of the battle as if it were happening in realtime. Note that there is a bug in older versions of Dolphin that seems to be frequently triggered when playing battle records, which causes the emulator to crash with the message `QObject::~QObject: Timers cannot be stopped from another thread`. To avoid this, use the latest version of Dolphin.

#### Tournaments

Tournaments work differently than they did on Sega's servers. Tournaments can be created with the `create-tournament` shell command, which enables players to register for them. (Use `help` to see all the arguments - there are many!) The `start-tournament` shell command starts the tournament (and prevents further registrations), but this doesn't schedule any matches. Instead, players who are ready to play their next match can all stand at the 4-player battle table near the lobby warp in the same CARD lobby, and the tournament match will start automatically.

These tournament semantics mean that there can be multiple matches in the same tournament in play simultaneously, and not all matches in a round must be complete before the next round can begin - only the matches preceding each individual match must be complete for that match to be playable.

The Meseta rewards for winning tournament matches can be configured in config.json.

#### Episode 3 files

Episode 3 state and game data is stored in the system/ep3 directory. The files in there are:
* card-definitions.mnr: Compressed card definition list, sent to Episode 3 clients at connect time. Card stats and abilities can be changed by editing this file.
* card-definitions.mnrd: Decompressed version of the above. If present, newserv will use this instead of the compressed version, since this is easier to edit.
* card-text.mnr: Compressed card text archive. Generally only used for debugging.
* card-text.mnrd: Decompressed card text archive; same format as TextCardE.bin. Generally only used for debugging.
* com-decks.json: COM decks used in tournaments. The default decks in this file come from logs from Sega's servers, so the file doesn't include every COM deck Sega ever made - the rest are probably lost to time.
* maps/: Online free battle and quest maps (.mnm/.bin/.mnmd/.bind files). newserv comes with all the original online and offline maps, including Story Mode quests. If you don't want the offline maps and quests to be playable online, delete the .bind files system/ep3/maps.
* maps-download/: Download maps and quests (.mnm/.bin/.mnmd/.bind files). There are two subcategories by default (download maps and Trial Edition download maps), but you can add more by editing QuestCategories in config.json. Categories that have flag 0x40 (Ep3 download) set are indexed from this directory; all others are indexed from system/quests/. Files in maps-download/ subdirectories have the same format as those in the maps/ directory, but should be named like `e###-gc3-LANGUAGE.EXT` (similar to how non-Episode 3 quests are named in the system/quests/ directory). If you want a map to be available for online play and for downloading, the file must exist in both maps/ and in a maps-download/ subdirectory (a symbolic link is acceptable).
* tournament-state.json: State of all active tournaments. This file is automatically written when any tournament changes state for any reason (e.g. a tournament is created/started/deleted or a match is resolved).

There is no public editor for Episode 3 maps and quests, but the format is described fairly thoroughly in src/Episode3/DataIndexes.hh (see the MapDefinition structure). You'll need to use `newserv decompress-prs ...` to decompress .bin or .mnm files before editing them, but you don't need to compress the files again to use them - just put the .bind or .mnmd file in the maps directory and newserv will make it available.

Like quests, Episode 3 card definitions, maps, and quests are cached in memory. If you've changed any of these files, you can run `reload ep3` in the interactive shell to make the changes take effect without restarting the server.

### Client patch directories

If you're not playing PSO Blue Burst on newserv, you can skip these steps.

newserv implements a patch server for PSO PC and PSO BB game data. Any file or directory you put in the system/patch-bb or system/patch-pc directories will be synced to clients when they connect to the patch server.

To make server startup faster, newserv caches the modification times, sizes, and checksums of the files in the patch directories. If the patch server appears to be misbehaving, try deleting the .metadata-cache.json file in the relevant patch directory to force newserv to recompute all the checksums. Also, in the case when checksums are cached, newserv may not actually load the data for a patch file until it's needed by a client. Therefore, modifying any part of the patch tree while newserv is running can cause clients to see an inconsistent view of it.

For BB clients, newserv reads some files out of the patch data to implement game logic, so it's important that certain game files are synchronized between the server and the client. newserv contains defaults for these files in the system/blueburst/map directory, but if these don't match the client's copies of the files, odd behavior will occur in games.

Specifically, the patch-bb directory should contain at least the data.gsl file and all map_*.dat files from the version of PSOBB that you want to play on newserv. You can copy these files out of the client's data directory from a clean installation, and put them in system/patch-bb/data.

Patch directory contents are cached in memory. If you've changed any of these files, you can run `reload patches` in the interactive shell to make the changes take effect without restarting the server.

### Memory patches and DOL files

Everything in this section requires resource_dasm to be installed, so newserv can use the PowerPC assembler and disassembler from its libresource_file library. If resource_dasm is not installed, newserv will still build and run, but these features will not be available.

In addition, these features are only supported for the following game versions:
* PSO GameCube Episodes 1&2 JP, USA, and EU (not Plus)
* PSO GameCube Episodes 1&2 Plus JP v1.04 (not v1.05)
* PSO GameCube Episode 3 Trial Edition
* PSO GameCube Episode 3 JP
* PSO GameCube Episode 3 USA (experimental; must be manually enabled in config.json)

You can put memory patches in the system/ppc directory with filenames like PatchName.patch.s and they will appear in the Patches menu for PSO GC clients that support patching. Memory patches are written in PowerPC assembly and are compiled when newserv is started. The PowerPC assembly system's features are documented in the comments in system/ppc/WriteMemory.s - this file is not a memory patch itself, but it describes how memory patches may be written and the restrictions that apply to them.

You can also put DOL files in the system/dol directory, and they will appear in the Programs menu. Selecting a DOL file there will load the file into the GameCube's memory and run it, just like the old homebrew loaders (PSUL and PSOload) did. For this to work, ReadMemoryWord.s, WriteMemory.s, and RunDOL.s must be present in the system/ppc directory. This has been tested on Dolphin but not on a real GameCube, so results may vary.

Like other kinds of data, functions and DOL files are cached in memory. If you've changed any of these files, you can run `reload functions` or `reload dol-files` in the interactive shell to make the changes take effect without restarting the server.

I mainly built the DOL loading functionality for documentation purposes. By now, there are many better ways to load homebrew code on an unmodified GameCube, but to my knowledge there isn't another open-source implementation of this method in existence.

### Using newserv as a proxy

If you want to play online on remote servers rather than running your own server, newserv also includes a PSO proxy. Currently this works with PSO GC and may work with PC and DC; it also works with some BB clients in specific situations.

To use the proxy for PSO DC, PC, or GC, add an entry to the corresponding ProxyDestinations dictionary in config.json, then run newserv and connect to it as normal (see below). You'll see a "Proxy server" option in the main menu, and you can pick which remote server to connect to.

To use the proxy for PSO BB, set the ProxyDestination-BB entry in config.json. If this option is set, it essentially disables the game server for all PSO BB clients - all clients will be proxied to the specified destination instead. Unfortunately, because PSO BB uses a different set of handlers for the data server phase and character selection, there's no in-game way to present the player with a list of options, like there is on PSO PC and PSO GC.

When you're on PSO DC, PC, or GC and are connected to a remote server through newserv's proxy, choosing the Change Ship or Change Block action from the lobby counter will send you back to newserv's main menu instead of the remote server's ship or block select menu. You can go back to the server you were just on by choosing it from the proxy server menu again.

There are many options available when starting a proxy session. All options are off by default unless otherwise noted. The options are:
* **Chat commands**: enables chat commands in the proxy session (on by default).
* **Chat filter**: enables escape sequences in chat messages and info board (on by default).
* **Player notifications**: shows a message when any player joins or leaves the game or lobby you're in.
* **Block pings**: blocks automatic pings sent by the client, and responds to ping commands from the server automatically. This works around a bug in Sylverant's login server.
* **Infinite HP**: automatically heals you whenever you get hit. An attack that kills you in one hit will still kill you, however.
* **Infinite TP**: automatically restores your TP whenever you use any technique.
* **Switch assist**: attempts to unlock doors that require two players in a one-player game.
* **Infinite Meseta** (Episode 3 only): gives you 1,000,000 Meseta, regardless of the value sent by the remote server.
* **Block events**: disables holiday events sent by the remote server.
* **Block patches**: prevents any B2 (patch) commands from reaching the client.
* **Save files**: saves copies of several kinds of files when they're sent by the remote server. The files are written to the current directory (which is usually the directory containing the system/ directory). These kinds of files can be saved:
    * Online quests and download quests (saved as .bin/.dat files)
    * GBA games (saved as .gba files)
    * Patches (saved as .bin files, and disassembled into PowerPC assembly if newserv is built with patch support)
    * Player data from BB sessions (saved as .bin files, which are not the same format as .nsc files)
    * Episode 3 online quests and maps (saved as .mnmd files)
    * Episode 3 download quests (saved as .mnm files)
    * Episode 3 card definitions (saved as .mnr files)
    * Episode 3 media updates (saved as .gvm, .bml, or .bin files)

The remote server will probably try to assign you a Guild Card number that doesn't match the one you have on newserv. On PSO DC, PC and GC, the proxy server rewrites the commands in transit to make it look like the remote server assigned you the same Guild Card number as you have on newserv, but if the remote server has some external integrations (e.g. forum or Discord bots), they will use the Guild Card number that the remote server believes it has assigned to you. The number assigned by the remote server is shown to you when you first connect to the remote server, and you can retrieve it in lobbies or during games with the $li command.

Some chat commands (see below) have the same basic function on the proxy server but have different effects or conditions. In addition, there are some server shell commands that affect clients on the proxy (run `help` in the shell to see what they are). If there's only one proxy session open, the shell's proxy commands will affect that session. Otherwise, you'll have to specify which session to affect with the `on` prefix - to send a chat message in LinkedSession:17205AE4, for example, you would run `on 17205AE4 chat ...`.

### Chat commands

newserv supports a variety of commands players can use by chatting in-game. Any chat message that begins with `$` is treated as a chat command. (If you actually want to send a chat message starting with `$`, type `$$` instead.)

Some commands only work on the game server and not on the proxy server. The chat commands are:

* Information commands
    * `$li`: Shows basic information about the lobby or game you're in. If you're on the proxy server, shows information about your connection instead (remote Guild Card number, client ID, etc.).
    * `$ping` (game server only): Shows round-trip ping time from the server to you.
    * `$what` (game server only): Shows the type, name, and stats of the nearest item on the ground.
    * `$matcount` (game server only): Shows how many of each type of material you've used.

* Debugging commands
    * `$debug` (game server only): Enable or disable debug. You need the DEBUG permission in your user license to use this command. When debug is enabled, you'll see in-game messages from the server when you take certain actions. You'll also be placed into the highest available slot in lobbies and games instead of the lowest, which is useful for finding commands for which newserv doesn't handle client IDs properly. This setting also disables certain safeguards and allows you to do some things that might crash your client.
    * `$quest <number>`: Load a quest by quest number. Can be used to load battle or challenge quests with only one player present.
    * `$qcall <function-id>`: Call a quest function on your client.
    * `$qset <flag-num>` or `$qclear <flag-num>`: Set or clear a global quest flag for everyone in the game.
    * `$qsync <reg-num> <value>`: Set a quest register's value on your client. `<reg-num>` should be either rXX (e.g. r60) or fXX (e.g. f60); if the latter, `<value>` is parsed as a floating-point value instead of as an integer.
    * `$gc` (game server only): Send your own Guild Card to yourself.
    * `$persist` (game server only): Enable or disable persistence for the current lobby or game. This determines whether the lobby/game is deleted when the last player leaves. You need the DEBUG permission in your user license to use this command because there are no game state checks when you do this. For example, if you make a game persistent, start a quest, then leave the game, the game can't be joined by anyone but also can't be deleted.
    * `$sc <data>`: Send a command to yourself.
    * `$ss <data>` (proxy server only): Send a command to the remote server.

* Personal state commands
    * `$arrow <color-id>`: Changes your lobby arrow color.
    * `$secid <section-id>`: Sets your override section ID. After running this command, any games you create will use your override section ID for rare drops instead of your character's actual section ID. To revert to your actual section id, run `$secid` with no name after it. On the proxy server, this will not work if the remote server controls item drops (e.g. on BB, or on Schtserv with server drops enabled). If the server does not allow cheat mode anywhere (that is, "CheatModeBehavior" is "Off" in config.json), this command does nothing.
    * `$rand <seed>`: Sets your override random seed (specified as a 32-bit hex value). This will make any games you create use the given seed for rare enemies. This also makes item drops deterministic in Blue Burst games hosted by newserv. On the proxy server, this command can cause desyncs with other players in the same game, since they will not see the overridden random seed. To remove the override, run `$rand` with no arguments. If the server does not allow cheat mode anywhere (that is, "CheatModeBehavior" is "Off" in config.json), this command does nothing.
    * `$ln [name-or-type]`: Sets the lobby number. Visible only to you. This command exists because some non-lobby maps can be loaded as lobbies with invalid lobby numbers. See the "GC lobby types" and "Ep3 lobby types" entries in the information menu for acceptable values here. Note that non-lobby maps do not have a lobby counter, so there's no way to exit the lobby without using either `$ln` again or `$exit`. On the game server, `$ln` reloads the lobby immediately; on the proxy server, it doesn't take effect until you load another lobby yourself (which means you'll like have to use `$exit` to escape). Run this command with no argument to return to the default lobby.
    * `$exit`: If you're in a lobby, sends you to the main menu (which ends your proxy session, if you're in one). If you're in a game or spectator team, sends you to the lobby (but does not end your proxy session if you're in one). Does nothing if you're in a non-Episode 3 game and no quest is in progress.
    * `$patch <name>`: Run a patch on your client. `<name>` must exactly match the name of a patch on the server.

* Blue Burst player commands (game server only)
    * `$bbchar <username> <password> <1-4>`: Use this command when playing on a non-BB version of PSO. If the username and password are correct, this command converts your current character to BB format and saves it on the server in the given slot. Any character already in that slot is overwritten.
    * `$edit <stat> <value>`: Modifies your character data. If the server does not allow cheat mode anywhere (that is, "CheatModeBehavior" is "Off" in config.json), this command does nothing.
    * `$save`: Saves your character, system, and Guild Card data immediately. (By default, your character is saved every 60 seconds while online, and your account and Guild Card data are saved whenever they change.)

* Game state commands (game server only)
    * `$maxlevel <level>`: Sets the maximum level for players to join the current game. (This only applies when joining; if a player joins and then levels up past this level during the game, they are not kicked out, but won't be able to rejoin if they leave.)
    * `$minlevel <level>`: Sets the minimum level for players to join the current game.
    * `$password <password>`: Sets the game's join password. To unlock the game, run `$password` with nothing after it.
    * `$itemtable`: Switches between using the client's or the server's drop table. No effect on BB (the server's drop table is always used). The server's rare tables are defined in JSON files in the system/item-tables directory.
    * `$drop`: Enables or disables all item drops from boxes and enemies in the current game.

* Episode 3 commands (game server only)
    * `$spec`: Toggles the allow spectators flag for Episode 3 games. If any players are spectating when this flag is disabled, they will be sent back to the lobby.
    * `$inftime`: Toggles infinite-time mode. Must be used before starting a battle. If infinite-time mode is enabled, the overall and per-phase time limits will be disabled regardless of the values chosen during battle setup. After completing a battle, infinite-time mode is reset to the server's default value (which can be set in Episode3BehaviorFlags in config.json).
    * `$defrange <min>-<max>`: Sets the DEF dice range for the next battle. If this is used, the dice range set during battle rules setup will apply only to ATK dice; DEF dice will use this range instead. Assist cards and other dice effects will still apply. Dice exchange also still applies if it is enabled.
    * `$stat <what>`: Shows a statistic about your player or team in the current battle. `<what>` can be `duration`, `fcs-destroyed`, `cards-destroyed`, `damage-given`, `damage-taken`, `opp-cards-destroyed`, `own-cards-destroyed`, `move-distance`, `cards-set`, `fcs-set`, `attack-actions-set`, `techs-set`, `assists-set`, `defenses-self`, `defenses-ally`, `cards-drawn`, `max-attack-damage`, `max-combo`, `attacks-given`, `attacks-taken`, `sc-damage`, `damage-defended`, or `rank`.
    * `$surrender`: Causes your team to immediately lose the current battle.
    * `$saverec <name>`: Saves the recording of the last battle.
    * `$playrec <name>`: Plays a battle recording. This command creates a spectator team and replays the specified battle log within it. There is a bug in Dolphin that makes use of this command unstable in emulation (see the "Battle records" section above).

* Cheat mode commands
    * `$cheat`: Enables or disables cheat mode for the current game. All other cheat mode commands do nothing if cheat mode is disabled. By default, cheat mode is off in new games but can be enabled; there is an option in config.json that allows you to disable cheat mode entirely, or set it to on by default in new games.
    * `$infhp` / `$inftp`: Enables or disables infinite HP or TP mode. Applies to only you. In infinite HP mode, one-hit KO attacks will still kill you.
    * `$warpme <floor-id>`: Warps yourself to the given floor.
    * `$warpall <floor-id>`: Warps everyone in the game to the given floor. You must be the leader to use this command, unless you're on the proxy server.
    * `$next`: Warps yourself to the next floor.
    * `$swa`: Enables or disables switch assist. When enabled, the server will attempt to automatically unlock two-player doors in solo games if you step on both switches sequentially.
    * `$item <desc>` (or `$i <desc>`): Create an item. `desc` may be a description of the item (e.g. "Hell Saber +5 0/10/25/0/10") or a string of hex data specifying the item code. Item codes are 16 hex bytes; at least 2 bytes must be specified, and all unspecified bytes are zeroes. If you are on the proxy server, you must not be using Blue Burst for this command to work. On the game server, this command works for all versions.
    * `$unset <index>`: In an Episode 3 battle, removes one of your set cards from the field. `<index>` is the index of the set card as it appears on your screen - 1 is the card next to your SC's icon, 2 is the card to the right of 1, etc. This does not cause a Hunters-side SC to lose HP, as they normally do when their items are destroyed.

* Configuration commands
    * `$event <event>`: Sets the current holiday event in the current lobby. Holiday events are documented in the "Using $event" item in the information menu. If you're on the proxy server, this applies to all lobbies and games you join, but only you will see the new event - other players will not.
    * `$allevent <event>` (game server only): Sets the current holiday event in all lobbies.
    * `$song <song-id>` (Episode 3 only): Plays a specific song in the current lobby.

* Administration commands (game server only)
    * `$ann <message>`: Sends an announcement message. The message text is sent to all players in all games and lobbies.
    * `$ax <message>`: Sends a message to the server's terminal. This cannot be used to run server shell commands; it only prints text to stderr.
    * `$silence <identifier>`: Silences a player (remove their ability to chat) or unsilences a player. The identifier may be the player's name or Guild Card number.
    * `$kick <identifier>`: Disconnects a player. The identifier may be the player's name or Guild Card number.
    * `$ban <identifier>`: Bans a player. The identifier may be the player's name or Guild Card number.

### Connecting local clients

#### PSO DC

Some versions of PSO DC will connect to a private server if you just set their DNS server address (in the network configuration) to newserv's address, and enable newserv's DNS server. This will not work for other versions; for those, you'll need a cheat code. Creating such a code is beyond the scope of this document.

If you're emulating PSO DC or have a disc image, you can patch the appropriate files within the disc image to make it connect to any address you want. Creating such a patch is also beyond the scope of this document.

#### PSO DC on Flycast

If you're emulating PSO DC, all versions will connect to newserv by setting the following options in Flycast's `emu.cfg` file under `[network]`:
- DNS = Your newserv's server address (newserv's DNS server must be running on port 53)
- EmulateBBA = yes
- Enable = yes

It is also necessary to save any DNS information to the flash memory of the Dreamcast to use the BBA - the easiest way to do this is to use the website option in USv2 and then choose the save to flash option.

Once set up, the EU and US versions will work without any extra set up (other than the HL Check Disable code for USv2), while the JP versions require HL Check Disable codes to be running.

If the server is running on the same machine as Flycast, this might not work, even if you point Flycast's DNS queries at your local IP address (instead of 127.0.0.1). In this case, you can modify the loaded executable in memory to make it connect anywhere you want. There is a script included with newserv that can do this on macOS; a similar technique could be done manually using scanmem on Linux or Cheat Engine on Windows. To use the script, do this:
1. Build and install memwatch (https://github.com/fuzziqersoftware/memwatch).
2. Start Flycast and run PSO. (You must start PSO before running the script; it won't work if you run the script before loading the game.)
3. Run `sudo patch_flycast_memory.py <original-destination>`. Replace `<original-destination>` with the hostname that PSO wants to connect to (you can find this out by using Wireshark and looking for DNS queries). The script may take up to a minute; you can continue using Flycast while it runs, but don't start an online game until the script is done.
4. Run newserv and start an online game in PSO.

If you use this method, you'll have to run the script every time you start PSO in Flycast, but you won't have to run it again if you start another online game without restarting emulation.

#### PSO PC

The version of PSO PC I have has the server addresses starting at offset 0x29CB34 in pso.exe. Using a hex editor, change those to "localhost" (without quotes) if you just want to connect to a locally-running newserv instance. Alternatively, you can add an entry to the Windows hosts file (C:\Windows\System32\drivers\etc\hosts) to redirect the connection to 127.0.0.1 (localhost) or any other IP address.

#### PSO GC on a real GameCube

You can make PSO connect to newserv by setting its default gateway and DNS server addresses to newserv's address. newserv's DNS server must be running on port 53 and must be accessible to the GameCube.

If you have PSO Plus or Episode III, it won't want to connect to a server on the same local network as the GameCube itself, as determined by the GameCube's IP address and subnet mask. In the old days, one way to get around this was to create a fake network adapter on the server (or use an existing real one) that has an IP address on a different subnet, tell the GameCube that the server is the default gateway (as above), and have the server reply to the DNS request with its non-local IP address. To do this with newserv, just set LocalAddress in the config file to a different interface. For example, if the GameCube is on the 192.168.0.x network and your other adapter has address 10.0.1.6, set newserv's LocalAddress to 10.0.1.6 and set PSO's DNS server and default gateway addresses to the server's 192.168.0.x address. This may not work on modern systems or on non-Windows machines - I haven't tested it in many years.

#### PSO GC on Dolphin

If you're using the HLE BBA type, set the BBA's DNS server address to newserv's IP address and it should work. (If newserv is on the same machine as Dolphin, try your local IP address or 127.0.0.1.) In PSO, use the example values below in PSO's network configuration.

If you're using the TAP BBA type, you'll have to set PSO's network settings appropriately for your tap interface. Set the DNS server address in PSO's network settings to newserv's IP address.

If you're using a version of Dolphin with tapserver support, you can make it connect to a newserv instance running on the same machine via the tapserver interface. You do not need to install or run tapserver. To do this:
1. Set Dolphin's BBA type to tapserver (Config -> GameCube -> SP1).
2. Enable newserv's IP stack simulator according to the comments in config.json and start newserv.
3. In PSO's network settings, enable DHCP ("Automatically obtain an IP address"), set DNS server address to "Automatic", and leave DHCP Hostname as "Not set". Leave the proxy server settings blank.
4. Start an online game.

#### PSO BB

The PSO BB client has been modified and distributed in many different forms. newserv supports most, but not all, of the common distributions. Unlike other versions, it's important that the client and server have the same map files, so make sure to set up the patch directory based on the client you'll be using with newserv. (See the "Client patch directories" section for instructions on setting this up.)

The original Japanese and US versions of PSO BB should work, but you'll have to modify your hosts file or edit psobb.exe to point to your newserv instance. The original versions are packed, so this is a more involved process than simply opening the executable in a hex editor and finding/replacing some strings.

Alternatively, you can use the Tethealla client (https://archive.org/details/psobb-tethealla-client); you can find the connection addresses starting at 0x56D724 in psobb.exe. Overwrite these addresses with your server's hostname or IP address, and you should be able to connect.

### Connecting external clients

If you want to accept connections from outside your local network, you'll need to set ExternalAddress to your public IP address in the configuration file, and you'll likely need to open some ports in your router's NAT configuration - specifically, all the TCP ports listed in PortConfiguration in config.json.

For GC clients, you'll have to use newserv's built-in DNS server or set up your own DNS server as well. If you want external clients to be able to use your DNS server, you'll have to forward UDP port 53 to your newserv instance. Remote players can then connect to your server by entering your DNS server's IP address in their client's network configuration.

### Non-server features

newserv has many CLI options, which can be used to access functionality other than the game and proxy server. Run `newserv help` to see these options and how to use them. The non-server things newserv can do are:

* Compress or decompress data in PRS, PR2, or BC0 format (`compress-prs`, `compress-pr2`, `compress-bc0`, `decompress-prs`, `decompress-pr2`, `decompress-bc0`)
* Compute the decompressed size of compressed PRS data without decompressing it (`prs-size`)
* Encrypt or decrypt data using any PSO version's network encryption scheme (`encrypt-data`, `decrypt-data`)
* Encrypt or decrypt data using Episode 3's trivial scheme (`encrypt-trivial-data`, `decrypt-trivial-data`)
* Encrypt or decrypt data using the Challenge Mode text algorithm (`encrypt-challenge-data`, `decrypt-challenge-data`)
* Encrypt or decrypt PSO GC save data (.gci files) (`encrypt-gci-save`, `decrypt-gci-save`)
* Convert a PSO GC or Episode 3 snapshot file to a BMP image (`decode-gci-snapshot`)
* Find the likely round1 or round2 seed for a corrupt save file (`salvage-gci`)
* Run a brute-force search for a decryption seed (`find-decryption-seed`)
* Convert quests in .gci, .vms, .dlq, or .qst format to .bin/.dat format (`decode-gci`, `decode-vms`, `decode-dlq`, `decode-qst`)
* Convert quests in .bin/.dat to .qst format (`encode-qst`)
* Convert text archives (e.g. TextEnglish.pr2) to JSON and vice versa (`decode-text-archive`, `encode-text-archive`)
* Disassemble quest scripts (`disassemble-quest-script`)
* Format Episode 3 game data in a human-readable manner (`show-ep3-maps`, `show-ep3-cards`)
* Convert item data to a human-readable description, or vice versa (`describe-item`, `encode-item`)
* Connect to another PSO server and pretend to be a client (`cat-client`)
* Replay a session log for testing (`replay-log`)
* Extract the contents of a .gsl or .bml archive (`extract-gsl`, `extract-bml`)
