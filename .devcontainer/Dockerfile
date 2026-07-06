FROM mcr.microsoft.com/vscode/devcontainers/base:ubuntu-22.04

# Avoid prompts from apt
ENV DEBIAN_FRONTEND=noninteractive

# Install system dependencies
RUN apt-get update && apt-get install -y --no-install-recommends \
    build-essential \
    curl \
    git \
    wget \
    xz-utils \
    m4 \
    unzip \
    pkg-config \
    zlib1g-dev \
    libomp-dev \
    coreutils \
    inotify-tools \
    libgmp-dev \
    libssl-dev \
    libev-dev \
    libc++-15-dev \
    libc++abi-15-dev \
    libx11-dev \
    ncurses-dev \
    autoconf \
    && rm -rf /var/lib/apt/lists/*

# Install Clang 15.0.0 from binaries
# Note: Hardcoded for x86_64 as per INSTALL.md instructions
RUN wget https://github.com/llvm/llvm-project/releases/download/llvmorg-15.0.0/clang+llvm-15.0.0-x86_64-linux-gnu-rhel-8.4.tar.xz \
    && tar -xf clang+llvm-15.0.0-x86_64-linux-gnu-rhel-8.4.tar.xz \
    && mv clang+llvm-15.0.0-x86_64-linux-gnu-rhel-8.4 /opt/clang-15.0.0 \
    && rm clang+llvm-15.0.0-x86_64-linux-gnu-rhel-8.4.tar.xz

# Configure update-alternatives for clang and llvm-config
RUN update-alternatives --install /usr/bin/clang clang /opt/clang-15.0.0/bin/clang 100 \
    && update-alternatives --install /usr/bin/clang++ clang++ /opt/clang-15.0.0/bin/clang++ 100 \
    && update-alternatives --install /usr/bin/llvm-config llvm-config /opt/clang-15.0.0/bin/llvm-config 100 \
    && ln -s /opt/clang-15.0.0/bin/llvm-config /usr/bin/llvm-config-15

# Set environment variables for Clang
ENV PATH="/opt/clang-15.0.0/bin/:${PATH}"

# Install opam as root
RUN printf '\n' | bash -c "sh <(curl -fsSL https://opam.ocaml.org/install.sh)"

# Switch to vscode user
USER vscode
WORKDIR /home/vscode

# Initialize opam and set up the switch
# Disable sandboxing as it often fails in Docker
RUN opam init --disable-shell-hook --disable-sandboxing -y \
    && opam switch create 4.14.2+options --packages=ocaml-variants.4.14.2+options,ocaml-option-flambda \
    && opam pin add -y dune 3.18.2 \
    && opam pin add -y menhirLib 20210419 \
    && opam pin add -y pprint 20220103 \
    && opam pin add -y conf-libclang 15 --no-depexts \
    && opam pin add -y clangml 4.8.0 --no-depexts --yes --no-action \
    && opam install -y dune refl pprint menhir menhirLib base64 ocamlbuild ocaml-lsp-server ppx_deriving graphics odoc lambdasoup dream \
    && opam install -y --verbose clangml

# Add opam env to bashrc
RUN echo 'eval $(opam env)' >> /home/vscode/.bashrc

# Default workspace
WORKDIR /workspaces/optitrust
