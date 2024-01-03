# Rust on Linux Workshop

## Overview

Directory structure:

| **Directory** | **Content**                      |
| ---           | ---                              |
| docker        | Dockerfile for build environment |
| initramfs     | Created on build, minimal rootfs |
| linux         | The Linux kernel                 |
| linux-config  | Minimalistic Linux build configs |
| scripts       | Scripts for different tasks      |

## Prepare

Install the required tools

```sh
# Debian/Ubuntu
apt-get install busybox-static git gzip docker.io qemu-system-x86
```

Clone the repository

```
git clone --recurse-submodules https://vcs.malbolge.net/chaosox/clt2024-rust-on-linux-workshop.git
```

### Containered tools

Build container with all required tools

```sh
./scripts/build-builder
```

## Development cycle

Build the kernel and modules and install the modules into initramfs

```sh
./scripts/build-kernel
```

Generate a new *initramfs* image

```sh
./scripts/gen-initramfs
```

Run the built kernel with initramfs

```sh
./scripts/run-kernel
```
