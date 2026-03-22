# 🔧 Claw-SOS

**Emergency recovery tool for [OpenClaw](https://github.com/openclaw/openclaw) bots.**

When your bot stops responding on Telegram, Discord, or any channel — SSH in, type `sos`, and it handles the rest.

![License](https://img.shields.io/badge/license-MIT-green)
![Platform](https://img.shields.io/badge/platform-Linux%20%7C%20macOS-blue)
![Shell](https://img.shields.io/badge/shell-bash-orange)

---

## Why?

Your OpenClaw bot is down. You SSH into the machine. Now what?

You could try to remember the right `systemctl` commands, check logs, figure out if it's a config issue or a crash... or you could just type:

```bash
sos
```

## Install

**One line:**

```bash
curl -fsSL https://raw.githubusercontent.com/clawsos/claw-sos/main/install.sh | bash
```

Or with wget:

```bash
wget -qO- https://raw.githubusercontent.com/clawsos/claw-sos/main/install.sh | bash
```

**Manual:**

```bash
curl -fsSL https://raw.githubusercontent.com/clawsos/claw-sos/main/sos.sh -o /usr/local/bin/sos
chmod +x /usr/local/bin/sos
```

## Usage

### Interactive Menu (arrow keys)

```bash
sos
```

Opens a menu you can navigate with arrow keys. AUTOFIX is selected by default — just press Enter.

### Quick Commands

```bash
sos auto     # Diagnose + fix automatically (no menu)
sos net      # Network diagnostics (DNS, Telegram API, latency)
sos tg       # Send a test message through Telegram
sos fix      # Same as auto
sos --help   # Show all options
sos --version
```

## What AUTOFIX Does

When you run `sos auto` (or select AUTOFIX from the menu), it runs through this sequence and **stops as soon as the bot is healthy**:

```
Step 0: Diagnose (process, RAM, disk)
Step 0.5: Fix resources if critical (clear caches, free disk)
Step 1: Check network (DNS, internet, Telegram API)
Step 2: Run openclaw doctor --fix
Step 3: Quick health check — maybe it's already fine
Step 4: Graceful restart
Step 5: Force kill + restart
Step 6: Nuclear (kill all + systemd reload + start)
Step 7: Give up — show manual recovery steps
```

If the problem is network (DNS broken, Telegram blocked), it tells you immediately instead of wasting time restarting.

## All Options

| Option | What it does |
|--------|-------------|
| **AUTOFIX** | Diagnose + repair automatically (recommended) |
| Check status | Show if gateway is running, RAM, disk, version |
| Restart | Graceful gateway restart |
| Force kill | Kill hung process + restart |
| Rollback | Undo a bad update (restores config backup) |
| View logs | Last 50 gateway log lines + SOS action log |
| Full diagnostics | Telegram, RAM, disk, sessions, process memory |
| Backup config | Save config + version before risky changes |
| Self-test | Verify SOS itself works |
| Nuclear | Kill everything, reload systemd, start fresh |
| Network check | Test DNS, internet, Telegram/Anthropic/OpenAI APIs |
| Telegram test | Send a real test message to verify delivery |

## Features

- **Arrow-key menu** via whiptail/dialog (falls back to text if unavailable)
- **Auto-escalation** — tries gentle fixes first, escalates only if needed
- **Network-aware** — detects connectivity issues before wasting time restarting
- **Config backup/restore** — automatic before updates, manual via menu
- **Full logging** — every action timestamped in `~/.openclaw/backups/sos.log`
- **Cross-platform** — Linux (systemd) + macOS (launchctl)
- **No dependencies** — pure bash + optional whiptail/dialog for the menu
- **Safe** — no hardcoded IPs, passwords, or personal data
- **Future-proof** — auto-detects service type, paths, and OS

## Platform Support

| Platform | Service Manager | Menu | Status |
|----------|----------------|------|--------|
| Linux (Ubuntu/Debian) | systemd (user + system) | whiptail | ✅ Fully tested |
| Linux (other) | systemd / manual | whiptail/dialog | ✅ Should work |
| macOS | launchctl | dialog (`brew install dialog`) | ✅ Supported |
| Docker | manual (no systemd) | text fallback | ✅ Graceful fallback |
| Windows/WSL | systemd (WSL2) | text fallback | 🔶 Untested |

## Logging

Every SOS session is logged to `~/.openclaw/backups/sos.log`:

```
[2026-03-21 20:34:29 UTC] [INFO] === SOS session started by root from 192.168.1.5 ===
[2026-03-21 20:34:29 UTC] [INFO] OpenClaw version: 2026.3.13, Service: user
[2026-03-21 20:34:29 UTC] [ACTION] User selected option: 10
[2026-03-21 20:34:29 UTC] [AUTOFIX] === Autofix started ===
[2026-03-21 20:34:35 UTC] [AUTOFIX] Already healthy — no action needed
[2026-03-21 20:34:35 UTC] [INFO] === SOS session ended ===
```

Your OpenClaw bot can read this log to understand what happened while it was down.

## Config Backups

SOS keeps rolling backups of your `openclaw.json`:

```bash
sos            # → option 7 (Backup config NOW)
```

Backups are stored in `~/.openclaw/backups/` with timestamps. Only the last 5 are kept.

Rollback restores both the config file AND the OpenClaw version.

## Uninstall

```bash
curl -fsSL https://raw.githubusercontent.com/clawsos/claw-sos/main/uninstall.sh | bash
```

Or manually:

```bash
rm /usr/local/bin/sos
```

Log files and config backups in `~/.openclaw/backups/` are kept.

## FAQ

**Q: Does it work without OpenClaw installed?**
A: It installs, but most features need OpenClaw. The network check and system diagnostics work standalone.

**Q: Will AUTOFIX break anything?**
A: It starts with non-destructive checks and escalates gradually. It won't rollback versions or change config automatically — only restart/kill processes.

**Q: Can I use it with Docker-based OpenClaw?**
A: Yes. It detects when systemd isn't available and falls back to manual process management.

**Q: What about the nuclear option?**
A: Requires typing YES in caps. It kills all OpenClaw processes, reloads the service manager, and starts fresh. Use only if restart and force-kill both failed.

## Contributing

PRs welcome! Especially for:
- Windows/WSL testing
- Additional service managers (OpenRC, runit, s6)
- Translations
- Integration with OpenClaw's built-in health system

## License

MIT — see [LICENSE](LICENSE)

---

Built by the [ClawSOS](https://github.com/clawsos) community.
