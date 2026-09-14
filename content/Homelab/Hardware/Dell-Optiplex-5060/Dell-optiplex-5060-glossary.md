---
title: Dell Optiplex 5060 Glossary
description: Terms used across the Dell Optiplex 5060 bucket.
tags:
  - glossary
---
## ECC RAM

RAM that can detect and correct in-memory data corruption using an extra parity chip per module, which matters most for servers and other machines expected to stay up and error-free for long stretches. It requires explicit motherboard/CPU support — it's not interchangeable with non-ECC-only platforms, and mixing the two doesn't work. I learned this the hard way: the cheapest used DDR4 kit I could find turned out to be ECC RAM, and it produced a BIOS POST beep-code RAM fault on this consumer board that has no ECC support, forcing me to trade it for a non-ECC kit instead. See [ECC memory (Wikipedia)](https://en.wikipedia.org/wiki/ECC_memory).
