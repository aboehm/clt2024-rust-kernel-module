# Rust For Linux

Alexander Böhm & Antonia Siegert

Chemnitzer Linux Tage 2024

---

## Plan

* Tooling aufsetzen
* Linux Kernel kompilieren
* Eigenes Kernelmodul
* Interaktion mit dem Modul
* Kennenlernen der Rust-Integration

---

## Zu uns

---

### Alexander Böhm

* Softwareentwickler
* hauptsächlich Rust & Python
* Kein Kernel Maintainer
* Mehr Hobby-Projekt

---

### Antonia Siegert

---

* Softwareentwickler
* hauptsächlich C/C++ & Python
* Kein Kernel Maintainer


## Tooling aufsetzen

* git
* `docker` oder `podman`
* QEMU

---

### Notwendige Programme

```sh
# Debian/Ubuntu
apt-get install git podman qemu-system-x86
```

---

## Projekt

```sh
git clone --recurse-submodules \
    https://github.com/aboehm/clt2024-rust-kernel-module.git
```

---

### Struktur

```text
├── build
│   ├── bzImage         -- Kernel
│   ├── initramfs       -- Minimales RootFS
│   └── initramfs.cpio  -- RAM-Dateisystem
├── docker              -- Container-Beschreibung
├── linux               -- Linux Kernel
├── linux-config        -- Kernel konfiguration
├── scripts             -- Skripte
└── slides              -- Vortragsfolien
```

---

## Vorbereitung

---

### Build-Umgebung

* Notwendige Tools enthalten
* Start [Container](https://hub.docker.com/r/aboehm/clt2024-rust-on-linux-workshop)

```sh
./scripts/enter-build-env
```

```text
root@localhost:/usr/src#
```

---

### Konfiguration

* Konfiguration kopieren 

  ```sh
  cp linux-config/v6.7-rust-clt24 linux/.config
  ```

-> Minimaler Kernel mit Rust-Unterstützung

---

### Kernel-Build

* Vorbereitung

  ```sh
  mkdir -p build/initramfs
  cd linux
  ```

* Kernel & Module kompilieren

  ```sh
  make -j`nproc` LLVM=1 all
  ```

* Kernel kopieren

  ```sh
  cp arch/x86/boot/bzImage ../build/bzImage
  ```

* Module kopieren

  ```sh
  make LLVM=1 INSTALL_MOD_PATH=../build/initramfs modules_install
  ```

oder mit einem Skript

```sh
./scripts/build-kernel
```

---

### RootFS

* Dateisystem
* Initramfs
* `cpio`-Archiv
* Busybox
* Minimaler Init-Prozess
* Skript
  ```sh
  ./scripts/gen-initramfs
  ```

---

### Kernel starten

```sh
qemu-system-x86_64 \
    -monitor none -serial stdio \
    --vga none -display none \
    --kernel build/bzImage \
    --initrd build/initramfs.cpio \
    --append "console=ttyS0 debug"
```

---

### Alles in einem Schritt

```sh
./scripts/dev-cycle
```

* Baut Kernel & Module
* Generiert Initramfs neu
* Startet Kernel mit QEMU

---

## Das erste Modul

---

### Dokumentation

* Datei anlegen<br><br>
  `linux/samples/rust/rust_clt_workshop.rs`:

```rust
// SPDX-License-Identifier: GPL-2.0

//! Rust module for CLT 2024

use kernel::prelude::*;
```

---

### Modul definieren

```rust
module! {
    type: RustCltModule,
    name: "rust_clt_workshop",
    author: "Alexander Böhm & Antonia Siegert",
    description: "Rust Module for CLT 2024",
    license: "GPL v2",
}

struct RustCltModule {}
```

---

### Loading 

```rust
impl kernel::Module for RustCltModule {
    fn init(
        _module: &'static ThisModule
    ) -> Result<Self> {
        pr_info!("Hello CLT!");
        Ok(Self {})
    }
}
```

---

### Unloading

```rust
impl Drop for RustCltModule {
    fn drop(&mut self) {
        pr_info!("Goodbye CLT");
    }
}
```

---

### Kernel config erweitern

Option hinzufügen in<br><br>`samples/rust/Kconfig`

```text
config SAMPLE_RUST_CLT_WORKSHOP
	tristate "CLT Workshop"
	help
	  Simple Rust module for the workshop.

	  If unsure, say N.
```

---

### Build dependencies anpassen

Hinzufügen in<br><br>`samples/rust/Makefile`

```
obj-$(CONFIG_SAMPLE_RUST_CLT_WORKSHOP) += rust_clt_workshop.o
```

---

### Modul aktivieren

```sh
cd linux
make LLVM=1 menuconfig
```
```
Location:
  -> Kernel hacking
    -> Sample kernel code
      -> Rust samples
```
![Modul in Konfiguration aktivieren](media/enable-module.png)

---

### Nach build nutzen

* Laden

```text
# insmod /lib/modules/6.7.0+/kernel/samples/rust/rust_clt_workshop.ko 
[   15.361326] rust_clt_workshop: Hello CLT!
```

* Entladen

```text
# rmmod rust_clt_workshop
[   22.364618] rust_clt_workshop: Goodbye CLT!
```

---

## Rust Integration

TBD

---

![Rust kernel infrastructure](media/rust-infrastructure.png)

<small>

*Quelle: [Rust for Linux, Miguel Ojeda, Wedson Almeida Filho (März 2022)](https://www.youtube.com/watch?v=fVEeqo40IyQ&list=PL85XCvVPmGQgL3lqQD5ivLNLfdAdxbE_u)*

</small>

---

![Rust kernel infrastructure](media/rust-module-design.png)

<small>

*Quelle: [Rust for Linux, Miguel Ojeda, Wedson Almeida Filho (März 2022)](https://www.youtube.com/watch?v=fVEeqo40IyQ&list=PL85XCvVPmGQgL3lqQD5ivLNLfdAdxbE_u)*

</small>

---

## Anhang

---

### Warum kein Modul?

* Rust basiert auf LLVM
* Kernel mit *clang* bauen
* Fehlende Funktionalitäten
  * PoC bei Rust-For-Linux für v5.19
  * Heutige Basis v6.7 => Patch

---

### Debug the kernel

```
file vmlinux
break misc_open
break misc_register
target remote localhost:1234
```

---

### Tooling ohne Container

```sh
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs |sh

rustup toolchain install 1.73.0
cargo install --locked --version 0.65.1 bindgen-cli
rustup component add clippy rustfmt rust-src

apt-get update
apt-get install \
    build-essential gcc kmod \
    bc bison dwarves flex \
    libelf-dev libncurses-dev libssl-dev \
    clang clang-14 lld llvm llvm-14 zlib1g \
    busybox-static cpio \
    python3 \
    qemu-system-x86
```

---

### Links

* [Memory Layout (x86-64)](https://www.kernel.org/doc/Documentation/x86/x86_64/mm.txt)
