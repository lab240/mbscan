# mbscan — Fast Modbus RTU Bus Scanner

A lightweight command-line utility that scans a Modbus RTU serial bus for connected slave devices. Opens the serial port once and probes a range of addresses by sending FC03 (Read Holding Registers) requests.

## Features

- **Fast** — scans 247 addresses in ~2.5 seconds at 115200 baud (10ms timeout)
- **Cross-platform** — Linux (x86_64, aarch64), Windows (x86_64), OpenWrt, Raspberry Pi, Armbian
- **Zero dependencies** — pure C, no libmodbus required
- **Own CRC16** — built-in Modbus CRC16 implementation
- **Configurable** — baud rate, data format, address range, timeout, register count

## Quick Start

### Linux

```bash
# Scan all addresses on /dev/ttyUSB0 (default: 115200-8N1, timeout 100ms)
mbscan -p /dev/ttyUSB0

# Fast scan with 10ms timeout
mbscan -p /dev/ttyUSB0 -o 10

# Scan specific range, read 4 registers
mbscan -p /dev/ttyUSB0 -f 1 -t 30 -c 4

# 9600 baud, 8E1 parity
mbscan -p /dev/ttyS1 -b 9600 -d 8E1
```

### Windows

```cmd
rem Scan COM3 with default settings
mbscan.exe -p COM3

rem Fast scan, 10ms timeout
mbscan.exe -p COM3 -o 10

rem Scan addresses 1-30, read 4 registers, 9600 baud
mbscan.exe -p COM5 -b 9600 -d 8E1 -f 1 -t 30 -c 4

rem High-numbered COM port (above COM9)
mbscan.exe -p COM15
```

## Output Example

### Linux

```
mbscan: scanning /dev/ttyUSB0 115200-8N1, addresses 1-247, timeout 100ms
mbscan: reading 4 register(s) starting at 0

Found slave 125: [0]=125 [1]=1 [2]=830 [3]=794

mbscan: done. Found 1 device(s).
```

### Windows

```
mbscan: scanning COM3 115200-8N1, addresses 1-247, timeout 100ms
mbscan: reading 4 register(s) starting at 0

Found slave 125: [0]=125 [1]=1 [2]=830 [3]=794

mbscan: done. Found 1 device(s).
```

## Usage

```
mbscan -p PORT [options]

Options:
  -p PORT    Serial port (required)
             Linux: /dev/ttyUSB0, /dev/ttyS1
             Windows: COM3, COM10, \\.\COM15
  -b BAUD    Baud rate (default: 115200)
  -d PARAMS  Data format: 8N1, 8E1, 8O1, 7E1, etc. (default: 8N1)
  -f FROM    Start address (default: 1)
  -t TO      End address (default: 247)
  -o MS      Timeout per address in ms (default: 100)
  -r REG     Start register, 0-based (default: 0)
  -c COUNT   Number of registers to read (default: 1)
  -v         Verbose output
  -h         Show help
```

## Prebuilt Binaries

Prebuilt binaries are available on the [Releases](../../releases) page:

| File | Platform | Notes |
|---|---|---|
| `mbscan-linux-x86_64` | Linux x86_64 | Static binary, no dependencies |
| `mbscan-linux-aarch64` | Linux aarch64 (ARM64) | Static binary |
| `mbscan.exe` | Windows x86_64 | Static binary, no dependencies |

## Build from Source

### Linux — native

```bash
cd src
gcc -O2 -Wall -o mbscan mbscan.c
```

Static build (portable, no dependencies):

```bash
gcc -O2 -Wall -static -o mbscan mbscan.c
```

### Linux ARM64 — native on device

On Armbian, Raspberry Pi OS, or any ARM64 Linux with gcc:

```bash
# Install gcc if needed
sudo apt install gcc

cd src
gcc -O2 -Wall -o mbscan mbscan.c
```

### Linux ARM64 — cross-compile from x86_64 host

