# Makefile for RISC-V ISA Manuals
#
# This work is licensed under the Creative Commons Attribution-ShareAlike 4.0
# International License. To view a copy of this license, visit
# http://creativecommons.org/licenses/by-sa/4.0/ or send a letter to
# Creative Commons, PO Box 1866, Mountain View, CA 94042, USA.
#
# SPDX-License-Identifier: CC-BY-SA-4.0
#
# Description:
# 
# This Makefile is designed to automate the process of building and packaging 
# the documentation for RISC-V ISA Manuals. It supports multiple build targets 
# for generating documentation in various formats (PDF, HTML).

# Build Targets
TARGETS := unpriv
STAMP := stamp
BUILD_DOCKER_NAME := rvdocs

ifdef realpath
MANUAL_ROOT_DIR := $(realpath $(PWD)/..)
else
MANUAL_ROOT_DIR := $(shell cd .. && pwd)
endif

ifeq ($(wildcard $(MANUAL_ROOT_DIR)/src/riscv-unprivileged.adoc),)
  $(error MANUAL_ROOT_DIR=$(MANUAL_ROOT_DIR) is not a RISCV-ISA-MANUAL repo)
endif


# Declare phony targets
.PHONY: all $(TARGETS) clean

# Default target builds all
all: $(TARGETS)

# Build with preinstalled docker container;
# We prevent auto pulling the image by using --pull=never
# first install it with:
#   docker pull riscvintl/riscv-docs-base-container-image:latest
$(STAMP)/container:
	@echo "Craeting container" $(BUILD_DOCKER_NAME)
	@mkdir -p $(STAMP)
	@docker rm $(BUILD_DOCKER_NAME) >/dev/null 2>&1 || true
	@docker create --name $(BUILD_DOCKER_NAME) --pull=never \
		-v $(MANUAL_ROOT_DIR):/build riscvintl/riscv-docs-base-container-image:latest \
		/bin/sh -c 'cd ./build; make'
	@touch $(STAMP)/container

docker:$(STAMP)/container
	@echo "Building with docker"
	@docker start -i $(BUILD_DOCKER_NAME)

# Asciidoctor options
ASCIIDOCTOR_OPTS := -a compress \
                    --attribute=mathematical-format=svg \
                    --failure-level=ERROR \
                    --require=asciidoctor-bibtex \
                    --require=asciidoctor-diagram \
                    --require=asciidoctor-mathematical \
					-a scripts=cjk \
                    --trace

# Source directory
SRCDIR := $(MANUAL_ROOT_DIR)/src

# Unprivileged ISA build
unpriv: unpriv-isa-asciidoc.pdf

unpriv-isa-asciidoc.pdf: $(SRCDIR)/riscv-unprivileged.adoc $(SRCDIR)/*.adoc
	@echo "Building Unprivileged ISA"
	rm -f $@.tmp
	asciidoctor-pdf $(ASCIIDOCTOR_OPTS) --out-file=$@.tmp $<
	mv $@.tmp $@

clean:
	@if [ -f unpriv-isa-asciidoc.pdf ]; then \
		echo "Removing unpriv-isa-asciidoc.pdf"; \
		rm -f unpriv-isa-asciidoc.pdf; \
	fi
	@docker rm $(BUILD_DOCKER_NAME) >/dev/null 2>&1 && rm $(STAMP)/container || true