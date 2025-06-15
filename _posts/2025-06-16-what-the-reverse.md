---
title: Everything is in reverse
date: 2025-06-15
author: grga
categories: [raspberry-pi, bare-metal]
tags: [rpi, aarch64, mailbox]
description: Relief and frustration in one picture
---

So while playing around with mailbox and trying to write to the screen, after a few tries i ended up with this:

![Desktop View](/assets/img/mailbox_reverse.png)


Turns out, my hardcoded font array was LSB first.
So i stole IBM PC 3dfx8x8.bin font, loaded it using ``pub const FONT: &[u8] = include_bytes!("3dfx8x8.bin");`` and BAM

![Desktop View](/assets/img/mailbox.png)
