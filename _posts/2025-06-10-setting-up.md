---
title: Setting up environment
date: 2025-06-14
author: grga
categories: [rust, bare-metal]
tags: [wsl, rust, aarch64]
description: 
---

## In WSL, Ubuntu 22.04

To setup VSCode with WSL I followed this [ link ](https://learn.microsoft.com/en-us/windows/wsl/tutorials/wsl-vscode)

install rust:
```shell
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
```

add target:
```shell
rustup target add aarch64-unknown-none
```

already installed from before:
```shell
sudo apt install gcc-aarch64-linux-gnu binutils-aarch64-linux-gnu qemu-system-arm
```

install cargo-binutils (objdump, objcopy, etc.) to create final kernel8.img file:
```shell
cargo install cargo-binutils
rustup component add llvm-tools-preview

create new binary project:
cargo new --bin rpi-rust-os
```

## Manual building

to build run ``cargo build``, this will create an elf file in `target/aarch64-unknown-none/debug/rpi-rust-os`

after building create raw binary, no need to specify path:
```shell
cargo objcopy -- -O binary target/kernel8.img
```

to test in qemu:
```shell
qemu-system-aarch64 -M raspi3b -kernel target/kernel8.img -serial stdio
```

might be needed for "-display sdl" in qemu, but im gonna use gtk anyways:
```shell
sudo apt install libsdl1.2-dev
```

> All of this could be nicely combined in build.rs and vscode tasks.
{: .prompt-tip}

## Some additional things for bare metal in Rust

- `#![no_std]` don't link standard library
- `#![no_main]` don't generate main() entry point
- `#[panic_handler]` Rust requires a function to call if the program panics, so provide a simple halt
- `#[no_mangle]` prevent Rust name mangling for external variables, symbols
- `unsafe { core::ptr::write_volatile(...) }`
  - interacting with hardware memory (MMIO) is unsafe because compiler can't verify it, wrapping it in an unsafe block acknowledges this
  - `write_volatile` is the correct way to perform MMIO writes, preventing unwanted compiler optimizations.
