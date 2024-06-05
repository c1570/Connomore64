# Connomore64
https://github.com/c1570/Connomore64

Realtime cycle exact emulation of the [Commodore 64](https://en.wikipedia.org/wiki/Commodore_64) using multiple microcontrollers in parallel.

**This is a proof of concept and not end user ready.**

## In a nutshell, this is (or aims to be)…
* …a cycle exact Commodore 64 homecomputer emulator…
* …running on 8 ARM Cortex M0+ CPU cores of 4 [RP2040](https://en.wikipedia.org/wiki/RP2040) microcontrollers (<1€ each!)…
* …interconnected using a multiplexed 8 bit bus running at about 8 MHz effectively…
* …using DVI/HDMI as video and audio output…
* …with microsecond exact true to original real world signal timing…
* …able to **interface with original hardware** such as the C1541 floppy drive.

## Additionally, this is…
* …an example of running compute intensive software on cheap low powered interconnected microcontrollers…
* …an emulation framework to test and debug projects using several RP2040s in parallel on a PC.

## But why?
First and foremost, this was started as a holiday project (“porting an existing C64 emulator to a 400MHz ARM platform cannot take that long, can it?”) with a focus on playing a bit with the RP2040 microcontroller and getting an idea of what its PIOs (i.e., its GPIO handling coprocessors) are capable of.
While some emulators (even of the C64, even on the RP2040) exist, those are using extremely simplified models of the C64, e.g., not emulating the exact steps and timing of MOS 6510 CPU opcodes, and not emulating the VIC-II video chip exactly but doing video line based emulation instead which limits compatibility.
Accurate emulators of the C64 exist for FPGA platforms, but those platforms strike yours truly as quite overpowered and expensive, not as hackable as would be nice, and typically come with a rather unwieldy (and typically closed source) toolchain.

The holiday of this “holiday project” was christmas 2022, and it turns out a single 400MHz M0+ core is definitely not enough to emulate a C64, if accuracy is some priority…

## Building blocks
### rp2040 emulator
For developing this project, the [rp2040js](https://github.com/wokwi/rp2040js) emulator project by Uri Shaked of [Wokwi.com](https://wokwi.com) fame has been extended. Very little debugging on real hardware happened. No logic analyzers were necessary at all!

You can find my fork of rp2040js [here](https://github.com/c1570/rp2040js/).
* cycle exact timing of PIOs has been added
* [VCD](https://en.wikipedia.org/wiki/Value_change_dump) trace files can get generated for the GPIO signals
* a simple framework for running multiple rp2040 emulation instances interfacing to each was written ([WIP code](https://github.com/c1570/rp2040js/blob/c1570/WIP/demo/ntc-run.ts))
* simulation of input/output latency of the RP2040’s GPIOs
* inter-core FIFO functionality has been added
* functionality to output statistics and monitor any PIO stalls is present in the emulation runner that has been customized for this project

### C64 emulator
The C64 emulation code is based on the “[chips](https://github.com/floooh/chips)” emulation library by Andre Weissflog. A lot of performance optimizations have been done:
* rewritten VIC-II graphics rendering code (running 5-10 times as fast as the previous code while sacrificing some compatibility)
* rewritten Sprite rendering (in contrast to the original VIC-II, sprites are rendered into a buffer in the offscreen time)
* faster CIA emulation using look up tables

### Video output
HDMI/DVI output is based on the [PicoDVI](https://github.com/Wren6991/PicoDVI) library by Luke Wren.

## Prototype hardware
So far, the Connomore64 consists of three stacked RP2040 boards which are interconnected using the 8 bit bus mentioned before. Video output is done using a passive DVISock adapter. Using a small passive “hat”, it can connect to an original C64 keyboard, joysticks, as well as a floppy drive. Loading programs from the drive is possible using the original CBM routines as well as the much faster JiffyDOS or other speeders.

At some point, it would be sensible to create a dedicated board for the project.
It could be made very cheap and small as only one flash chip and crystal would be needed, and setting up the microcontrollers could be done via SWD.
Total cost including connectors etc. below 20€ might be possible.

## Missing features
Some features are still missing - this project is work in progress.
* Only the CPU half of each C64 cycle is emulated, limiting potential compatibility with C64 expansion port cartridges.
* Userport software and hardware is not done (this should be easy though)
* Expansion port firmware and hardware is not done (needs some thought)
* SID audio is not done (it should be relatively straightforward to port [SIDKick pico](https://github.com/frntc/SIDKick-pico) though)

## License
At the moment, the project and code is in no way release ready.
I'll likely release it as (possibly non commercial) strong copyleft open source code at some point.
If you are interested in the project or in contributing, please contact me.
