# Intro

What are the interfaces that we need to fully document in a dom0less
deployment for safety certifications?

Dom0 and Xen userspace tools, such as xl and libxl, are out of scope for
now, we are only discussing the hypervisor. vm_assist, xsm, argo are
also out of scope.


# From the user perspective

- Xen hypervisor command line options
- Dom0less device tree configuration


# From a DomU perspective

- kernel and ramdisk image format
- boot protocol
- device tree
- memory map: location of memory and other resources
- exposed devices
  - GIC interrupt contoller
  - Generic Timer
  - virtual UART (PL011)
- PSCI


# Hypercalls exposed to all DomUs

These hypercalls are unused by dom0less DomUs, however, they are
still exposed, at least partially, to all domUs.

- memory_op
- sched_op
- xen_version
- hvm_op
- multicall
- platform_op
- vcpu_op


# PV drivers interfaces

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
