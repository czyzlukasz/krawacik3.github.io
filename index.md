# Deploying Rust code on ARM
This page is dedicated for people starting with embedded development in Rust. There are some tutorials on the internet touching this topic, but none of them goes deeply into the details of how and why it works.

## Tooling
The examples below were done on Linux machine. The target microcontroller is STM32F103 (commonly known as Blue Pill). It's an ARM Cortex-M3 with 64kB of FLASH and 20kB of SRAM, which is large enough for most purposes. For programming and debugging I'm using ST-LINK V2 clone.

## Software
### Compiler
For now, I've been successful with only Nightly Rust. To install nightly, use 
```bash
rustup install nightly
```
Verify the rust version with 'rustc -V'. Try to stick to the newest version.
Next, You'll need to add your target:
```bash
rustup target add thumbv7m-none-eabi
```

### GDB
To debug the processor you'll need to install the 'gdb-multiarch' tool, because bare GDB does not fully support ARM architecture.

### OpenOCD
OpenOCD is responsible for interfacing with On-chip debugger. It's sort of an middleman between GDB and target.
OpenOCD needs configuration to run properly.
For this, You'll need two configuration files: one for interface and one for target. [This repo](https://github.com/hikob/openocd/tree/master/tcl) supports most programmers and targets. In my example, corresponding files were used:
'interface/stlink-v2.cfg'
'target/stm32f1x.cfg'

#### Config file for OpenOCD
For convenience, You can create config file which will be loaded at startup of OpenOCD.
openocd.cfg:
```Bash
set CHIPNAME stm32f1x
source [find interface/stlink-v2.cfg]
source [find target/stm32f1x.cfg]

init
```
Starting at the top `set CHIPNAME stm32f1x` sets the variable CHIPNAME to the proper value (same as the target .cfg).
`source [find interface/stlink-v2.cfg]` and `source [find target/stm32f1x.cfg]` are relative paths to your target and debugger files.
`init` terminates the configuration stage and enters the run stage.

Example of OpenOCD properly configured and connected to target:
```console
$ openocd -f openocd/openocd.cfg 
Open On-Chip Debugger 0.10.0
Licensed under GNU GPL v2
For bug reports, read
	http://openocd.org/doc/doxygen/bugs.html
Info : auto-selecting first available session transport "hla_swd". To override use 'transport select <transport>'.
Info : The selected transport took over low-level target control. The results might differ compared to plain JTAG/SWD
adapter speed: 1000 kHz
adapter_nsrst_delay: 100
none separate
Info : Unable to match requested speed 1000 kHz, using 950 kHz
Info : Unable to match requested speed 1000 kHz, using 950 kHz
Info : clock speed 950 kHz
Info : STLINK v2 JTAG v17 API v2 SWIM v4 VID 0x0483 PID 0x3748
Info : using stlink api v2
Info : Target voltage: 3.235403
Info : stm32f1x.cpu: hardware has 6 breakpoints, 4 watchpoints
```

# Basics of microcontroller booting process
To properly run the program on your microcontroller, You'll need to understand how the boot sequence. For now, I'll omit most details. After reset, the Program Counter (PC) starts at 0x00000000.It loads the value at this memory location to Main Stack Pointer (MSP) thus, marking the beginning of stack. Stack, usually, starts at the highest address of RAM and grows down. Next memory region after MSP is dedcated to Vector Table. For now, only one vector is needed, the Reset Vector. It's located at location 0x00000004.
Vectors are just simple functions that are called by hardware and/or software. The Reset Vector is called just after the reset or power on.

# Linker script
Linker scripts can be really useful for embedded programming, because code can be split in multiple sections, and the memory address and location (RAM, FLASH, external cached/uncached RAM) can be chosen by the programmer.

Small example of linker script can look like this:
memory.x
```
MEMORY
{
  FLASH : ORIGIN = 0x08000000, LENGTH = 64K
  RAM : ORIGIN = 0x20000000, LENGTH = 20K
}

ENTRY(Reset);
EXTERN(RESET_VECTOR);
SECTIONS
{
  . = ORIGIN(FLASH);
  .vector_table ORIGIN(FLASH) :
  {
    /*First, the stack pointer*/
    LONG(ORIGIN(RAM) + LENGTH(RAM));
    /*Then, the address of the reset vector*/
    .vector_table.reset_vector;
  } > FLASH

  .text :
  {

  } > FLASH

  .rodata :
  {

  } > RAM

   .ARM.exidx :
  {

  } > RAM
}
```
You can freely edit and modify this script and inspect the resulting binary to get hold of how linker works. Script is divided into two main items: `MEMORY` and `SECTIONS`. The `MEMORY` defines the physical layout of memory.
```
MEMORY
{
  FLASH : ORIGIN = 0x08000000, LENGTH = 64K
  RAM : ORIGIN = 0x20000000, LENGTH = 20K
}
```
This defines the FLASH and RAM memory regions, with its origins and sizes. If You use microcontroller with different memory size, You'll need to adjust these numbers according to the reference manual.

*Reminder:* Do not forget about space before the colon, otherwise Your script will not work!
Syntax of the `SECTIONS` item is as follows:
```
OutputSection :
{
ItemToIncludeInThisSection;
ItemToIncludeInThisSection;
} > MemoryRegion
```
TODO: describe ORIGIN(FLASH) 
The first item `. = ORIGIN(FLASH)` sets the current location in memory, see [this page](http://www.scoberlin.de/content/media/http/informatik/gcc_docs/ld_3.html) for more explanation.
Let's translate this script to *human readable* form:
1. Create the section at the beginning of FLASH that contains:
 1. Pointer to the end of the RAM section (as mentioned before, this is the start of stack)
 2. Pointer to Reset vector
2. Then just after that load code into the flash.
3. Load .rodata (Read-only data) to the RAM.
4. Load the .ARM.exidx (this is the info for stack unwinding in case of the fault) into the RAM.

# Putting it all together
Now, with working connection to the microcontroller and with working environment, You can proceed to writing Rust code.
```rust

```
