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
version from **Dashboard → Updates**, what you did, and what happened
instead is enough to start with.

`landing/` holds the source of [aurora-media.shop](https://aurora-media.shop).
---

## Contents

- [Requirements](#requirements) - and which download you want
- [Windows, step by step](#windows-step-by-step)
- [Raspberry Pi, Debian, Ubuntu, Mint](#raspberry-pi-debian-ubuntu-mint--step-by-step)
- [macOS](#macos-step-by-step)
- [Docker](#docker)
- [Install without an installer](#install-without-an-installer-any-platform)
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
| **OS** | Windows 10, macOS 11, any Linux with systemd, or Docker | 64-bit. ARM64 works on Linux (Raspberry Pi 4/5, ARM NAS) and macOS (Apple silicon). |
| **FFmpeg** | any recent build | **Required.** It is what plays anything your browser cannot open by itself. The Windows installer, the Linux packages and the Docker image all take care of it; only the plain archives leave it to you. |
| **RAM** | 1 GB free | A Raspberry Pi 4 with 2 GB is comfortable. |
| **Disk** | ~150 MB, plus cache | The transcode cache is capped at 10 GB by default and is configurable. |
| **Browser** | anything from the last few years | Chrome, Edge, Firefox, Safari, and the browser built into most televisions. |

Aurora does **not** need a GPU. It uses one for transcoding if it finds one, and gets
along without.

**Which download do I want?** The short answer:

| You have | Take this |
|---|---|
| A Windows PC | `aurora-vX.Y.Z-windows-setup.exe` |
| A Raspberry Pi (Raspberry Pi OS) | `aurora-vX.Y.Z-linux-arm64.deb` |
| Debian, Ubuntu, Mint on a normal PC | `aurora-vX.Y.Z-linux-amd64.deb` |
| Fedora, RHEL, Rocky | `aurora-vX.Y.Z-linux-x86_64.rpm` |
| A Mac | `aurora-vX.Y.Z-macos-applesilicon.tar.gz` (M1 or later) or `-macos-intel` |
| A NAS, Unraid, CasaOS, or you already use Docker | [the container image](#docker) |

Everything is on the [latest release page](https://github.com/aurora-media/aurora-releases/releases/latest),
under **Assets**. If the list is collapsed, click *Assets* to open it.

---

## Windows, step by step

**1. Download the installer.** Open the
[latest release](https://github.com/aurora-media/aurora-releases/releases/latest),
click **Assets**, and click `aurora-vX.Y.Z-windows-setup.exe`. It lands in your
Downloads folder.

**2. Run it.** Double-click the file.

**3. Get past the blue warning.** Windows shows **"Windows protected your PC"**. Click
**More info**, then **Run anyway**.

> This appears because the installer is not code-signed. A certificate costs a few
> hundred euros a year and will be bought when that is worth it. Until then, this is
> the honest state of things rather than something to discover halfway through. The
> file you are running is the one published on this page and nowhere else.

**4. Approve the admin prompt.** Windows asks *"Do you want to allow this app to make
changes?"* — **Yes**. Aurora is installed as a Windows **service**, and registering a
service, opening the firewall and writing to `Program Files` all need administrator
rights. This is asked once, by the installer, and never again by Aurora itself.

**5. Click through the wizard.** Four questions, all with sensible answers already
filled in:

| Page | What it asks | Just press Next unless… |
|---|---|---|
| Install location | Where the program goes | …you want it off the system drive. `C:\Program Files\Aurora` is right for almost everyone. |
| Data folder | Where your library **data** goes — database, accounts, watch history, artwork, cache | …that drive is short of space. This is **not** your films; it is what Aurora writes *about* them. |
| Port | Which port Aurora answers on | …something else already uses **8096**. Jellyfin does, so if you run Jellyfin on the same machine, use `8097`. |
| Almost there | Two tick boxes | Leave both ticked. See below. |

The two tick boxes on the last page:

- **Allow other devices on my network to reach Aurora** — adds a Windows Firewall
  rule. Without it Aurora works on this PC and is invisible to every phone, tablet and
  television in the house. This is the single most common "it does not work".
- **Download and install FFmpeg** — only offered when the machine has none. It fetches
  the standard Windows build (~90 MB) from [gyan.dev](https://www.gyan.dev/ffmpeg/builds/)
  and puts it beside Aurora, where Aurora looks for it. Without FFmpeg, Aurora plays
  only the files your browser can already open by itself.

**6. Finish.** Leave **Open Aurora** ticked and click Finish. Your browser opens at
`http://localhost:8096`. Go to [First run](#first-run).

### What the Windows installer actually did

| Thing | Where |
|---|---|
| Program | `C:\Program Files\Aurora` |
| Your data | `C:\ProgramData\Aurora` — database, accounts, history, artwork, cache, `aurora.log` |
| Settings | `C:\Program Files\Aurora\aurora.conf` — port, data folder, web folder |
| Service | **Aurora Media Server**, startup type *Automatic*, restarts itself if it crashes |
| Firewall | Inbound rule *Aurora Media Server* on the port you chose, private and domain networks |
| Start menu | *Aurora*, *Aurora data folder*, *Aurora log*, *Uninstall Aurora* |

It starts with Windows and needs **nobody logged in** — that is what "service" means
here. To check on it: press `Win+R`, type `services.msc`, find **Aurora Media Server**.
Right-click for Start, Stop and Restart.

To change the port or the data folder later, edit `aurora.conf` and restart the
service from `services.msc`.

**Uninstalling:** *Settings → Apps → Installed apps → Aurora → Uninstall*. The service
and the firewall rule are removed, and you are **asked** whether to delete your data
folder — say No and a reinstall picks up exactly where you left off, accounts and
watch history included.

> **Media on a NAS or network share?** A Windows service runs as *Local System*, which
> has no access to your mapped network drives. Open `services.msc` → **Aurora Media
> Server** → *Properties* → **Log On** tab → *This account*, and give it a Windows
> account that can reach the share. Then use the full `\\server\share\...` path when
> adding the library, not the mapped letter.

---

## Raspberry Pi, Debian, Ubuntu, Mint — step by step

**1. Download the package** on the Pi or PC itself:

```bash
# Raspberry Pi 4/5 and other ARM boards (64-bit Raspberry Pi OS)
wget https://github.com/aurora-media/aurora-releases/releases/latest/download/aurora-vX.Y.Z-linux-arm64.deb

# ordinary 64-bit PCs
wget https://github.com/aurora-media/aurora-releases/releases/latest/download/aurora-vX.Y.Z-linux-amd64.deb
```

Replace `vX.Y.Z` with the version on the
[latest release page](https://github.com/aurora-media/aurora-releases/releases/latest).

**2. Install it.** `apt` pulls in FFmpeg for you:

```bash
sudo apt install ./aurora-vX.Y.Z-linux-arm64.deb
```

**3. That is it.** The package created an `aurora` system user, enabled the service and
started it. It printed the address to open. Check it is running:

```bash
systemctl status aurora          # should say active (running)
journalctl -u aurora -f          # watch what it is doing, Ctrl+C to stop watching
```

**4. Open it** at `http://<the-machine's-ip>:8096`. Find the IP with `hostname -I`.

| Thing | Where |
|---|---|
| Program | `/usr/bin/aurora`, web app in `/usr/share/aurora/web` |
| Your data | `/var/lib/aurora` — database, accounts, history, artwork, cache |
| Service | `/lib/systemd/system/aurora.service`, runs as the `aurora` user |
| Logs | `journalctl -u aurora` |

**Permissions.** Aurora runs as the `aurora` user, so that user has to be able to
*read* your media folder. If a scan finds nothing, that is almost always why:

```bash
sudo usermod -aG <group-that-owns-the-media> aurora
sudo systemctl restart aurora
```

**Uninstalling:** `sudo apt remove aurora`. `/var/lib/aurora` is deliberately left
alone, so your accounts and history survive. `sudo apt purge aurora` if you really
want it gone, and delete `/var/lib/aurora` by hand.

### Fedora, RHEL, Rocky

```bash
sudo dnf install ./aurora-vX.Y.Z-linux-x86_64.rpm
```

Everything else is identical to the above.

---

## macOS, step by step

**1. Download** `aurora-vX.Y.Z-macos-applesilicon.tar.gz` (M1 or newer) or
`aurora-vX.Y.Z-macos-intel.tar.gz` from the
[latest release](https://github.com/aurora-media/aurora-releases/releases/latest).

**2. Install FFmpeg**, once, in Terminal:

```bash
brew install ffmpeg
```

No Homebrew? Install it from [brew.sh](https://brew.sh) first — one pasted command.

**3. Unpack and run:**

```bash
cd ~/Downloads
tar xzf aurora-vX.Y.Z-macos-applesilicon.tar.gz
cd aurora-vX.Y.Z-macos-applesilicon
./aurora -web ./web -data ~/Library/Application\ Support/Aurora
```

macOS may say the binary is from an unidentified developer: *System Settings → Privacy
& Security → Open Anyway*, once.

**4. Open** `http://localhost:8096`.

**5. To keep it running** after you close Terminal, and to start it with your session,
save this as `~/Library/LaunchAgents/shop.auroramedia.server.plist`, with the two paths
corrected, then `launchctl load -w ~/Library/LaunchAgents/shop.auroramedia.server.plist`:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
  <key>Label</key><string>shop.auroramedia.server</string>
  <key>ProgramArguments</key>
  <array>
    <string>/Users/YOU/Aurora/aurora</string>
    <string>-web</string><string>/Users/YOU/Aurora/web</string>
    <string>-data</string><string>/Users/YOU/Library/Application Support/Aurora</string>
  </array>
  <key>RunAtLoad</key><true/>
  <key>KeepAlive</key><true/>
</dict>
</plist>
```

A signed `.dmg` that does all of this for you is coming; it needs a paid Apple
developer account, which Aurora does not have yet.

---

## Docker

The image bundles FFmpeg and MKVToolNix, so there is nothing else to install. It runs
on amd64 and arm64 (Raspberry Pi included).

**One command:**

```bash
docker run -d --name aurora --restart unless-stopped \
  -p 8096:8096 \
  -v aurora-data:/data \
  -v /path/to/your/media:/path/to/your/media:ro \
  ghcr.io/aurora-media/aurora:latest
```

**Or with compose** — take [`docker-compose.yml`](docker-compose.yml) from this repository, or save this and run `docker compose up -d`:

```yaml
services:
  aurora:
    image: ghcr.io/aurora-media/aurora:latest
    container_name: aurora
    restart: unless-stopped
    ports:
      - "8096:8096"
    volumes:
      # Aurora's own data: database, accounts, history, artwork, cache.
      - ./aurora-data:/data
      # Your media, read-only. Mount it at the SAME path inside the container
      # as it has on the host: library paths are stored in the database, and
      # keeping the paths identical is what makes them survive a move.
      - /path/to/your/media:/path/to/your/media:ro
    environment:
      - TZ=Europe/Madrid
```

Then open `http://<the-machine's-ip>:8096`.

| Setting | How |
|---|---|
| Different host port | `-p 8097:8096` (change only the left number) |
| Hardware transcoding on Intel/AMD | add `--device /dev/dri:/dev/dri` |
| Where its data lives | the `/data` volume; back that up and you have backed up everything |
| Metadata key | set it in the app later, or pass `-e TMDB_API_KEY=...` |

**Updating:** `docker compose pull && docker compose up -d`, or
`docker pull ghcr.io/aurora-media/aurora:latest` and recreate the container. Updating
from inside the app is deliberately disabled in a container — there, the image *is*
the unit of deployment.

**Unraid / CasaOS / Synology:** any of them will take the compose file above. Point the
media volume at your share, keep `:ro`, and use `http://<host>:8096/icon.svg` if it
asks for an icon.

---

## Install without an installer (any platform)

The archives contain one static binary and the web app. Nothing is installed, nothing
is registered, and deleting the folder removes every trace. This is the right choice
for a NAS, a USB stick, or anyone who would rather run things by hand.

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

FFmpeg has to be on your `PATH`, **or** simply drop `ffmpeg.exe` and `ffprobe.exe` into
the same folder as `aurora.exe` (or an `ffmpeg\` subfolder) — Aurora looks there too.

| Flag | Default | What it does |
|---|---|---|
| `-addr` | `:8096` | address and port to listen on |
| `-web` | `./web` | the web app that ships in the archive |
| `-data` | `./data` | database, transcode cache and backups |
| `-tls-cert`, `-tls-key` | none | serve HTTPS directly instead of plain HTTP |

The same three settings can come from `aurora.conf` beside the binary, or from the
environment — useful when the command line is not yours to change:

```ini
# aurora.conf
addr = :8096
data = /var/lib/aurora
web  = /usr/share/aurora/web
```

```bash
AURORA_ADDR=:8096 AURORA_DATA=/srv/aurora ./aurora
```

A flag beats the environment, which beats the file.

Which archive to take:

| File | For |
|---|---|
| `linux-x86_64` | ordinary Linux PCs and servers |
| `linux-arm64` | Raspberry Pi 4/5, ARM NAS boxes |
| `macos-applesilicon` | M1 and later |
| `macos-intel` | Intel Macs |
| `windows-x86_64` | Windows 10/11 |

### Running it as a service yourself

If you unpacked an archive but still want it to survive a reboot, the unit file the
Linux packages use is in [`aurora.service`](aurora.service). Copy it to
`/etc/systemd/system/`, fix the paths, then:

```bash
sudo systemctl daemon-reload
sudo systemctl enable --now aurora
```

`Restart=always` is not decoration — it is what makes updating from inside the app end
in a running server rather than a stopped one.

---

## First run

1. Open `http://localhost:8096`, or `http://<this-machine-ip>:8096` from another
   device on the same network.
2. Register. **The first account becomes the admin.**
3. **Settings → Add library**, pick a type, and browse to the folder.

The folder picker browses the **server's** disk, not the disk of the device you are
browsing from. If Aurora runs on a Pi, you are looking at the Pi.

| Library type | Expects |
|---|---|
| Movies | `Film (2019)/film.mkv`, or loose files named `Film (2019).mkv` |
| Series / Anime | `Show/Season 1/Show S01E02.mkv`, and most of the other shapes releases use |
| Documentaries | either of the above |
| Music | `Artist/Album (1997)/03 - Song.flac` — see below |

Scanning is immediate; artwork and metadata arrive in the background. Music skips the
scrapers entirely.

---

## Updating

**Dashboard → Updates.**

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
own player handled it, **Settings → Video → External player** hands VLC or mpv the
original file: they decode DTS, TrueHD, HDR and PGS subtitles that no browser will
touch, and the server does nothing but serve bytes.

### …watch something with someone who does not have Aurora

**Settings → Video → Watch together → Create a link.** Send them the link. They need no
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

Add a library of type **Music** pointed at your records. The player lives in the app
shell, so a record keeps going while you browse.

**Where the names come from.** There is no music database and nothing is sent
anywhere. Two sources, in this order:

1. **The folders**, which the person who owns the collection arranged on purpose:

   ```
   Music/
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

**Dashboard → Users and profiles.** Each account gets its own watch history, resume points
and lists, and you can limit which libraries it sees at all.

### …change how it looks

**Settings → Appearance.** Presets, plus a CSS box that can override anything on the page.

### …skip intros

**Settings → Show** while an episode is playing. Set the markers by hand, or let Aurora
find them by comparing the audio of two episodes.

### …make a playlist

**Listas** in the sidebar. Create one, then right-click anything in the library and
choose *Add to a list*. A list is a view, not a folder: removing something from
one never touches the file, and lists belong to the account that made them, so two
people in a house do not share them.

### …sync with Trakt

**Dashboard → Plugins → Trakt** takes a Client ID and Client Secret. Aurora
ships neither: an application that embeds its own OAuth secret has published it to
everyone who downloads it. Create a free application at *trakt.tv → Settings → Your
API Apps*, Redirect URI `urn:ietf:wg:oauth:2.0:oob`, and paste the two values.

Each person then links their own account in **Account → Trakt**: Aurora shows a short
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

Paste the TMDB key in **Dashboard → Plugins**, or set `TMDB_API_KEY` (see
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
Check it is running.
*Windows:* `Win+R` → `services.msc` → **Aurora Media Server** should say *Running*; if
it does not, right-click → Start, and read `C:\ProgramData\Aurora\aurora.log`.
*Linux:* `systemctl status aurora` and `journalctl -u aurora -n 50`.
*macOS:* `~/Library/Application Support/Aurora/aurora.log`.
*Docker:* `docker logs aurora`.
If the log says the address is already in use, something else has the port — Jellyfin
uses 8096 too. Change `addr` in `aurora.conf` (Windows), the unit file (Linux) or the
`-p` mapping (Docker).

**Another device on the network cannot reach it**
Use the machine's IP, not `localhost` — `localhost` on a phone means the phone.
On Windows this is nearly always the firewall: rerun the installer and leave *Allow
other devices on my network to reach Aurora* ticked, or add the rule by hand in an
**administrator** PowerShell:

```powershell
netsh advfirewall firewall add rule name="Aurora Media Server" dir=in action=allow protocol=TCP localport=8096
```

Both devices also have to be on the same network: a phone on mobile data, or on a
guest Wi-Fi, cannot see your PC.

**A file will not play**
Almost always FFmpeg missing. `ffmpeg -version` in a terminal should print something.
On Windows you can also drop `ffmpeg.exe` and `ffprobe.exe` next to `aurora.exe` in
`C:\Program Files\Aurora`, or in an `ffmpeg\` folder there, and restart the service.
The player shows what it is doing — *Direct*, *Remux* or *Transcoding* — which is the
first thing worth putting in an issue.

**Playback stutters on a small machine**
*Direct* and *Remux* cost the server almost nothing. *Transcoding* means a full
re-encode, which a Raspberry Pi cannot do in real time for 1080p or 4K. If you see it
often, the device doing the watching probably cannot decode the codec — an external
player usually can.

**The scan found nothing**
The folder picker browses the server's disk. Check the path is the one the *server*
sees, not the one your laptop sees. On Linux, Aurora runs as the `aurora` user, which
has to be able to read the folder: `sudo -u aurora ls /your/media` tells you in one
line. On Windows, a service cannot see mapped network drives — see the note at the end
of the Windows section.

**It cannot play anything on a Raspberry Pi except by transcoding, and that stutters**
A Pi has no video encoder, so a real re-encode is never going to keep up. What it does
brilliantly is *Direct* and *Remux*, which cost nothing. If a specific file always
transcodes, it is the watching device that cannot decode it — an external player, or a
different browser, usually fixes it.

**A subtitle is out of step**
Aurora tried and was not confident enough to move it, which it says in the log. Use the
offset buttons in **Settings → Subtitles**.

---
