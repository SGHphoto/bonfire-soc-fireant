# Bonfire RISC-V processor on FireAnt

This repo contains an implementation of [Bonfire Basic SOC](https://github.com/bonfireprocessor/bonfire-basic-soc)  for the [FireAnt FPGA](https://www.crowdsupply.com/xips-technology/fireant) board. The FireAnt contains a Efinix Trion T8 FPGA. This FPGA is not well suited for Soft CPU implementations, because it is quite limited in Block RAM. But for a proof of concept it is sufficent.

## Features
  * [Bonfire  CPU Core](https://github.com/bonfireprocessor/bonfire-cpu), supporting RV32IM and a subset of RISC-V privileged
  * 1 UART connected to pins P16 (rxd) and P17 (txd) of the FireAnt
  * GPIO Port connected to the 4 LEDs of the FireAnt  
  * 25Mhz CPU Clock (generated by PLL)

  ### Ressource utilization on the T8 FPGA


  | Ressource      | used | max  |
  |----------------|------|------|
  | Logic Elements | 4203 | 7384 |
  | Memory Blocks  | 21   | 24   |
  | Multipliers    | 4    | 8    |
  | Clock Networks | 1    | 15   |


  The FireAnt contains only a single channel FTDI232 chip which is used to configure the FPGA or load the Flash with SPI. In principle the FTDI232 can also work as UART and the FPGAs pins can be mapped accordingly, but switching the FTDI to UART mode will hold the FPGA in continous reset (because of the pin mapping of the FTDI DTR line to the FPGA Reset pin).
  For this Reason the UART pins are mapped to the board headers P16 and P17, and can be wired to any  UART breakout board with 3.3V levels(I use a Digilent PMOD UART).

  The supplied demo program can also be tested without a UART connection, it animates the LEDs on the FireAnt.

## Building
The Repo contains a complete project for the Efinix IDE. So just git clone, open the project file (bonfire_basic_soc.xml) and run the complete build flow in the Efinix IDE.

## Running
The bitstream can be loaded to the FireAnt with the Efinix Programmer either in "SPI passive" mode (volatile, just loaded to the FPGA) or in "SPI active mode" (non-volatile, loaed to flash). Active mode often requires pressing the Reset button of the FireAnt after loading.
### LEDs
The LEDs will show the bit pattern 1--1 for a short moment, and then a "running light" of one LED is shown.
### UART
UART runs with 38400 Baud.
On Reset the following information is emitted:
```
MIMPID: 0001001f                                                                
MISA: 40001100                                                                  
UART Divisor: 649  
```

Then after every run trough the 4 LEDs the uptime of the board in seconds is emitted.

## Directory structure and hints for changing the project
```


├── local_src
├── outflow
├── src
│   ├── bonfire-basic-soc_0
│   │   ├── compiled_code
│   │   ├── memory
│   │   └── tb
│   ├── bonfire-cpu_0
│   │   └── rtl
│   ├── bonfire-dcache_0
│   │   └── spartan6
│   ├── bonfire-gpio_0
│   ├── bonfire-soc-io_0
│   ├── bonfire-spiflash_0
│   │   └── spi
│   ├── bonfire-util_0
│   └── zpuino-uart_0
├── work_pnr
└── work_syn
```
(outflow, work_pnr and work_syn directories are not part of the repo, they are generated by the Efinix IDE)
Everything below src is the rtl form the Bonfire project. Bonfire uses FueseSoC for building, because there is no FuseSoc support for Efinix yet, I have just copied a  generated FuseSoC build tree as a whole into src. It contains everything to build every configuration of [bonfire-basic-soc](https://github.com/bonfireprocessor/bonfire-basic-soc), so there are also source files for things wich are not active in the FireAnt image now.

compiled_code contains a Hex dump of the sim_hello program from  [bonfire-software](https://github.com/bonfireprocessor/bonfire-software)
Unfortunately I had no time yet to document how to build code from bonfire-software.

local_src contains the toplevel file for the FireAnt implementation.

## FireAnt specifics
The Block RAMs of the FireAnt does not have byte select write lines, and instantiating Memory from the standard "MainMemory.vhd" File does not work. Therefore in the memory subdirectory an implementation of "Laned" Memory is provided, which defines 4  ram8 (n*8 Bits) instances for a 32 Bit word.
The default configuration is 8KBytes of RAM, synthesis maps this to  16 2048 x 2 Bit BRAMs.

Currently there are no optimizations for the Trion FPGAs. While the Bonfire-CPU easily runs with around 100 Mhz on a Spartan-6, the maximum Frequency on the T8C2 is ~30- 32Mhz.
The 25Mhz of the default configuration is on the safe side.

FirAnt does not use HDL primtives to define I/O blocks or PLLs, instead the Interface designer is used. Refer to Efinix documentation for details. The toplevel ports sysclk, o_resetn and i_locked are used to interface the PLL.
