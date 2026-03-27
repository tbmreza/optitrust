FROM mcr.microsoft.com/devcontainers/base:ubuntu-22.04

# Build argument to install extra tools (hwloc, valgrind, mpi, jemalloc, fftw)
ARG INSTALL_EXTRAS=false

# Install system dependencies
# Including libssl-dev and libev-dev for 'dream' package
RUN apt-get update && export DEBIAN_FRONTEND=noninteractive \
    && apt-get -y install --no-install-recommends \
    libomp-dev pkg-config zlib1g-dev coreutils inotify-tools \
    meld wget xz-utils build-essential ca-certificates \
    libc++-15-dev libc++abi-15-dev git curl m4 unzip sudo \
    libgmp-dev libssl-dev libev-dev \
    && if [ "$INSTALL_EXTRAS" = "true" ]; then \
    apt-get -y install --no-install-recommends \
    hwloc valgrind libopenmpi-dev openmpi-bin libjemalloc-dev libfftw3-dev; \
    fi \
    && apt-get clean && rm -rf /var/lib/apt/lists/*

# Install Clang 15.0.0 exactly as specified in INSTALL.md
RUN wget https://github.com/llvm/llvm-project/releases/download/llvmorg-15.0.0/clang+llvm-15.0.0-x86_64-linux-gnu-rhel-8.4.tar.xz \
    && tar -xf clang+llvm-15.0.0-x86_64-linux-gnu-rhel-8.4.tar.xz \
    && mv clang+llvm-15.0.0-x86_64-linux-gnu-rhel-8.4 /opt/clang-15.0.0 \
    && rm clang+llvm-15.0.0-x86_64-linux-gnu-rhel-8.4.tar.xz

# Set up update-alternatives
RUN update-alternatives --install /usr/bin/clang clang /opt/clang-15.0.0/bin/clang 100 \
    && update-alternatives --install /usr/bin/llvm-config llvm-config /opt/clang-15.0.0/bin/llvm-config 100 \
    && ln -s /opt/clang-15.0.0/bin/llvm-config /usr/bin/llvm-config-15

# Update PATH
ENV PATH="/opt/clang-15.0.0/bin/:${PATH}"

# Switch to the non-root user provided by the base image
USER vscode

# Install opam
RUN curl -fsSL https://raw.githubusercontent.com/ocaml/opam/master/shell/install.sh | sh -s -- --install-eventual-opam-binary-host-sys-package

# Initialize opam and install OCaml dependencies
# Fix: --disable-sandboxing instead of --disabled-sandboxing
RUN opam init --disable-sandboxing --shell-setup --yes \
    && opam switch create 4.14.2+options --packages=ocaml-variants.4.14.2+options,ocaml-option-flambda --yes \
    && eval $(opam env) \
    && opam pin add -y dune 3.18.2 \
    && opam pin add -y menhirLib 20210419 \
    && opam pin add -y pprint 20220103 \
    && opam pin add -y conf-libclang 15 --no-depexts \
    && opam pin add -y clangml 4.8.0 --no-depexts --yes --no-action \
    && opam install -y dune refl pprint menhir menhirLib base64 ocamlbuild ocaml-lsp-server ppx_deriving \
    && opam install -y clangml --verbose \
    && opam install -y odoc lambdasoup dream

# Ensure opam environment is loaded for the user
RUN echo "eval \$(opam env)" >> /home/vscode/.bashrc
