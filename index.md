# Deploying Rust code on ARM
This page is dedicated for people starting with embedded development in Rust. There are some tutorials on the internet touching this topic, but none of them goes deeply into the details of how and why it works.

## Tooling
The examples below were done on Linux machine. The target microcontroller is STM32F103 (commonly known as Blue Pill). It's an ARM Cortex-M3 with 64kB of FLASH and 20kB of SRAM, which is large enough for most purposes. For programming and debugging I'm using ST-LINK V2 clone.

## Software
### Compiler
For now, I've been successful with only Nightly Rust. To install nightly, use 'rustup install nightly'.
