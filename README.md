# Jetson Orin Nano Development Environment

This repository provides a reproducible development environment for building and testing kernel images, device-tree sources/overlays, and camera drivers for Jetson Orin Nano targets running L4T (JetPack 6 / R36.x).

Goals:
- Build the Linux kernel and device tree for Jetson Orin Nano
- Build kernel modules and camera drivers (V4L2 / tegracam)
- Provide a VS Code Dev Container workflow for reproducible builds

Supported target
- Jetson Orin Nano
- L4T / JetPack 6 (R36.x)
- Upstream-based kernel 5.15 (as used in L4T R36.x)

Prerequisites
- Host OS: Linux (Ubuntu recommended) or Windows with WSL2. The devcontainer workflow expects Docker and VS Code.
- Docker (for Dev Container) or native toolchain installed on host
- Visual Studio Code with the Remote - Containers (Dev Containers) extension
- Sufficient disk space (several GBs) and RAM (8+ GB recommended)

What you need from NVIDIA
NVIDIA does not publish source blobs directly in this repo. To build L4T/kernel artifacts you must download the official NVIDIA packages and place them into `l4t/`.
- Download the matching Jetson Linux and Sample-Root-Filesystem packages for your target (e.g. `Jetson_Linux_r36.4.4_aarch64.tbz2` and `Tegra_Linux_Sample-Root-Filesystem_r36.4.4_aarch64.tbz2`) from NVIDIA's developer site.
- Place those files into the `l4t/` directory at the repo root (they are large; do not commit them to git).

Quickstart (Dev Container)
1. Open this repository in VS Code.
2. Install the Dev Containers extension if prompted.
3. Reopen in Container: click the green >< icon in the lower-left and select "Reopen in Container".
4. Inside the container shell, source the environment helper:

```bash
. ./scripts/env.sh
```

High-level build steps (inside dev container)
- Prepare NVIDIA sources: ensure the downloaded NVIDIA archives live in `l4t/` and run any sync scripts you use to extract them into `l4t/Linux_for_Tegra/`.
- Configure the kernel: edit or copy the defconfig you need (device-specific configs are under `l4t/Linux_for_Tegra/`).
- Build kernel image, device trees and modules. Example:

```bash
# set output/build directory
OUT_DIR=$PWD/out
mkdir -p "$OUT_DIR"

# build kernel image and modules (example targets)
make -C Linux_for_Tegra/source/kernel/kernel-jammy-src O="$OUT_DIR/kernel" -j$(nproc) Image dtbs modules
```

Notes:
- The exact `make` invocation and kernel tree path may differ depending on your L4T version; check the `l4t/Linux_for_Tegra/` tree for the appropriate kernel source and board config files.
- Use `O=$OUT_DIR` to keep build artifacts outside the source tree.

Building device-tree overlays
- Device tree sources (DTS) and overlays should be compiled to DTB/DTSB and placed in the boot directory of the target rootfs. Example:

```bash
make -C Linux_for_Tegra/source/kernel/kernel-jammy-src O="$OUT_DIR/kernel" dtbs
```

Building camera drivers
- If you need tegracam or other camera drivers, build them as kernel modules and install into a rootfs or deploy via `depmod`/`modprobe` on the board. Follow driver README or camera repo instructions when present.

Deploying to the device
- Transfer kernel, modules and DTBs to the Jetson via `scp` or an SD/USB boot rootfs. Typical flow:

```bash
# copy kernel and DTBs (example)
scp "$OUT_DIR/kernel/arch/arm64/boot/Image" user@jetson:/boot/Image
scp "$OUT_DIR/kernel/arch/arm64/boot/dts/*your-board*.dtb" user@jetson:/boot/

# copy modules and install on device
scp -r "$OUT_DIR/kernel/lib/modules/$(uname -r)" user@jetson:/lib/modules/
ssh user@jetson 'sudo depmod -a && sudo update-initramfs -u'
```

Common paths in this repo
- `l4t/` — place NVIDIA archives here (do not commit)
- `l4t/Linux_for_Tegra/` — generated/extracted NVIDIA source tree (board configs, scripts)
- `scripts/env.sh` — helper to set environment variables inside the container

Troubleshooting
- Missing NVIDIA files: ensure the TBZ2 files are downloaded and placed in `l4t/`.
- Build failures: confirm you are using the correct L4T/kernel source for your JetPack version.
- Container issues: ensure Docker has sufficient disk and memory, and that VS Code Dev Containers is up-to-date.

Next steps / TODOs
- Customize kernel config for your board in `l4t/Linux_for_Tegra/`.
- Add build scripts for specific modules you need to develop.

License & Contact
- This repo contains helper scripts and documentation. Check third-party licenses for NVIDIA packages you download separately.
- For questions, open an issue in this repository.

