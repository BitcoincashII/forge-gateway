# Forge Gateway: release notes

Each section is the release page text for that version. A `v<version>` tag in this repository
builds the release, from the forge-solo commit in `FORGE_SOLO_COMMIT`.

## 1.1.1

Forge Gateway now credits a mistyped address as a worker name in every form, and an address with a
worker name straight after it to that address; its status page and tray say only what is so. It
is built from Forge Solo 1.0.15's source.

- **A mistyped address is a worker name, with or without its prefix.** A username that is a BCH2
  address without the `bitcoincashii:` prefix, or with the `bitcoinii:` one some miners send, is
  now checked as one with the prefix is. Before, a mistyped one was taken as an address: the
  gateway sent that miner's shares to the pool under it, the pool refused every one, and the work
  was credited to no one. Now it is a worker name, credited to the payout address, as a mistyped
  address with the prefix already was. Keep the `bitcoincashii:` prefix all the same.
- **An address with a worker name straight after it** (`bitcoincashii:q…rig1`, with no dot) is now
  credited to that address, with `rig1` as the worker name. 1.1.0 credited it to the payout
  address, with the whole username as the worker name. Without the prefix, such a username was
  already credited to its address.
- **What a block pays:** the status page said every block pays Forge Pool's TIDES split. A block
  found on work the pool registered does; one found while mining solo pays your payout address in
  full, and the page now says so.
- **When the pool refuses:** a pool that answers but will not take your node's work was reported
  as one that cannot be reached. The status page now says Forge Pool cannot be reached, or will not
  take your node's work; the tray says Forge Pool is unavailable; and the Pool only note in
  Settings covers both.
- **"Cannot reach your node" for 10 seconds:** about 2 minutes after Forge Gateway started, the
  status page and the tray said for 10 seconds that it could not reach your node, and the log had
  a warning ending in `EOF`. They no longer do: the node closes a connection left unused for 30
  seconds, and Forge Gateway now closes its own after 20, before the node does.
- **Miners on other devices:** with no shares yet, the status page says whom a username is
  credited to, and that on Windows, miners on other devices can connect only while the PC's network
  profile is Private or Domain. Setup's Ready page says the same before it adds the firewall rule.
- **The log:** the payout address's public key hash no longer goes straight to the console (and so,
  on Windows, into `forge-gateway.log`) at every start and settings change. It is in the gateway's
  own log, at debug level.
- **Built with Go 1.27.2** instead of 1.26.8: it has security fixes to Go's crypto/tls,
  html/template, net/http, net/textproto and os packages. On Windows, `forge-gateway.exe` and the
  tray app are built with golang.org/x/sys 0.49.0 instead of 0.48.0.

## 1.1.0

Forge Gateway for Windows now installs and runs like Forge Solo for Windows: an installer, an icon
in the taskbar, and its settings in the status page instead of a JSON file.

- **Windows installer:** `ForgeGateway-Setup-1.1.0.exe` installs Forge Gateway for your Windows
  account. One permission prompt (Windows Command Processor) adds a firewall rule for miners on
  port 3333, on private and domain networks, for Forge Gateway alone. An option starts it when you
  sign in. Updates close it cleanly first; uninstalling asks whether to keep your data folder.
- **Tray icon:** open the status page, copy the settings password, restart or quit Forge Gateway.
  Its tooltip says what Forge Gateway is doing, or what is wrong. If Forge Solo already uses port
  3333 on the PC, it says so.
- **Settings in the status page:** your node's address and login (or its cookie file), your payout
  address, the coinbase tag and pool only, saved with the settings password, in effect at once:
  your miners stay connected and get new work within seconds. A new payout address makes them
  reconnect once, so that miners logged in with a worker name are credited to it. A new install
  opens the page at Settings.
- **A Forge Gateway 1.0.0 service** on the PC: the installer offers to stop it cleanly and remove
  it, and removes the firewall rule 1.0.0's guide had you add, which let any program in on port
  3333.
