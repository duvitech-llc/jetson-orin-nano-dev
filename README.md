# Jetson Orin Nano Development Environment

Cross-compilation environment for:
- Linux kernel
- Device tree overlays
- Camera drivers (tegracam / V4L2)

## Target
- Jetson Orin Nano
- JetPack 6 (L4T R36.x)
- Kernel 5.15

## Requirements
- Windows 10/11
- Docker Desktop (WSL2 backend)
- VS Code + Dev Containers extension

## Usage
1. Open repo in VS Code
2. Reopen in Dev Container
3. Sync NVIDIA sources into `l4t/`
4. Build kernel & modules
5. Deploy via SSH

