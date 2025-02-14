ARG VCS_REF=master
ARG BUILD_DATE=""
ARG REGISTRY_PATH=docker.io/paritytech

FROM ${REGISTRY_PATH}/base-ci:latest

# metadata
LABEL io.parity.image.authors="devops-team@parity.io" \
	io.parity.image.vendor="Parity Technologies" \
	io.parity.image.title="${REGISTRY_PATH}/contracts-ci-linux" \
	io.parity.image.description="Inherits from base-ci:latest. \
llvm-dev, clang, zlib1g-dev, npm, yarn, wabt, binaryen. \
rust nightly, rustfmt, clippy, rust-src, solang, substrate-contracts-node" \
	io.parity.image.source="https://github.com/paritytech/scripts/blob/${VCS_REF}/\
dockerfiles/contracts-ci-linux/Dockerfile" \
	io.parity.image.documentation="https://github.com/paritytech/scripts/blob/${VCS_REF}/\
dockerfiles/contracts-ci-linux/README.md" \
	io.parity.image.revision="${VCS_REF}" \
	io.parity.image.created="${BUILD_DATE}"

WORKDIR /builds

ENV SHELL /bin/bash
ENV DEBIAN_FRONTEND=noninteractive

# install tools and dependencies
RUN set -eux; \
	apt-get -y update && \
	apt-get install -y --no-install-recommends \
		zlib1g-dev npm wabt llvm-dev && \
	npm install --ignore-scripts -g yarn && \
# `binaryen` is needed by `cargo-contract` for optimizing Wasm files.
# We fetch the latest release which contains a Linux binary.
	curl --silent https://api.github.com/repos/WebAssembly/binaryen/releases | \
		egrep --only-matching 'https://github.com/WebAssembly/binaryen/releases/download/version_[0-9]+/binaryen-version_[0-9]+-x86_64-linux.tar.gz' | \
		head -n1 | \
		xargs curl -L -O && \
	tar -xvzf binaryen-*-x86_64-linux.tar.gz  && \
	rm binaryen-*-x86_64-linux.tar.gz && \
	chmod +x binaryen-*/bin/wasm-opt && \
	mv binaryen-*/bin/wasm-opt /usr/local/bin/ && \
	rm -rf binaryen-*/ && \
# Installs the latest common nightly for the listed components,
# adds those components, wasm target and sets the profile to minimal
	rustup toolchain install nightly --target wasm32-unknown-unknown \
		--profile minimal --component rustfmt clippy rust-src && \
	rustup default nightly && \
	cargo install pwasm-utils-cli --bin wasm-prune && \
	cargo install cargo-dylint dylint-link && \
	cargo install cargo-contract && \
# tried v0.1.5 and the latest master - both fail with https://github.com/hyperledger-labs/solang/issues/314
	cargo install --git https://github.com/hyperledger-labs/solang --tag v0.1.2 && \
# download the latest `substrate-contracts-node` binary
	curl -L "https://gitlab.parity.io/parity/substrate-contracts-node/-/jobs/artifacts/master/raw/artifacts/substrate-contracts-node-linux/substrate-contracts-node?job=build-linux" \
		-o /usr/local/cargo/bin/substrate-contracts-node && \
	chmod +x /usr/local/cargo/bin/substrate-contracts-node && \
# We use `estuary` as a lightweight cargo registry in the CI to test if
# publishing `cargo-contract` to it and installing it from there works.
	cargo install --git https://github.com/onelson/estuary.git --force && \
# versions
	yarn --version && \
	rustup show && \
	cargo --version && \
	solang --version && \
	echo $( substrate-contracts-node --version | awk 'NF' ) && \
	estuary --version && \
# cargo clean up
# removes compilation artifacts cargo install creates (>250M)
	rm -rf "${CARGO_HOME}/registry" "${CARGO_HOME}/git" /root/.cache/sccache && \
# apt clean up
	apt-get autoremove -y && \
	apt-get clean && \
	rm -rf /var/lib/apt/lists/*

# TODO: https://gitlab.parity.io/parity/cargo-contract/-/jobs/958744, https://gitlab.parity.io/parity/cargo-contract/-/jobs/958745
# USER nonroot:nonroot
