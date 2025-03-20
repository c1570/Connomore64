# Connomore64
https://github.com/c1570/Connomore64

Realtime cycle exact emulation of the [Commodore 64](https://en.wikipedia.org/wiki/Commodore_64) using multiple microcontrollers in parallel.

**This is a proof of concept and not end user ready.**

![Connomore64 breadbox v0 prototype](/images/CNM64_Breadbox_v0_prototype.jpg)

**[July 2024 poster](/2024-07-Connomore64_poster.pdf)** (old prototype)

## In a nutshell, this is (or aims to be)…
* …a **cycle exact Commodore 64 homecomputer emulator**…
* …an (almost) chip-by-chip rebuild using multiple [RP2040](https://en.wikipedia.org/wiki/RP2040)/[RP2350](https://en.wikipedia.org/wiki/RP2350) microcontrollers (<1€ per chip!)…
* …interconnected using a multiplexed **8 bit bus** running at about 8 MHz effectively…
* …using **DVI/HDMI** as video and audio output…
* …with microsecond exact true to original real world signal timing…
* …able to **interface with original hardware**…
  * …such as the C1541 floppy drive, including fastloaders…
  * …as well as userport and (some) expansion port based hardware.

### Additionally, this is…
* …an example of running **compute intensive software on cheap low powered interconnected microcontrollers**…
* …an **emulation framework** to test and debug projects using several RP2040/RP2350s in parallel on a PC.

## But why?
* First and foremost, this was started as a holiday project (“porting an existing C64 emulator to a 400MHz ARM platform cannot take that long, can it?”) with a focus on playing a bit with the RP2040 microcontroller and getting an idea of what its PIOs (i.e., its GPIO handling coprocessors) are capable of.
  * The holiday of this “holiday project” was christmas 2022, and it turns out a single 400MHz M0+ core is definitely not enough to emulate a C64, if accuracy is some priority…
* While a C64 emulator on the RP2040 already [exists](https://github.com/silvervest/c64pico), it is using an extremely simplified model of the C64, e.g., not emulating the exact steps and timing of MOS 6510 CPU opcodes, and not emulating the VIC-II video chip exactly but doing video line based emulation instead which limits compatibility.
* PC based emulators (VICE, BMC etc.) reach high emulation quality but do not feature perfect realtime timing. That means you cannot use physical retro hardware (floppy drives, etc.) with those emulators in general.
* FPGA based emulators of the C64 reach high emulation quality and realtime accuracy, but those platforms strike yours truly as quite overpowered and expensive, not as hackable as would be nice, and typically come with a rather unwieldy (and typically closed source) toolchain.

## Building Blocks
### RP2040/RP2350 Emulator
For developing this project, the [rp2040js](https://github.com/wokwi/rp2040js) emulator project by Uri Shaked of [Wokwi.com](https://wokwi.com) fame has been extended. Very little debugging on real hardware happened. No logic analyzers were necessary at all!

You can find my fork of rp2040js [here](https://github.com/c1570/rp2040js/). It features:
* RP2350 support (RISC-V/Hazard3 only)
* cycle exact timing of PIOs
* [VCD](https://en.wikipedia.org/wiki/Value_change_dump) trace files can get generated for the GPIO signals
* a simple framework for running multiple chip emulation instances interfacing to each other ([WIP code](https://github.com/c1570/rp2040js/blob/rp2350js/WIP/demo/ntc-run.ts))
* simulation of input/output latency of GPIOs
* functionality to output statistics and monitor any PIO stalls is present in the emulation runner that has been customized for this project

### C64 Emulator Code
The C64 emulation code is based on the “[chips](https://github.com/floooh/chips)” emulation library by Andre Weissflog. A lot of optimizations have been done:
* speed optimized VIC-II code
  * rewritten graphics rendering code (running 5-10 times as fast as the previous code while sacrificing some compatibility)
  * rendering bitmap/text mode is done using PIO/DMA
  * rewritten Sprite rendering (in contrast to the original VIC-II, sprites are rendered into a buffer in the offscreen time)
* fixed a few [VIC-II emulation bugs](https://github.com/floooh/chips/issues/99) (VSP/AGSP works now)
* faster CIA emulation using look up tables
* replaced `uint64_t pins` interface with 32 bits pins/direct variables

### Video Output
HDMI/DVI output is based on the [PicoDVI](https://github.com/Wren6991/PicoDVI) library by Luke Wren.

### Audio Output
Provided by a port of the [SIDKick pico](https://github.com/frntc/SIDKick-pico) firmware (by Carsten Dachsbacher, using reSID by Dag Lem) to the Connomore64 bus system.

## Hardware
### Breadbox v0 prototype (latest)
This is a custom PCB that fits into the C64 case (shown on top of the page).
It comes with the usual C64 ports (userport, IEC, expansion port, joystick ports) plus HDMI and audio jack.
Currently, the userport, IEC, and joystick ports are functional.
Support for select expansion port cartridges (e.g., Magic Desk) is upcoming.

Goals of this prototype:
* All original C64 connectors available
* Simple to build
* Easy migration to RP2350 in development
* Testbed for SWD-based firmware handling

### Stacked Pico boards (early prototype)
The first prototype was several stacked RP2040 boards.
Video output is done using a passive DVISock adapter.
Using a small passive “hat”, it can connect to an original C64 keyboard, joysticks, as well as a floppy drive.
Loading programs from the drive is possible using the original CBM routines as well as the much faster JiffyDOS or other speeders.

![Early prototype](/images/CNM64_Prototype_1.jpg)

### General
Smaller custom PCBs for the Connomore are possible.
Those could be made very cheap and small as only one flash chip and crystal would be needed, and setting up the microcontrollers could be done via SWD.
Total cost below 20€ might be possible.

## Status
### Compatibility
* Runs almost all games just fine
  * Mayhem in Monsterland, Hawkeye, Armalyte, Katakis, R-Type, Bubble Bobble, Turrican, ...
* Runs a good portion of demos (e.g., Layers)
* Fastloaders work just fine (tested with JiffyDOS and [Transwarp](https://www.c64-wiki.com/wiki/Transwarp))
* User port hardware works (tested with WiC64)

### Missing Features
* Only the CPU half of each C64 cycle is emulated, limiting potential compatibility with C64 expansion port cartridges.
  * There is code for Phi low but RP2040/RP2350s are not fast enough for that.
* Expansion port firmware and hardware is not done (needs some thought)

## License
At the moment, the project and code is in no way release ready.
I'll likely release it as (possibly non commercial) strong copyleft open source code at some point.
If you are interested in the project or in contributing, please contact me.
