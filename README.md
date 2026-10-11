# Forge Gateway

Mine into **Forge Pool's TIDES window from your own BCH2 node**, without Forge Solo.

Your node builds every block template. Your miners connect to Forge Gateway. Forge Pool registers
each job and credits the shares your miners find that reach the job's share difficulty. Every
block found on a job Forge Pool registered (a DATUM block), by any Forge Gateway or Forge Solo in
TIDES mode, pays its TIDES split **straight from the coinbase**: no pool fee and no payout
threshold; only amounts under 546 satoshis wait for a later block.
If Forge Pool cannot be reached, or will not take your node's work, the gateway keeps your miners
busy mining solo on your node, and a block found then pays only your payout address (with
`pool_only`, it sends them to their backup pool instead). It rejoins by itself when the pool takes
its work again.

Forge Solo's TIDES mode is the same gateway, built in. Use Forge Gateway when you run your own
node and your own mining setup instead.

It is a DATUM-style gateway. DATUM and TIDES were designed by [OCEAN](https://ocean.xyz); see
[Credits](#credits).

## Download

From the [latest release](https://github.com/BitcoincashII/forge-gateway/releases/latest):

| System | File |
|---|---|
| Linux, 64-bit PC or server (x86_64) | `forge-gateway-<version>-linux-amd64.tar.gz` |
| Linux, 64-bit ARM (a Raspberry Pi with a 64-bit OS, ARM servers) | `forge-gateway-<version>-linux-arm64.tar.gz` |
| Windows, 64-bit | `ForgeGateway-Setup-<version>.exe`: installs Forge Gateway with an icon in the taskbar and Settings in its status page |
| Windows, 64-bit, as a service or in a Command Prompt | `forge-gateway-<version>-windows-amd64.zip`, with a signed `forge-gateway.exe` |

The zip and the Linux downloads each hold the program, this README, the license and
`forge-gateway.example.json`; the Linux ones also hold `forge-gateway.service`. Check your
download against `SHA256SUMS`.

There is no Mac download: on a Mac, [build it yourself](#on-a-mac-build-it-yourself).

## What you need

- A fully synced **Bitcoin Cash II node** on mainnet with RPC enabled.
- A **BCH2 address** for your payouts (`bitcoincashii:q…`).
- Miners that speak stratum V1 (any SHA-256 ASIC, Bitaxe, NerdQAxe, a rental proxy…).

In your node's config file, enable RPC for the gateway (use your own long random password):

```
server=1
rpcuser=forgegateway
rpcpassword=CHANGE-THIS-TO-A-LONG-RANDOM-PASSWORD
rpcbind=127.0.0.1
rpcallowip=127.0.0.1
```

Restart the node after changing it. The gateway can use the node's `.cookie` file instead
(`rpc_cookie_file`). A node writes a new cookie every time it restarts; Forge Gateway reads the new
one by itself.

## Windows: install

Forge Gateway for Windows needs 64-bit Windows (Windows 10 or 11 on an x64 PC, or Windows 11 on
ARM) and what is in [What you need](#what-you-need): a synced BCH2 node with RPC enabled, on this
PC or on another one on your network.

1. Download `ForgeGateway-Setup-<version>.exe` from the
   [latest release](https://github.com/BitcoincashII/forge-gateway/releases/latest) and run it.
   When SmartScreen says "Windows protected your PC", choose **More info**, then **Run anyway**.
   If Smart App Control on Windows 11 blocks it, turn Smart App Control off: **Windows Security →
   App & browser control → Smart App Control settings**. Windows 10 has no Smart App Control.
2. Windows asks once whether Windows Command Processor may make changes to your device: choose
   **Yes**. That adds one firewall rule, for port 3333 on private and domain networks, that lets
   in `forge-gateway.exe` only. For miners on other devices, set your network to **Private**: in
   Windows Settings, under **Network & internet**, open your connection's properties and set its
   network profile to Private. Windows 11 makes new networks Public, and neither the tray nor the
   status page can tell when that keeps your miners out.
3. Forge Gateway starts, puts its icon in the notification area of the taskbar, and opens its
   status page at **Settings**. Enter your node's RPC address and login (its RPC user and
   password, or the full path of its `.cookie` file) and your BCH2 payout address. To save,
   right-click the Forge Gateway icon (if it is not there, click the **^** arrow in the
   notification area first), choose **Copy Settings Password**, paste it into the password box and
   choose **Save settings**. Forge Gateway mines as soon as the settings are saved: nothing needs
   restarting. A payout address saved later makes your miners reconnect once, so that those
   logged in with a worker name are credited to the new address.
4. Point your miners at `stratum+tcp://<PC-IP>:3333`, where `<PC-IP>` is the PC's address on your
   network (see [Your miners](#your-miners)).

Right-click the icon for **Open Status Page**, **Copy Settings Password**, **Restart Forge
Gateway**, **Open Data Folder** and **Quit Forge Gateway**. Its tooltip says what Forge Gateway is
doing: not set up yet, your node cannot be reached, refuses the login or refuses this PC, your
node is still syncing, Forge Pool is unavailable, this PC's clock is off, or mining into the
TIDES window.

Forge Gateway keeps its files in `%APPDATA%\ForgeGateway`: `forge-gateway.json` (the settings),
`forge-gateway.key` (this gateway's identity at Forge Pool), `secrets.env` (the settings password)
and its logs, `launcher.log` and `forge-gateway.log`. To keep the identity of a gateway you ran
before, quit Forge Gateway (right-click its icon, **Quit Forge Gateway**), copy your
`forge-gateway.key` over the one in that folder, and start Forge Gateway again. Enter your node and
payout address in Settings rather than copying an old `forge-gateway.json`: a copied file with
`"status": {"listen": "off"}` or a `log_file` keeps the status page or the log from working with
the tray.

- **Start with Windows:** tick **Start Forge Gateway when I sign in** in the installer. Run the
  installer again to change it.
- **Update:** run the new installer. It closes Forge Gateway cleanly first.
- **Uninstall:** Windows Settings, **Apps**. It asks whether to delete the data folder; **No**
  keeps it.
- **Forge Solo on the same PC:** both use port 3333, so run one of the two. Forge Solo's TIDES mode
  is the same gateway, built in. The tray says when Forge Solo holds the port.
- **A Forge Gateway 1.0.0 service on the same PC:** the installer says so and offers to stop and
  remove it. Choose **Yes**: Windows asks once, for the firewall rule and the service together.
  Its config and key stay where you put them (`C:\ForgeGateway` in 1.0.0's guide): enter the same
  node and payout address in Settings, and copy the key as above to keep the gateway's identity.
  The installer also deletes the rule named "Forge Gateway" that 1.0.0's guide added, which let any
  program in on port 3333.
- **A node on another computer:** in the node's config file add `rpcbind=<the node's address on
  your network>` and `rpcallowip=<this PC's address>` beside the `127.0.0.1` lines, restart the
  node, and enter `http://<the node's address>:8342` as the RPC address in Settings.
- **"Your node refuses this computer":** the node answered HTTP 403, whatever the login: it lets
  RPC in only from the addresses in its `rpcallowip` lines, and with none of them only at an address
  given as a number. Add this PC's address as above, or for a node on this PC enter
  `http://127.0.0.1:8342`, not `localhost`. Typing the password again does not help.
- **"This PC's clock is ... off":** Forge Pool refuses requests whose time is more than 2 minutes off
  its own, so Forge Gateway mines solo (with Pool only on, turns miners away) until the clock is
  right. Turn on **Set time automatically** in Windows Settings, **Time & language**; Forge
  Gateway goes back to the pool by itself within a minute.

## Set up

1. Copy `forge-gateway.example.json` to `forge-gateway.json` and fill in at least
   `node.rpc_user`, `node.rpc_password` and `mining.payout_address`.
2. Check everything before you leave it running:

   ```
   forge-gateway -check -config forge-gateway.json
   ```

   It checks the config, logs in to your node, and asks Forge Pool for its TIDES window. It also
   creates `forge-gateway.key` next to the config: this gateway's identity at the pool.
3. Run it (`forge-gateway -config forge-gateway.json`), or install it as a service, below.
4. Point your miners at `stratum+tcp://<this machine's LAN address>:3333`.

### Linux: run as a service

The service runs as its own user, which can write only to `/var/lib/forge-gateway`. So first set
`"key_file": "/var/lib/forge-gateway/forge-gateway.key"` in `forge-gateway.json`: the service
creates its identity at the pool there at its first start (the key file `-check` made next to
your own copy of the config is not the service's). Then:

```
sudo useradd --system --home /var/lib/forge-gateway --shell /usr/sbin/nologin forge-gateway
sudo install -m 755 forge-gateway /usr/local/bin/
sudo install -d -m 750 -o root -g forge-gateway /etc/forge-gateway
sudo install -m 640 -o root -g forge-gateway forge-gateway.json /etc/forge-gateway/
sudo install -m 644 forge-gateway.service /etc/systemd/system/
sudo systemctl daemon-reload
sudo systemctl enable --now forge-gateway
journalctl -u forge-gateway -f
```

The service cannot read files under `/home`, so give it `rpc_user` and `rpc_password` rather
than a node's `.cookie` file there. To check its own setup once it has started:
`sudo -u forge-gateway forge-gateway -check -config /etc/forge-gateway/forge-gateway.json`

### Windows: run as a service

For a service, or to run it in a Command Prompt, use the zip instead of the installer. Use one or
the other: the installer offers to remove a ForgeGateway service it finds, and a service and the
installed Forge Gateway cannot both have port 3333.

Put `forge-gateway.exe` and `forge-gateway.json` in a folder of their own, for example
`C:\ForgeGateway`. From an **Administrator** Command Prompt:

```
cd C:\ForgeGateway
forge-gateway.exe -check -config C:\ForgeGateway\forge-gateway.json
forge-gateway.exe install -config C:\ForgeGateway\forge-gateway.json
```

The service starts now and with Windows, and Windows starts it again if it crashes. If it stops
instead, the config, the node login or a port is wrong: run
`forge-gateway.exe -config C:\ForgeGateway\forge-gateway.json` in the Command Prompt and it says
what. Its log is `C:\ForgeGateway\forge-gateway.log` (or the `log_file` you set). To remove the
service: `forge-gateway.exe uninstall`. You can also just run that command in a Command Prompt
window instead of installing the service.

`forge-gateway.exe` is signed, but not yet by a certificate Windows trusts. If Smart App Control
on Windows 11 blocks it, turn Smart App Control off: **Windows Security → App & browser control →
Smart App Control settings**. Windows 10 has no Smart App Control.

If your miners are on other machines, allow the stratum port through Windows Firewall for your
local network, from the same Administrator prompt:

```
netsh advfirewall firewall add rule name="Forge Gateway" dir=in action=allow protocol=TCP localport=3333 profile=private,domain
```

## On a Mac (build it yourself)

There is no Mac download. On a Mac you build Forge Gateway from its source and sign it yourself,
ad hoc, as Bitcoin Cash II's own Mac app is signed. You need macOS 13 Ventura or later, on Apple
silicon or Intel, and what is in [What you need](#what-you-need). These steps put Forge Gateway in
`~/ForgeGateway` and its source in `~/forge-solo`. Type the commands in Terminal.

### Tools

Install Apple's command line tools, which bring `git`. If they are installed already, it says so.

```
xcode-select --install
```

Then install Go: the `.pkg` from [go.dev/dl](https://go.dev/dl/) (Apple macOS, ARM64 for Apple
silicon, x86-64 for an Intel Mac), or `brew install go` if you use Homebrew. Open a new Terminal
window afterwards. Any Go 1.21 or newer will do: the `GOTOOLCHAIN` line below has Go build with the
version the release is built with, which the `toolchain` line of forge-solo's `go.mod` names
(go1.27.2 for 1.1.1), and Go downloads that version by itself if it is not the one you installed.

### Build and sign

`V` is the release to build: `FORGE_SOLO_COMMIT` at its tag names the forge-solo commit it is
built from, and the `go build` line is the release workflow's.

<!-- mac-build:start -->
```
V=1.1.1
git clone https://github.com/BitcoincashII/forge-solo ~/forge-solo
cd ~/forge-solo
git checkout "$(curl -fsSL https://raw.githubusercontent.com/BitcoincashII/forge-gateway/v$V/FORGE_SOLO_COMMIT)"
export GOTOOLCHAIN="$(sed -n 's/^toolchain //p' go.mod)"
CGO_ENABLED=0 go build -trimpath -ldflags "-s -w -X main.version=$V" -o forge-gateway ./cmd/forge-gateway
codesign --force --sign - --identifier org.bitcoincashii.forge-gateway forge-gateway
codesign --verify --strict --verbose=2 forge-gateway
./forge-gateway -version
mkdir -p ~/ForgeGateway
rm -f ~/ForgeGateway/forge-gateway
cp forge-gateway cmd/forge-gateway/forge-gateway.example.json ~/ForgeGateway/
```
<!-- mac-build:end -->

`codesign --sign -` signs ad hoc: with no certificate, the signature seals the program's exact
bytes. Go already signs a program for Apple silicon that way, under the name `a.out`, and does not
sign one for an Intel Mac at all. `codesign` signs both the same way under Forge Gateway's own
name, `org.bitcoincashii.forge-gateway`, as Bitcoin Cash II's Mac app is signed under
`org.bitcoincashII.BitcoinCashII-Qt`. Run it again after anything changes the file. The check
prints `forge-gateway: valid on disk` and `forge-gateway: satisfies its Designated Requirement`,
and `-version` prints `forge-gateway 1.1.1`.

A program you build yourself is not quarantined as a download is, so Gatekeeper lets it run
without Apple's notarization. The `rm` matters when you update: macOS can kill a program copied
over one that has run, because it keeps what it knew of the old one's signature.

### Your BCH2 node

Bitcoin Cash II's Mac download is its BitcoinCashII-Qt app. Its `bitcoincashIId` you build
yourself, as the [macOS steps in BCH2's README](https://github.com/BitcoincashII/bitcoincashII-core/blob/v27.0.2/README.md#macos)
say. Both keep their files in `~/Library/Application Support/Bitcoin Cash II` unless you chose
another folder (BCH2's README says `BitcoinCashII`; version 27.0.2 uses `Bitcoin Cash II`). Turn
its RPC on in its config file there, `bitcoincashII.conf`:

<!-- mac-build:start -->
```
mkdir -p ~/Library/Application\ Support/Bitcoin\ Cash\ II
echo server=1 >> ~/Library/Application\ Support/Bitcoin\ Cash\ II/bitcoincashII.conf
```
<!-- mac-build:end -->

Then quit the node and start it again. The app answers RPC only with `server=1`; `bitcoincashIId`
does without it. Unless its config sets `rpcpassword`, the node writes a new login to the `.cookie`
file in that folder each time it starts, and deletes it when it stops. Forge Gateway reads it from
there.

### Configure

```
cd ~/ForgeGateway
cp forge-gateway.example.json forge-gateway.json
echo "$HOME/Library/Application Support/Bitcoin Cash II/.cookie"
nano forge-gateway.json
```

Set `node.rpc_user` and `node.rpc_password` to `""`, `node.rpc_cookie_file` to the path `echo`
printed (written out in full: the config does not expand `~`), and `mining.payout_address` to your
`bitcoincashii:q…` address. With `rpcuser` and `rpcpassword` in the node's config, set those two
instead and leave the cookie file `""`. Save with Control-O and Return, and quit with Control-X.
With the cookie, the node's part reads:

```
  "node": {
    "rpc_url": "http://127.0.0.1:8342",
    "rpc_user": "",
    "rpc_password": "",
    "rpc_cookie_file": "/Users/you/Library/Application Support/Bitcoin Cash II/.cookie"
  },
```

### Check and run

```
./forge-gateway -check -config forge-gateway.json
./forge-gateway -config forge-gateway.json
```

`-check` checks the config, logs in to your node and asks Forge Pool for its TIDES window. It also
creates `forge-gateway.key` in `~/ForgeGateway`: this gateway's identity at the pool. With the node
not running, or running without `server=1`, it says
`✗ node login: node.rpc_cookie_file: open ...: no such file or directory` (there is no cookie), or
with a user and password, `✗ node http://127.0.0.1:8342: ... connection refused`.

The second line runs Forge Gateway in this Terminal window until you press Control-C; it then
closes your miners' connections and sends Forge Pool the shares it still holds. Its status page is
`http://127.0.0.1:3090/`. Point your miners at `stratum+tcp://<the Mac's address>:3333`: the
address is under **Details** of your connection in System Settings, **Network**. A Mac that sleeps
stops your miners; `sudo pmset -c sleep 0` keeps it awake while it is on power.

If the macOS firewall is on (System Settings, **Network**, **Firewall**), macOS asks the first time
Forge Gateway waits for miners: *Do you want the application “forge-gateway” to accept incoming
network connections?* Choose **Allow**; with **Deny**, miners on other devices cannot connect. To
change it later, see the list under **Options** there. The firewall knows a program by its
signature, and an ad-hoc signature fits only the one file it was made for: after you build a new
version, macOS can ask again.

### Start at login

Stop Forge Gateway in Terminal first (Control-C): the one launchd starts needs port 3333. This
installs forge-solo's LaunchAgent for it with your home folder in its paths (launchd expands
neither `~` nor `$HOME`), and checks it:

<!-- mac-build:start -->
```
mkdir -p ~/Library/LaunchAgents ~/Library/Logs/ForgeGateway
curl -fsSL https://raw.githubusercontent.com/BitcoincashII/forge-solo/main/cmd/forge-gateway/org.bitcoincashii.forge-gateway.plist | sed "s|/Users/YOU|$HOME|g" > ~/Library/LaunchAgents/org.bitcoincashii.forge-gateway.plist
plutil -lint ~/Library/LaunchAgents/org.bitcoincashii.forge-gateway.plist
```
<!-- mac-build:end -->

`plutil` says `OK`. Then start it:

```
launchctl bootstrap gui/$(id -u) ~/Library/LaunchAgents/org.bitcoincashii.forge-gateway.plist
```

Forge Gateway runs now, at every login, and again whenever it stops. Its log is
`~/Library/Logs/ForgeGateway/forge-gateway.log`; to follow it:
`tail -f ~/Library/Logs/ForgeGateway/forge-gateway.log`. macOS says a background item was added:
leave it on under **Allow in the Background** in System Settings, **General**, **Login Items**
(**Login Items & Extensions** on macOS 15 and later).

- **Restart** it, after you change `forge-gateway.json`:
  `launchctl kill SIGTERM gui/$(id -u)/org.bitcoincashii.forge-gateway`. It stops cleanly, as
  on Control-C, and launchd starts it again within 10 seconds.
- **Stop** it: `launchctl bootout gui/$(id -u)/org.bitcoincashii.forge-gateway`. It starts again
  at your next login; to stop that too, also
  `rm ~/Library/LaunchAgents/org.bitcoincashii.forge-gateway.plist`.
- **If the status page does not answer**, the config, the node login or a port is wrong, and the
  last lines of the log say which. launchd starts it again every 10 seconds, so a gateway started
  at login before the node has written its cookie runs once the node has.
- **A node on another computer:** macOS 15 and later keep a program started at login this way from
  other devices on your network until it is allowed in System Settings, **Privacy & Security**,
  **Local Network**. With such a node, run Forge Gateway in Terminal instead, where that does not
  apply. A node on this Mac, Forge Pool and your miners' connections are not affected.

### Update

Delete the source (`rm -rf ~/forge-solo`), run [Build and sign](#build-and-sign) again with the
new version in its first line, and restart Forge Gateway: in Terminal, Control-C and run it again;
started at login, as under **Restart** above. Your `forge-gateway.json` and `forge-gateway.key`
stay as they are.

## Your miners

| Setting | Value |
|---|---|
| Pool URL | `stratum+tcp://<gateway machine>:3333` |
| Username | your BCH2 address, optionally with `.workername`; or just a worker name |
| Password | anything (`x`) |

A username that is a BCH2 address (`bitcoincashii:q…`), with or without a worker name after it
(`.workername`, or the name straight after the address with no dot), is credited to **that
address** at the pool; any other username, a legacy `1…` address included, is credited to the
gateway's `payout_address`, with the username as the worker name. So one gateway can serve several
people, each paid to their own address by DATUM blocks; a block found while the gateway mines solo
pays only its `payout_address` (see [How you are paid](#how-you-are-paid)). An address with a typo
in it is not an address, with or without its `bitcoincashii:` prefix, so it counts as a worker
name: the gateway's log says, at each miner's login, which address it is credited to.

Forge Gateway 1.1.0 and older differ in two ways. They credit `bitcoincashii:q…` with a worker name
straight after it to `payout_address`, with the whole username as the worker name. And they take a
mistyped address without its prefix as an address: the pool refuses that miner's shares, so its
work is credited to no one. Keep the `bitcoincashii:` prefix all the same.

## Status page

Open `http://127.0.0.1:3090/` on the gateway machine: the pool connection, your node, what a block
found now would pay your payout address, each worker's hashrate and the address it is credited
to, and blocks found. The same data is at `/api/status` as JSON. It has no login, so it listens on
this machine only; set `status.listen` to `0.0.0.0:3090` only on a network you trust.

It has a Settings panel for the node, the payout address, the coinbase tag and pool only. It works
only in a page opened on the gateway machine itself, at `127.0.0.1` or `localhost`. Forge Gateway
started with `SETTINGS_PASSWORD` (16 characters or more; the Windows tray app does this) saves them
with that password, and they take effect at once. Started without it, the panel only shows them:
edit `forge-gateway.json` and restart.

To start on a new block the moment your node has it (instead of within a second), add to the
node's config: `blocknotify=curl -s -X POST http://127.0.0.1:3090/notify`

## How you are paid

Forge Pool's TIDES window holds up to 8 × the network difficulty (of the block being mined) of the
most recent credited share work, from all gateways together: Forge Gateways and Forge Solo
installs in TIDES mode. Your part of it is your work in it divided by all the work in it.

A block found on a job Forge Pool registered (a DATUM block), by your gateway or anyone else's,
pays each address in the window its part of the block's value (the subsidy plus the fees of the
transactions in it), plus any amount carried for it, directly in that block's coinbase. There is
no pool fee. Amounts under 546 satoshis get no output: they are carried forward and paid in a
later DATUM block in which the address has work in the window and is due at least 546 satoshis.
Blocks found by miners on the pool's own stratum ports are paid by the pool's usual payouts, not
from this window.

The payout is an output of the block itself, and can be spent 100 blocks after it (the network's
coinbase maturity rule). Forge Pool counts a DATUM block as confirmed at 2 confirmations and checks
it until its coinbase can be spent: one that drops off the chain is marked orphaned, pays nothing,
and its effect on carried amounts is undone. Forge Pool's
[TIDES guide](https://pool.bch2.org/tides-guide.html) has the details, and its
[TIDES page](https://pool.bch2.org/tides) shows the window.

Each job's share difficulty is the higher of the pool's own difficulty for your gateway and the one
the job commits to: the power of two at or above twice the highest difficulty your miners work at,
never above the network difficulty. A difficulty a miner sets itself (`d=` in its password) counts
once it has sent a share at it. A miner that connects, or whose difficulty rises, is covered from
the next job, within about 15 seconds. Only the shares that reach the job's share difficulty go to
the pool, which credits each at that difficulty, so each miner's work counts in full on average;
the status page shows both counts.

While Forge Pool cannot be reached, or will not take your node's work (your node is still
catching up after a restart, say), the gateway mines **solo**: a block found then is not a DATUM
block, and pays your `payout_address` the whole reward. A job the pool already took stays in use
while it cannot be refreshed, until it is 45 seconds to about a minute old. The gateway tries the
pool again every minute and moves your miners back as soon as the pool takes its work. With
`"pool_only": true` (**Pool only** in Settings) it turns miners away instead, so they fail over to
their backup pool; set that on a gateway that serves other people's addresses, because a solo
block pays only `payout_address`.

## Config reference

Every key is optional except `node.rpc_user`/`rpc_password` (or `rpc_cookie_file`) and
`mining.payout_address`. `forge-gateway.example.json` lists every key with its default. A
misspelt key is an error, not silently ignored. Relative paths are relative to the config file.

| Key | Default | Meaning |
|---|---|---|
| `node.rpc_url` | `http://127.0.0.1:8342` | your node's RPC address |
| `node.rpc_user`, `node.rpc_password` | required (or the cookie file) | RPC login |
| `node.rpc_cookie_file` | (none) | read the login from the node's `.cookie` instead |
| `mining.payout_address` | required | credited for worker-name logins; paid solo blocks |
| `mining.coinbase_tag` | `Forge Gateway` | text in your blocks' coinbase: up to 24 printable ASCII characters (a longer one is cut to 24; over 32 is refused) |
| `mining.pool_only` | `false` | turn miners away instead of mining solo while the pool cannot be reached or will not take your node's work |
| `stratum.listen` | `0.0.0.0:3333` | where miners connect |
| `stratum.min_difficulty` | `1024` | lowest share difficulty a miner is given |
| `stratum.max_difficulty` | `1e12` | highest |
| `stratum.target_share_seconds` | `5` | vardiff aims for a share this often per miner |
| `stratum.retarget_seconds` | `10` | how often vardiff may adjust |
| `stratum.max_connections` | `256` | connections in total |
| `stratum.max_connections_per_ip` | half of `max_connections` (`128`) | connections from one address |
| `pool.url` | `https://pool.bch2.org` | Forge Pool. Must be `https://`: the pool's answers carry the payout split (plain `http://` only for a pool on this machine) |
| `pool.key_file` | `forge-gateway.key` | this gateway's identity at the pool, created on first start; keep it |
| `status.listen` | `127.0.0.1:3090` | status page; `off` disables it |
| `log_file` | console (a Windows service: `forge-gateway.log` next to the config) | where the log goes. A log file is kept under 20 MB; the older part moves to `<file>.1` |
| `log_level` | `info` | `debug`, `info`, `warn` or `error` |

The installer's Forge Gateway keeps this file in `%APPDATA%\ForgeGateway`, and Settings writes its
node and mining keys. A `bitcoincashii:p...` payout address is refused: the gateway pays only
`bitcoincashii:q...` addresses. Exit codes: 3 when the config or `SETTINGS_PASSWORD` is wrong, 4
when a listen address is taken, 1 for anything else; `-check` exits 1 on any problem.

## Source

Forge Gateway's source is in [BitcoincashII/forge-solo](https://github.com/BitcoincashII/forge-solo/tree/main/cmd/forge-gateway),
where it shares the stratum and TIDES code with Forge Solo's TIDES mode; its Windows installer and
tray app are in `windows/gateway` there. Each release is built from the forge-solo commit in
[`FORGE_SOLO_COMMIT`](FORGE_SOLO_COMMIT), and its release page says which.

To build it yourself you need Git, curl and Go 1.21 or newer, and Docker for the Windows installer.
forge-solo's `go.mod` names the Go the releases are built with (its `toolchain` line), and an older
Go downloads that one by itself. `V` is the release to build: `FORGE_SOLO_COMMIT` at its tag names
its commit. These are the release workflow's commands (`.github/workflows/release.yml` here);
`-X main.version` sets the version `forge-gateway -version` prints, and `GOARCH=arm64` builds for
64-bit ARM:

```
V=1.1.1
git clone https://github.com/BitcoincashII/forge-solo && cd forge-solo
git checkout "$(curl -fsSL https://raw.githubusercontent.com/BitcoincashII/forge-gateway/v$V/FORGE_SOLO_COMMIT)"
CGO_ENABLED=0 go build -trimpath -ldflags "-s -w -X main.version=$V" -o forge-gateway ./cmd/forge-gateway
CGO_ENABLED=0 GOOS=windows GOARCH=amd64 go build -trimpath -ldflags "-s -w -X main.version=$V" -o forge-gateway.exe ./cmd/forge-gateway
```

The Windows installer is built in the same checkout, from `windows/gateway/bin`: the tray app,
`forge-gateway.exe` and this repository's LICENSE. Inno Setup compiles it in Docker, with no
network, in the image the release workflow pins (`INNOSETUP_IMAGE`). The image runs as uid 1000,
so its folder is made writable for it, as the release does:

```
go install github.com/akavel/rsrc@v0.10.2
(cd windows/gateway/launcher && "$(go env GOPATH)/bin/rsrc" -ico forge-gateway.ico -arch amd64 -o rsrc.syso)
mkdir -p windows/gateway/bin
(cd windows/gateway/launcher && CGO_ENABLED=0 GOOS=windows GOARCH=amd64 go build -trimpath -ldflags "-H=windowsgui -s -w -X main.version=$V" -o ../bin/forge-gateway-tray.exe .)
cp forge-gateway.exe windows/gateway/bin/
curl -fsSL -o windows/gateway/bin/LICENSE.txt https://raw.githubusercontent.com/BitcoincashII/forge-gateway/v$V/LICENSE
chmod a+w windows/gateway
docker run --rm --network none -v "$PWD":/work amake/innosetup:innosetup6@sha256:81713b854eb12278021045dcb57701fe35312030b2dc1d37710184f294a23f81 "/DMyAppVersion=$V" windows/gateway/forge-gateway.iss
```

The installer is `windows/gateway/ForgeGateway-Setup-<version>.exe`. A release signs
`forge-gateway.exe` before it goes in, and the installer after; a build of your own is not signed.
[windows/gateway/README.md](https://github.com/BitcoincashII/forge-solo/blob/main/windows/gateway/README.md)
in forge-solo says more about the tray app and the installer.

## Credits

DATUM (Decentralized Alternative Templates for Universal Mining) and TIDES were designed and
built by OCEAN: <https://ocean.xyz/docs/datum>, <https://ocean.xyz/docs/tides>, and the original
[DATUM Gateway](https://github.com/OCEAN-xyz/datum_gateway) (Copyright (c) 2024-2025 Bitcoin
Ocean, LLC, Jason Hughes, and individual contributors; MIT License). Forge Gateway follows their
design (your node builds the template, and the coinbase pays the pool's miners directly) as an
independent implementation for Bitcoin Cash II and Forge Pool: it contains no DATUM Gateway code
and does not speak OCEAN's DATUM Protocol. This project is not affiliated with or endorsed by
OCEAN. See the credits and notices in [LICENSE](LICENSE).

MIT License: see [LICENSE](LICENSE).
