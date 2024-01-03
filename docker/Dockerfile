FROM docker.io/library/rust:1.75.0
RUN rustup override set 1.73.0
RUN cargo install --locked --version 0.65.1 bindgen-cli
RUN rustup component add clippy rustfmt rust-src
RUN apt-get update && \
    DEBCONF_FRONTEND=noninteractive apt-get install --yes \
        bc \
        bison \
        build-essential \
        clang \
        clang-14 \
        cpio \
        dwarves \
        flex \
        gcc \
        kmod \
        libelf-dev \
        libncurses-dev \
        libssl-dev \
        lld \
        llvm \
        llvm-14 \
        zlib1g
