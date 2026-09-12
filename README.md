# RV1103 Starter Template

Welcome! Whether you're a **DIY enthusiast**, a **hobbyist**, or a **professional**, this repo is for you. It's a minimal, working **C++17 cross-compilation template** for the **Luckfox Pico (RV1103)** board — so you can skip the toolchain boilerplate and start shipping code the moment you clone it.

Everything is wired up for you: compilers, linker, sysroot, build system, IDE support, and a one-command deploy-to-board workflow via USB. Add your own `src/` files and go.

---

## 💡 Blog Post — Coming Soon

I'm writing a deep dive on how this setup came together and *why* it works the way it does. Written with beginners in mind, it walks through:

- Why `make` and CMake are both used, and what `CMakeLists.txt` actually does
- How compilers, linkers, and library files (sysroot) fit together
- What clang/clangd brings to the table
- How to make it all play nicely with VS Code

> **Blog link will appear here once published.**

---

## Features

- **Zero-boilerplate cross-compilation** — one `make` command builds a Luckfox Pico binary
- **Toolchain file** (`cmake/rv1103-toolchain.cmake`) — targets the RV1103 (ARMv7-a, NEON, hard-float) with the official Luckfox GCC toolchain
- **`make dev` workflow** — build, upload, and run on the board over USB RNDIS (scp + ssh)
- **IDE-ready** — clangd + VS Code integration with full cross-compile awareness (IntelliSense, go-to-definition, diagnostics)
- **Simple layout** — `src/`, `include/`, `cmake/`, and three small config files. That's it.

---

## Prerequisites

- A Linux host (the workflow assumes `bash` and `sudo` for network setup)
- A **Luckfox Pico** (RV1103) board with the USB RNDIS interface enabled
- The **Luckfox SDK** / official toolchain installed
- CMake ≥ 3.20, `make`, `scp`, `ssh`

This template expects the toolchain at:

```
~/.luckfox-pico-toolchain/arm-rockchip830-linux-uclibcgnueabihf
```

(Adjust the path in `env.sh` if yours lives somewhere else.)

---

## Quick Start

```bash
# 1. Load the cross-compile environment
source env.sh

# 2. Configure the build (toolchain + compile_commands.json)
make init

# 3. Bring up the USB RNDIS link and assign a host IP
make connect

# 4. Build the firmware
make build

# 5. Upload the binary to the board
make upload

# 6. Run it
make run
```

Or, once the USB link is up, the whole loop in one shot:

```bash
make dev        # = build + upload + run
```

---

## Make Targets

| Target      | What it does                                              |
|-------------|-----------------------------------------------------------|
| `make init`   | Configure CMake with the RV1103 toolchain file            |
| `make build`  | Compile and link the firmware                             |
| `make clean`  | Remove the `build/` directory                             |
| `make rebuild`| `clean` + `init` + `build` in one go                      |
| `make connect`| Configure the USB RNDIS interface + host IP              |
| `make upload` | `scp` the binary to the board                             |
| `make run`    | `ssh` into the board and execute the binary               |
| `make shell`  | Drop into a `ssh` shell on the board                      |
| `make dev`    | `build` + `upload` + `run`                                |

---

## How It Fits Together

```
env.sh                     exports TOOLCHAIN_ROOT, SYSROOT, and
                           the cross compiler/linker binaries
        │
        ▼
cmake/rv1103-toolchain.cmake   tells CMake to use the Arm toolchain,
                           sysroot, and arch flags for RV1103
        │
        ▼
CMakeLists.txt             defines your executable, sources, and
                           headers (C++17, exports compile_commands)
        │
        ▼
build/compile_commands.json    generated for clangd to index the
                           project with correct cross-compile flags
        │
        ▼
clangd (VS Code)               full IntelliSense on your host editor
```

### Key files

| File | Role |
|------|------|
| `env.sh` | Sets the toolchain root, sysroot, and `CC`/`CXX`/`AR`/`LD`/… to the Arm cross tools |
| `cmake/rv1103-toolchain.cmake` | CMake toolchain file: system name, arch flags, sysroot, find-root rules |
| `CMakeLists.txt` | Project definition — sources, includes, C++17, `pthread` link |
| `Makefile` | Human-friendly commands wrapping CMake + USB network + scp/ssh |
| `.vscode/` | VS Code tasks + clangd settings |
| `.clangd` | Points clangd at the compile database |

---

## Configuration

The board IP, host interface, and host IP live at the top of the **`Makefile`**:

```make
TARGET_IP  := 172.32.0.81   # address of the Luckfox Pico
HOST_IFACE := enp0s20f0u7u4 # your USB RNDIS interface name
HOST_IP    := 172.32.0.24/24 # host-side static IP + netmask
```

`TARGET_IP` must match your board's static IP. If your RNDIS interface shows a different name (`ip link`), change `HOST_IFACE` to match.

---

## VS Code + clangd

The repo ships with:

- **`.vscode/tasks.json`** — one-click tasks for every `make` target (`RV1103: Build`, `RV1103: Upload`, …)
- **`.vscode/settings.json`** — clangd configured with `--query-driver` so it can read the cross-compiler's include paths
- **`.clangd`** — points clangd at `build/compile_commands.json` (plus a small diagnostic suppression)

To get cross-compile-aware IntelliSense in VS Code:

1. Install the **clangd** extension (and disable the C/C++ MS extension's IntelliSense)
2. Run `make init` once so `build/compile_commands.json` exists

---

## Directory Layout

```
├── CMakeLists.txt                  build definition for the RVFirmware executable
├── Makefile                        daily-driver commands (init/build/upload/run/...)
├── env.sh                          loads the cross-compilation environment
├── commands.sh                     reference: raw/cmake/network commands
├── .clangd                         clangd compile-database config
├── .gitignore                       ignores build/ and luckfox-pico/
├── cmake/
│   └── rv1103-toolchain.cmake      the RV1103 Arm toolchain file
├── include/
│   └── system_info.hpp             example header
├── src/
│   ├── main.cpp                    example entry point
│   └── system_info.cpp             example implementation
├── .vscode/
│   ├── settings.json               clangd query-driver config
│   └── tasks.json                  VS Code tasks for make targets
└── build/                          generated at build time (git-ignored)
```

---

## Troubleshooting

- **`make upload` / `make run` can't reach the board** — confirm `make connect` succeeded, the board is plugged in via USB RNDIS, and `TARGET_IP` matches the board's configured IP.
- **Interface name differs** — RNDIS interfaces get named based on physical port; list yours with `ip link` and update `HOST_IFACE` in the Makefile.
- **clangd shows no IntelliSense** — run `make init` (or `make rebuild`) so `build/compile_commands.json` exists, then reload the window.
- **Toolchain not found at `/home/.../arm-rockchip830-...`** — this repo expects the toolchain under `~/.luckfox-pico-toolchain/`; change `TOOLCHAIN_ROOT` in `env.sh` if yours differs.

---

## Contributing

Found a bug, or have a nicer twist on the setup? Issues, PRs, and ideas are very welcome — this template is meant to grow with the community around it.

---

## License

MIT — use it freely, fork it, build your projects on it. See `LICENSE` (recommend adding one) if you distribute this as-is.