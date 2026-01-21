# notes about L4T version & source sync
https://developer.nvidia.com/embedded/jetson-linux-r3644

## Extract Linux for Tegra
```bash
cd l4t

wget https://developer.nvidia.com/downloads/embedded/l4t/r36_release_v4.4/release/Jetson_Linux_r36.4.4_aarch64.tbz2
tar xvf Jetson_Linux_r36.4.4_aarch64.tbz2
```

## Setup Source Environment

### To Sync the Kernel Sources with Git
```bash
cd Linux_for_Tegra/source
# just kernel sources use -k and use correct release tag 
./source_sync.sh -t jetson_36.4.4

# if your version is newer checkout the GA release first then the version you have
./source_sync.sh -t jetson_36.4.7

```

### To Manually Download and Expand the Kernel Sources
```bash
wget https://developer.nvidia.com/downloads/embedded/l4t/r36_release_v4.4/sources/public_sources.tbz2
tar xf public_sources.tbz2
cd Linux_for_Tegra/source
tar xf kernel_src.tbz2
tar xf kernel_oot_modules_src.tbz2
tar xf nvidia_kernel_display_driver_source.tbz2
```

## Setup Root File system

### Manually build the root file system
```bash
cd Linux_for_Tegra/tools/samplefs
sudo ./nv_build_samplefs.sh --abi aarch64 --distro ubuntu --flavor desktop --version jammy
```

### Extract the root file system
```bash
cd l4t
wget https://developer.nvidia.com/downloads/embedded/l4t/r36_release_v4.4/release/Tegra_Linux_Sample-Root-Filesystem_r36.4.4_aarch64.tbz2
sudo tar xpf Tegra_Linux_Sample-Root-Filesystem_r36.4.4_aarch64.tbz2 -C Linux_for_Tegra/rootfs/

```
