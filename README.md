# Rust on Linux Workshop

Files for Rust for Linux Workshop at the Chemnitzer Linux Tage 2024. The slides are located under [slides/SLIDES.md](slides/SLIDES.md).

You will find the solution for the workshop task in [`samples/rust/rust_chrdev_solution.rs`](linux/samples/rust/rust_chrdev_solution.rs)

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
apt-get install git podman qemu-system-x86
```

Clone the repository

```
git clone --depth 1 --recurse-submodules --shallow-submodules https://github.com/aboehm/clt2024-rust-kernel-module.git 
```

### Containered tools

The environment require `podman`/`buildah` or `docker`. The scripts will check which one installed.

It's optional to build the build environment container. You can pull it from [docker hub](https://hub.docker.com/r/aboehm/clt2024-rust-on-linux-workshop) instead.

To build the container with all required tools run


```sh
./scripts/build-builder
# Enforce to use buildah
DOCKER_BIN=buildah ./scripts/build-builder
# Enforce to use docker
DOCKER_BIN=docker ./scripts/build-builder
```

To enter the build environment run

```sh
./scripts/enter-build-env
# Enforce to use podman
DOCKER_BIN=podman ./scripts/enter-build-env
# Enforce to use docker
DOCKER_BIN=docker ./scripts/enter-build-env
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

### Permission issues

When using docker without the provided scripts file will be created or written as root. Change the user by running

```sh
sudo chown -R $(id -u):$(id -g) .
```

### Version errors during the build of the kernel module

Remove the already build kernel under `build/bzImage`.

### There're a lot of old files in the booted RootFS

Files in the initramfs directory will overwritten but not deleted. Delete all files of `build/initramfs`.

### Rust option is missing in the kernel configuration dialog

Maybe some build commands were executed without `LLVM=1`. Copy the configuration again and run `make LLVM=1 menuconfig` again to select your module. Ensure you add the LLVM parameter to each call of `make`.
