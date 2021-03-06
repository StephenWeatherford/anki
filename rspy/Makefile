SHELL := /bin/bash
FIND := $(if $(wildcard /bin/find),/bin/find,/usr/bin/find)

ifndef OS
	OS := unknown
endif

ifeq ($(OS),Windows_NT)
	ifndef PYTHON_BIN
		PYTHON_BIN := python
	endif
else
	ifndef PYTHON_BIN
		PYTHON_BIN := python3
	endif
endif

.SHELLFLAGS := -eu -o pipefail -c
.DELETE_ON_ERROR:
MAKEFLAGS += --warn-undefined-variables
MAKEFLAGS += --no-builtin-rules

$(shell mkdir -p .build ../dist)

OUTDIR := ../dist
BUILDFLAGS := --release --strip
RSPY_TARGET_DIR ?= target

QT_FTL_TEMPLATES := ../qt/ftl
QT_FTL_LOCALES := ../qt/ftl/repo/desktop

.PHONY: all develop build check fix clean

all: develop

develop: .build/develop

DEPS := .build/tools .build/vernum ../meta/buildhash \
	$(wildcard $(QT_FTL_TEMPLATES)/*.ftl) \
	$(wildcard $(QT_FTL_LOCALES)/*/*.ftl) \
	$(shell ${FIND} ../rslib/src -name '*.rs') $(wildcard ../proto/*) \
	$(shell ${FIND} ../rslib/ftl -type f) \
	$(shell ${FIND} ./src -type f) 

.build/develop: $(DEPS)
	touch ../proto/backend.proto
	FTL_TEMPLATE_DIRS="$(QT_FTL_TEMPLATES)" FTL_LOCALE_DIRS="$(QT_FTL_LOCALES)" \
		CARGO_TARGET_DIR="$(RSPY_TARGET_DIR)" maturin develop
	touch $@

build: $(DEPS)
	rm -rf $(OUTDIR)/ankirspy*
	touch ../proto/backend.proto
	FTL_TEMPLATE_DIRS="$(QT_FTL_TEMPLATES)" FTL_LOCALE_DIRS="$(QT_FTL_LOCALES)" \
		maturin build -i $(shell which ${PYTHON_BIN}) -o $(OUTDIR) $(BUILDFLAGS)

check: .build/check

fix:
	cargo fmt

clean:
	rm -rf .build target

# nightly currently required for ignoring files in rustfmt.toml
RUST_TOOLCHAIN := $(shell cat rust-toolchain)

.build/tools: requirements.txt rust-toolchain
	python -m pip install -r requirements.txt
	rustup toolchain install $(RUST_TOOLCHAIN)
	rustup component add rustfmt-preview --toolchain $(RUST_TOOLCHAIN)
	rustup component add clippy-preview --toolchain $(RUST_TOOLCHAIN)
	@touch $@

.build/check: build
	cargo fmt -- --check
	cargo clippy -- -D warnings
	@touch $@

VER := $(shell cat ../meta/version)
.build/vernum: ../meta/version
	sed -i.bak 's/.*automatically updated.*/version = "$(VER)" # automatically updated/' Cargo.toml
	rm Cargo.toml.bak
	@touch $@
