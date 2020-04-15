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
Generic Interrupt Controller [GIC]. The GIC version exposed to the DomU
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

A timer compatible with [Arm Arch Timer][TIMER] is exposed to all DomUs.
Both the virtual timer and the physical timer interfaces are provided.
Only the CP15 and SYSREG interfaces are made avialble to DomUs, no MMIO
regions.

--------------  --
virt_timer PPI  27
phys_timer PPI  30
--------------  --

### PL011

Xen can provide a UART [PL011] to DomUs, including its MMIO region and
interrupt. It is not enabled by default.

----    -----------------------
MMIO    0x22000000 - 0x22001000
SPI     32
----    -----------------------

## PSCI

The Power State Coordination Interface [PSCI] is a standard interface
for power management. Rich operating systems, like Linux and Windows,
use PSCI for CPU and overall system power management.

Xen exposes a virtual PSCI interface to DomUs for them to bring up
secondary virtual CPUs. PSCI supports multiple transports, Xen chooses
the `hvc` instruction as transport for DomUs.


## Device Tree

The information listed above is also advertised via [device tree][DT].
At boot time, a device tree binary (FDT binary blob) is mapped into domU
memory. The address in guest memory of the binary is passed to the domU
kernel via the `x21` register.

The following is a sample device tree for a DomU, converted to device
tree source (DTS) form:


    /dts-v1/;

    / {
            #address-cells = <0x2>;
            model = "XENVM-4.13";
            #size-cells = <0x2>;
            interrupt-parent = <0xfde8>;
            compatible = "xen,xenvm-4.13", "xen,xenvm";

            interrupt-controller@3001000 {
                    #address-cells = <0x0>;
                    compatible = "arm,cortex-a15-gic", "arm,cortex-a9-gic";
                    #interrupt-cells = <0x3>;
                    reg = <0x0 0x3001000 0x0 0x1000 0x0 0x3002000 0x0 0x2000>;
                    phandle = <0xfde8>;
                    linux,phandle = <0xfde8>;
                    interrupt-controller;
            };

            memory@40000000 {
                    device_type = "memory";
                    reg = <0x0 0x40000000 0x0 0x64000000>;
            };

            psci {
                    method = "hvc";
                    compatible = "arm,psci-1.0", "arm,psci-0.2", "arm,psci";
                    cpu_on = <0x2>;
                    cpu_off = <0x1>;
            };

            timer {
                    interrupts = <0x1 0xd 0xf08 0x1 0xe 0xf08 0x1 0xb 0xf08>;
                    interrupt-parent = <0xfde8>;
                    compatible = "arm,armv8-timer";
            };

            chosen {
                    linux,initrd-end = <0x0 0x57774000>;
                    bootargs = "console=hvc0 root=/dev/ram0";
                    linux,initrd-start = <0x0 0x48000000>;
            };

            cpus {
                    #address-cells = <0x1>;
                    #size-cells = <0x0>;

                    cpu@0 {
                            device_type = "cpu";
                            compatible = "arm,armv8";
                            reg = <0x0>;
                            enable-method = "psci";
                    };
            };
    };


The number of CPUs and the amount of memory vary depending on the
virtual machine configuration. The version of the GIC also varies and
depends on the underlying physical hardware as well as the virtual
machine configuration. The `chosen` node contains the location of the
Linux initrd in guest memory and the DomU kernel command line arguments
as specified in the virtual machine configuration file.

[GIC]: https://developer.arm.com/ip-products/system-ip/system-controllers/interrupt-controllers
[TIMER]: https://developer.arm.com/architectures/learn-the-architecture/generic-timer/what-is-the-generic-timer
[PL011]: http://infocenter.arm.com/help/topic/com.arm.doc.ddi0183f/DDI0183.pdf
[PSCI]: https://developer.arm.com/architectures/system-architectures/software-standards/psci
[DT]: https://www.devicetree.org/
