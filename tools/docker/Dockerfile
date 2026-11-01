# Linux ARM64 build and test environment, for building on an Apple Silicon Mac.
FROM rust:1.89-bookworm
RUN apt-get update && apt-get install -y --no-install-recommends \
    ninja-build nasm pkg-config python3-pip python3-venv zip file binutils llvm \
    libasound2-dev libvulkan-dev mesa-vulkan-drivers libclang-dev clang
RUN python3 -m venv /opt/py && /opt/py/bin/pip install --quiet ziglang meson
RUN cargo install --locked cargo-zigbuild
RUN apt-get install -y --no-install-recommends fonts-noto-cjk
ENV PATH=/opt/py/bin:$PATH
ENV CARGO_TARGET_DIR=/cache/target
ENV LD_LIBRARY_PATH=/src/third_party/ffmpeg/aarch64/lib
WORKDIR /src
