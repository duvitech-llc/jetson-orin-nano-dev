FROM ubuntu:22.04

ENV DEBIAN_FRONTEND=noninteractive

# Core build dependencies
RUN apt-get update && apt-get install -y \
    build-essential \
    bc \
    flex \
    bison \
    libssl-dev \
    libelf-dev \
    dwarves \
    git \
    device-tree-compiler \
    pkg-config \
    python3 \
    python3-pip \
    rsync \
    sudo \
    vim \
    less \
    wget \
    ca-certificates \
    gcc-aarch64-linux-gnu \
    g++-aarch64-linux-gnu \
    && rm -rf /var/lib/apt/lists/*

# Create non-root user (matches VS Code expectations)
ARG USERNAME=dev
ARG UID=1000
ARG GID=1000

RUN groupadd --gid ${GID} ${USERNAME} \
    && useradd --uid ${UID} --gid ${GID} -m ${USERNAME} \
    && echo "${USERNAME} ALL=(ALL) NOPASSWD:ALL" >> /etc/sudoers

USER dev
WORKDIR /workspace

# Useful defaults
ENV ARCH=arm64
ENV CROSS_COMPILE=aarch64-linux-gnu-
