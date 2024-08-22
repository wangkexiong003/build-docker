FROM alpine/git AS tool
WORKDIR /
ARG DENO_VERSION=1.5.4
RUN git clone --recurse-submodules https://github.com/denoland/deno.git \
    && cd deno && git checkout tags/v${DENO_VERSION}

FROM centos:7 AS base
COPY --from=tool /deno /deno
COPY resource/vault.repo /etc/yum.repos.d/vault.repo
##
## 1. lexical-core build error
##   https://users.rust-lang.org/t/expected-u32-found-usize-in-lexical-core/62820
##
ARG RUSTC_VERSION=1.49
RUN find /etc/yum.repos.d -type f ! -name "vault.repo" -delete \
    && yum groupinstall -y "Development Tools" \
    && yum install -y epel-release \
    && yum install -y jq \
    && URL=$(curl -s https://api.github.com/repos/kitware/cmake/releases/latest | jq -r .assets[].browser_download_url | grep linux-x86_64.tar.gz) \
    && curl -fsSL ${URL} | tar --transform='s,^[^/]*/,,' -C /usr -xzf - \
    && curl https://sh.rustup.rs -sSf | sh -s -- --default-toolchain ${RUSTC_VERSION} -y \
    && . ~/.cargo/env \
    && cd /deno \
    && cargo build --release

FROM centos:7
LABEL maintainer=wangkexiong
COPY --from=base /deno/target/release/deno /usr/local/bin