- **It says what is wrong:** the status page and the tray say when Forge Gateway is not set up, when
  your node cannot be reached, refuses the login or refuses the PC (its `rpcallowip`), when it is
  still syncing, when Forge Pool cannot be reached, and when the PC's clock is off, which makes
  Forge Pool refuse its requests. Started from the tray, a wrong node login no longer stops Forge
  Gateway: fix it in Settings. With a cookie file, Forge Gateway reads the node's new cookie by
  itself when the node restarts.
- **Console and service:** the zip, as before. Started with `SETTINGS_PASSWORD` (16 characters or
  more), the console program has the same Settings. Exit code 3 now means the config or
  `SETTINGS_PASSWORD` is wrong, 4 that a port is taken.
- **A bitcoincashii:p... payout address is refused at start.** Forge Gateway pays only
  bitcoincashii:q... addresses; with a p... address 1.0.0 started but got no work from the node.
- **Signed:** the installer and `forge-gateway.exe`; the tray app inside the installer is not
  signed. If Smart App Control on Windows 11 blocks it, turn Smart App Control off (Windows
  Security → App & browser control → Smart App Control settings). Windows 10 has no Smart App
  Control.

## 1.0.0

The first release of **Forge Gateway**: mine into Forge Pool's TIDES window from your own BCH2
node, without Forge Solo. It is built from Forge Solo 1.0.13's source.

Your node builds every block template, your miners connect to the gateway, and Forge Pool
registers each job and counts your miners' shares. Every block found by a Forge Gateway or by a
Forge Solo in TIDES mode pays everyone with work in the TIDES window, straight from its coinbase:
no pool fee, no pool balance, no minimum payout. It is the same gateway as Forge Solo's TIDES mode,
on its own, for people who run their own node and their own mining setup.

- **Downloads:** Linux (x86_64 and arm64, with a systemd unit) and Windows (x86_64, signed;
  `forge-gateway.exe install` sets it up as a Windows service). Each holds the program, its
  README, an example config and the license. Check them against `SHA256SUMS`.
- **Windows 11:** `forge-gateway.exe` is signed, but not yet by a certificate Windows trusts. If
  Smart App Control blocks it, turn Smart App Control off (Windows Security → App & browser
  control → Smart App Control settings). Windows 10 has no Smart App Control.
- **What you need:** a fully synced Bitcoin Cash II node with RPC enabled, a BCH2 payout address,
  and miners that speak stratum V1.
- **Your miners' usernames:** a username that is a BCH2 address is credited to that address; any
  other username is credited to the gateway's payout address, as a worker name. So one gateway can
  serve several people.
- **Your miners' work counts in full:** each job commits to a share difficulty above the highest
  any of your miners is on (the power of two at or above twice it, never above the network's), and
  the pool credits every share that reaches it at that difficulty, so your miners' work counts in
  full on average.
- **When Forge Pool cannot be reached,** or will not take your node's work (your node still
  catching up after a restart, say), the gateway mines solo on your node (a block found then pays
  your payout address in full), tries the pool again every minute and goes back by itself. With
  `pool_only` set, it turns miners away instead, so they fail over to their backup pool.
- **Status page** at http://127.0.0.1:3090 (and `/api/status`): the pool connection, your node,
  what a block found now would pay you, each worker, and blocks found.
- **The pool connection is HTTPS only.** The pool's answers carry the payout split, so `pool.url`
  must be `https://` (plain `http://` only for a pool on the same machine).
- **As a service:** on Windows, the service starts with Windows, and Windows starts it again after
  a crash; a wrong config, node login or port stops it instead, and running the program in a
  Command Prompt says what is wrong. On Linux, the systemd unit runs the gateway as its own user,
  which can write only to `/var/lib/forge-gateway`: set `key_file` there, as the README says.
- A log file is kept under 20 MB; the older part moves to `<file>.1`.
- `forge-gateway -check` checks the config, logs in to your node and asks Forge Pool for its
  window before you leave it running.

DATUM and TIDES were designed by OCEAN; Forge Gateway is an independent implementation for
Bitcoin Cash II, not affiliated with or endorsed by OCEAN. See LICENSE.
