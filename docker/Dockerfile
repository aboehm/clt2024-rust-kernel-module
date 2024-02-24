FROM docker.io/library/rust:1.73.0
RUN cargo install --locked --version 0.65.1 bindgen-cli
RUN rustup component add clippy rustfmt rust-src
RUN apt-get update && \
    DEBCONF_FRONTEND=noninteractive apt-get install --yes \
        bc \
        bison \
        build-essential \
        busybox-static \
        clang \
        clang-14 \
        cpio \
        dvipng \
        dwarves \
        flex \
        fonts-noto-cjk \
        gdb \
        gcc \
        graphviz \
        kmod \
        latexmk \
        libelf-dev \
        libncurses-dev \
        librsvg2-bin \
        libssl-dev \
        lld \
        llvm \
        llvm-14 \
        python3 python3-sphinx python3-venv \
        texlive-lang-chinese texlive-xetex \
        zlib1g
