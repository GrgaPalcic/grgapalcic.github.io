---
title: Brief device tree structure
date: 2025-06-15
author: grga
categories: [raspberry-pi, bare-metal]
tags: [rpi, aarch64, dtb]
description: This one was confusing at first but ultimately very interesting to learn about.
---

Device Tree blob is flat binary (big-endian) encoding of device tree data. There is a lot of things in there, but for now i only need this:


First, ``address-cells`` and ``size-cells`` are contained in the root node, and they represent encoding of ``reg`` properties of it's child nodes. Former defines number of u32 cells (32 bit values) needed to form the base address part, and later the size part. If child node contains ``address-cells`` and ``size-cells`` they will overwrite parent ones for that node.


e.g.
```
Node:
    #address-cells: 00 00 00 01 
    #size-cells: 00 00 00 01 
    Node: memory@0
        reg: 00 00 00 00 1c 00 00 00 
        device_type: 6d 65 6d 6f 72 79 00
```
Here, both address and size are one u32 value, 4 bytes wide, of the reg so ``0x0000_0000`` and size is ``0x1c00_0000`` resulting in ``469762048`` byter from the address 0x0. This is in the **CPU physical space**.


However, nodes containing ``ranges`` property, such as ``soc``, means peripherals sit behind the bus bridge, and one has to translate child bus address to the parent physical cpu address.

``ranges = < child_address parent_address size> * N``


e.g.
```
soc {
        compatible = "simple-bus";
        #address-cells = <0x00000001>;
        #size-cells = <0x00000001>;
        ranges = <0x7e000000 0x3f000000 0x01000000 0x40000000 0x40000000 0x00001000>;
        dma-ranges = <0xc0000000 0x00000000 0x3f000000 0x7e000000 0x3f000000 0x01000000>;
        phandle = <0x00000041>;
        serial@7e201000 {
            compatible = "arm,pl011", "arm,primecell";
            reg = <0x7e201000 0x00000200>;
            interrupts = <0x00000002 0x00000019>;
```

In this particular case there are two mappings (N = 2).

I care for the first one, so bus (child) address being ``0x7e000000``, cpu (parent) address being ``0x3f00_0000``, and size of ``0x0100_0000``.

Size means that translation mapping covers address ranges from ``0x7e00_0000`` to ``0x7eff_ffff``, as serial address falls into that range it will be translated from ``0x7e20_1000`` to ``0x3f20_1000``.

In code:
```
phy_addr = parent_address + (device_address - child_address)
```

> It's important to note that range covers adresses ``[start, start+size-1]`` because range ends on the last byte of the mapped region, and not on the next unmapped byte.
{: .prompt-info}

## Sources:
- https://devicetree-specification.readthedocs.io/en/stable/devicetree-basics.html
- https://docs.kernel.org/devicetree/usage-model.html
