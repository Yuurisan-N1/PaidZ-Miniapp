<div align="center"> <img width="100%" alt="header" src="https://capsule-render.vercel.app/api?type=waving&height=210&text=PaidZ%20Bot&fontAlign=50&fontAlignY=36&fontSize=56&desc=Auto%20Ads%20%7C%20Multi-Network%20%7C%20Token%20Cache%20%7C%20Multi-Account&descAlign=50&descAlignY=58"/> <img alt="typing" src="https://readme-typing-svg.demolab.com?font=Inter&size=18&duration=3000&pause=650&center=true&vCenter=true&width=900&lines=Auto+Watch+Ads+%7C+Claim+Coins+from+Multiple+Networks;Live+Ad+Countdown+%7C+Waits+Minimum+Duration+Per+Ad;Token+Caching+%7C+Device+Fingerprint+Persisted+Per+Account;Proxy+Support+%7C+Fallback+to+Direct+on+Failure;Multi-Account+%7C+All+Accounts+Per+Cycle"/> <p>
<img alt="platform" src="https://img.shields.io/badge/Platform-PaidZ%20Miniapp-111111"/>
  <img alt="multi-account" src="https://img.shields.io/badge/Multi--Account-Supported-111111"/>
  <img alt="proxy" src="https://img.shields.io/badge/Proxy-Supported-111111"/>
  <img alt="author" src="https://img.shields.io/badge/by-Yuurisandesu-111111"/>
</p> <p>
  <b>PaidZ Bot</b> is a full automation bot for the PaidZ Telegram Miniapp.

It handles the complete daily cycle: fetching all available ad networks, watching ads from each network in sequence with a live per-ad countdown, and claiming coin rewards all running automatically across multiple accounts with token and device caching, proxy support with automatic fallback, and a live countdown between cycles.Built and distributed by <b>Yuurisandesu</b>.

</p> </div>

---

## Table of Contents

