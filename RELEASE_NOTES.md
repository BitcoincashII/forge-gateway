# Forge Gateway: release notes

Each section is the release page text for that version. A `v<version>` tag in this repository
builds the release, from the forge-solo commit in `FORGE_SOLO_COMMIT`.

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
