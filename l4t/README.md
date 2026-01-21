# notes about L4T version & source sync
https://developer.nvidia.com/embedded/jetson-linux-r3644

## Extract Linux for Tegra
```bash
cd l4t

wget https://developer.nvidia.com/downloads/embedded/l4t/r36_release_v4.4/release/Jetson_Linux_r36.4.4_aarch64.tbz2
# L4T build notes and kernel build workflow

This file documents how to extract NVIDIA L4T packages, sync or extract kernel sources, and build the kernel, device trees and modules for Jetson Orin (L4T R36.x). It assumes this repository layout where `l4t/` lives at the repo root.

Prerequisites
- Download the matching NVIDIA archives for your JetPack/L4T version (for example `Jetson_Linux_r36.4.4_aarch64.tbz2` and `Tegra_Linux_Sample-Root-Filesystem_r36.4.4_aarch64.tbz2`) from NVIDIA Developer and place them into the `l4t/` directory.

Extract L4T (example)
```bash
cd l4t
wget https://developer.nvidia.com/downloads/embedded/l4t/r36_release_v4.4/release/Jetson_Linux_r36.4.4_aarch64.tbz2
tar xvf Jetson_Linux_r36.4.4_aarch64.tbz2
```

Sync or extract kernel sources
- To sync official kernel sources using NVIDIA's `source_sync.sh` (recommended when available):

```bash
cd Linux_for_Tegra/source
# sync kernel sources for a specific release tag
./source_sync.sh -t jetson_36.4.4
```

- To manually extract the public source archive:

```bash
cd l4t
wget https://developer.nvidia.com/downloads/embedded/l4t/r36_release_v4.4/sources/public_sources.tbz2
tar xf public_sources.tbz2
cd Linux_for_Tegra/source
tar xf kernel_src.tbz2
tar xf kernel_oot_modules_src.tbz2
tar xf nvidia_kernel_display_driver_source.tbz2
```

Setup the root filesystem (optional)
- Build a sample rootfs (if needed) or extract NVIDIA's sample rootfs archive into `Linux_for_Tegra/rootfs/` and run the pre-requisite scripts:

```bash
cd Linux_for_Tegra/tools/samplefs
sudo ./nv_build_samplefs.sh --abi aarch64 --distro ubuntu --flavor desktop --version jammy

# or extract the provided sample rootfs
cd l4t
wget <Tegra_Linux_Sample-Root-Filesystem...tbz2>
sudo tar xpf Tegra_Linux_Sample-Root-Filesystem_...tbz2 -C Linux_for_Tegra/rootfs/
cd Linux_for_Tegra
sudo ./tools/l4t_flash_prerequisites.sh
sudo ./apply_binaries.sh
```

Environment helper
- Source the repository helper that sets `L4T_ROOT`, `KERNEL_SRC`, `OUT_DIR`, `ARCH` and `CROSS_COMPILE`:

```bash
cd /workspace
source scripts/env.sh
```

Kernel configuration
- Get the running kernel config from a Jetson device (preferred when you want an identical config):

```bash
# on device
zcat /proc/config.gz > /tmp/orin-kernel.config
# copy to host
scp user@orin:/tmp/orin-kernel.config /workspace/out/orin-kernel.config

# prepare out dir and use the device config
mkdir -p "$OUT_DIR/kernel"
cp /workspace/out/orin-kernel.config "$OUT_DIR/kernel/.config"
# create an O-based config (do not run mrproper in the source tree unless you know what it removes)
make -C "$KERNEL_SRC" O="$OUT_DIR/kernel" olddefconfig
```

- Or use the Tegra-provided defconfig:

```bash
mkdir -p "$OUT_DIR/kernel"
make -C "$KERNEL_SRC" O="$OUT_DIR/kernel" tegra_defconfig
```

Build kernel, device trees and modules
- Recommended single-line build (builds Image, dtbs and modules into `OUT_DIR`):

```bash
make -C "$KERNEL_SRC" O="$OUT_DIR/kernel" -j$(nproc) Image dtbs modules
```

- To build only modules:

```bash
make -C "$KERNEL_SRC" O="$OUT_DIR/kernel" -j$(nproc) modules
```

Verify build artifacts

```bash
ls -l "$OUT_DIR/kernel/arch/arm64/boot/Image"
ls -l "$OUT_DIR/kernel/arch/arm64/boot/dts"/*.dtb
ls -l "$OUT_DIR/kernel/lib/modules"
```

Installing modules to a rootfs (staged)
- Install modules into a staged rootfs directory (useful for creating rootfs tarballs or copying to an SD card):

```bash
# create a staging rootfs
mkdir -p "$OUT_DIR/rootfs"
make -C "$KERNEL_SRC" O="$OUT_DIR/kernel" INSTALL_MOD_PATH="$OUT_DIR/rootfs" modules_install

# then package or copy $OUT_DIR/rootfs/lib/modules to the target
```

Deploying kernel and DTBs to the Jetson
- Example using `scp` (adjust paths and user@host):

```bash
scp "$OUT_DIR/kernel/arch/arm64/boot/Image" user@jetson:/boot/Image
scp "$OUT_DIR/kernel/arch/arm64/boot/dts"/*your-board*.dtb user@jetson:/boot/

# copy modules (either scp the staged rootfs modules or install on device)
scp -r "$OUT_DIR/rootfs/lib/modules/" user@jetson:/lib/modules/
ssh user@jetson 'sudo depmod -a'
```

Notes & tips
- Use `O=$OUT_DIR` to keep build artifacts out of the source directory.
- Avoid running `make mrproper` in the source tree unless you understand it will remove extra files; prefer using `O=` builds.
- If you cross-compile on the host, ensure `CROSS_COMPILE` is correct and the cross-toolchain is installed.
- Kernel build can be memory- and disk-intensive; use `-j` appropriate for your host.