- [Requirements](#requirements )
- [Installation](#installation)
- [Configuration](#configuration)
- [Building from Source](#building-from-source)
- [Running the Bot](#running-the-bot)
- [Features](#features)
- [File Structure](#file-structure)
- [Disclaimer](#disclaimer)

---

## Requirements

No installation needed. The bot is distributed as prebuilt binaries. Download the one for your platform and run it directly.

---

## Installation

**Clone the repository:**

```bash
git clone https://github.com/Yuurisan-N1/PaidZ-Miniapp.git
cd PaidZ-Miniapp
```

Build the binary for your platform first, see [Building from Source](#building-from-source ) below, then place the binary in the same folder as your `data.txt`, `proxy.txt`, and `config.json` before running.

---

## Configuration

### 1. Accounts (data.txt)

Fill `data.txt` with Telegram WebApp `initData` for each account, one per line:

```
user=%7B%22id%22...&hash=abc123
user=%7B%22id%22...&hash=def456
```

> `initData` can be obtained from the browser DevTools when opening PaidZ on Telegram Web.

### 2. Proxy (proxy.txt)

Fill `proxy.txt` with proxies, one per line (optional, leave empty to run without proxy):

```
host:port
host:port:user:pass
http://user:pass@host:port
```

Proxies are assigned to accounts by index in round-robin order. If a proxy fails, the bot automatically falls back to a direct connection for that request.

### 3. Bot Settings (config.json  )

`sleep_seconds` controls how many seconds the bot waits between cycles. If `config.json` is missing, the bot falls back to a default of `3600` seconds.

---

## Building from Source

Build only the assembly file that matches the target operating system and CPU architecture. Run the commands from the folder that contains the assembly file.

### Ubuntu or Linux x86_64

The supplied Linux assembly requires Python 3.12. Install the compiler and Python 3.12 development files:

```bash
sudo apt update
sudo apt install -y gcc software-properties-common
sudo add-apt-repository -y ppa:deadsnakes/ppa
sudo apt update
sudo apt install -y python3.12 python3.12-dev
```

Check the Python linker flags:

```bash
python3.12-config --embed --ldflags
```

The output must contain `-lpython3.12`. Assemble the Linux source:

```bash
gcc -c paidz-linux-amd64.s -o paidz-linux-amd64.o
```

Link the binary:

```bash
gcc paidz-linux-amd64.o -o paidz-linux-amd64 $(python3.12-config --embed --ldflags)
```

### Windows x86_64

Install Visual Studio Build Tools with the **Desktop development with C++** workload and Python 3.12. Open **x64 Native Tools Command Prompt for VS 2022**.

Set the Python installation directory:

```cmd
for /f "delims=" %i in ('py -3.12 -c "import sys; print(sys.prefix)"') do set "PYTHON_ROOT=%i"
```

Assemble the Windows source:

```cmd
ml64 /c paidz-windows-x64.asm /Fo paidz-windows-x64.obj
```

Link the binary:

```cmd
link paidz-windows-x64.obj /OUT:paidz-windows-x64.exe /LIBPATH:"%PYTHON_ROOT%\libs" python312.lib
```

### Android ARM64 with Termux

Check that the device uses ARM64:

```bash
uname -m
```

The expected result is `aarch64` or `arm64`. Install the compiler and Python package:

```bash
pkg update
pkg install clang python
```

Assemble the Android source:

```bash
clang -c paidz-android-arm64.s -o paidz-android-arm64.o
```

Set the Termux Python configuration command:

```bash
PYCFG=$(command -v python3-config || command -v python-config)
```

Link the binary:

```bash
clang paidz-android-arm64.o -o paidz-android-arm64 $($PYCFG --ldflags)
```


The output binary is `paidz-android-arm64`. If the linker reports missing Python symbols or cannot find a Python library, the installed Termux Python package does not provide the development library required by this assembly file.

---

## Running the Bot

After building, make the binary executable on Linux and Android first:

**Linux:**

```bash
chmod +x paidz-linux-amd64
./paidz-linux-amd64
```

**Android (Termux):**

```bash
chmod +x paidz-android-arm64
./paidz-android-arm64
```

**Windows:**

```bash
.\paidz-windows-x64.exe
```

Press `Ctrl+C` at any time to stop the bot cleanly.

---

## Features

### Auto Ads Multi-Network

The bot fetches all available ad networks for the account and checks how many ads remain for each network today. It then watches ads from every network in round-robin order until all daily slots are used. Each ad session goes through a start request, a live countdown matching the server-required minimum duration plus a buffer, and a watch completion request to claim the coin reward. Each successful claim is logged with the network name and result.

### Live Ad Countdown

While waiting for each ad's minimum duration, the bot displays a live `HH:MM:SS` countdown in the terminal. After the countdown finishes, the completion request is sent automatically.

### Token and Device Caching

After the first login, the authentication token and device fingerprint are cached locally. On subsequent cycles, the cached token is validated first. If it is still active, login is skipped entirely and the cached device is reused. If the token has expired, the bot re-authenticates and updates the cache. This keeps the device fingerprint consistent across cycles for each account.

### Proxy with Automatic Fallback

Proxies are loaded from `proxy.txt` and assigned to accounts by position. For every request, the bot tries the assigned proxy first. If the proxy fails, it automatically retries the same request without a proxy before moving on. Proxy credentials are masked in log output.

### Multi Account

All accounts in `data.txt` are processed sequentially within every cycle. Each account runs its full daily cycle before moving to the next.

### Auto Countdown

After all accounts complete a cycle, the bot displays a live countdown until the next cycle starts.

---

## File Structure

```
PaidZ-Miniapp/
├── paidz-linux-amd64.s       # Assembly source for Linux x86_64
├── paidz-android-arm64.s     # Assembly source for Android ARM64
├── paidz-windows-x64.asm     # Assembly source for Windows x86_64
├── config.json               # Sleep duration between cycles
├── data.txt                  # Account initData, one per line
└── proxy.txt                 # Proxy list, one per line (optional)
```

---

## Disclaimer

This tool is built for educational and technical exploration purposes. Use it wisely and at your own responsibility.

---

<div align="center">
<img width="100%" alt="footer" src="https://capsule-render.vercel.app/api?type=waving&height=120&section=footer"/>
</div>
