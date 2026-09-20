In this guide we will cover **setting up**, **configuring** and **listing** a [Hytale](https://hytale.com/) server on **Windows**, **Linux** and **macOS**.

Hytale servers are a bit different from the other games we write server guides for, for one reason: **every Hytale world is already a server**. When you create a singleplayer world, the client spins up a local server and joins it. Multiplayer is the same thing with other people connecting. There is no separate "server mode" to switch into.

That also means mods are a server concern rather than a client one, which is covered in our [Hytale mod guide](https://moddingcommunity.com/blog/how-to-install-mods-in-hytale/).

**A note on how current this is:** Hytale has been in early access since January 2026 and Hypixel Studios have not yet published a standalone dedicated server setup document. What follows is built from their official docs, patch notes and Server Operator Policies, and it says clearly where the official guidance stops.

[**View Guide On TMC (Recommended Due To Better Formatting)**](https://moddingcommunity.com/blog/how-to-setup-a-hytale-server/)

## Table Of Contents
* [Requirements](#requirements)
* [Understanding Hytale's Server Model](#understanding-hytales-server-model)
    * [Universes, Worlds And Instances](#universes-worlds-and-instances)
* [Hosting From The Client](#hosting-from-the-client)
* [Running A Standalone Server](#running-a-standalone-server)
    * [Where The Server Files Live](#where-the-server-files-live)
    * [Linux Prerequisites](#linux-prerequisites)
* [Installing Mods And Plugins](#installing-mods-and-plugins)
* [Configuration](#configuration)
* [Getting Listed In Server Discovery](#getting-listed-in-server-discovery)
    * [Heartbeats](#heartbeats)
* [Server Operator Policies](#server-operator-policies)
    * [Monetisation And Revenue Share](#monetisation-and-revenue-share)
* [Port Forwarding](#port-forwarding)
* [Keeping It Running On Linux](#keeping-it-running-on-linux)
* [Backups](#backups)
* [Writing Your Own Plugins](#writing-your-own-plugins)
* [Troubleshooting](#troubleshooting)
* [Conclusion](#conclusion)
* [See Also](#see-also)

## Requirements
Hytale's published minimum requirements for the **client** are:

| | Minimum |
| --- | --- |
| OS | Windows 11 or Windows 10 x64 (1809), Linux x64 (kernel 6.15), or Apple Silicon with macOS Tahoe 26.0 |
| CPU | Intel Core i5-7500, AMD Ryzen 3 1200, or Apple M1 |
| RAM | 8 GB |
| GPU | Intel UHD 620, AMD Radeon Vega 6, Apple M1, or a GTX 900 / Radeon 400 series card |

For a server, ignore the GPU line and weight everything else upward:

* **RAM is the main constraint.** The server is Java, it holds loaded chunks and entities in memory, and asset packs add to that. 8 GB is a floor for a small group, and a busy server with mods wants considerably more.
* **Single-thread performance matters.** As with most block games, a lot of the tick work is not parallel.
* **Disk** depends entirely on how much world gets explored. World saves grow steadily.
* A **Hytale licence**, bought from [hytale.com](https://hytale.com/). The game is not on Steam.

**NOTE** - Hytale is Apple Silicon only on macOS. There is no Intel Mac build.

## Understanding Hytale's Server Model
Kevin Carstens, Hytale's Technical Director, put it plainly when the studio published their modding strategy: Hytale is different in that even when you join singleplayer, you join a local server that is just for yourself. When the docs talk about servers, they mean both singleplayer and multiplayer.

Practically, that has a few consequences worth internalising:

* **Mods are installed once, by the host.** Players joining your server install nothing. Their vanilla client renders whatever your server tells it to.
* **Your singleplayer knowledge transfers.** Configuring a world in singleplayer and configuring a server are close to the same skill.
* **There is no modded client to distribute.** The studio has said they do not intend to support client mods, precisely so that a fragmented ecosystem of per-server clients never develops.

### Universes, Worlds And Instances
Hytale organises things in a hierarchy that shows up throughout the configs and the patch notes.

A **universe** contains worlds. A **world** is what players think of as the map. **Instances** are separate copies of a world, used for things like the Portal Devices, and they can inherit settings such as fall damage and PvP from the universe's default world.

You will see this vocabulary in plugin configuration, so it is worth knowing before you start reading config files.

## Hosting From The Client
For a small group, this is the whole answer and it needs no extra software.

Create a world in the client, select the mods and asset packs you want for it in the **New World** options, and share an invite code with your friends. They connect and play with a completely vanilla client.

This suits a group of friends and it is how most people will run their first Hytale multiplayer. The limits are the obvious ones: the world is only up when your client is running, it competes for resources with your own game, and it is tied to your machine.

## Running A Standalone Server
For something persistent that runs without you logged in, you want the server on its own.

**This is where the official documentation currently stops.** Hypixel Studios ship [a Java Server API reference](https://docs.hytale.com/api/) and [asset documentation](https://docs.hytale.com/assets/), and they run [Server Discovery](https://accounts.hytale.com/servers) with a full policy document behind it, so self-hosted servers are plainly expected and supported. What they have not yet published is a step-by-step "download this, run this command" guide of the kind Valheim or ARK have.

Until that lands, the practical routes are:

* **The Hytale launcher and account manager.** Server distribution is handled through Hytale's own platform rather than through a third-party store, so [accounts.hytale.com](https://accounts.hytale.com/) is the place to check first.
* **The official Discord.** The [Hytale Discord](https://discord.com/invite/hytale) has the studio's developers in it and is where current server operators are. For something moving this fast, that is more reliable than any written guide, including this one.
* **A managed host.** Several game server hosts added Hytale support during 2026 and handle the setup for you.

**NOTE** - The server is not obfuscated, and the studio committed to releasing the server source code under the **Hytale Shared Source** programme, which went live in June 2026. Licensed players can access the full server source, network protocol and assets on GitHub, with access modelled on Epic's Unreal Engine source programme. If you are technical, that is the most complete documentation that exists.

### Where The Server Files Live
On a client install, the server side of things lives under `UserData`:

```
C:\Users\<user>\AppData\Roaming\Hytale\UserData
```

with `Mods` for globally available mods and `Saves\<World Name>` for each world, each of which has its own `mods` folder.

Linux and macOS use the equivalent application data directory for the platform, with the same structure underneath.

A standalone server keeps the same shape: a `mods` directory for plugins and asset packs, and world data alongside it.

### Linux Prerequisites
Hytale's server is Java, so you need a JVM plus the usual server housekeeping tools.

**Debian and Ubuntu:**

```bash
sudo apt update
sudo apt install -y openjdk-21-jre-headless screen curl unzip
```

**Fedora, RHEL and Rocky:**

```bash
sudo dnf install -y java-21-openjdk-headless screen curl unzip
```

Check what you have with:

```bash
java -version
```

**TIP** - Check the Java version the current Hytale build expects before committing to a JVM. It has moved during early access, and a mismatch produces an unhelpful error.

Hytale's Linux requirement is a **kernel 6.15** x64 system, which is newer than several long-term-support distributions ship by default. Check with `uname -r` before you start.

## Installing Mods And Plugins
Mods go in the server's `mods` directory and the server loads them at startup.

1. Download the plugin or asset pack, usually from [CurseForge](https://www.curseforge.com/hytale).
2. Put it in the server's `mods` directory. Plugins are Java `.jar` files; asset packs are a zip or a folder with a `manifest.json` inside.
3. Restart the server.

Well-built plugins create their own folder under `mods` for configuration and data. [BetterMap](https://www.curseforge.com/hytale/mods/bettermap), for instance, creates `mods/bettermap/` with a `config.json` in it, and registers `/bm` commands including `/bm reload` to pick up config changes without a restart.

Players need nothing installed. That is the entire point of the server-side model.

Our [Hytale mod guide](https://moddingcommunity.com/blog/how-to-install-mods-in-hytale/) goes into the four mod categories and how asset packs are put together.

## Configuration
Hytale configuration is JSON, and it is spread across a few places rather than sitting in one `server.properties` style file.

* **World settings** are chosen when a world is created, covering things like fall damage and PvP, and instances can inherit them from the universe's default world.
* **Gameplay configs** live in JSON under the server's config directories. Patch notes reference paths such as `GameplayConfigs/Portal.json`, which gives you a sense of the layout.
* **Plugin configs** live under each plugin's own folder in `mods`.

Because the format and paths are still shifting between updates, the [patch notes on hytale.com](https://hytale.com/news) are worth reading whenever you update. Update 7's notes, for example, renamed several config keys outright.

**WARNING** - Read the patch notes before updating a live server. Hytale runs weekly pre-release patches bundled into stable releases every few weeks, and config keys have been renamed and removed between them. A rename you did not notice means a setting silently reverts to its default.

## Getting Listed In Server Discovery
Server Discovery is the in-game server browser, which shipped with **Update 5** in May 2026. Listing is free and the process is fully documented.

1. Sign in to the [Account Manager](https://accounts.hytale.com/) and open the [Server Profiles](https://accounts.hytale.com/servers) page.
2. Click **Create server**.
3. Fill in your details:
   * **Name and description.** This is what players judge you on, so make it say what your server actually is.
   * **Server type**, from Survival, Adventure/RPG, Creative, PvP, Minigames, Roleplay, Social, Sandbox or Other.
   * **Audience tag**: Everyone, Teen or Mature.
   * **Regions you serve**, from ten options across NA, South America, EU, the Middle East, Asia and Oceania.
   * **Domain verification** via a TXT DNS record, so players know the listing is genuinely yours.
4. Submit and wait for moderation. A moderator checks the details are accurate, and the status on your listing card updates as it moves through review.
5. Once accepted, configure your server to send heartbeats using the discovery token on your listing card.

Listings are reviewed in the order they arrive. The studio also hand-picks **featured servers** to highlight to the community.

### Heartbeats
Your server proves it is alive by sending heartbeats using the discovery token from your listing card. Generating that token is a simple command you run as an OP, in either the console or in chat, to complete the link.

**If your server stops sending heartbeats for more than two minutes, it is hidden from listings until it comes back.** That is a sensible design, and it means a flaky server quietly disappears from Discovery rather than frustrating players who try to join it.

**NOTE** - The first version of Discovery is deliberately minimal. Listings are name, description and tags, with no custom icons, banners or rich media. There is no live player count or rating either, which the studio held back specifically so servers could not spoof or manipulate them.

## Server Operator Policies
Hypixel Studios publish [Server Operator Policies](https://hytale.com/server-policies) that apply to **anyone** hosting a Hytale server, not just listed ones. They are worth reading properly if you plan to run anything public. The headlines:

* **Baseline rules apply to every operator.** Listed servers in Discovery take on extra obligations on top.
* **Listed servers** need verified operator details, a 24-hour response target for urgent safety reports where reasonably practicable, technical and quality minimums, and accurate content self-rating with parental controls.
* **Prohibited on all servers**: sexual content, NFTs, exploitative blockchain or crypto schemes, and real money gambling. Accepting cryptocurrency as a payment method is allowed.
* **Listed servers must support official cosmetics**, with limited theme overrides permitted if disclosed.

### Monetisation And Revenue Share
If you take money for anything on your server, this section applies to you.

* Monetisation has to be truthful and lawful. All Ages servers may carry only family-safe ads. Teen, mature and adult servers must disclose gameplay-affecting purchases.
* **Paid random items** need published numerical odds, an "expected cost" disclosure, and twelve months of recordkeeping. **Double random purchases are banned outright.**
* **Revenue share applies to all monetising servers, and it is 0% for the first two years, until 13 January 2028.**

That last point is a genuine deadline worth diarising if you are building something commercial.

**NOTE** - Where these policies conflict with the Hytale EULA or Terms of Service, the EULA and ToS take precedence, except where the policies are expressly designated as Additional Terms for listed servers.

## Port Forwarding
Hytale has not published a default server port in its public documentation, so check your server's own configuration rather than trusting a number you read somewhere.

The general shape is the same as any game server:

1. Find the port your server binds to, from its config or its startup output.
2. Forward it on your router to the server machine's internal IP.
3. Allow it through the OS firewall.

```bash
# Debian/Ubuntu with ufw, substituting your actual port
sudo ufw allow <port>/tcp

# Fedora/RHEL with firewalld
sudo firewall-cmd --permanent --add-port=<port>/tcp
sudo firewall-cmd --reload
```

Domain verification for Discovery uses a **TXT DNS record** rather than anything network-facing, so that part needs no ports opened.

## Keeping It Running On Linux
Once you have a working start command, wrap it so it survives disconnects and reboots.

The quick version with `screen`:

```bash
screen -S hytale ./start.sh
```

Detach with `CTRL` + `A` then `D`, reattach with `screen -r hytale`.

The proper version, as a systemd unit at `/etc/systemd/system/hytale.service`:

```ini
[Unit]
Description=Hytale Dedicated Server
After=network.target

[Service]
Type=simple
User=hytale
WorkingDirectory=/home/hytale/server
ExecStart=/home/hytale/server/start.sh
Restart=on-failure
RestartSec=15
KillSignal=SIGINT
TimeoutStopSec=120

[Install]
WantedBy=multi-user.target
```

```bash
sudo systemctl daemon-reload
sudo systemctl enable --now hytale
```

`KillSignal=SIGINT` and a generous `TimeoutStopSec` give the server time to save the world on shutdown rather than being killed mid-write. That matters more than it sounds.

Follow the log with `journalctl -u hytale -f`.

## Backups
Hytale does not advertise an automatic backup system the way Valheim does, so treat backups as your job.

Back up the whole server data directory, worlds and `mods` included, on a schedule. A cron job with `rsync` or `tar` to a different disk is plenty:

```bash
0 */6 * * * tar czf /backups/hytale-$(date +\%Y\%m\%d-\%H\%M).tar.gz /home/hytale/server/UserData
```

Two Hytale-specific reasons this matters more than usual:

* **Early access.** Updates arrive weekly during pre-release and things do break.
* **Mods write into worlds.** Removing a mod that added blocks or entities can leave a world in a poor state, exactly as in any block game.

Back up before every update, not just on a schedule.

## Writing Your Own Plugins
If you are going to run a server seriously, you will end up writing something for it.

Hytale plugins are Java, and the [Server API documentation](https://docs.hytale.com/api/) is generated from the server itself, covering packages under `com.hypixel.hytale`. The [asset reference](https://docs.hytale.com/assets/) documents the JSON format of every asset the server loads, generated from the server's own codecs.

Two things worth knowing about the direction:

* There is **no text-based scripting** and there will not be. The studio argued that Lua-style scripting is a false compromise that leaves programmers juggling two languages and designers still learning to program.
* **Visual scripting is the plan** instead, in the spirit of Unreal Blueprints, with programmers extending it by exposing new nodes from Java. The Trigger Volume tool that shipped in Update 5 is an early piece of this, and it was explicitly built with mod support so modders can add their own effects.

## Troubleshooting
**I cannot find the dedicated server download.** As covered above, there is no published standalone server setup guide yet. Check [accounts.hytale.com](https://accounts.hytale.com/) and ask in the [official Discord](https://discord.com/invite/hytale).

**My listing is not showing up in Discovery.** Check its status on the listing card. It may still be in the moderation queue, which can be slow during a launch wave. If it was accepted and then vanished, your server has probably stopped sending heartbeats.

**Players say they need mods to join.** They do not. Hytale modding is server-side, and anyone telling you otherwise is describing a different game.

**A config setting reverted after an update.** Read the patch notes. Config keys have been renamed and removed between updates, and a renamed key means your value is being ignored.

**The server will not start after an update.** Check the Java version the current build expects, and read the patch notes for breaking changes. If it started right after you added a plugin, pull the plugin out and test again.

**Linux kernel too old.** Hytale asks for kernel 6.15 x64. Several LTS distributions ship older. Check with `uname -r`.

**Can I run a server on macOS?** The client is Apple Silicon only on macOS, and the server side is the least documented part of an already young game. A Linux VM or a rented box is the path of least resistance.

## Conclusion
Hytale's server model is the cleanest of any game in these guides, once the core idea lands: every world is a server, mods are installed once by the host, and players join with a vanilla client and nothing else.

For a group of friends, hosting from the client with an invite code is genuinely enough. For something persistent and public, the parts that are fully documented today are Server Discovery and the Server Operator Policies, both of which are worth reading carefully before you launch anything. The standalone server setup itself is the piece still waiting on official documentation, and the Discord is the honest answer until that arrives.

We will rewrite this section properly when Hypixel Studios publish it.

## See Also
* [Hytale Documentation](https://docs.hytale.com/) - The official docs hub for running servers and building mods.
* [Hytale Server API reference](https://docs.hytale.com/api/) - The Java API, for plugin developers.
* [Asset reference](https://docs.hytale.com/assets/) - The JSON format of every asset the server loads.
* [Server Operator Policies](https://hytale.com/server-policies) - Required reading for anyone hosting publicly.
* [Hytale Account Manager](https://accounts.hytale.com/) - Where Server Profiles and Discovery listings live.
* [Official Server Lists announcement](https://hytale.com/news/2026/4/official-server-lists) - The full Discovery process, and the source for that section.
* [Hytale Modding Strategy and Status](https://hytale.com/news/2025/11/hytale-modding-strategy-and-status)
* [Official Hytale Discord](https://discord.com/invite/hytale)
* [Hytale Wiki](https://hytale.fandom.com/wiki/Hytale_Wiki)
* [TMC App](https://github.com/modcommunity/tmc-app) - Our own app, with a server browser, live latency graphs and RCON. Open source under GPL-3.0 and in very early development, so trying it or leaving feedback is genuinely appreciated.

Hytale is in early access and moving quickly, so this guide will need updating more often than most. If you find an instruction that no longer matches what you are seeing, or you have set up a standalone server and can fill in the gaps above, please report it or open a [pull request](https://github.com/modcommunity/how-to-setup-a-hytale-server/pulls) on this guide's GitHub repository. Contributions to this one in particular would be genuinely useful.

Join our [Discord server](https://discord.moddingcommunity.com) if you have any questions or want help with anything server or modding related!
