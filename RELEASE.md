# mbscan v1.0.2  — Cross-platform release

**Windows support added.** Single source file now builds natively on Linux, Windows, and cross-compiles for ARM64.

## What's New

- **Windows support** — native Win32 serial API (CreateFile, DCB, ReadFile/WriteFile), no POSIX emulation layer
- **Built-in getopt** for MSVC (no external dependencies on Windows)
- **COM port handling** — `\\.\` prefix added automatically, COM10+ works without special syntax
- **ARM64 builds** — cross-compile with standard `gcc-aarch64-linux-gnu`, no OpenWrt toolchain required

## Downloads

| File | Platform | How to build |
|---|---|---|
| `mbscan-linux-x86_64` | Linux x86_64 | `gcc -O2 -Wall -static -o mbscan-linux-x86_64 mbscan.c` |
| `mbscan-linux-aarch64` | Linux ARM64 | `aarch64-linux-gnu-gcc -O2 -Wall -static -o mbscan-linux-aarch64 mbscan.c` |
| `mbscan.exe` | Windows x86_64 | `x86_64-w64-mingw32-gcc -O2 -Wall -static -o mbscan.exe mbscan.c` |

All binaries are statically linked — no runtime dependencies, just copy and run.

## Usage

```bash
# Linux
mbscan -p /dev/ttyUSB0
mbscan -p /dev/ttyUSB0 -b 9600 -d 8E1 -f 1 -t 30 -c 4

# Windows
mbscan.exe -p COM3
mbscan.exe -p COM5 -b 9600 -d 8E1 -f 1 -t 30 -c 4
```

## Note on timeouts

Default timeout is 100ms per address (~25 sec full scan). For faster scans use `-o 20` (minimum recommended). Timeout `-o 10` may miss devices, especially on Windows due to USB-Serial driver latency.

---

Full documentation: [README](https://github.com/lab240/mbscan#readme)