```bash
sudo apt install gcc-aarch64-linux-gnu
aarch64-linux-gnu-gcc -O2 -Wall -static -o mbscan-linux-aarch64 src/mbscan.c
```

The `-static` flag produces a self-contained binary that works on any aarch64 Linux (Armbian, Raspberry Pi OS, Ubuntu, OpenWrt, etc.) without runtime dependencies.

### Windows — cross-compile from Linux

No need to install anything on Windows:

```bash
sudo apt install gcc-mingw-w64-x86-64
x86_64-w64-mingw32-gcc -O2 -Wall -static -o mbscan.exe src/mbscan.c
```

The resulting `mbscan.exe` runs on any Windows x86_64 machine without dependencies.

### Windows — native with MinGW/MSYS2

Install [MSYS2](https://www.msys2.org/), then:

```bash
pacman -S mingw-w64-x86_64-gcc
gcc -O2 -Wall -o mbscan.exe mbscan.c
```

### Windows — native with MSVC

Open "Developer Command Prompt for VS" and:

```cmd
cl src\mbscan.c /Fe:mbscan.exe /O2
```

### OpenWrt Package

Copy `mbscan` directory into OpenWrt package tree and build:

```bash
cp -r mbscan /path/to/openwrt/package/
cd /path/to/openwrt
echo "CONFIG_PACKAGE_mbscan=y" >> .config
make package/mbscan/compile -j$(nproc)
```

The resulting `.ipk` / `.apk` package will be in `bin/packages/*/base/`.

### Cross-compile for aarch64 via OpenWrt toolchain

```bash
/path/to/openwrt/staging_dir/toolchain-aarch64_generic_gcc-*/bin/aarch64-openwrt-linux-gcc \
  -O2 -Wall -static -o mbscan-linux-aarch64 src/mbscan.c
```

## How It Works

1. Opens the serial port and configures baud rate, parity, data/stop bits
2. For each address in the range:
   - Flushes stale data from the port
   - Sends an 8-byte Modbus RTU FC03 request with CRC16
   - Waits for response with configurable timeout
   - Validates response: CRC, slave address, function code
   - Reports found devices with register values
3. Observes Modbus inter-frame delay (3.5 character times) between requests

## Performance

| Timeout | Full scan (1–247) | Notes |
|---|---|---|
| 10 ms | ~2.5 sec | Minimum, may miss devices (see below) |
| 20 ms | ~5 sec | Minimum recommended |
| 50 ms | ~12 sec | Recommended for most setups |
| 100 ms | ~25 sec | Default, reliable |
| 200 ms | ~50 sec | Long RS-485 lines |

> **⚠️ Note on short timeouts:** With `-o 10` some devices may not be detected — the 10ms window can be too short for a slave to form and send a response, especially on Windows where USB-Serial driver latency is higher than on Linux. The minimum recommended timeout is **20ms**. If a device is not found, try increasing the timeout before checking wiring or address.

## Windows Notes

- COM port names are passed as-is: `-p COM3`. The `\\.\` prefix is added automatically, so high-numbered ports (COM10+) work without special syntax.
- Find your COM port number in Device Manager → Ports (COM & LPT).
- Common USB-Serial adapters (CH341, CP2102, FTDI, PL2303) install their drivers automatically on Windows 10/11.

## Integration

`mbscan` is used as the backend for the **Scan Bus** tab in [luci-app-mbpoll](https://github.com/lab240/luci-app-mbpoll) — a LuCI web interface for Modbus device polling and bus scanning on OpenWrt.

## Hardware

Tested on:

- [NapiLab Napi](https://napiworld.ru) — industrial IoT gateways (RK3308, aarch64, OpenWrt)
- Standard Linux x86_64 with USB-RS485 adapters (CH341, CP2102, FTDI)
- Windows 10/11 x86_64 with USB-RS485 adapters

## License

GPL-2.0