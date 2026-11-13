# RingMinusOne

**Nothing escapes.** Below the OS. Beyond its reach.

We build **Umbra**, a bare-metal hypervisor. We also port the tools people already use so they work from underneath
Windows instead of through it.

[ringminusone.com](https://ringminusone.com/) · [Discord](https://ringminusone.com/go/discord/) · [Contact](https://ringminusone.com/contact/)

> [!IMPORTANT]
> **Umbra is in private beta.** It isn't on sale yet. To apply as a tester, open a ticket on our
> [Discord](https://ringminusone.com/go/discord/), choose **Apply as Tester**, and tell us your Windows version,
> motherboard and CPU.

## The hypervisor

Umbra is a Type-1 hypervisor. It starts before Windows and runs at ring −1, underneath the kernel, on AMD SVM. Your
existing Windows install boots on top of it, from its own SSD, inside a machine Umbra controls completely.

- **A whole virtual platform.**
  - **Firmware:** a complete UEFI environment, emulated from SEC to the boot manager.
  - **TPM:** a full TPM 2.0 in software.
  - **Secure Boot:** keys, databases and signatures emulated end to end.
  - **IOMMU:** a virtual one, whose DMA remapping the guest can verify for itself.
  - **Disk and GPU:** the disk goes through a virtual controller; the graphics card is passed through and fenced by the
    IOMMU.
- **Out of Windows' reach.** Umbra's memory, its sealed store and the physical TPM sit outside anything Windows can
  address. The sealed store is a 128 MiB private partition for TPM state and firmware variables.
- **Hyper-V nested inside, with VBS and HVCI on.** Umbra hides the hypervisor bit and CPUID leaves and handles AMD-V
  for Hyper-V. Hyper-V is never told it has company.
- **Memory, seen from below.**
  - It reads and writes pages.
  - It resolves a process's address space from its CR3 page-table root.
  - It traps writes or execution from beneath the guest's own page tables.
  - **Cloak:** everyone else reads a decoy; only your process sees what's really there.
- **A gate.** Only trusted programs get in: a measured binary, a clean process, an elevated session. That is why the
  tools below talk to Umbra through a tiny trusted broker instead of directly.

**Requirements:**

- AMD CPU with SVM and an IOMMU (Intel is coming soon).
- Windows 10 22H2 or Windows 11 25H2.
- UEFI, an NVMe SSD and a dedicated GPU.
- Secure Boot and BitLocker off; SVM and IOMMU on.

## Projects

| Project | What it is | Source |
|---|---|---|
| **[cheat-engine](https://github.com/RingMinusOne-hv/cheat-engine)** | Cheat Engine with Umbra as its memory backend and debugger. See the list below. | public |
| **[x64dbg](https://github.com/RingMinusOne-hv/x64dbg)** | x64dbg with **HV Mode**. See the list below. | public |
| **Kratos** | Disassembler, decompiler, graph view and debugger, with packer and compiler detection, an in-app AI agent, an MCP server and embedded Python. See the list below. [More on the site](https://ringminusone.com/). | private for now |

**cheat-engine** runs scans, reads, writes, "find what writes / accesses", breakpoints, stepping, native trace and
pause through the hypervisor. It uses no `ReadProcessMemory`, no handle, no driver and no debug port.

**x64dbg** with HV Mode attaches without a debug port. It uses no `int3`, no debug registers and no handle on the target.
Its breakpoints, stepping, memory breakpoints, pause and run trace are all held by the hypervisor.

**Kratos** is free. With Umbra, its debugger works on any process, a driver or the kernel, with no handle and no debug
port.

In the repositories the hypervisor goes by its codename, **turing-hv**. Umbra's own source is not public.
