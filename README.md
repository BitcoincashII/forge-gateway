# Forge Gateway

Mine into **Forge Pool's TIDES window from your own BCH2 node**, without Forge Solo.

Your node builds every block template. Your miners connect to Forge Gateway. Forge Pool registers
each job and counts the shares your miners find, and every block found through the pool's DATUM
gateways pays its TIDES split **straight from the coinbase**: no pool fee, no pool balance, no
minimum payout.
If Forge Pool cannot be reached, the gateway keeps your miners busy mining solo on your node (or,
with `pool_only`, sends them to their backup pool) and rejoins by itself when the pool is back.

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
   network profile to Private. Windows 11 makes new networks Public.
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
doing: not set up yet, your node cannot be reached or refuses the login, your node is still
syncing, Forge Pool cannot be reached, or mining into the TIDES window.

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

## Your miners

| Setting | Value |
|---|---|
| Pool URL | `stratum+tcp://<gateway machine>:3333` |
| Username | your BCH2 address, optionally with `.workername`; or just a worker name |
| Password | anything (`x`) |

A username that is a BCH2 address is credited to **that address** at the pool; any other username
is credited to the gateway's `payout_address`, with the username as the worker name. So one
gateway can serve several people, each paid to their own address. An address with a typo in it is
not an address, so it counts as a worker name: the gateway's log says, at each miner's login,
which address it is credited to.

## Status page

Open `http://127.0.0.1:3090/` on the gateway machine: the pool connection, your node, what a block
found now would pay you, each worker's hashrate, and blocks found. The same data is at
`/api/status` as JSON. It has no login, so it listens on this machine only; set `status.listen`
to `0.0.0.0:3090` only on a network you trust.

It has a Settings panel for the node, the payout address, the coinbase tag and pool only. Forge
Gateway started with `SETTINGS_PASSWORD` (16 characters or more; the Windows tray app does this)
saves them with that password, and they take effect at once. Started without it, the panel only
shows them: edit `forge-gateway.json` and restart.

To start on a new block the moment your node has it (instead of within a second), add to the
node's config: `blocknotify=curl -s -X POST http://127.0.0.1:3090/notify`

## How you are paid

Forge Pool's TIDES window is the most recent shares from every DATUM gateway: Forge Gateways and
Forge Solo installs in TIDES mode. Every block any of them finds, yours included, pays each
address in the window its share of the reward, directly in that block's coinbase. (Blocks found
by the pool's own stratum miners are paid by the pool's usual payouts, not this window.) The
payout reaches your address when the block is mined, and can be spent after the usual 100-block
coinbase maturity.

Each job commits to a share difficulty above the highest any of your miners is on (the power of
two at or above twice it, never above the network's). Only the shares that reach it go to the
pool, which credits each at that difficulty, so your miners' work counts in full on average; the
status page shows both counts.

While Forge Pool cannot be reached, or will not take your node's work (your node is still
catching up after a restart, say), the gateway mines **solo**: a block found then pays your
`payout_address` the whole reward. It tries the pool again every minute and moves your miners
back as soon as the pool takes its work. With `"pool_only": true` it turns miners away instead,
so they fail over to their backup pool; set that on a gateway that serves other people's
addresses, because a solo block pays only `payout_address`.

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
| `mining.coinbase_tag` | `Forge Gateway` | text in your blocks' coinbase, up to 32 characters |
| `mining.pool_only` | `false` | turn miners away instead of mining solo while the pool is down |
| `stratum.listen` | `0.0.0.0:3333` | where miners connect |
| `stratum.min_difficulty` | `1024` | lowest share difficulty a miner is given |
| `stratum.max_difficulty` | `1e12` | highest |
| `stratum.target_share_seconds` | `5` | vardiff aims for a share this often per miner |
| `stratum.retarget_seconds` | `10` | how often vardiff may adjust |
| `stratum.max_connections` | `256` | connections in total |
| `stratum.max_connections_per_ip` | `128` | connections from one address |
| `pool.url` | `https://pool.bch2.org` | Forge Pool. Must be `https://`: the pool's answers carry the payout split (plain `http://` only for a pool on this machine) |
| `pool.key_file` | `forge-gateway.key` | this gateway's identity at the pool, created on first start; keep it |
| `status.listen` | `127.0.0.1:3090` | status page; `off` disables it |
| `log_file` | console (a Windows service: `forge-gateway.log`) | where the log goes. A log file is kept under 20 MB; the older part moves to `<file>.1` |
| `log_level` | `info` | `debug`, `info`, `warn` or `error` |

The installer's Forge Gateway keeps this file in `%APPDATA%\ForgeGateway`, and Settings writes its
node and mining keys. A `bitcoincashii:p...` payout address is refused: the gateway pays only
`bitcoincashii:q...` addresses. Exit codes: 3 when the config or `SETTINGS_PASSWORD` is wrong, 4
when a listen address is taken, 1 for anything else.

## Source

Forge Gateway's source is in [BitcoincashII/forge-solo](https://github.com/BitcoincashII/forge-solo/tree/main/cmd/forge-gateway),
where it shares the stratum and TIDES code with Forge Solo's TIDES mode; its Windows installer and
tray app are in `windows/gateway` there. Each release is built from the forge-solo commit in
[`FORGE_SOLO_COMMIT`](FORGE_SOLO_COMMIT), and its release page says which. To build it yourself,
with Go (the version in forge-solo's `go.mod`):

```
git clone https://github.com/BitcoincashII/forge-solo && cd forge-solo
git checkout <the commit in FORGE_SOLO_COMMIT>
go build -trimpath -o forge-gateway ./cmd/forge-gateway
GOOS=windows GOARCH=amd64 go build -trimpath -o forge-gateway.exe ./cmd/forge-gateway
```

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
