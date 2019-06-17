# Intro

This document covers the external interfaces in a dom0less deployment
that we need to document for safety certifications.

Dom0 and Xen userspace tools, such as xl and libxl, are out of scope for
now, we are only discussing the hypervisor. vm_assist, xsm, argo are
also out of scope: the assumption is that they will disabled via
kconfig.


# From the user perspective

- Xen hypervisor command line options
- Dom0less device tree configuration


# Xen - Bootloader interfaces

- multiboot
- Xen boot protocol


# From a DomU perspective

- kernel image format
- boot protocol
- device tree
- memory map: location of memory and other resources
- exposed devices
  - GIC interrupt controller
  - Generic Timer
  - virtual UART (PL011)
- PSCI
- hypercall protocol (i.e. registers, etc.)
- memory sharing (i.e. memory/cache attributes)


# Hypercalls exposed to all DomUs

These hypercalls are unused by dom0less domUs, however, they are
still exposed to all domUs.

- memory_op
- sched_op
- xen_version
- hvm_op
- multicall
- platform_op
- vcpu_op
- physdev_op (NOP on Arm)


# PV drivers interfaces

Dom0less domUs cannot use PV drivers today. However, these interfaces
are still exposed to them, and one day they might be able to use them
correctly:

- hypercalls
  - console_io
  - grant_table_op
  - event_channel_op
- xenstore
  - xenstore initialization
  - xenstore protocol
    - message format
    - notifications
  - xenstore as a bus
- PV protocols
  - PV console
  - PV network
  - PV block
  

# Privileged hypercalls (dom0 and toolstack)

- domctl
- sysctl
