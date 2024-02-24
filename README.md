# Rust on Linux Workshop

Files for Rust for Linux Workshop at the Chemnitzer Linux Tage 2024

## Overview

Directory structure

```text
├── build               -- Temporary build directory
│   ├── bzImage         -- The kernel
│   ├── initramfs       -- Our minimal RootFS directory
│   └── initramfs.cpio  -- The packed minimal RootFS
├── docker
│   └── Dockerfile      -- Container specification of the build environment
├── linux               -- Linux kernel sources
├── linux-config        -- Kernel configs
├── rust-module         -- Rust kernel module template
└── scripts             -- Some useful scripts to build the project
```

## Prepare

Install the required tools

```sh
# Debian/Ubuntu
apt-get install git docker.io qemu-system-x86
```

Clone the repository

```
git clone --recurse-submodules https://vcs.malbolge.net/chaosox/clt2024-rust-on-linux-workshop.git
```

### Containered tools

Build container with all required tools. This is optional. You can pull it from docker hub instead.

```sh
./scripts/build-builder
```

After that you can enter the build environment with

```sh
./scripts/build-env
```

## Development

### Configure the kernel

Copy the prepared kernel config from [linux-config/v6.7-rust-clt24](linux-config/v6.7-rust-clt24) to `linux/.config`.
After you can modify the configuration.

```sh
make LLVM=1 menuconfig
```

### Build everything

Start the build process

```
# Build the kernel and modules
make -j`nproc` LLVM=1 all
# Install the module into 
make LLVM=1 INSTALL_MOD_PATH=<path to initramfs> modules_install
```

After some minutes the should be store under `arch/x86/boot/bzImage`.

You can run the kernel with [QEMU](https://qemu.org)

```sh
qemu-system-x86_64 --kernel arch/x86/boot/bzImage
```

### A simple root file system

TBD

### Repetitive steps via helper scripts

There're several helper scripts to run some required commands.

Build the kernel and modules and install the modules into initramfs

```sh
./scripts/build-kernel
```

Generate a new *initramfs* image

```sh
./scripts/gen-initramfs
```

Build and install the rust module

```sh
./scripts/build-module
```

Run the built kernel with initramfs

```sh
./scripts/run-kernel
```

You get a serial console on the terminal and can load the module.

All these commands can be run at once with

```sh
./scripts/dev-cycle
```

## Trouble shooting

### Version errors during the build of the kernel module

Remove the already build kernel under `build/bzImage`.

### There're a lot of old files in the booted RootFS

Files in the initramfs directory will overwritten but not deleted. Delete all files of `build/initramfs`.
