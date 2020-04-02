DomU Memory, MMIO, and Interrupts
=================================

This document details where normal memory and MMIO regions of devices
are located in the address space of a DomU. It also details interrupts
available, if any. The most relevant header file is
xen/include/public/arch-arm.h in xen.git.


## Normal Memory

The first bank is up to 3GB in size and starts at 0x40000000. The second
bank is up to 1016GB in size and starts at 0x0200000000.

--------------------------      ----------------------------
First bank (up to 3GB)          0x40000000 - 0x100000000    
Second bank (up to 1016GB)      0x0200000000 - 0x10000000000
--------------------------      ----------------------------


## Devices

### GIC

Xen provides to DomUs an interrupt controller compatible with the Arm
Generic Interrupt Controller (GIC). The GIC version exposed to the DomU
matches the version of the physical GIC on the platform: if the hardware
is GICv2 a domU will get a GICv2; if the hardware is GICv3 a domU will
get a GICv3.

#### GICv2

---------    -----------------------
GICD MMIO    0x03001000 - 0x03002000
GICC MMIO    0x03002000 - 0x03004000
---------    -----------------------

#### GICv3

---------    -----------------------
GICD MMIO    0x03001000 - 0x03011000
GICR MMIO    0x03020000 - 0x04020000
---------    -----------------------

### Arch Timer

A timer compatible with Arm Arch Timer is exposed to all DomUs. Both the
virtual timer and the physical timer interfaces are provided. Only the
CP15 and SYSREG interfaces are made avialble to DomUs, no MMIO regions.

--------------  --
virt_timer PPI  27
phys_timer PPI  30
--------------  --

### PL011

Xen can provide a PL011 UART to DomUs, including its MMIO region and
interrupt. It is not enabled by default.

----    -----------------------
MMIO    0x22000000 - 0x22001000
SPI     32
----    -----------------------
