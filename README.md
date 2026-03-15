# mbscan — Fast Modbus RTU Bus Scanner
 
A lightweight command-line utility that scans a Modbus RTU serial bus for connected slave devices. Opens the serial port once and probes a range of addresses by sending FC03 (Read Holding Registers) requests.
 
## Features
 
- **Fast** — scans 247 addresses in ~2.5 seconds at 115200 baud (10ms timeout)
- **Zero dependencies** — pure POSIX C, no libmodbus required
- **Own CRC16** — built-in Modbus CRC16 implementation
- **Portable** — works on Linux (x86_64, aarch64), OpenWrt, Raspberry Pi, any POSIX system
- **Configurable** — baud rate, data format, address range, timeout, register count
 
## Quick Start
 
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
 
## Output Example
 
```
mbscan: scanning /dev/ttyUSB0 115200-8N1, addresses 1-247, timeout 100ms
mbscan: reading 4 register(s) starting at 0
 
Found slave 125: [0]=125 [1]=1 [2]=830 [3]=794
 
mbscan: done. Found 1 device(s).
```
 
## Usage
 
```
mbscan -p PORT [options]
 
Options:
  -p PORT    Serial port (required, e.g. /dev/ttyUSB0)
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
| `mbscan-linux-aarch64` | Linux aarch64 (ARM64) | Static binary, built via OpenWrt toolchain |
 
## Build from Source
 
### Native (Linux)
 
```bash
cd src
gcc -O2 -Wall -o mbscan mbscan.c
```
 
Or static build:
 
```bash
gcc -O2 -Wall -static -o mbscan mbscan.c
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
 
### Cross-compile for aarch64
 
Using OpenWrt toolchain:
 
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
| 10 ms | ~2.5 sec | Works on short cables |
| 50 ms | ~12 sec | Recommended for most setups |
| 100 ms | ~25 sec | Default, reliable |
| 200 ms | ~50 sec | Long RS-485 lines |
 
## Integration
 
`mbscan` is used as the backend for the **Scan Bus** tab in [luci-app-mbpoll](https://github.com/lab240/luci-app-mbpoll) — a LuCI web interface for Modbus device polling and bus scanning on OpenWrt.
 
## Hardware
 
Tested on:
 
- [NapiLab Napi](https://napiworld.ru) — industrial IoT gateways (RK3308, aarch64, OpenWrt)
- Standard Linux x86_64 with USB-RS485 adapters (CH341, CP2102, FTDI)
 
## License
 
GPL-2.0
