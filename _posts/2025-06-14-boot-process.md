---
title: RPi Boot Process
date: 2025-06-14
author: grga
categories: [raspberry-pi, bare-metal]
tags: [rpi, aarch64, boot]
description: Very brief overview of the boot process
---

## Raspberry Pi Boot Sequence

### Primary Bootloader (PBL)

- Hardcoded into GPU silicon
- Loads **Second Stage Bootloader (SBL)**: `bootcode.bin` from FAT32 partition (SD card)

### Second Stage Bootloader (SBL)

- Initializes:
  - SDRAM controller
  - Minimal hardware setup
- Loads **Third Stage Bootloader**: `start4.elf`

### Third Stage Bootloader (`start4.elf`)

- Handles full boot logic:
  - Reads `config.txt` for boot parameters
  - Loads `fixup4.dat` for GPU/CPU memory split
- Loads Device Tree Blob (DTB)
- Applies Device Tree overlays (if any) as specified in `config.txt`
- Places finalized DTB in RAM at a known address for kernel use

### Kernel Load

- GPU loads `kernel8.img` to physical address `0x80000` (RPi specific, AArch64 mode).
- Sets up initial CPU state and registers:
  - `x0`: DTB physical address
  - `x1`: Resevred
  - `x2`: Resevred
  - `x3`: Resevred
- Loads command line parameters from `cmdline.txt`
- Releases CPUs from reset
- CPU begins execution at kernel entry point (`_start` in `boot.S`)

### Files Involved

- Most boot files are proprietary, so naturally stolen from Raspberry Pi OS:
  - `bootcode.bin`, `start4.elf`, `fixup4.dat`, etc.
- Files under my control:
  - `boot.S` kernel entry code
  - `config.txt`, `cmdline.txt` kernel arguments, memory settings

> On kernel entry, DTB address is in x0. Validity is confirmed by reading first 4 bytes for 0xD00DFEED magic. More on that later
{: .prompt-info}
