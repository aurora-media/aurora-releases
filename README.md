# Aurora

Self-hosted media server with a console-style interface. Films, series, anime,
documentaries and music, scanned from your own folders and streamed to any device on
your network.

One Go binary plus a web app. No cloud, no account anywhere but your own machine, no
telemetry, and nothing to sign into.

**Free, all of it.** Download from [Releases](https://github.com/aurora-media/aurora-releases/releases/latest) · [aurora-media.shop](https://aurora-media.shop)

Aurora is not sold anywhere. Do not pay for access or download builds claiming to be
Aurora from anywhere but this repository.

> This repository is where Aurora is published and documented, and where to report a
> problem. The source code is not here: Aurora is licensed software, not open source.
> Installation documentation and help are below.

**Found a bug, or something behaving oddly?** [Open an issue](../../issues/new) — the
version from **Panel de control → Actualizaciones**, what you did, and what happened
instead is enough to start with.

`landing/` holds the source of [aurora-media.shop](https://aurora-media.shop).
---

## Contents

- [Requirements](#requirements)
- [Install with an installer](#install-with-an-installer)
- [Install without an installer](#install-without-an-installer)
- [Docker](#docker)
- [First run](#first-run)
- [Updating](#updating)
- [Where should the server run?](#where-should-the-server-run)
- [Watching on a TV, a phone or a tablet](#watching-on-a-tv-a-phone-or-a-tablet)
- [How to…](#how-to) — including [watching from outside the house](#watch-from-outside-my-house)
- [API keys](#api-keys)
- [Troubleshooting](#troubleshooting)

---

## Requirements

|  | Minimum | Notes |
|---|---|---|
| **OS** | Windows 10, macOS 11, or any Linux with systemd | 64-bit. ARM64 is supported on Linux (Raspberry Pi 4/5) and macOS (Apple silicon). |
| **FFmpeg** | any recent build | **Required.** It is what plays anything your browser cannot open by itself. The Linux packages install it for you; on Windows and macOS you install it once, see below. |
| **RAM** | 1 GB free | A Raspberry Pi 4 with 2 GB is comfortable. |
| **Disk** | ~80 MB, plus cache | The transcode cache is capped at 10 GB by default and is configurable. |
| **Browser** | anything from the last few years | Chrome, Edge, Firefox, Safari, and the browser built into most televisions. |

Aurora does **not** need a GPU. It uses one for transcoding if it finds one, and gets
along without.

### Installing FFmpeg

```bash
# Debian / Ubuntu / Raspberry Pi OS — the .deb does this for you
sudo apt install ffmpeg

# macOS
brew install ffmpeg

# Windows, with winget
winget install Gyan.FFmpeg
```

On Windows you can also download a build from
[gyan.dev](https://www.gyan.dev/ffmpeg/builds/) and put the folder containing
`ffmpeg.exe` on your `PATH`.

---

## Install with an installer

Download from [Releases](https://github.com/aurora-media/aurora-releases/releases) and run it. Each installer registers Aurora
to start with the machine, so your library is there whenever the machine is on.

### Windows

Run `aurora-vX.Y.Z-windows-setup.exe`. It installs to `Program Files`, registers a
Windows service, and opens `http://localhost:8096` when it finishes.

> **"Windows protected your PC"** — the installer is not code-signed yet. Click **More
> info → Run anyway**. A signing certificate costs a few hundred euros a year and will
> be bought when it is worth it; until then this is the honest state of things rather
> than something to discover halfway through.

Data lives in `C:\ProgramData\Aurora`. Uninstalling from *Add or remove programs*
leaves it there, so your library, accounts and watch history survive a reinstall.

### Linux

```bash
sudo apt install ./aurora-vX.Y.Z-linux-amd64.deb     # Debian, Ubuntu, Raspberry Pi OS
sudo dnf install ./aurora-vX.Y.Z-linux-x86_64.rpm    # Fedora, RHEL
```

FFmpeg comes with it as a declared dependency. The service is enabled and started
automatically:

```bash
systemctl status aurora        # is it running
journalctl -u aurora -f        # watch what it is doing
```

Data lives in `/var/lib/aurora` and is **not** removed when the package is.

### macOS

Open `aurora-vX.Y.Z-macos.dmg` and drag Aurora to Applications. Launching it registers
a login item so it starts with your session, then opens your library. It has no window
of its own — Aurora is a server, and the browser is its interface.

> **"Aurora can't be opened because it is from an unidentified developer"** — the app
> is not notarised, which needs a paid Apple developer account. Right-click the app →
> **Open** → **Open**, once. macOS remembers.

Data lives in `~/Library/Application Support/Aurora`.

---

## Install without an installer

The archives contain a single static binary and the web app. Nothing is installed,
nothing is registered, and deleting the folder removes it completely. This is the
right choice for a NAS, a container, or anyone who would rather run it by hand.

**Linux / macOS**

```bash
tar xzf aurora-vX.Y.Z-linux-arm64.tar.gz
cd aurora-vX.Y.Z-linux-arm64
./aurora -addr :8096 -web ./web -data ./data
```

**Windows** — unzip, then in PowerShell:

```powershell
.\aurora.exe -addr :8096 -web .\web -data .\data
```

| Flag | Default | What it does |
|---|---|---|
| `-addr` | `:8096` | address and port to listen on |
| `-web` | `./web` | the web app that ships in the archive |
| `-data` | `./data` | database, transcode cache and backups |

Which archive to take:

| File | For |
|---|---|
| `linux-x86_64` | ordinary Linux PCs and servers |
| `linux-arm64` | Raspberry Pi 4/5, ARM NAS boxes |
| `macos-applesilicon` | M1 and later |
| `macos-intel` | Intel Macs |
| `windows-x86_64` | Windows 10/11 |

### Running it as a service yourself

If you unpacked the archive but still want it to survive a reboot, the unit file the
package uses is in [`aurora.service`](aurora.service).
Copy it to `/etc/systemd/system/`, fix the paths, then:

```bash
sudo systemctl daemon-reload
sudo systemctl enable --now aurora
```

`Restart=always` is not decoration — it is what makes updating from inside the app
end in a running server rather than a stopped one.

---

## Docker

An official image is not published yet. Public archives are also pending; there is
currently no supported public download to install.

---

## First run

1. Open `http://localhost:8096`, or `http://<this-machine-ip>:8096` from another
   device on the same network.
2. Register. **The first account becomes the admin.**
3. **Ajustes → Añadir biblioteca**, pick a type, and browse to the folder.

The folder picker browses the **server's** disk, not the disk of the device you are
browsing from. If Aurora runs on a Pi, you are looking at the Pi.

| Library type | Expects |
|---|---|
| Películas | `Film (2019)/film.mkv`, or loose files named `Film (2019).mkv` |
| Series / Anime | `Show/Season 1/Show S01E02.mkv`, and most of the other shapes releases use |
| Documentales | either of the above |
| Música | `Artist/Album (1997)/03 - Song.flac` — see below |

Scanning is immediate; artwork and metadata arrive in the background. Music skips the
scrapers entirely.

---

## Updating

**Panel de control → Actualizaciones.**

Aurora checks [aurora-releases](https://github.com/aurora-media/aurora-releases/releases)
for a newer build **of its own platform**, shows what changed,
and one button downloads it, swaps itself and restarts. The browser waits for the
server to come back and offers a reload.

The previous binary is kept beside the new one as `aurora.old`, so recovering from a
release that will not start is renaming one file rather than finding another computer.

Builds made from a working tree report themselves as `dev` and are never offered a
replacement — that would throw away whatever they were built to test.

---

## Where should the server run?

Aurora scans the filesystem of whatever machine it runs on, so run it on the machine
your media is actually attached to.

| Setup | Notes |
|---|---|
| **Desktop or laptop** | Simplest. The machine has to be on for anyone to watch. |
| **NAS or home server** | Best of both: always on, drives attached directly. |
| **Raspberry Pi 4/5** | Cheap and silent. Use `linux-arm64`. Direct play and remuxing are instant; a full re-encode of 4K HEVC is not something a Pi can do in real time, which is fine because it almost never has to. |
| **Server on your PC, media on a NAS** | Works — mount the share (SMB/NFS) and point a library at the mounted path. Reads cross the network, so the PC must stay on. |

---

## Watching on a TV, a phone or a tablet

**On a television**, open `http://<server-ip>:8096` in the TV's own browser. The whole
interface is built for a remote: arrows move, OK selects, back goes back. There is no
box to buy and nothing to sideload.

**On a phone or tablet**, the browser works, and you can add Aurora to the home screen
for a full-screen app. A native Android app is in progress and is deliberately not
published yet.

---

## How to…

### …play a file my browser cannot open

Nothing to do — Aurora notices and remuxes or transcodes it. If you would rather your
own player handled it, **Ajustes → Vídeo → Reproductor externo** hands VLC or mpv the
original file: they decode DTS, TrueHD, HDR and PGS subtitles that no browser will
touch, and the server does nothing but serve bytes.

### …watch something with someone who does not have Aurora

**Ajustes → Vídeo → Ver juntos → Crear enlace.** Send them the link. They need no
account, no app and nothing installed; they get their own volume and their own
fullscreen, and you keep the controls. The link works for that one title and lapses
after twelve idle hours.

### …get subtitles

Embedded tracks appear on their own. **Buscar online** searches OpenSubtitles (needs a
free key, see below).

Either way Aurora lines them up against the film's own audio: a subtitle timed for a
different cut is put back where it belongs without anybody nudging it. One that cannot
be matched confidently is left exactly as it came, because a confident wrong answer is
worse than none. The manual offset is still there for the rest.

### …add music

Add a library of type **Música** pointed at your records. The player lives in the app
shell, so a record keeps going while you browse.

**Where the names come from.** There is no music database and nothing is sent
anywhere. Two sources, in this order:

1. **The folders**, which the person who owns the collection arranged on purpose:

   ```
   Música/
     Eminem/                              ← artist
       Curtain Call_ The Hits - Eminem/   ← album; "_ " becomes ": "
         01 - Eminem - Intro.flac         ← track 1, "Intro"
       The Marshall Mathers LP (2000)/    ← year in brackets is read
         CD2/                             ← disc 2
           06 - Stan.flac
   ```

   An artist repeated in the album folder or the filename is removed once it is
   known from the folder above, so nothing ends up saying "Eminem" three times.

2. **The file's own tags**, asked for only where the folders left a gap — title,
   artist, album, track, disc, date. That is a separate process per file, so it is
   the second question and not the first. The year is asked once per album folder
   rather than once per song.

**Album art** is `cover.jpg` (or `folder`, `front`, `album`, `artwork` — any of
`.jpg/.jpeg/.png/.webp`) sitting in the album folder, and failing that the sleeve
embedded in the file itself.

If a record comes out wrong, fixing the folder or the tags and rescanning fixes the
library — a music rescan always rewrites what it finds, because there is no scraped
metadata to protect.

### …stop a housemate seeing everything

**Panel de control → Usuarios.** Each account gets its own watch history, resume points
and lists, and you can limit which libraries it sees at all.

### …change how it looks

**Ajustes → Temas.** Presets, plus a CSS box that can override anything on the page.

### …skip intros

**Ajustes → Serie** while an episode is playing. Set the markers by hand, or let Aurora
find them by comparing the audio of two episodes.

### …make a playlist

**Listas** in the sidebar. Create one, then right-click anything in the library and
choose *Añadir a una lista*. A list is a view, not a folder: removing something from
one never touches the file, and lists belong to the account that made them, so two
people in a house do not share them.

### …sync with Trakt

**Panel de control → Plugins → Trakt** takes a Client ID and Client Secret. Aurora
ships neither: an application that embeds its own OAuth secret has published it to
everyone who downloads it. Create a free application at *trakt.tv → Settings → Your
API Apps*, Redirect URI `urn:ietf:wg:oauth:2.0:oob`, and paste the two values.

Each person then links their own account in **Cuenta → Trakt**: Aurora shows a short
code, you type it into trakt.tv on whatever device is nearest, and the page notices
by itself. After that, finishing something here marks it there, and **Traer mi
historial de Trakt** brings an existing history the other way.

Everything is matched on TMDB ids, so a remake or a translated title cannot be
confused for the original. Anything Aurora never identified has no id to send and is
skipped.

### …watch from outside my house

The honest starting point: **do not forward a port**. Putting a media server
straight onto the public internet means anyone who scans that port finds a login
page, and the only thing between them and your library is a password. Every
option below avoids that, and none of them needs a fixed IP address.

**Tailscale — the easy one, and the one to try first.**

It builds a small private network between your own devices. Your server and your
phone both join it, and the phone can reach the server as if it were on the sofa
next to it — from anywhere, over mobile data, in a hotel. Nothing is exposed to
the internet, because there is nothing listening on it: the two devices find each
other and talk directly, encrypted.

```bash
# on the server
curl -fsSL https://tailscale.com/install.sh | sh
sudo tailscale up
```

Install the Tailscale app on the phone or laptop, sign into the same account, and
Aurora is at `http://<server-name>:8096`. Free for personal use, up to 100
devices. Turn on **MagicDNS** in the Tailscale admin panel and the name works
without remembering an address.

The trade-off, stated plainly: everyone who watches has to install Tailscale
once. For a household that is fine. For sending a link to someone who does not
have it, use [watch together](#watch-something-with-someone-who-does-not-have-aurora) instead — that needs nothing
installed at all.

**Cloudflare Tunnel — a real address, nothing to install for viewers.**

The server opens an outbound connection to Cloudflare, and `aurora.yourdomain.com`
arrives back down it. No port is opened, your home IP is never published, and
anyone with the link can reach it in a browser. You need a domain on Cloudflare
(free plan is enough).

```bash
cloudflared tunnel login
cloudflared tunnel create aurora
cloudflared tunnel route dns aurora aurora.yourdomain.com
cloudflared tunnel run --url http://localhost:8096 aurora
```

Then run it as a service so it survives a reboot: `cloudflared service install`.

The trade-off: your library is now reachable by anyone who knows the address, so
the account passwords are doing real work. Cloudflare's own free tier also
discourages sustained video through a tunnel — fine for a few viewers, not a
public service.

**WireGuard, if you would rather run it yourself.** Same idea as Tailscale
without the coordinating service, and one forwarded UDP port instead of a web
port. More setup, nothing in the middle.

| | Viewers install something | Port opened | Own domain | Good for |
|---|---|---|---|---|
| **Tailscale** | yes, once | no | no | your own devices, your household |
| **Cloudflare Tunnel** | no | no | yes | sharing a link, family elsewhere |
| **WireGuard** | yes, once | one UDP | no | people who want no third party |
| **Port forwarding** | no | yes | optional | **not recommended** |


### …back up

Everything is in the data directory: `/var/lib/aurora`, `C:\ProgramData\Aurora`,
`~/Library/Application Support/Aurora`, or wherever `-data` points. Aurora also writes
a database snapshot there daily and keeps a week of them. Copy the folder; that is the
whole backup.

---

## API keys

Aurora ships none and shares none. Every install uses its own, and every key lives in
your own database or environment, never in this repo.

| Service | Needed for | Where to get it |
|---|---|---|
| TMDB | posters, synopses, cast, the Streaming catalogue | free and instant at [themoviedb.org](https://www.themoviedb.org/settings/api) |
| AniList | anime metadata | none needed |
| TheTVDB | anime episode ordering (optional) | [thetvdb.com](https://thetvdb.com/subscribe) |
| OpenSubtitles | subtitles in any language (optional) | free account at [opensubtitles.com](https://www.opensubtitles.com/consumers) |

Paste the TMDB key in **Panel de control → Plugins**, or set `TMDB_API_KEY` (see
`.env.example`). Without one Aurora still runs; titles just keep their filename.

The language metadata comes back in — titles, synopses, collection names — is the
**Metadata language** field on the same panel, or `TMDB_LANGUAGE` (`es-ES`, `en-US`,
`fr-FR`…). The panel wins over the variable. It defaults to `es-ES`; after changing it,
rescrape a library to apply it to what is already catalogued.

There is no IMDb API to use: IMDb sells bulk datasets and has no free public API. TMDB
is the free equivalent, and it hands back IMDb ids, which is what external source
adapters index on.

---

## Troubleshooting

**Nothing at `localhost:8096`**
Check it is running: `systemctl status aurora`, *Services* on Windows, or
`~/Library/Application Support/Aurora/aurora.log` on macOS.

**Another device on the network cannot reach it**
Use the machine's IP, not `localhost`. On Windows, allow `aurora.exe` through the
firewall when it asks — if you dismissed that prompt, add the rule by hand.

**A file will not play**
Almost always FFmpeg missing from `PATH`. `ffmpeg -version` should print something. The
player also shows what it is doing — *Directo*, *Remux* or *Transcodificando* — which
is the first thing worth reporting in an issue.

**Playback stutters on a small machine**
*Directo* and *Remux* cost the server almost nothing. *Transcodificando* means a full
re-encode, which a Raspberry Pi cannot do in real time for 1080p or 4K. If you see it
often, the device doing the watching probably cannot decode the codec — an external
player usually can.

**The scan found nothing**
The folder picker browses the server's disk. Check the path is the one the *server*
sees, not the one your laptop sees.

**A subtitle is out of step**
Aurora tried and was not confident enough to move it, which it says in the log. Use the
offset buttons in **Ajustes → Subtítulos**.

---
