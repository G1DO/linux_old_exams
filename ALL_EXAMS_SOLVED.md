# Embedded Linux — All Old Final Exams, Solved & Explained

**Course:** CSY441 "Recent Topics in Computer Systems" — Ain Shams University, Faculty of Computer & Information Sciences.
**Generated:** 2026-06-05. Worked solutions for all six past papers, written to teach each concept from scratch and grounded in the nine course lectures.

> **How these answers were decided.** Ground truth = the course **lecture** PDFs (via slide-cited study notes). The two files named `*_Solution.pdf` in the exam folder are **AI-generated, not official keys** (each is footer-stamped "prepared by an AI tool"); they were used only as a cross-check. Wherever the AI solution disagreed with the lectures, the **lecture answer is used** and the block carries a `[KEY vs LECTURE]` note.
>
> **Two things to know:** (1) `Final Exam.pdf` is **identical** to the `Linux_23` paper, so its section repeats those solutions. (2) A few items (shared memory, `fork`/`exec`/`wait`, IPC comparison) come from a **Processes/IPC lecture that is not in the course folder**; those answers are standard POSIX/Linux and are marked `[DERIVED — verify]`.

## Table of Contents

| # | Exam | Paper | Questions |
|---|------|-------|:--:|
| 1 | [Linux_23](#exam-linux-23) | Final 10/6/2023 (Dr. Karim Emara) | 22 |
| 2 | [Linux_24](#exam-linux-24) | Final 9/6/2024 (Prof. Eman Shaaban & Dr. Karim Emara) | 22 |
| 3 | [Linux_25](#exam-linux-25) | Final 27/5/2025 (Prof. Said Ghoniemy & Dr. Karim Emara) | 19 |
| 4 | [Final Exam](#exam-final) | = Linux_23 (duplicate) | 22 |
| 5 | [Sample 1](#exam-sample-1) | Spring 2023 sample | 21 |
| 6 | [Sample 2](#exam-sample-2) | Spring 2023 sample 2 | 20 |

**Total: 126 question blocks** (104 unique + 22 in the Final Exam duplicate).

---

<a id="exam-linux-23"></a>

## Linux_23 — CSY441 Final Exam (10/6/2023, Dr. Karim Emara) — Ain Shams FCIS

Total questions: 22

*Exam header: FCIS – Ain Shams University; CSY441: Recent Topics in Computer Systems; Final 10/6/2023; Level 3rd & 4th; Examiner: Dr. Karim Emara; Academic year 2nd term 2022–2023; duration 2 hours. "Answer the following 4 questions."*

---

### Q1.1. Choose the correct answer [7 marks, 10 min]. The ___ is the part of the Linux system that is responsible for loading kernel into memory after system reset and firmware initialization. a) Init program  b) Kernel program  c) Bootloader  d) Board firmware

**Answer:** c) Bootloader

**Explanation:**
When you power on an embedded board, the hardware cannot just "run Linux" — there is a fixed chain of hand-offs, each stage bringing more of the system to life:

1. **Board firmware / ROM code** — tiny code baked into the SoC. It runs first, initializes the bare minimum (clocks, a little on-chip SRAM), and fetches the next stage. It does *not* know what Linux is.
2. **Bootloader** (e.g. U-Boot) — this is the stage that "knows about the kernel." Its job: bring the hardware to a known state, set up RAM, place the kernel parameters (device tree + command line) in memory, then **load the kernel image into RAM and jump to it**.
3. **Kernel** — once loaded and running, it mounts the root filesystem and starts the init program.
4. **Init program** (PID 1) — the first user-space process; it launches all the system daemons and services.

So the component whose specific responsibility is *loading the kernel into memory after reset and firmware init* is the **bootloader**.

- a) Init program — wrong: init is started *by* the kernel, long after the kernel is already in memory. It cannot load the thing that starts it.
- b) Kernel program — wrong: the kernel is the *thing being loaded*; it cannot load itself into memory.
- d) Board firmware — wrong: firmware runs first and only initializes the SoC and loads the *next bootstrap stage*; it is not the component that loads the Linux kernel.

*Source: Lecture 03 — Bootloader (role: init HW, place kernel parameters, load kernel & jump).*

---

### Q1.2. The root filesystem is mounted to the system through the ____. a) mount command  b) root= kernel option  c) bootloader ROM Code  d) init program

**Answer:** b) root= kernel option

**Explanation:**
The **root filesystem (rootfs)** is the top-level "/" directory tree that holds everything user space needs: programs, libraries, configuration, and device nodes. There is a chicken-and-egg problem: the normal way to attach a filesystem is the `mount` command — but `mount` is itself a program that lives *inside* the root filesystem. You cannot run a program from a filesystem that is not mounted yet.

Linux solves this by having the **kernel mount the very first filesystem itself**, before any user-space program runs. The kernel needs to be told *which* device/partition holds the rootfs, and that is exactly what the **`root=` kernel command-line option** does, e.g. `root=/dev/mmcblk0p2`. If `root=` points at the wrong place (or nothing), the kernel cannot find an init program and **panics**.

- a) mount command — wrong: `mount` lives inside the rootfs, so it cannot be used to mount the rootfs itself (it is only used for *additional* filesystems afterward).
- c) bootloader ROM Code — wrong: ROM code runs far earlier and only bootstraps the next stage; it has no concept of a Linux filesystem.
- d) init program — wrong: init is started *after* the rootfs is already mounted; mounting rootfs is a precondition for running init.

*Source: Lecture 05 — Root filesystem (rootfs mounted directly by the kernel per the `root=` option).*

---

### Q1.3. In the filesystem hierarchy standard, the _____ directory stores the devices nodes defined for the hardware attached to the system. a) /sys  b) /driver  c) /bin  d) /dev

**Answer:** d) /dev

**Explanation:**
The **Filesystem Hierarchy Standard (FHS)** is the agreed-upon convention for what each top-level directory in a Linux system contains, so software knows where to find things. Key ones:

- **/dev** — **device nodes**. A device node is a *special file* that represents a piece of hardware (or a virtual device). Opening/reading/writing that file is how user space talks to the driver. Each node carries a type (`c` = character, `b` = block) plus a **major** number (which driver) and **minor** number (which instance/partition). Example: `/dev/ttyS0`, `/dev/null`, `/dev/mmcblk0`.
- /bin — essential user programs (ls, cp, sh).
- /etc — configuration files.
- /lib — shared libraries.
- /sys — exposes the kernel's *driver model* (sysfs), not the device nodes themselves.
- /proc — exposes *process* and kernel info.

So the directory holding the **device nodes** is **/dev**.

- a) /sys — wrong: sysfs exposes kernel/driver-model *information* (attributes, hierarchy), not the actual device-node special files used for I/O.
- b) /driver — wrong: there is no such standard FHS directory.
- c) /bin — wrong: /bin holds executable programs, not device nodes.

*Source: Lecture 05 — FHS / device nodes (/dev).*

---

### Q1.4. The system V init program begins reading the ______ file which defines rules to start programs at boot up and stop them at shutdown. a) /conf/init.d  b) /sbin/init  c) /sbin/boot  d) /etc/inittab

**Answer:** d) /etc/inittab

**Explanation:**
**System V init** (and the BusyBox init used on embedded systems) is the first user-space process, PID 1. To know *what* to start and stop, it reads a configuration file at boot: **`/etc/inittab`**.

Each line in `/etc/inittab` has the format `<id>:<runlevels>:<action>:<process>`, telling init which program to run and *how*:
- `sysinit` — run this before everything else (typically `::sysinit:/etc/init.d/rcS`, which runs the boot-time shell commands).
- `respawn` — keep this running; restart it if it dies (e.g. a getty login prompt).
- `wait` / `once` — run once.

So init **begins by reading `/etc/inittab`**, which defines the rules for starting programs at boot and stopping them at shutdown.

Note the classic trap: `/sbin/init` is the init *executable itself*, not its config file. And the *shell commands* to run at boot go in `/etc/init.d/rcS`, which is *invoked from* inittab — but the file init **reads first** is inittab.

- a) /conf/init.d — wrong: not a standard path; the real script directory is `/etc/init.d/`.
- b) /sbin/init — wrong: that is the init *binary*, not the rules file it reads.
- c) /sbin/boot — wrong: no such standard file.

*Source: Lecture 06 — Init (reads /etc/inittab; entry format `id:runlevels:action:process`).*

---

### Q1.5. In block devices, the minor number identifies the ______ of the device. a) partition  b) device driver  c) interface  d) none of the above

**Answer:** a) partition

**Explanation:**
Every device node carries two numbers:
- **Major number** — identifies the *driver / device class* (which kernel driver handles this device). Same for all instances served by that driver.
- **Minor number** — identifies *which specific thing* within that driver's domain.

What the minor number "means" depends on the **type** of device:
- **Block devices** (mass storage: SD cards, disks): the minor number identifies the **partition**. Example: `mmcblk0` (major 179) → minor 0 = the whole device, and the following minors = `mmcblk0p1`, `mmcblk0p2`, … the individual partitions.
- **Character devices**: the minor number identifies the *instance/interface* (e.g. which serial port).

The question specifically says *block devices*, so the minor number = **partition**.

- b) device driver — wrong: that is what the **major** number identifies, not the minor.
- c) interface — wrong: "interface/instance" is the meaning of the minor number for **character** devices, not block devices (this is the deliberate distractor — note the 2025 paper flipped the question to the character case).
- d) none of the above — wrong: "partition" is exactly correct for block devices.

*Source: Lecture 08 — Major/minor numbers (block minor = partition).*

---

### Q1.6. If the system has no MMU, the best C library can be used is _____. a) musl libc  b) libm  c) uClibc-ng  d) glibc

**Answer:** c) uClibc-ng

**Explanation:**
The **C library** is the layer between your program and the kernel's system calls (it provides `printf`, `malloc`, `open`, etc.). On embedded systems you pick the C library to match the hardware and size constraints.

An **MMU (Memory Management Unit)** is the hardware that provides virtual memory — separate address spaces per process, `fork()` with copy-on-write, etc. Some small microcontrollers/SoCs have **no MMU**; these run a variant of Linux historically called **uClinux**. Without an MMU you need a C library designed to work in that constrained, no-virtual-memory environment.

The decision flowchart taught in the course:
- **No MMU (uClinux)** → **uClibc-ng** ← (this question)
- Has MMU but very small RAM (< ~32 MiB) → **musl**
- Otherwise (general purpose, full features) → **glibc**

So for a **no-MMU** system, the best fit is **uClibc-ng**.

- a) musl libc — wrong: musl is the pick for *small-RAM but MMU-equipped* systems, not the no-MMU case.
- b) libm — wrong: `libm` is just the *math* library (sin, cos, sqrt) that accompanies a C library; it is not a full C library and is the obvious distractor.
- d) glibc — wrong: glibc is large and feature-rich, intended for general systems *with* an MMU; it is the worst fit for a tiny no-MMU target.

*Source: Lecture 02 — Toolchain / C library selection (no-MMU → uClibc-ng).*

---

### Q1.7. In JFFS2 filesystem, the ____ block contains only valid nodes. a) Free  b) Erased  c) Clean  d) Ful

**Answer:** c) Clean

**Explanation:**
**JFFS2** is a **log-structured** flash filesystem. Instead of overwriting data in place (which raw flash cannot do — flash must be erased a whole block at a time before rewriting), JFFS2 **appends** every change as a new "node" to the log. When a file is updated, the *old* node becomes **obsolete** and a new node is written elsewhere.

Because of this, JFFS2 classifies each erase block by what it currently holds:
- **Free** — empty, contains **no nodes** at all; ready to be written.
- **Clean** — contains **only valid (still-in-use) nodes**, no obsolete ones. ← this question
- **Dirty** — contains **at least one obsolete node** (wasted space that garbage collection can reclaim).
- **Open block** — the single block currently being appended to (receiving new writes).

When the filesystem runs low on free space, a kernel garbage-collection thread copies the valid nodes out of *dirty* blocks into the open block, then erases the dirty block to make it free again (this also provides simple wear leveling).

So the block that contains **only valid nodes** is the **Clean** block.

- a) Free — wrong: a free block contains *no* nodes, not "only valid" ones.
- b) Erased — wrong: not a JFFS2 block category; an erased block is effectively a free block (no nodes).
- d) Ful (Full) — wrong: not a JFFS2 block state; fullness alone says nothing about whether nodes are valid or obsolete.

*Source: Lecture 09 — JFFS2 blocks (Free / Clean / Dirty + open block).*

---

### Q2.1. Complete the following sentences by a suitable word/phrase [12 marks, 15 min]. The Linux command that removes all files under a directory dir1 is ____ .

**Answer:** `rm -rf dir1` (equivalently `rm -r -f dir1`)

**Explanation:**
`rm` is the Linux command to *remove* (delete) files. By itself, `rm` only deletes plain files and refuses to touch directories. To delete an entire directory and everything inside it, you combine two flags:

- **`-r`** (recursive) — descend into the directory and delete its contents (subdirectories and files), then the directory itself. This is what makes it remove *all files under* `dir1`.
- **`-f`** (force) — do not prompt for confirmation, and do not error out on missing/write-protected files. This makes the deletion non-interactive.

So `rm -rf dir1` wipes `dir1` and its whole subtree in one shot. (Order of flags does not matter; `-rf`, `-fr`, or `-r -f` are all equivalent.)

Caution worth teaching: `rm -rf` is irreversible — there is no recycle bin. Running it on the wrong path (especially `/`) is the classic catastrophic mistake.

*Source: Lecture 01 — Basic shell commands (`rm -r -f`).*

---

### Q2.2. The bootloader part that loads another small chunk of code from preprogrammed locations into SRAM is called ____ while the part that actually loads the kernel into the DRAM is called ___.

**Answer:** Blank 1 = **ROM code** ; Blank 2 = **TPL** (the full/third-stage bootloader, i.e. full U-Boot).

`[KEY vs LECTURE]` The AI-generated solution wrongly put **SPL** in blank 1. Per the lecture, the stage that "loads a small chunk of code from preprogrammed locations into **SRAM**" is the **ROM code** — the SPL is what *gets loaded into SRAM*, it is not the loader described here.

**Explanation:**
Boot on a real SoC is a multi-stage relay because, right after reset, the large external **DRAM** is not yet usable (its controller has not been configured). Only a small on-chip **SRAM** is available. So the boot proceeds in stages, each one setting up enough hardware to load the next:

1. **ROM code** — permanently burned into the SoC's mask ROM; the very first code to run. DRAM is *not* available yet, so it loads a **small chunk of code from preprogrammed locations into on-chip SRAM**. That small chunk is the SPL. → **blank 1**
2. **SPL** (Secondary Program Loader) — runs from SRAM; its main job is to **set up the DRAM/memory controller** so DRAM becomes usable, then load the next, larger stage into DRAM.
3. **TPL / full U-Boot** (Tertiary Program Loader) — now running from DRAM, this is the full-featured bootloader; it **loads the kernel image (plus the flattened device tree) into DRAM** and jumps to it. → **blank 2**

Disambiguator to remember: identify each stage by *who does the loading and into which memory*. "Into SRAM" ⇒ ROM code did it. "Loads the kernel into DRAM" ⇒ the full bootloader (TPL).

*Source: Lecture 03 — Boot phases (ROM code → SPL → TPL).*

---

### Q2.3. In device trees, the ____ property is an empty property that declares a node as a device that receives interrupt signals.

**Answer:** **interrupt-controller**

`[KEY vs LECTURE]` The AI-generated solution wrongly answered **interrupt-parent**. That is incorrect: `interrupt-parent` is *not* empty — it takes a **phandle** value pointing at the controller. The property that is an **empty** flag declaring a node *receives* interrupts (i.e. it *is* an interrupt controller) is **`interrupt-controller`**.

**Explanation:**
A **device tree (DT)** is a data structure that describes the hardware to the kernel (since non-discoverable hardware like memory-mapped peripherals can't be probed automatically). Interrupt wiring is described with a small family of properties:

- **`interrupt-controller`** — an **empty** (boolean/flag) property. Placing it in a node declares "this node is an interrupt controller — it *receives* interrupt signals from other devices." Because it is just a flag, it has no value (written simply as `interrupt-controller;`). ← this question
- **`#interrupt-cells`** — how many 32-bit cells make up one interrupt specifier for this controller.
- **`interrupt-parent`** — a **phandle** (a reference) pointing to the controller that a device's interrupts are routed to. (Has a value → not empty.)
- **`interrupts`** — on a device node, the list of interrupt specifiers describing which line(s) it uses.

So the *empty* property that marks a node as a receiver of interrupts is **`interrupt-controller`**.

*Source: Lecture 03 — Device tree interrupt properties.*

---

### Q2.4. The ___ feature in the flash translation layer maximizes the lifespan of a chip by making each block is erased roughly the same number of times.

**Answer:** **wear leveling**

**Explanation:**
Flash memory cells **wear out**: each erase block can only survive a limited number of erase/program cycles (e.g. tens of thousands) before it fails. If some blocks were erased far more often than others, those "hot" blocks would die quickly while the rest of the chip is still fresh — wasting most of the chip's life.

**Wear leveling** is the **Flash Translation Layer (FTL)** feature that spreads erase/write activity **evenly across all blocks**, so every block is erased roughly the same number of times. This maximizes the overall lifespan of the chip.

It works by decoupling the *logical* address the OS uses from the *physical* block actually written: the FTL remaps writes to less-used blocks and migrates data around. (Other FTL features — distractors — include sub-allocation, garbage collection, bad-block handling, and robustness against power loss.)

*Source: Lecture 09 — Flash Translation Layer (wear leveling).*

---

### Q2.5. In root filesystem, device nodes can be created manually using ______ or automatically on demand using ______.

**Answer:** Manually = **`mknod`** ; Automatically on demand = **udev** (or its embedded equivalents **mdev** / **devtmpfs**).

**Explanation:**
A **device node** is the special file in `/dev` that maps a name to a `(type, major, minor)` triple so user space can reach a driver. There are two ways these nodes come into existence:

- **Manually — `mknod`**: you create each node by hand, e.g. `mknod /dev/null c 1 3` (type `c` = character, major 1, minor 3). This is static: you must know every device in advance and create it explicitly. Fine for a tiny fixed embedded rootfs (which needs at least `console` and `null`).
- **Automatically on demand — udev / mdev / devtmpfs**: the kernel announces hardware as it appears (and disappears), and a manager **creates and removes the matching `/dev` nodes dynamically**. **udev** is the full desktop solution; **mdev** is BusyBox's lightweight version; **devtmpfs** is a kernel-populated `/dev` filesystem. These handle hot-plug (e.g. plugging in a USB stick) without you pre-creating anything.

So: manual = **mknod**, automatic-on-demand = **udev/mdev/devtmpfs**.

*Source: Lecture 05 — Device nodes (manual `mknod` vs automatic udev/mdev/devtmpfs).*

---

### Q2.6. In yocto project, BitBake is the task scheduler that parses the ______ and _____ to create a dependency tree and execute the building process.

**Answer:** **recipes** and **configuration files** (together these make up the *metadata*).

**Explanation:**
The **Yocto Project** builds a complete custom Linux distribution from source. Its build engine is **BitBake** — think of it as a much more powerful `make`. BitBake itself doesn't hard-code how to build anything; instead it reads **metadata** that you and the layers provide:

- **Recipes** (`.bb` files) — each recipe is "a list of settings and tasks (instructions) for building **one package/component**": where to fetch the source, how to configure, compile, and install it (`do_fetch`, `do_unpack`, `do_configure`, `do_compile`, `do_install`, …).
- **Configuration files** (`.conf`) — global/user settings and hardware (machine) configuration: which packages to include, target architecture, build options, etc.

BitBake **parses the recipes and configuration files**, works out all the inter-package dependencies into a **dependency tree**, and then **executes the tasks in order** to download, configure, build, and package everything into the final images.

So the two things BitBake parses are **recipes** and **configuration files**.

*Source: Lecture 07 — Yocto / BitBake (parses recipes + configuration files → dependency tree).*

---

### Q2.7. In Kconfig language, to define a config option that can be compiled as a kernel module, it should be defined as ____ option type.

**Answer:** **tristate**

**Explanation:**
The Linux kernel is configured through **Kconfig**, the language behind `make menuconfig`. Each configurable feature is a *config option* with a **type** that determines what values it can take:

- **bool** — two states: **y** (built **into** the kernel) or **n** (left out). On/off only.
- **tristate** — **three** states: **y** (built-in), **m** (compiled as a loadable **kernel module**, a separate `.ko` file you can insert/remove at runtime), or **n** (not built). ← this question
- Other types: `string`, `int`, `hex` for textual/numeric values.

Because building a feature as a **module** is a distinct third choice (`m`), any option that *can* be compiled as a module **must** be declared **tristate** — `bool` does not offer the `m` option.

*Source: Lecture 04 — Kconfig (tristate = y / m / n).*

---

### Q2.8. The purpose of the ___ and ____ pseudo filesystems is to expose information about processes and kernel driver model to user space, respectively.

**Answer:** **/proc** and **/sys** (sysfs) — respectively.

**Explanation:**
These are **pseudo (virtual) filesystems**: they contain no data on disk. Instead, the kernel generates their "files" on the fly, so reading/writing them is actually a window into kernel internals. The word **"respectively"** in the question fixes the order:

- **/proc** (procfs) — exposes information about **processes** (and assorted kernel/system info). Each running process appears as `/proc/<PID>/` with details about it; files like `/proc/cpuinfo`, `/proc/meminfo` give system info. → matches "information about **processes**".
- **/sys** (sysfs) — exposes the kernel's **driver/device model**: the hierarchy of devices, buses, drivers, and their attributes (e.g. the GPIO interface lives under `/sys/class/gpio`). → matches "kernel **driver model**".

So, respectively: **/proc** (processes) and **/sys** (kernel driver model).

*Source: Lecture 04 — /proc vs /sys (processes vs kernel driver model, "respectively").*

---

### Q3.A.1. [20 marks, 60 min] For the following output of the `ls -l /bin/tracert` command — `-rwxr-xr-x 1 root root 35712 Feb 6 09:15 /bin/tracert` — answer: Is it allowed for the user john (not a root group member) to execute /bin/tracert? If so, what will be the effective UID of the process when john runs it? [5 marks]

**Answer:** **Yes**, john can execute it. The **effective UID of the process is `john`** (the invoking user) — there is no setuid bit, so the program simply runs with the privileges of whoever launched it.

**Explanation:**
A long listing's first field is the file type + **permission triad**:

```
-  rwx  r-x  r-x   1 root root ...
^   |    |    |
|  owner group others
type
```

Read it in three groups of three (`r`=read/4, `w`=write/2, `x`=execute/1):
- **owner** (`root`) → `rwx` — read, write, execute.
- **group** (`root`) → `r-x` — read, execute (no write).
- **others** (everyone else) → `r-x` — read, execute (no write).

Now classify **john**: he is *not* the owner (owner is root) and *not* in the file's group (the group is `root`, and the question says john is not a root-group member). So john falls into the **"others"** category, whose permission is `r-x` → **execute is allowed**. Hence **john can run /bin/tracert**.

**Effective UID:** the *effective UID* is the identity the kernel uses for permission checks while the process runs. Normally a program runs with the **effective UID of the user who started it**. Here there is **no setuid ("s") bit** on the owner triad (it shows plain `rwx`, not `rws`), so no privilege elevation happens. Therefore when john runs it, the process's effective UID = **john**.

*Source: Lecture 05 — File permissions / effective UID.*

---

### Q3.A.2. What will be the changes on the effective UID after the root execute the following command: `chmod 4755 /bin/tracert`?

**Answer:** `chmod 4755` sets the **setuid (SUID) bit**, so the listing becomes **`-rwsr-xr-x`**. From then on, **any** user who executes the file runs it with **effective UID = root (the file owner, UID 0)** instead of their own UID. (So when john runs it now, his process's effective UID becomes **root**, not john.)

**Explanation:**
`chmod 4755` is an octal mode with **four** digits:
- The leading **`4`** is the **special-permissions** digit, and `4` = the **setuid (SUID)** bit.
- The remaining **`755`** are the normal permissions: owner `rwx` (7), group `r-x` (5), others `r-x` (5).

When the SUID bit is set on an **executable owned by root**, the owner's execute slot is displayed as **`s`** instead of `x`:

```
-rwsr-xr-x   ← the 's' shows SUID is on
```

The meaning of SUID: when the program is launched, the kernel sets the process's **effective UID to the file's owner**, *regardless of who actually started it*. So the program runs with the **owner's** privileges. Since the owner here is **root (UID 0)**, every user — john included — who runs `/bin/tracert` now executes it with **effective UID root**.

(This is exactly how tools like `ping` and `traceroute` work: they need root-level access to raw network sockets, so they are installed setuid-root so ordinary users can run them. It is also why SUID-root binaries are a security-sensitive item.)

So the change: effective UID goes from "the invoking user" to **root (the owner)** for anyone who runs it.

*Source: Lecture 05 — SUID bit / effective UID elevation.*

---

### Q3.B.1. [15 marks] A push button is connected to a GPIO pin (gpio 48) in an embedded board and pulled down to ground. Write the shell commands required to make this pin an input and enable the rising-edge interrupt.

**Answer:**
```sh
echo 48 > /sys/class/gpio/export
echo in > /sys/class/gpio/gpio48/direction
echo rising > /sys/class/gpio/gpio48/edge
```

**Explanation:**
Linux exposes GPIO pins to user space through the **sysfs gpiolib interface** under `/sys/class/gpio`. You control a pin by writing plain text to files. The three steps:

1. **Export the pin** — `echo 48 > /sys/class/gpio/export`. This asks the kernel to hand control of GPIO number 48 to user space. The kernel responds by creating a directory `/sys/class/gpio/gpio48/` containing the control files (`direction`, `value`, `edge`, `active_low`).

2. **Set the direction to input** — `echo in > /sys/class/gpio/gpio48/direction`. Writing `in` configures the pin as an input so we can *read* the button state. (Inputs are actually the default after export, but it is good practice — and what the question asks — to set it explicitly. Writing `out` would make it an output.)

3. **Arm the rising-edge interrupt** — `echo rising > /sys/class/gpio/gpio48/edge`. The `edge` file controls *interrupt* (edge-detection) behavior; valid values are `none`, `rising`, `falling`, or `both`. Writing **`rising`** tells the kernel to flag the pin whenever its level transitions from **0 → 1**. This is what lets a `poll()` call in a program block efficiently and wake up exactly when the edge occurs (instead of busy-looping on `value`).

Context for *why rising* here: the button is **pulled down to ground**, so the line idles at logic **0**; pressing the button drives it to **1**. That press is a **rising edge (0 → 1)**, which is why we enable the **rising** edge.

*Source: Lecture 08 — GPIO via sysfs (export / direction / edge).*

---

### Q3.B.2. The following program should wait for the logic level on the button and print a message when the button is pressed, but parts [1]–[5] are missing. Write the lines that contain the missing parts with their line numbers.

Given program:
```c
1: int main(int argc, char *argv[])
2: {
3:   int f;
4:   struct pollfd poll_fds[1];
5:   int ret;
6:   char value[4];
7:   int n;
8:   f = ..[1]..;
9:   if (f == -1) { perror("Can't open gpio48"); return 1; }
10:  poll_fds[0].fd = f;
11:  ..[2]...;
12:  while (1) {
13:    printf("Waiting\n");
14:    ret = ..[3]..(poll_fds, 1, -1);
15:    if (ret > 0) {
16:      n = ..[4]..;
17:      printf("Button pressed: %d bytes, value=%c\n", n, value[0]);
18:      ..[5]..;
19:    }
20:  }
21:  return 0;
22: }
```

**Answer:**
```c
8:   f = open("/sys/class/gpio/gpio48/value", O_RDONLY);   /* [1] */
11:  poll_fds[0].events = POLLPRI | POLLERR;                /* [2] */
14:  ret = poll(poll_fds, 1, -1);                           /* [3] */
16:  n = read(f, &value, sizeof(value));                    /* [4] */
18:  lseek(f, 0, SEEK_SET);                                 /* [5] */
```

**Explanation:**
This is the standard pattern for **waiting on a GPIO edge with `poll()`** instead of busy-polling. Going line by line through the blanks:

- **[1] (line 8) — `f = open("/sys/class/gpio/gpio48/value", O_RDONLY);`**
  We open the pin's **`value`** attribute file (read-only) to get a file descriptor `f`. This is the file whose readable content is the pin's current logic level ('0' or '1'), and — crucially — the file `poll()` will watch. The check on line 9 (`if (f == -1)`) confirms blank [1] must be an `open()` that returns a descriptor or `-1`.

- **[2] (line 11) — `poll_fds[0].events = POLLPRI | POLLERR;`**
  `poll()` watches an array of `struct pollfd`; each entry needs `.fd` (set on line 10) and **`.events`** — the events we want to wait for. For a GPIO sysfs edge, the kernel signals readiness via **`POLLPRI`** ("there is exceptional/priority data" — this is *how a sysfs GPIO edge is reported*, not normal `POLLIN`), and we also watch **`POLLERR`**. So `.events = POLLPRI | POLLERR;`.

- **[3] (line 14) — `poll(poll_fds, 1, -1)`**
  The call itself is **`poll`**. Arguments: the array, `1` (one descriptor), and a timeout of **`-1`** which means **block forever** until an edge arrives. It returns `> 0` when the watched event fires (handled by the `if (ret > 0)` on line 15).

- **[4] (line 16) — `n = read(f, &value, sizeof(value));`**
  After `poll()` wakes us, we **`read`** the current level from the `value` file into the `value[]` buffer; `n` is the number of bytes read, and `value[0]` is the level character printed on line 17.

- **[5] (line 18) — `lseek(f, 0, SEEK_SET);`**
  This **rewinds** the file offset back to the beginning. A sysfs attribute is re-read from the start each time; after a `read()` the offset sits at end-of-file, so without `lseek(f, 0, SEEK_SET)` the *next* `read()` in the loop would return 0 bytes (and `poll()` would not behave correctly). Rewinding makes the next loop iteration re-read the fresh value.

Two concepts the question is really testing: **`POLLPRI`** is the mechanism by which sysfs reports a GPIO edge (so it goes in `.events`), and a **timeout of `-1`** makes `poll()` block indefinitely. (Note: although the prompt says the level should "fall from 1 to 0," Q3.B.1 armed the **rising** edge per its own wording; the C-program blanks above are the canonical lecture answers regardless.)

*Source: Lecture 08 — GPIO button poll() program (slide ~50).*

---

### Q4.A. [11 marks, 35 min] When building BusyBox, it generates a single executable file. Describe briefly how it can offer multiple system tools using this single file.

**Answer:**
BusyBox is **one binary that contains the code of many small tools ("applets")** — `ls`, `cp`, `cat`, `mount`, `sh`, etc. Each applet is implemented as a function named like `ls_main`, `cp_main`, and so on, all linked into the single `busybox` executable. When BusyBox is installed, it creates **symbolic links** in the system directories — `/bin/ls`, `/bin/cat`, `/sbin/mount`, … — that all **point to the one `busybox` binary**. At runtime, when you invoke (say) `ls`, the program starts and inspects **`argv[0]`** — the name it was called by. It looks that name up in its internal **applet table**, finds the matching `*_main` function, and dispatches to it. So the *same* file behaves as whichever tool its invocation name says, letting one executable provide dozens of commands.

**Explanation:**
The motivation is **size**. On a tiny embedded system, shipping the full GNU coreutils/util-linux as dozens of separate programs — each carrying its own copy of common code and linking overhead — wastes precious flash and RAM. BusyBox instead compiles a streamlined version of each tool into a **single shared executable**, so common code is written once and there is only one program image.

The mechanism that makes "one file = many tools" work rests on two facts:

1. **`argv[0]` carries the invocation name.** When the shell runs a command, the first element of the program's argument vector (`argv[0]`) is the name used to launch it. If you call the binary as `ls`, `argv[0]` is `"ls"`; if you call it as `cp`, `argv[0]` is `"cp"`.

2. **Symlinks make every tool name resolve to BusyBox.** `busybox --install` (or the build) populates `/bin`, `/sbin`, `/usr/bin`, … with symlinks such as `ls -> /bin/busybox`. So no matter which command name the user types, the kernel actually executes the *same* `busybox` file — but with `argv[0]` set to the name that was typed.

Putting them together: BusyBox's `main()` reads `argv[0]`, strips the directory path to get the base name, searches its **applet table** for that name, and calls the corresponding `<applet>_main()` function. (If you run `busybox ls -l` directly, it uses `argv[1]` as the applet name instead.) That table-driven dispatch on the invocation name is how a single executable masquerades as many independent system tools.

*Source: Lecture 05 — BusyBox (applets + symlinks + argv[0] dispatch).*

---

### Q4.B. System V init supports the concept of runlevels. List the 8 run levels showing when each level is executed.

**Answer:**
A **runlevel** is a numbered system state that defines *which set of services is running*. System V init defines 8:

| Runlevel | When it is executed / what it means |
|---|---|
| **S** (or **s**) | **Startup / single-user** — the first state entered at boot; runs the system's startup/initialization tasks. |
| **0** | **Halt** — shut the system down / power off. |
| **1** | **Single-user mode** — minimal, one user, for maintenance/recovery (no networking, no daemons). |
| **2** | **Multi-user mode without networking** — multiple users, but network services not started. |
| **3** | **Multi-user mode with networking** — full multi-user text mode with network services (typical server default). |
| **4** | **Unused / user-definable** — not assigned by default; free for custom configuration. |
| **5** | **Multi-user mode with networking + graphical (GUI)** — like 3 plus the display manager / graphical login. |
| **6** | **Reboot** — restart the system. |

**Explanation:**
The idea behind runlevels is to bundle "what should be running" into named modes you can switch between. init reads `/etc/inittab` to know which programs belong to each runlevel, then starts/stops services accordingly.

How to read the list:
- **S** is special: it is the **startup** state entered first, where the one-time initialization (mounting filesystems, running `rcS`) happens. It is also the single-user bring-up state.
- **0 and 6 are the "transition" levels**: entering **0** halts the machine and entering **6** reboots it (you never "stay" in them).
- **1 → 2 → 3 → 5** is a ladder of increasing capability: single-user maintenance (1) → multi-user but no network (2) → multi-user with network (3) → all that plus a graphical desktop (5).
- **4** is intentionally left **unused/user-defined** so administrators can craft their own state.

When the system **switches** runlevels, init runs the **K** ("kill") scripts to stop services not wanted in the new level, then the **S** ("start") scripts to launch the ones that are. (BusyBox init understands the same concepts but ignores the runlevel field in inittab.)

*Source: Lecture 06 — Runlevels (S, 0–6).*

---

### Q4.C. Compare between NAND and NOR Flash in terms of erase block size, reliability and controller complexity.

**Answer:**

| Aspect | **NOR Flash** | **NAND Flash** |
|---|---|---|
| **Erase block size** | Relatively small erase blocks (~**128 KB**). Accessed at fine granularity — **word/byte-addressable**, read word-by-word; can be **memory-mapped and executed in place (XIP)**. | Larger erase blocks, organized as **16 KB–512 KB** blocks made of **pages**; I/O is **page-based (2–4 KB pages**, e.g. 2048 B data + 64 B spare/OOB). Not directly random-addressable like RAM. |
| **Reliability** | **Highly reliable**; high endurance, roughly **100K–1M** erase cycles. Few/no bad blocks. | **Less reliable**: ships with (and develops) **bad blocks**, and suffers **bit flips**, so it **requires error-correcting codes (ECC)** and bad-block management. Lower endurance. |
| **Controller complexity** | **Simple** — essentially "no initialization, just wiring and address mapping"; behaves like memory. | **Complex hardware + software controller** — must handle **bad-block management, ECC, and a flash translation layer**; needs dedicated NAND controller plus bootloader/kernel driver support. |

`[KEY vs LECTURE]` The AI-generated solution printed **reversed/different size ranges** (e.g. "NAND 64 KB–2 MB vs NOR 64–128 KB"). Use the **lecture's** numbers above (NOR ≈128 KB word-addressable; NAND 16 KB–512 KB page-based). The marks really hinge on the *word-vs-page access*, *reliability/ECC*, and *controller-complexity* rows rather than the exact KB figures.

**Explanation:**
Both NOR and NAND are non-volatile flash, but they are built and used very differently — and the exam wants the trade-off along three axes.

- **Why the access/erase difference matters:** **NOR** is wired so the CPU can read any address directly, like ROM/RAM. That makes it **memory-mappable** and able to run code straight out of flash — **eXecute-In-Place (XIP)** — which is why bootloaders often live in NOR. Its erase blocks are modest in size. **NAND** trades random access for **density and low cost**: you read/write whole **pages** and erase whole **blocks**, so it is great for *bulk storage* but cannot be executed in place.

- **Why reliability differs:** NAND's tight, high-density cells are inherently noisier — it is **shipped with some bad blocks**, develops more over time, and individual bits can flip. So NAND **mandates ECC** (e.g. Hamming for SLC, BCH for MLC/TLC) and **bad-block tracking**. NOR is more robust with much higher endurance and effectively no bad-block problem.

- **Why controller complexity differs:** Because NOR "just works" like addressable memory, its controller is **trivial** (wiring + address decode). NAND needs a **substantial controller** — in hardware and in the OS — to manage bad blocks, compute/verify ECC on every page, and provide a flash translation layer (wear leveling, mapping). That is the price of NAND's density.

In short: **NOR = simple, reliable, memory-mappable (XIP), smaller capacity** → good for *boot code*; **NAND = dense, cheaper, page-based, needs ECC + complex controller** → good for *mass data storage*.

*Source: Lecture 09 — NAND vs NOR flash (erase block size, reliability, controller complexity).*

---

<a id="exam-linux-24"></a>

## Linux_24 — CSY441 Final Exam (9/6/2024, Prof. Eman Shaaban & Dr. Karim Emara) — Ain Shams FCIS

Total questions: 22

*Exam info: FCIS – Ain Shams University; CSY441; Final 9/6/2024; Levels 3,4; Total Marks 50; 2nd term 2023-2024; 2 hours. Question (1) = 10 marks/25 min · Question (2) = 5 marks/15 min · Question (3) = 20 marks/40 min · Question (4) = 15 marks/40 min.*

---

### Q1.1. The ____ toolchain runs on the different type of system than the target machine. a) cross-platform  b) cross-compiled  c) native  d) normal

**Answer:** **b) cross-compiled**

**Explanation:**
A *toolchain* is the set of programs (compiler `gcc`, assembler/linker `binutils`, C library, debugger `gdb`) used to turn source code into an executable. The key question is always: **on which machine does the toolchain RUN, and for which machine does it PRODUCE code?**

- A **native** toolchain runs on the *same type* of system that it produces binaries for. Example: gcc running on an x86 PC building programs that run on that same x86 PC.
- A **cross-compiled** (cross) toolchain *runs on one type of machine but produces binaries for a different type*. This is the normal situation in embedded Linux: you build on a powerful x86_64 host PC but the output runs on a small ARM target board. The host is fast and has lots of RAM/disk; the target is too slow/small to compile its own software efficiently. Hence the verbatim stem "runs on the different type of system than the target" = the cross-compiled toolchain.

Why the other options are wrong:
- **a) cross-platform** — not a term used in this course for toolchains. It is generic marketing language and does not name the specific concept. [KEY vs LECTURE: the AI solution wrongly chose "cross-platform"; the lecture's term is "cross-compiled".]
- **c) native** — the opposite case: same system type as the target, not different.
- **d) normal** — not a defined toolchain category.

*Source: Lecture 02 — Toolchain (native vs cross-compiled).*

---

### Q1.2. The bootloader code that setups the memory controller and load the full bootloader into DRAM is called ____ . a) TPL  b) ROM Code  c) SoC  d) SPL

**Answer:** **d) SPL** (Secondary Program Loader)

**Explanation:**
On most SoCs, booting happens in stages because **DRAM is not usable immediately after reset** — the DRAM (main memory) controller has not been configured yet, so there is nowhere big to load a large bootloader. The chain solves this with progressively larger stages:

1. **ROM code** — burned into the SoC's mask ROM. It is tiny and runs first, but it *cannot initialise DRAM*. It copies a small chunk of code from a fixed location into the small on-chip **SRAM**.
2. **SPL (Secondary Program Loader)** — runs from SRAM. Its job is to **set up the memory controller (initialise DRAM)** and then **load the full/main bootloader (TPL) into DRAM**. This matches the stem exactly.
3. **TPL (Tertiary Program Loader)** — the full bootloader (e.g. U-Boot), now running from DRAM. It loads the kernel image + device tree into DRAM and provides a command line.

Why the other options are wrong:
- **a) TPL** — that is the *full* bootloader that SPL loads; TPL runs *after* DRAM is already set up, it doesn't set up the memory controller.
- **b) ROM Code** — runs before SPL and *cannot* configure DRAM; it only loads code into SRAM.
- **c) SoC** — System-on-Chip, the hardware itself, not a bootloader stage.

*Source: Lecture 03 — Bootloader & Device Tree (boot phases ROM code → SPL → TPL).*

---

### Q1.3. The ____ is a compressed version of the Linux kernel image that is self-extracting, while ____ is an image file that has a U-Boot wrapper. a) zImage, uimage  b) uImage, zImage  c) uImage, cpio  d) zImage, cpio

**Answer:** **a) zImage, uImage**

**Explanation:**
After the kernel is compiled, it can be packaged in several formats:

- **zImage** — a *compressed* kernel image that is **self-extracting**: a small uncompressed stub is prepended that decompresses the rest into RAM at boot, then jumps to it. It saves storage/transfer space while still being directly bootable on many platforms.
- **uImage** — a zImage (or other payload) wrapped with a **U-Boot header** added by the U-Boot tool `mkimage`. That 64-byte header records the load address, entry point, type, and a CRC, so the U-Boot bootloader knows how to handle and verify the image.

So zImage = self-extracting compressed kernel; uImage = zImage + U-Boot mkimage wrapper.

Why the other options are wrong:
- **b) uImage, zImage** — reversed; the U-Boot-wrapped one is uImage, not zImage.
- **c) uImage, cpio** — `cpio` is the archive format used for an *initramfs/ramdisk*, not a kernel image; and the first blank should be zImage.
- **d) zImage, cpio** — first blank correct, but the second should be uImage, not cpio.

*Source: Lecture 04 — Kernel (zImage vs uImage).*

---

### Q1.4. The purpose of the ___ and ____ pseudo filesystems is to expose information about processes and kernel driver model to user space, respectively. a) /dev, /proc  b) /proc, /dev  c) /proc, /sysfs  d) /sysfs, /proc

**Answer:** **c) /proc, /sysfs**

**Explanation:**
A *pseudo (virtual) filesystem* contains no real files on disk; the kernel generates its contents on the fly so user space can read/write kernel state as if it were files. The two asked here, matched to the word "**respectively**":

- **/proc** — exposes information about **processes** (and some kernel/system info). Each running process has a directory `/proc/<pid>/` with its status, command line, memory map, open files, etc.
- **/sysfs** (mounted at `/sys`) — exposes the **kernel driver model**: the hierarchy of buses, devices, drivers and their attributes (e.g. `/sys/class/gpio`, `/sys/block`).

The order in the stem is "processes" then "kernel driver model", so the answer must be /proc then /sysfs.

Why the other options are wrong:
- **a) /dev, /proc** — `/dev` holds *device nodes*, it does not expose process info; order/content wrong.
- **b) /proc, /dev** — first is right, but `/dev` is not the driver-model filesystem; `/sys` is.
- **d) /sysfs, /proc** — correct two filesystems but in the *wrong order* for "processes … driver model, respectively".

*Source: Lecture 04 — Kernel (/proc vs /sys purpose).*

---

### Q1.5. If the mode of an executable owned by root is -rwxr-xr-x and user1 runs it, the executable will run with the privilege of the ____ . a) user1's group  b) root's group  c) user1  d) root

**Answer:** **c) user1**

**Explanation:**
When a program runs, its *effective user ID* (the identity used for permission checks) normally equals the **invoking user** — the person who launched it — regardless of who *owns* the file. The owner only matters for the special **setuid (SUID)** bit.

Read the mode `-rwxr-xr-x`:
- Owner (root): `rwx`
- Group: `r-x`
- Others: `r-x`
- The owner's execute field is a plain **`x`**, *not* an **`s`** — so **the SUID bit is NOT set**.

Because there is no SUID bit, when **user1** runs it, the process runs as **user1**, not as root. (If the mode were `-rwsr-xr-x`, i.e. `chmod 4755`, the `s` would set effective UID = file owner = root for any runner — but that is not this case.)

Why the other options are wrong:
- **a) user1's group** — the effective *user* is user1, but the privilege is the user identity, not "user1's group"; group is a separate concept and not what determines this.
- **b) root's group** — would require special bits and is not how SUID works anyway.
- **d) root** — would only be true *with* the SUID bit (`s`), which is absent here.

*Source: Lecture 05 — Root Filesystem (permissions, SUID / effective UID).*

---

### Q1.6. BusyBox init begins by reading ____ file which defines rules to start programs with their corresponding actions and runlevels, one per line. a) /etc/init  b) /etc/inittab  c) /etc/init.d/rcS  d) /etc/startup

**Answer:** **b) /etc/inittab**

**Explanation:**
`init` is the very first user-space program (PID 1). The System V-style / BusyBox `init` **begins by reading the `/etc/inittab` file**. Each line is a *rule* in the form `<id>:<runlevels>:<action>:<process>`, telling init which program to start, with which action (e.g. `sysinit`, `respawn`, `askfirst`), and for which runlevels. (BusyBox ignores the runlevels field but keeps the format.) The default first entry is typically `::sysinit:/etc/init.d/rcS`.

Note the contrast that exams love to test: init *reads* `/etc/inittab`; the *boot-time shell commands* themselves are put in `/etc/init.d/rcS` (which inittab's sysinit line invokes).

Why the other options are wrong:
- **a) /etc/init** — not the config file name; not what init reads.
- **c) /etc/init.d/rcS** — this is the *script of boot commands* that inittab calls via the sysinit action; it is not the file init "begins by reading", and it is not a list of rules with actions/runlevels.
- **d) /etc/startup** — not a standard init file.

*Source: Lecture 06 — Init & Runlevels (inittab vs rcS).*

---

### Q1.7. The Yocto project component that includes recipes, configuration files, commands that control the building process is called ___ . a) layers  b) OE-Core  c) poky  d) metadata

**Answer:** **d) metadata**

**Explanation:**
In Yocto/OpenEmbedded, **metadata** is the umbrella term for *all the input that tells the build system what and how to build*: it is made up of **recipes** (`.bb` files), **configuration files**, and the **commands/classes that control the building process**. BitBake (the build engine) reads this metadata to figure out the dependency tree and run the tasks.

Why the other options are wrong:
- **a) layers** — a *layer* is a directory that *groups* related metadata (recipes/config/classes) so it can be added or overridden modularly; the metadata is the *content*, the layer is the *container/organisation*.
- **b) OE-Core** — the specific set of validated *base* layers shared by all OpenEmbedded systems; it is one body of metadata, not the general term for "recipes + config + commands".
- **c) poky** — the Yocto *reference distribution* (a collection of projects/tools to bootstrap a distro), not the term for the build-control inputs.

*Source: Lecture 07 — Build Systems (Yocto glossary: metadata).*

---

### Q1.8. In block devices, the minor number identifies the ______ of the device. a) partition  b) interface  c) device driver  d) none of the above

**Answer:** **a) partition**

**Explanation:**
Every device node has a **major** number and a **minor** number.
- The **major** number identifies the **device class / driver** that handles the device (e.g. `mmcblk` = 179, `sd` disks = 8).
- The **minor** number identifies a *specific instance* within that driver. For **block devices** (mass storage like SD cards and disks), the minor number distinguishes the **partition**: typically the first minor is the whole device and the following minors are its partitions (e.g. `mmcblk0`, `mmcblk0p1`, `mmcblk0p2`).

So for block devices, minor = partition.

Why the other options are wrong:
- **b) interface** — "interface/instance" is the wording used for *character* devices, not block. (This is the classic 2025 "flip" trap.)
- **c) device driver** — that is what the **major** number identifies, not the minor.
- **d) none of the above** — incorrect because (a) is exactly right.

*Source: Lecture 08 — Device Drivers (major/minor numbers; block minor = partition).*

---

### Q1.9. To read NAND flash, the _____ tool should be used to skip bad blocks. a) cp  b) nanddump  c) nandwrite  d) nandcp

**Answer:** **b) nanddump**

**Explanation:**
NAND flash is not perfectly reliable: it ships with, and develops, **bad blocks** that must be skipped during access. Ordinary file tools do not understand this. The MTD utilities are NAND-aware:
- **nanddump** — *reads* NAND, **skipping bad blocks** (and can include/exclude the OOB area). This is the correct tool for reading.
- **nandwrite** — the matching tool to *write* to NAND, also skipping bad blocks (write to an erased region first via `flash_erase`).

Why the other options are wrong:
- **a) cp** — the standing trap: `cp` is *not* NAND-aware and **fails at the first bad block**, producing corrupt/incorrect data.
- **c) nandwrite** — correct family, but it *writes*; the question asks to **read**.
- **d) nandcp** — not a real MTD tool.

*Source: Lecture 09 — Flash & MTD (nanddump / nandwrite skip bad blocks).*

---

### Q1.10. The ___ is a collection of projects and tools, used to bootstrap a new distribution based on the Yocto Project. a) metadata  b) poky  c) BitBake  d) OE-Core

**Answer:** **b) poky**

**Explanation:**
**Poky** is the Yocto Project's **reference distribution** — described (verbatim) as "a collection of projects and tools, used to bootstrap a new distribution." It bundles BitBake (the build engine), OE-Core (base metadata), and example/reference policies so you have a working starting point that you then customise into your own distro.

Why the other options are wrong:
- **a) metadata** — the *inputs* (recipes + config + commands) that describe what to build; not a "collection of projects and tools to bootstrap a distro".
- **c) BitBake** — the *build engine / task scheduler* that executes the metadata; it is one tool inside Poky, not the whole bootstrapping collection.
- **d) OE-Core** — the validated *base layers* shared by OpenEmbedded systems; it is part of what Poky includes, not the bootstrapping distribution itself.

*Source: Lecture 07 — Build Systems (Yocto glossary: Poky).*

---

### Q2.1. In a device tree, the ___ property is used by the operating system to decide which device driver to bind to a device, while the ___ shows the base address and address range for the node.

**Answer:** **compatible** (first blank) / **reg** (second blank)

**Explanation:**
A *device tree* (DT) is a data structure that describes the hardware to the kernel so the same kernel can boot on different boards. Each hardware block is a *node* with *properties*:

- **compatible** — a string (or list) of the form `"vendor,model"`. The OS uses the `compatible` value to **decide which device driver to bind** to that device: it matches the node's `compatible` string against the strings each driver advertises. This is the mandatory binding key.
- **reg** — describes the device's **register region(s)** as tuples of `<base-address length>`. It tells the kernel the **base address and address range** (size) the node occupies in the bus/memory map. (How many cells make up the address and the length is set by the parent's `#address-cells` / `#size-cells`.)

So: `compatible` → driver binding; `reg` → base address + range. Example: `serial@48020000 { compatible = "arm,pl011"; reg = <0x48020000 0x1000>; };`.

*Source: Lecture 03 — Bootloader & Device Tree (compatible + reg).*

---

### Q2.2. In Linux Kernel build system, ____ files define each config option and its attributes, while ____ files store each the selected values of the config symbols.

**Answer:** **Kconfig** files (first blank) / **.config** file (second blank)

**Explanation:**
The Linux kernel is highly configurable; the build system (Kconfig) separates *what options exist* from *what you chose*:

- **Kconfig** files — scattered through the source tree, these **define each configuration option and its attributes**: the symbol name, its type (`bool`, `tristate`, `string`, `int`…), its prompt text, default value, help text, and dependencies (`depends on`, `select`). They are the menu *definitions* you see in `make menuconfig`.
- **.config** file — a single generated file at the top of the build tree that **stores the selected value of every config symbol** (e.g. `CONFIG_FOO=y`, `CONFIG_BAR=m`, `# CONFIG_BAZ is not set`). Tools like `menuconfig` or `defconfig` edit it, and the build reads it to decide what to compile.

So: Kconfig defines the options; .config stores the chosen values.

*Source: Lecture 04 — Kernel (Kconfig vs .config).*

---

### Q2.3. After a successful build of Buildroot, the ____ directory contains the results of the build while the _____ directory is the staging area for the root directory.

**Answer:** **images** directory (first blank) / **target** directory (second blank)

**Explanation:**
After Buildroot finishes, the `output/` tree holds several sub-directories. The two asked here:

- **images/** — contains the **results of the build**: the deliverables you actually deploy — the bootloader, the kernel (and its DTBs), and the **root-filesystem image(s)** (e.g. `rootfs.ext2`, `rootfs.cpio`).
- **target/** — the **staging area for the root directory**: the root filesystem tree exactly as it will appear on the target board, assembled file-by-file. It is **not directly bootable** as-is (it lacks proper device nodes and the files are owned by the host user); Buildroot applies a *device table* to fix ownership/permissions and create nodes when it packs `target/` into the final image in `images/`.

(For context, the other output sub-dirs: `staging/` = a symlink to the toolchain's sysroot (headers+libs); `build/` = per-package work dirs; `host/` = host tools including the toolchain.)

*Source: Lecture 07 — Build Systems (Buildroot output dirs: images / target).*

---

### Q2.4. Device drivers are identified in user space by a special file called _____.

**Answer:** **device node** (a special file in `/dev`)

**Explanation:**
The kernel exposes a driver/device to user space through a **device node** — a *special file* (it has no data blocks of its own) that lives under **/dev**. The node maps a human-readable name to a `(type, major, minor)` triple:
- **type** = `c` (character) or `b` (block),
- **major** = which driver/device class,
- **minor** = which specific instance/partition.

When a program does `open("/dev/...")`, the kernel uses the node's major number to route the call to the right driver. Nodes can be created manually with **`mknod <name> <c|b> <major> <minor>`**, or automatically/on-demand by **udev / mdev / devtmpfs**. (Note: *network* devices are the exception — they have no device node and are accessed via sockets.)

*Source: Lecture 05 / Lecture 08 — Root Filesystem & Device Drivers (device nodes in /dev).*

---

### Q2.5. A shared memory segment is created using the ____ function. To expand its size, the ____ function should be called.

**Answer:** **shmget()** to create / **ftruncate()** to expand its size

**Explanation:**
*Shared memory* is the fastest IPC mechanism: two or more processes map the **same** region of physical memory into their address spaces and read/write it directly (no copying through the kernel per message).

- **shmget()** — *creates* (or obtains) a shared memory segment, returning an identifier. It is the System V shared-memory "get" call that allocates the segment for a given key and size.
- **ftruncate()** — sets/changes the **size** of the underlying object; to **expand (grow) the segment's size** you call `ftruncate(fd, new_size)` so the mapping can cover more bytes.

So: create with `shmget()`, resize/expand with `ftruncate()`.

`[DERIVED — IPC lecture not in course folder; standard POSIX/Simmonds. Note: in pure POSIX the creation call is normally `shm_open()` paired with `ftruncate()`; the locked exam key uses `shmget()` + `ftruncate()`, which mixes the System V "get" with the POSIX "truncate" — answer per the key.]`

*Source: Processes/IPC lecture (not in course folder) — standard POSIX shared memory (shmget/ftruncate).*

---

### Q3.A. Write a sequence of Linux shell commands that sets the GPIO ports 12 and 13 as output and port 53 as input. Enable the interrupt for port 53 on the falling edge.

**Answer:**
```sh
echo 12 > /sys/class/gpio/export
echo 13 > /sys/class/gpio/export
echo out > /sys/class/gpio/gpio12/direction
echo out > /sys/class/gpio/gpio13/direction
echo 53 > /sys/class/gpio/export
echo in > /sys/class/gpio/gpio53/direction
echo falling > /sys/class/gpio/gpio53/edge
```

**Explanation:**
Linux exposes GPIO pins to user space through the **sysfs** interface under **`/sys/class/gpio`** (the legacy gpiolib sysfs ABI). You control a pin by writing plain text into attribute files — no C code needed. The workflow is always the same three steps:

1. **Export the pin** — `echo <N> > /sys/class/gpio/export` makes the kernel create a new directory `/sys/class/gpio/gpio<N>/` containing the control files (`direction`, `value`, `edge`, `active_low`). You must export a pin before you can configure it. Here we export 12, 13, and 53.
2. **Set the direction** — write to `gpio<N>/direction`:
   - `echo out > .../gpio12/direction` and `echo out > .../gpio13/direction` make pins 12 and 13 **outputs** (their level is then driven by writing `0`/`1` to `gpio12/value`).
   - `echo in > .../gpio53/direction` makes pin 53 an **input**. (Pins default to input, but writing it explicitly is correct and clear.)
3. **Arm the interrupt edge** — for an input you can ask the kernel to signal a level transition by writing to `gpio<N>/edge` one of `none | rising | falling | both`. `echo falling > .../gpio53/edge` enables an interrupt on the **falling edge** of pin 53. A user program can then `poll()` on `gpio53/value` (waiting for `POLLPRI`) to block until that edge occurs.

Order matters only in that **export must precede** any use of a pin's `direction`/`edge` files. Setting `edge` requires the pin to be an input, so direction-in is set before (or it is already the default).

*Source: Lecture 08 — Device Drivers & GPIO (GPIO sysfs export / direction / edge).*

---

### Q3.B. Assume user2 belongs to group1 and user3 belongs to group2, fill the following access matrix for the files uRamdisk and file.dtb for the 3 users, given the following output of ls command:
`-rw-r-xr-- 1 user1 group1 1734732 Nov 3 16:02 uRamdisk`
`-rwxrw-rw- 1 user1 group1 8075 Oct 28 07:41 file.dtb`
(Access matrix rows: user1, user2, user3; columns: uRamdisk, file.dtb.)

**Answer:**

| | uRamdisk | file.dtb |
|---|---|---|
| **user1** (owner) | r, w (`rw-`) | r, w, x (`rwx`) |
| **user2** (group1) | r, x (`r-x`) | r, w (`rw-`) |
| **user3** (others) | r (`r--`) | r, w (`rw-`) |

**Explanation:**
A Linux permission string has **10 characters**: `[type][owner triad][group triad][others triad]`, where each triad is `r` (read=4), `w` (write=2), `x` (execute=1) or `-` (none). Which triad applies to a given user is decided by this precedence:
- If the user **owns** the file → use the **owner** triad.
- Else if the user is a member of the file's **group owner** → use the **group** triad.
- Else → use the **others** triad.

Both files are owned by **user1** and have group owner **group1**. The three users map as follows for *both* files:
- **user1** = the **owner** → owner triad.
- **user2** ∈ **group1** = the files' group → **group** triad.
- **user3** ∈ group2 (which is *not* either file's group) → **others** triad.

Now read each file's triads:

`uRamdisk` = `-rw-r-xr--`
- owner `rw-` → user1: **r, w**
- group `r-x` → user2: **r, x**
- others `r--` → user3: **r**

`file.dtb` = `-rwxrw-rw-`
- owner `rwx` → user1: **r, w, x**
- group `rw-` → user2: **r, w**
- others `rw-` → user3: **r, w**

The common trap is to assume the same triad applies to a user across files — but you must re-check the *file's group owner* each time. Here both files happen to share group1, so user2 always takes the group triad and user3 always takes the others triad.

*Source: Lecture 05 — Root Filesystem (permission triads / access matrix).*

---

### Q3.C. This C program reads a shell command iteratively from the user and runs it as a new process. Complete the missing parts [1] to [6].
```c
1: int main(int argc, char *argv[]) {
2:   char command_str[128];
3:   int pid;
4:   int child_status;
5:   int wait_for = 1;
6:   while (1) {
7:     printf("sh> ");
8:     scanf("%s", command_str);
9:     pid = …[1]… ;
10:    if (pid …[2]…) {
11:      printf("cmd '%s'\n", command_str);
12:      …[3]…(command_str, command_str, (char *)NULL);
13:      …[4]…;
14:    }
15:    if (wait_for) {
16:      …[5]… (…[6]…, &child_status, 0);
17:      printf("Done, status %d\n", child_status);
18:    }
19:  }
20:  return 0;
21: }
```

**Answer:**

| Blank | Fill |
|---|---|
| **[1]** | `fork()` |
| **[2]** | `== 0` |
| **[3]** | `execl` |
| **[4]** | `exit(1)` |
| **[5]** | `waitpid` |
| **[6]** | `pid` |

Completed code:
```c
int main(int argc, char *argv[]) {
  char command_str[128];
  int pid;
  int child_status;
  int wait_for = 1;
  while (1) {
    printf("sh> ");
    scanf("%s", command_str);
    pid = fork();                                       /* [1] */
    if (pid == 0) {                                     /* [2] child branch */
      printf("cmd '%s'\n", command_str);
      execl(command_str, command_str, (char *)NULL);    /* [3] replace process image */
      exit(1);                                          /* [4] only reached if execl failed */
    }
    if (wait_for) {
      waitpid(pid, &child_status, 0);                   /* [5][6] reap the child */
      printf("Done, status %d\n", child_status);
    }
  }
  return 0;
}
```

**Explanation:**
This is the classic UNIX **fork–exec–wait** pattern that every shell uses to launch a command as a *new process*:

- **[1] `fork()`** — duplicates the calling process. After it returns there are **two** processes (parent and child) running the same code. Its return value is how each side knows who it is:
  - in the **child**, `fork()` returns **0**;
  - in the **parent**, it returns the **child's PID** (a positive number);
  - on failure it returns **-1**.
- **[2] `== 0`** — therefore `if (pid == 0)` selects the **child** branch. Only the child should go on to run the requested command.
- **[3] `execl`** — inside the child, `execl(path, arg0, …, (char*)NULL)` **replaces the child's process image** with the new program. Here `execl(command_str, command_str, (char *)NULL)` runs the typed command with itself as `argv[0]` and a NULL-terminated argument list. On **success `execl` never returns** — the child has become the new program.
- **[4] `exit(1)`** — this line is reached **only if `execl` failed** (e.g. command not found). The child then exits with a non-zero status to report the error instead of falling through and becoming a second shell.
- **[5] `waitpid`** / **[6] `pid`** — meanwhile the **parent** (where `pid` = child's PID, so it skips the `if`) reaches `if (wait_for)` and calls **`waitpid(pid, &child_status, 0)`** to **block until that specific child finishes** and to *reap* it (collect its exit status into `child_status`, preventing a zombie). It then prints the status.

So: `fork()` splits the process; the `pid == 0` child does `execl` (and `exit(1)` only on failure); the parent `waitpid(pid, …)` waits for and reaps the child.

`[DERIVED — processes lecture not in course folder; standard POSIX/Simmonds ch. 17.]`

*Source: Processes/IPC lecture (not in course folder) — standard POSIX fork/exec/wait.*

---

### Q4.A. Describe briefly the following Yocto project terms: BitBake, recipes and OE-Core.

**Answer:**
- **BitBake** — the Yocto/OpenEmbedded **build engine and task scheduler** (conceptually like `make`, but cross-architecture aware). It **parses the configuration files and recipes (the metadata)**, computes the **dependency tree**, and then **executes the tasks** needed to fetch, configure, compile, install and package every component, finally producing the package feeds and image(s).
- **Recipes** (`.bb` files) — a **recipe is a list of settings and tasks (instructions) for building ONE package/component**. It says where to get the source and how to build it through an ordered set of tasks: `do_fetch -> do_unpack -> do_patch -> do_configure -> do_compile -> do_install -> do_package`. Recipes live inside **layers**.
- **OE-Core** (OpenEmbedded-Core) — the **set of validated base layers (recipes, classes, common metadata) that are shared between all OpenEmbedded-based systems**. It is the common core formed when OpenEmbedded and Poky merged their shared components, providing the foundational recipes/classes every distribution builds on.

**Explanation:**
Yocto is not a distribution you install — it is a **build system that produces a custom embedded Linux distribution**. To understand it you separate the *engine* from the *inputs*:
- The **inputs** are collectively called **metadata** = recipes + configuration files + classes/commands. **Recipes** are the smallest unit: each one knows how to build a single piece of software. They are grouped into **layers** so features can be added or overridden modularly.
- The **engine** is **BitBake**: it reads all that metadata, works out what depends on what, and schedules the per-recipe tasks in the right order across all packages — exactly the role `make` plays for a single program, but scaled to a whole OS image and able to cross-compile.
- **OE-Core** is the *shared base* of metadata. Rather than every project reinventing recipes for libc, busybox, the kernel, etc., OE-Core provides a validated common set of base layers/classes that all OpenEmbedded-based builds (including Poky) reuse.

(Related terms for context: **Poky** = the reference distribution bundling BitBake + OE-Core + reference policies; **metadata** = the umbrella term for recipes/config/commands; **layers** = directories that group and can override metadata.)

*Source: Lecture 07 — Build Systems (Yocto glossary: BitBake, recipes, OE-Core).*

---

### Q4.B. Compare between NAND and NOR Flash in terms of erase block size, reliability and controller complexity.

**Answer:**

| Aspect | **NOR flash** | **NAND flash** |
|---|---|---|
| **Erase block size** | Larger erase blocks (~128 KB); accessed **word-at-a-time** (read/write individual words) and is **memory-mappable -> eXecute-In-Place (XIP)** is possible | Erased/written/read in **pages (about 2-4 KB)** grouped into blocks (about 16-512 KB); page = data area + spare **OOB** area; not directly memory-mappable |
| **Reliability** | **More reliable**; high endurance (~100K-1M erase cycles); no bad blocks in normal use | **Less reliable**: ships with and develops **bad blocks**, and suffers **bit flips**, so it **requires ECC** (error-correcting codes) and bad-block management; lower endurance |
| **Controller complexity** | **Simple** controller — "no initialisation needed, only wiring and address mapping" | **Complex** hardware+software controller required to handle **bad blocks and ECC** (NAND controller in the SoC plus driver support in bootloader/kernel) |

**Explanation:**
Both are non-volatile flash, but they trade off in opposite directions:
- **NOR** is laid out so the CPU can address it like memory: you can read individual words and even **run code directly from it (XIP)**, and it is highly reliable. The price is that it is more expensive per bit and slower to erase/write large amounts. Because it is so well-behaved, its **controller is trivial** — essentially just wiring and an address map, no special initialisation.
- **NAND** is denser and cheaper per bit (good for mass storage), but it is organised in **pages/blocks** and is **inherently less reliable**: blocks can be bad from the factory or wear out, and individual bits flip. To use it safely you need **ECC** to detect/correct bit errors and **bad-block management** to skip unusable blocks — which means a **much more complex controller** (hardware in the SoC plus software in the bootloader and kernel MTD layer). This is also why you read/write NAND with bad-block-aware tools like `nanddump`/`nandwrite` rather than `cp`.

The marks here are carried by the **page-vs-word access**, the **reliability/ECC/bad-block** point, and the **simple-vs-complex controller** point. `[KEY vs LECTURE: some AI keys printed reversed numeric size ranges; rely on the lecture's qualitative rows above — the reliability and controller-complexity contrasts are the load-bearing answers.]`

*Source: Lecture 09 — Flash & MTD (NAND vs NOR comparison).*

---

### Q4.C. Compare and differentiate between character and block Linux device drivers. What do major and minor numbers identify in each type?

**Answer:**

| | **Character (char) driver** | **Block driver** |
|---|---|---|
| Data model | **Unbuffered byte/stream I/O** — bytes flow in sequence | **Buffered, fixed-size block I/O** with **random access** |
| Typical use / device | Serial ports, terminals (tty), simple peripherals; a thin layer over the hardware | Mass storage: SD cards, disks (`mmcblk`, `sd...`) — used for partitioning, formatting, mounting |
| Node type in `/dev` | `c` (character special file) | `b` (block special file) |
| Access | `open / read / write / close` directly on the byte stream | Goes through the kernel's block layer / buffer cache; mounted as a filesystem |

**Major / minor numbers:**
- **Major number** — identifies the **device class / driver** in **BOTH** char and block types (which driver handles the node).
- **Minor number** — identifies the **specific instance**:
  - in a **character** device -> the particular **instance / interface** the driver manages;
  - in a **block** device -> the **partition** of the device.
- **Network devices are the exception**: they have **no device node and no major/minor number**; they are accessed via the socket API (e.g. `eth0`, `wlan0`).

**Explanation:**
Linux classifies most device drivers as either *character* or *block*, based on how data is accessed:
- A **character driver** presents the device as a **stream of bytes**, read/written sequentially and usually **unbuffered** — like reading from a serial line. It is a relatively **thin** layer sitting just above the hardware, exposed through a `c` node (e.g. a tty). You typically just `open`, `read`/`write` bytes, and `close`.
- A **block driver** presents the device as an array of **fixed-size blocks** that can be accessed in **any order (random access)** and is **buffered** by the kernel (the page/buffer cache). This model suits **mass storage**: you partition it, put a filesystem on it, and mount it. Its node type is `b` (e.g. `mmcblk0`).

For both, the kernel routes an `open("/dev/...")` to the right driver using the **major** number (the device class/driver), and then the driver uses the **minor** number to pick the exact thing it is talking to — an **instance/interface** for char devices, or a **partition** for block devices. Network interfaces don't fit the file model at all, so they have neither a node nor major/minor numbers and are reached through sockets instead.

*Source: Lecture 08 — Device Drivers (character vs block; major/minor numbers).*

---

### Q4.D. System V init supports the concept of runlevels. List the 8 run levels showing when each level is executed.

**Answer:**

| Runlevel | When it is executed / what it does |
|---|---|
| **S** (or s) | **Startup**: the first state entered at boot; runs system initialisation tasks before the normal runlevels |
| **0** | **Halt** — shuts the system down (powers off) |
| **1** | **Single-user mode** — minimal/maintenance mode, one user, no networking, for administration |
| **2** | **Multi-user mode, without networking** |
| **3** | **Multi-user mode, with networking** (full text/console multi-user) |
| **4** | **Not used / undefined** (available for custom use) |
| **5** | **Multi-user mode with networking and a graphical (GUI) login** |
| **6** | **Reboot** — restarts the system |

**Explanation:**
**System V init** organises the machine into a set of **runlevels** — named operating states, each defined by *which services should be running*. `init` brings the system to a target runlevel by running the appropriate scripts. The convention is 8 levels: a startup state **S** plus the numeric levels **0-6**.

- **S** is entered first and performs one-time boot-time initialisation.
- **0** and **6** are the "transition out" levels: **0 = halt** (power off) and **6 = reboot**.
- **1** is **single-user** maintenance mode (no network, minimal services) used for repairs.
- **2 / 3 / 5** are the everyday **multi-user** levels, differing by capability: **2** = multi-user but **no networking**, **3** = multi-user **with networking** (console), **5** = multi-user with networking **and a graphical desktop**.
- **4** is left **unused/undefined** for site-specific customisation.

When switching runlevels, init runs the **K (kill)** scripts to stop services not wanted in the new level, then the **S (start)** scripts to start the ones that are. Asking for runlevel 0 or 6 is therefore how you cleanly halt or reboot.

*Source: Lecture 06 — Init & Runlevels (System V runlevels S, 0-6).*

---

<a id="exam-linux-25"></a>

## Linux_25 — CSY441 Final Exam (27/5/2025, Prof. Said Ghoniemy & Dr. Karim Emara) — Ain Shams FCIS

Total questions: 19

---

### Q1.1. If the system has no MMU, the best C library can be used is ____ . a) musl libc  b) libm  c) uClibc-ng  d) glibc

**Answer:** c) uClibc-ng

**Explanation:**
A **C library** is the layer of code that sits between your application and the kernel. It implements the standard C functions (`printf`, `malloc`, `open`, `strcpy`, …) and wraps the raw kernel system calls so ordinary programs can use them. Every Linux program except a fully static "freestanding" one is linked against a C library, so choosing it is a core decision when you build an embedded toolchain.

An **MMU (Memory Management Unit)** is the hardware block that translates virtual addresses to physical addresses and gives every process its own private, protected address space. Big, full-featured CPUs have one; tiny/cheap microcontroller-class cores (the "no-MMU" world, historically run by **uClinux**) do **not**. Without an MMU there is no `fork()` with copy-on-write, no demand paging, and no per-process virtual memory — so a C library that assumes an MMU (like glibc) cannot run there.

The course gives a decision flowchart for picking the C library:
- **No MMU** → **uClibc-ng** (the maintained successor of the old uClibc, designed exactly for uClinux/no-MMU targets; it is small and avoids MMU-dependent features).
- Has MMU but very tight RAM (e.g. under ~32 MiB) → **musl** (small, modern, fast).
- Otherwise (desktop-class / plenty of resources) → **glibc** (the full GNU C library, the default on most distros).

So for a no-MMU system the correct answer is **uClibc-ng**.

Why the others are wrong:
- a) **musl libc** — small and modern, but it targets MMU-capable systems with tight RAM, not the no-MMU case.
- b) **libm** — this is *not* a general C library at all; it is the **math** library (`sin`, `cos`, `sqrt`, …) that complements libc. A distractor.
- d) **glibc** — the full GNU C library; it is large and assumes an MMU, so it is the wrong choice for no-MMU.

*Source: Lecture 02 — Toolchain (C library selection / uClibc-ng for no-MMU).*

---

### Q1.2. The linux command that lists files including hidden items is ____ . a) ls -l  b) ls -v  c) ls -h  d) ls -a

**Answer:** d) ls -a

**Explanation:**
`ls` is the basic command that lists the contents of a directory. By default it hides any entry whose name begins with a dot (`.`), e.g. `.bashrc`, `.config`, and the special `.` (current dir) and `..` (parent dir). These are **hidden / "dotfiles"** — a Unix convention used for configuration and bookkeeping files you do not normally want cluttering a listing.

The `-a` flag means **"all"**: it tells `ls` to stop hiding dotfiles and show *every* entry, including the hidden ones. So `ls -a` is exactly "list files including hidden items".

These letter flags are the everyday `ls` toolbox and each does something different:
- `-a` = **all** (show hidden dotfiles) ← the answer.
- `-l` = **long** listing (one file per line with permissions, owner, group, size, date).
- `-h` = **human-readable** sizes (e.g. `4.0K`, `2.1M`) — only meaningful together with `-l`.
- `-v` = **natural / version sort** of the names (so `file2` sorts before `file10`).

Why the others are wrong:
- a) `ls -l` — gives the detailed long listing but still **hides** dotfiles.
- b) `ls -v` — only changes the *sort order*; it does not reveal hidden files.
- c) `ls -h` — only makes sizes human-readable (and needs `-l` to even show sizes); it does not reveal hidden files.

*Source: Lecture 01 — Intro (shell / basic file commands).*

---

### Q1.3. The bootloader code stored in DRAM and loads the kernel image and device tree into DRAW is called ____ . a) ROM Code  b) TPL  c) SPL  d) SoC

**Answer:** b) TPL

**Explanation:**
*(Note: "DRAW" in the question is a typo for **DRAM** — the main dynamic RAM.)*

Booting an embedded SoC happens in **stages** because at power-on almost no hardware is initialised yet — in particular, the large external **DRAM** is not usable until its memory controller has been programmed. So the boot proceeds from the smallest, most primitive piece of code to progressively larger ones, each preparing the environment for the next. The course names three stages:

1. **ROM code** — burned permanently inside the SoC's mask ROM. It runs first, while DRAM is still dead, so it can only use the small on-chip **SRAM**. Its job is to load the next tiny stage from a preprogrammed boot source into SRAM.
2. **SPL (Secondary Program Loader)** — a small loader that runs from SRAM. Its key job is to **set up the memory controller (initialise DRAM)** and then load the full bootloader (TPL) into that now-working DRAM.
3. **TPL (Tertiary Program Loader)** — the **full bootloader** (e.g. full U-Boot), which is **already living in DRAM** by this point. It provides the interactive command line and, crucially, **loads the kernel image and the flattened device tree (FDT) into DRAM**, then jumps to the kernel.

The question's exact wording — "stored in DRAM and loads the kernel image and device tree into DRAM" — is the textbook description of the **TPL** stage. The disambiguator across years is *who does the loading and into which memory*: ROM code loads into SRAM; SPL initialises DRAM and loads TPL; TPL (in DRAM) loads the kernel + device tree.

Why the others are wrong:
- a) **ROM Code** — runs first from on-chip ROM into **SRAM**; it cannot use DRAM yet, and it does not load the kernel.
- c) **SPL** — its job is to *initialise the memory controller* and load **TPL** into DRAM, not the kernel.
- d) **SoC** — "System on Chip" is the chip itself, not a boot stage; pure distractor.

*Source: Lecture 03 — Bootloader (boot phases ROM code → SPL → TPL).*

---

### Q1.4. To define a config option that can be compiled as a kernel module, it should be defined as ____ option type. a) int  b) tristate  c) mod  d) bool

**Answer:** b) tristate

**Explanation:**
The Linux kernel's configuration system (**Kconfig**) describes each configurable feature as a *config option* with a **type**. The type determines what values the option can take. The most common types are `bool` (on/off) and `tristate` (on/off/module).

A **kernel module** is a piece of driver/feature code that is compiled **separately** and can be loaded into the running kernel on demand (`.ko` file, `insmod`/`modprobe`) instead of being baked into the kernel image. To allow a feature to be built as a module, its option must support **three** states, which is exactly what **`tristate`** provides:
- **`y`** (yes) — compiled **built-in** (statically linked into the kernel image, `vmlinux`).
- **`m`** (module) — compiled as a **loadable module** (`.ko`).
- **`n`** (no) — **not** compiled at all.

A plain `bool` option only offers `y`/`n` (built-in or off) — it has **no** module option. So any feature that you want to be able to build as a module *must* be declared `tristate`.

Why the others are wrong:
- a) **int** — holds a numeric value (e.g. a buffer size), not a build choice.
- c) **mod** — not a real Kconfig type; invented distractor.
- d) **bool** — only `y`/`n`; it **cannot** be built as a module.

*Source: Lecture 04 — Kernel & Kconfig (tristate option type).*

---

### Q1.5. In a really minimal root filesystem, two nodes are needed to boot with BusyBox: _____. a) console and null  b) console and urandom  c) console and tty  d) null and urandom

**Answer:** a) console and null

**Explanation:**
A **root filesystem (rootfs)** is the directory tree the kernel mounts at `/` after it boots; it contains the programs, libraries, config files and **device nodes** the system needs to actually run. A **device node** is a special file under `/dev` that represents a hardware (or virtual) device to user space — it carries a type (character `c` / block `b`), a **major** number (which driver) and a **minor** number (which instance). You create them with `mknod`.

Even the *smallest possible* BusyBox-based rootfs cannot boot without **two** specific device nodes:
- **`/dev/console`** — the system console. `init` (BusyBox) opens it to print boot messages and to give the shell somewhere to read/write. Without it, init has no controlling terminal and boot fails. (Created as `mknod -m 600 /dev/console c 5 1`.)
- **`/dev/null`** — the "bit bucket": writes are discarded, reads return end-of-file. Countless scripts and programs redirect to/from it, so it must exist very early. (Created as `mknod -m 666 /dev/null c 1 3`.)

Hence the minimal pair is **console and null**.

Why the others are wrong (all involve a node that, while useful, is *not* required for the minimal boot):
- b) console and **urandom** — `/dev/urandom` is a randomness source, not needed to boot.
- c) console and **tty** — `/dev/tty` is handy but not part of the absolute minimum.
- d) **null and urandom** — missing `/dev/console`, which is essential for init/the console.

*Source: Lecture 05 — Root Filesystem (minimal rootfs / device nodes).*

---

### Q1.6. In BusyBox init, shell commands that need to be performed at boot time should be placed in ______ file. a) /etc/init  b) /etc/startup  c) /etc/inittab  d) /etc/init.d/rcS

**Answer:** d) /etc/init.d/rcS

**Explanation:**
**`init`** is the very first user-space program the kernel starts (it runs as **PID 1**) and it is responsible for bringing the system up. In **BusyBox init**, the behaviour of init itself is controlled by a config file, while the *actual boot-time shell commands* (mount filesystems, set the hostname, start daemons, configure the network, …) are kept in a separate **shell script**.

That script is **`/etc/init.d/rcS`**. The course slide describes it as "the place to put initialization commands that need to be performed at boot." When init starts, its first action (a `sysinit` entry in the inittab) typically runs `/etc/init.d/rcS`, and *that* script is where you put your boot-time shell commands.

This is a classic exam trap. There are **two related but different** facts:
- *Where init begins reading its configuration* → **`/etc/inittab`** (this is what earlier years asked).
- *Where boot-time shell commands go* → **`/etc/init.d/rcS`** (this is what **this** question asks).
The question's stem is "shell commands … performed at boot time," so the answer is the **rcS** script, not inittab. Read the stem carefully.

Why the others are wrong:
- a) `/etc/init` — not a standard BusyBox config file/path for this purpose.
- b) `/etc/startup` — invented name; not used by BusyBox init.
- c) `/etc/inittab` — the file init *reads to know what to do*; it is not where you put a list of boot shell commands (it instead points at rcS via a `sysinit` action).

*Source: Lecture 06 — Init & Runlevels (inittab vs /etc/init.d/rcS).*

---

### Q1.7. In a character device driver, the major number maps the device node to a particular _____, while the minor number tells which ____ is being accessed of this device. a) device class, instance  b) interface, partition  c) device driver, partition  d) device class, driver

**Answer:** a) device class, instance

**Explanation:**
Every device node under `/dev` is identified by two numbers: a **major** and a **minor**. The kernel uses these to route operations (open/read/write) to the right driver and the right piece of hardware.

For a **character device driver** (a driver that handles byte-stream devices like serial ports, terminals, GPIO, etc.):
- The **major number** maps the node to a particular **device class / driver** — i.e. it tells the kernel *which* driver owns this kind of device.
- The **minor number** tells the driver *which specific* **instance** of that device class is being accessed (e.g. serial port 0 vs serial port 1, both handled by the same driver).

So major = "what kind of device / which driver," minor = "which one of those." For character devices the slide wording is **device class, instance**.

This is a deliberate flip of the *block*-device wording. For **block** devices the minor number usually identifies a **partition** of a disk — that is the "partition" answer that appears in other years. Here the question explicitly says **character** device, so "partition" is wrong and the correct minor meaning is **instance**.

Why the others are wrong:
- b) **interface, partition** — "partition" is the block-device meaning, not character.
- c) **device driver, partition** — again "partition" is block, not character.
- d) **device class, driver** — first word is fine, but the minor number does **not** select a "driver"; it selects an *instance*.

*Source: Lecture 08 — Device Drivers (major/minor numbers, char vs block).*

---

### Q1.8. In buildroot source tree, the ____ directory contains a set of predefined configurations, similar to the concept of defconfig in the kernel. a) system  b) configs  c) package  d) defconfig

**Answer:** b) configs

**Explanation:**
**Buildroot** is a build system that produces a complete embedded Linux image (cross-toolchain + bootloader + kernel + root filesystem) from one configuration. Like the kernel, Buildroot ships a set of **ready-made starting configurations** so you don't have to configure everything from scratch for a known board.

In the kernel source tree, a **defconfig** is a saved, minimal configuration file for a particular board/CPU that you load with `make <name>_defconfig`. Buildroot copies this idea: it keeps its **predefined board configurations** in the **`configs/`** directory (each file like `raspberrypi_defconfig`, `beaglebone_defconfig`, …). You select one with `make <name>_defconfig`, and it sets up Buildroot for that target. The slide describes `configs/` as "a set of predefined configurations, similar to the concept of defconfig in the kernel" — almost the exact wording of the question.

(For context, the Buildroot source tree also has `package/` ≈ 2000 packages, plus `boot/`, `fs/`, `linux/`, `system/`, `toolchain/` directories.)

Why the others are wrong:
- a) **system** — holds skeleton/system-level files (e.g. the rootfs skeleton), not the predefined configs.
- c) **package** — holds the recipes for the ~2000 buildable packages, not configurations.
- d) **defconfig** — that is the *kernel's* term/file; in **Buildroot** the directory is named **configs/**, so "defconfig" is the trap.

*Source: Lecture 07 — Build Systems (Buildroot source tree, configs/).*

---

### Q1.9. For a NAND flash of 4K page, the OOB area is ____ bytes per page. a) 16  b) 64  c) 32  d) 128

**Answer:** d) 128

**Explanation:**
**NAND flash** is organised into **pages** (the unit of read/write, typically 2 KB or 4 KB) grouped into **erase blocks**. Each page has, in addition to its main data area, a small extra region called the **OOB (Out-Of-Band) area** (also "spare area"). The OOB is *not* used for file data; it stores housekeeping information:
- the **ECC (Error-Correcting Code)** bytes needed because NAND bits flip and must be corrected,
- the manufacturer's **bad-block marker**,
- and filesystem metadata (e.g. JFFS2 clean markers).

NAND is intrinsically error-prone, so the OOB has to scale with page size. The course gives the rule for common **SLC** NAND: roughly **1 OOB byte for every 32 data bytes**. Apply it to a 4 KB page:

```
OOB = page_size / 32 = 4096 / 32 = 128 bytes
```

So a **4 K page → 128 bytes** of OOB. (As a sanity check, the same rule gives a 2 KB page → 2048/32 = **64** bytes, which is the well-known classic value.)

Why the others are wrong:
- a) **16** — far too small; that is closer to old 512-byte-page NAND.
- b) **64** — correct for a **2 KB** page, not a 4 KB page.
- c) **32** — too small; this is the *data-bytes-per-OOB-byte ratio*, not the total.

*Source: Lecture 09 — Flash & MTD (NAND OOB area / sizing).*

---

### Q1.10. The Yocto project component that contains a list of settings and tasks for building packages is called ___ . a) layers  b) Recipes  c) Configuration Files  d) metadata

**Answer:** b) Recipes

**Explanation:**
The **Yocto Project** builds custom Linux distributions from many small text descriptions collectively called **metadata**. Within that metadata, the unit that describes *how to build one piece of software* (one package) is a **recipe** (file extension `.bb`).

The course defines a **recipe** as exactly "a list of settings and tasks (instructions) for building packages" — matching the question word for word. A recipe says where to fetch the source (its *settings*: version, source URL, license, dependencies) and how to build it through a sequence of *tasks*: `do_fetch`, `do_unpack`, `do_patch`, `do_configure`, `do_compile`, `do_install`, `do_package`. Recipes are stored inside **layers**.

Why the others are wrong:
- a) **layers** — a layer is a *collection/repository* of metadata (recipes, classes, configs) that can be stacked and overridden; it is the container, not the per-package settings+tasks list.
- c) **Configuration Files** — these hold *global/user* variables and machine/distro settings, not the build instructions for an individual package.
- d) **metadata** — the umbrella term for *all* of the above (recipes + config files + classes); too broad — the specific "settings and tasks for one package" item is a **recipe**.

*Source: Lecture 07 — Build Systems (Yocto glossary: recipes).*

---

### Q1.11. In device tree, the ___ property is that used by the operating system to decide which device driver to bind to a device. a) node  b) ranges  c) reg  d) compatible

**Answer:** d) compatible

**Explanation:**
A **device tree (DT)** is a data structure that describes the hardware of a board to the kernel — which devices exist, where their registers live, which interrupts they use — *without hard-coding any of it into the kernel*. The kernel reads the DT at boot and, for each device node, must decide **which driver** should handle it.

It makes that decision using the **`compatible`** property. Each device node carries a `compatible = "vendor,model";` string (often a list, most-specific first, e.g. `"arm,pl011"`). Each driver advertises the `compatible` strings it supports. The kernel **matches** a node's `compatible` value against the drivers' supported strings and **binds** the matching driver to that device. So `compatible` is precisely "what the OS uses to decide which driver to bind."

Why the others are wrong:
- a) **node** — a "node" is the device entry itself, not a property used for driver matching.
- b) **ranges** — describes *address translation* between a bus and its parent, not driver binding.
- c) **reg** — gives the device's register **address and size** (so the driver knows where the hardware is), but it is *not* what selects the driver.

*Source: Lecture 03 — Device Tree (compatible / reg).*

---

### Q2.A. Assume user2 belongs to group1 and user3 belongs to group2, show the permissions (read, write and execute) for the two files test.txt and file.dtb for the three users (user1, user2, and user3), given the following output of ls command: [6 marks]

```
-rwxr-xr-- 1 user1 group1 1734732 Nov 3 16:02 test.txt
-rw-r-xr-- 1 user1 group2 8075 Oct 28 07:41 file.dtb
```

**Answer:**

How to read a permission string: the 10 characters are `[type][owner rwx][group rwx][others rwx]`. Each `rwx` triad applies to a different *category* of user, and you pick the triad based on the user's relationship **to that specific file**: owner → first triad; member of *that file's* group → middle triad; everyone else → last (others) triad.

Decode the two files:
- `test.txt` → type `-`, owner `rwx`, group `r-x`, others `r--`; owner **user1**, group owner **group1**.
- `file.dtb` → type `-`, owner `rw-`, group `r-x`, others `r--`; owner **user1**, group owner **group2**.

Memberships: user2 ∈ **group1**, user3 ∈ **group2**. The catch is the two files have **different group owners**, so the same user can fall in different categories per file.

| | test.txt (group1) | file.dtb (group2) |
|---|---|---|
| **user1** (owner of both) | r, w, x  (owner triad `rwx`) | r, w  (owner triad `rw-` → **no execute**) |
| **user2** (in group1) | r, x  (group triad of test.txt = `r-x`, since group1 is test.txt's group) | r only  (user2 is **not** in group2 → falls in **others** of file.dtb = `r--`) |
| **user3** (in group2) | r only  (user3 not in group1 → **others** of test.txt = `r--`) | r, x  (group triad of file.dtb = `r-x`, since group2 is file.dtb's group) |

**Explanation:**
Permissions are the Unix access-control mechanism. Three permission bits — **r** (read = 4), **w** (write = 2), **x** (execute = 1) — are stored separately for three categories: the file's **owner (user)**, the file's **group**, and **others** (everyone else). When a process tries to access a file, the kernel decides which single category applies to that user and checks only that triad:

1. If the user is the file's **owner**, use the **owner** triad (the other triads are ignored, even if they grant more).
2. Else if the user is a member of the file's **group owner**, use the **group** triad.
3. Else use the **others** triad.

The exam-critical subtlety here is step 2: you must look at **each file's own group owner**.
- For `test.txt` the group is **group1**. user2 *is* in group1 → user2 gets test.txt's **group** triad (`r-x` → r,x). user3 is *not* in group1 → user3 gets test.txt's **others** triad (`r--` → r only).
- For `file.dtb` the group is **group2**. user3 *is* in group2 → user3 gets file.dtb's **group** triad (`r-x` → r,x). user2 is *not* in group2 → user2 falls to file.dtb's **others** triad (`r--` → r only).

Also note user1 (the owner) can **read and write** both files but can only **execute** `test.txt` — `file.dtb`'s owner triad is `rw-`, with no `x`. This is the trap the examiner planted: same owner, same two "members," but the differing group ownership flips user2 and user3 between the group and others categories.

*Source: Lecture 05 — Root Filesystem (permissions matrix).*

---

### Q2.B. For the following shell command sequences, describe briefly the functions or the effect on the system after running each sequence. DON'T describe the function of each line. [8 marks]

```sh
# (1)
cd /sys/class/gpio/ ; echo 12 > export ; echo out > gpio12/direction
# (2)
mknod -m 666 dev/null c 1 3 ; mknod -m 600 dev/console c 5 1
# (3)
genext2fs -b 4096 -d rootfs -D device-table.txt -U rootfs.ext2
# (4)
arm-unknown-linux-gnueabi-readelf -a busybox | grep "interpreter" ; arm-unknown-linux-gnueabi-readelf -a busybox | grep "Shared"
```

**Answer:** _(effect of each sequence as a whole)_

**(1)** Exports **GPIO line 12** to user-space control and makes it an **output**. Writing `12` to `export` creates the directory `/sys/class/gpio/gpio12/`; writing `out` to its `direction` sets the pin as an output. **Net effect:** GPIO 12 is now controllable from user space, and its electrical level can be driven by writing `0`/`1` to `gpio12/value`.

**(2)** Creates the **two character device nodes a minimal BusyBox rootfs must have to boot**: `/dev/null` (character device, major **1**, minor **3**, mode **666** = world read/write) and `/dev/console` (character device, major **5**, minor **1**, mode **600** = root-only). **Net effect:** the rootfs gains the essential console and null devices needed for init to start.

**(3)** Builds a **~4 MB ext2 root-filesystem image** named `rootfs.ext2` (4096 blocks × 1 KB = 4 MB) from the staging directory `rootfs/`, applying `device-table.txt` to create the required **device nodes and to set correct ownership/permissions** — all **without needing root privileges**. **Net effect:** a ready-to-flash/copy rootfs image is produced (e.g. to write to an SD card or flash).

**(4)** Inspects the cross-compiled `busybox` binary to discover its **dynamic-link dependencies**: the first `grep "interpreter"` reveals the **program interpreter (the dynamic linker)**, e.g. `/lib/ld-linux.so.3`; the second `grep "Shared"` lists the **(NEEDED) shared libraries**, e.g. `libc.so`, `libm.so`. **Net effect:** you learn which dynamic loader and which shared libraries must be copied into the target root filesystem for busybox to run.

**Explanation:**
This question tests whether you can read a short command sequence and state the **overall outcome**, not narrate each line. Here is the "why" behind each:

**(1) GPIO via sysfs.** The kernel's `gpiolib` exposes GPIO pins at `/sys/class/gpio`. You "claim" a pin for user space by echoing its number into the `export` file; that makes a per-pin folder `gpioN/` appear, containing control files `direction`, `value`, `edge`, `active_low`. A freshly exported pin defaults to **input**; writing `out` to `direction` switches it to output so you can then drive it high/low via `value`. The combined effect of the three commands is "pin 12 is now a user-controllable output."

**(2) Device nodes for boot.** A device node is a special `/dev` file mapping a name to (type, major, minor). `mknod -m <mode> <name> c <major> <minor>` creates a **c**haracter node with given permission mode. The two nodes made here, `/dev/null` (1,3) and `/dev/console` (5,1), are precisely the minimum a BusyBox system needs: console so init has a terminal for messages/shell, null as the discard device used everywhere. So the sequence's effect is "the minimal rootfs now has its two mandatory device nodes."

**(3) Image build with a device table.** `genext2fs` builds an **ext2** filesystem image straight from a directory tree, on the build host, **as a normal user**. `-b 4096` = 4096 blocks (1 KB each → ~4 MB), `-d rootfs` = take the contents from `rootfs/`, `-D device-table.txt` = apply a *device table* (a text file listing nodes/owners/modes) so the image ends up with proper **root-owned files and device nodes even though you ran it unprivileged**, `-U` = make the directory owners root, and the final argument `rootfs.ext2` is the output image. Effect: one deployable rootfs image file.

**(4) Dependency discovery with readelf.** `readelf -a` dumps an ELF binary's headers. Filtering for `"interpreter"` shows the **`.interp`** entry — the path of the **dynamic linker/loader** that must run first to load the program. Filtering for `"Shared"` shows the dynamic section's **(NEEDED) shared library** entries. Together they tell you *exactly which loader and libraries* a dynamically-linked `busybox` requires, so you know what to copy into the target rootfs (a common cause of "not found" boot failures is a missing dynamic linker or library).

*Source: Lecture 08 — GPIO sysfs (1); Lecture 05 — device nodes (2) and readelf dependencies (4); Lecture 06 — device table + genext2fs (3).*

---

### Q2.C. For the given device tree, complete the missing parts [A]-[F] given: 32-bit address bus connecting: UART Controller (model pl011) at 0x48020000, size 0x1000, interrupt 32; Interrupt Controller (model pl190) at 0x47050000, size 0x1000. [6 marks]

```dts
/dts-v1/;
/ {
    #address-cells = __[A]__;
    __[B]__;
    interrupt-parent = <&intc>;
    serial@48020000 {
        compatible = "arm,pl011";
        __[C]__;
        __[D]__;
    };
    intc: interrupt-controller@47050000 {
        compatible = "arm,pl190";
        reg = ___[E]___;
        ____[F]____;
        #interrupt-cells = <1>;
    };
};
```

**Answer:** _(completed device tree)_

```dts
/dts-v1/;
/ {
    #address-cells = <1>;                 /* [A] 32-bit address bus -> 1 address cell */
    #size-cells = <1>;                    /* [B] 32-bit length -> 1 size cell */
    interrupt-parent = <&intc>;
    serial@48020000 {
        compatible = "arm,pl011";
        reg = <0x48020000 0x1000>;        /* [C] base address + size */
        interrupts = <32>;                /* [D] one cell, because #interrupt-cells = <1> */
    };
    intc: interrupt-controller@47050000 {
        compatible = "arm,pl190";
        reg = <0x47050000 0x1000>;        /* [E] base address + size */
        interrupt-controller;             /* [F] the empty property: this node receives interrupts */
        #interrupt-cells = <1>;
    };
};
```

- **[A]** `<1>`
- **[B]** `#size-cells = <1>;`
- **[C]** `reg = <0x48020000 0x1000>;`
- **[D]** `interrupts = <32>;`
- **[E]** `<0x47050000 0x1000>`
- **[F]** `interrupt-controller;`

**Explanation:**
Device-tree authoring is governed by a few precise rules; each blank tests one.

**Cells (`#address-cells` / `#size-cells`).** A device's `reg` property is written as pairs of `<address length>`, but the **number of 32-bit "cells"** used for the address and for the length is set by the **parent node** via `#address-cells` and `#size-cells`. The bus here is **32-bit**, so one 32-bit cell is enough to hold an address and one cell for a length:
- **[A]** `#address-cells = <1>` (a 32-bit address fits in one cell).
- **[B]** the matching `#size-cells = <1>;` (a 32-bit size fits in one cell). These two properties live on the root `/` node and govern its children's `reg` format.

**`reg` (where the hardware is).** With 1 address cell + 1 size cell, each `reg` is `<base size>`:
- **[C]** the UART is at base `0x48020000`, size `0x1000` → `reg = <0x48020000 0x1000>;`.
- **[E]** the interrupt controller is at base `0x47050000`, size `0x1000` → `reg = <0x47050000 0x1000>;`.
Note the node names (`serial@48020000`, `interrupt-controller@47050000`) use the **unit address = the first `reg` address**, which is why they match.

**`interrupts` (which IRQ).** The UART signals interrupt **32**. The *width* of an interrupt specifier is dictated by the interrupt controller's **`#interrupt-cells`**, which is `<1>` here — so the specifier is a single number:
- **[D]** `interrupts = <32>;` (one cell). `interrupt-parent = <&intc>` (inherited from root) tells the UART that this number is interpreted by `intc`.

**`interrupt-controller;` (the empty property).** A node that *is* an interrupt controller must declare itself with the **empty (valueless) property** `interrupt-controller;`. It takes no value — its mere presence flags "this node receives/dispatches interrupt signals." That is blank **[F]**. (Do not confuse it with `interrupt-parent`, which is a *phandle* pointing at a controller; that property is already present on the root.)

*Source: Lecture 03 — Device Tree authoring (cells, reg, interrupts, interrupt-controller).*

---

### Q3.A. Describe briefly the content of the following Buildroot output sub-directories: images, staging, and target.

**Answer:**
After a Buildroot build, results land under `output/`. The three asked sub-directories are:

- **`output/images/`** — the **deployable build results**: the bootloader, the kernel image (plus its device-tree blobs / DTBs), and the **root-filesystem images** (e.g. `rootfs.ext2`, `rootfs.tar`, `rootfs.ubi`). This is the directory you actually take and flash/copy to the target board.
- **`output/staging/`** — a **symbolic link to the cross-toolchain's sysroot**. It holds the target's **headers and libraries** in their normal `/usr/include`, `/usr/lib` layout, and exists so you can **compile (cross-compile) code against** the target's libraries. It is *not* copied to the target.
- **`output/target/`** — the **staging area for the root directory**: the rootfs tree exactly as it will appear on the target (`/bin`, `/etc`, `/lib`, …). However, it is **not directly bootable on its own**, because as a plain host directory it has **no real device nodes** and its files are **owned by the host user**, not root. Buildroot fixes ownership/permissions and adds device nodes by applying a **device table** when it builds the final image in `images/`.

**Explanation:**
Buildroot separates concerns into different output directories so each part of the cross-development workflow has a clear home. Understanding the three above (and why `target/` alone won't boot) is the heart of this question.

- **images/** is the *output you ship*. Everything a board needs to start — bootloader, kernel + DTB, and a filesystem image — is gathered here in flashable form.
- **staging/** is the *development sysroot*. When you cross-compile your own application, the compiler needs the target's headers and `.so` libraries to link against; `staging/` (a symlink into the toolchain's sysroot) provides exactly that. It deliberately keeps full headers and unstripped libraries that you would not necessarily put on the device.
- **target/** is the *assembled rootfs tree*, the intermediate from which the rootfs **image** is generated. The crucial exam point is **why it is not bootable directly**: a normal directory on the build host cannot contain genuine device nodes (creating them needs root), and its files carry host ownership. So Buildroot does not boot `target/` as-is; instead it runs the image step, applying a **device table** to set root ownership/correct permissions and to create device nodes inside the generated image. That generated image is what appears in `images/`.

(For completeness, `output/` also contains `build/` = per-package source/build directories, and `host/` = the host tools, including the cross-toolchain itself under `output/host/usr/bin`.)

*Source: Lecture 07 — Build Systems (Buildroot output/ tree).*

---

### Q3.B. Describe briefly the following yocto project terminology: BitBake, recipes and OE-Core.

**Answer:**

- **BitBake** — the Yocto **build engine / task scheduler** (conceptually like `make`, but more powerful). It **parses the configuration files and recipes** (the *metadata*), works out the **dependency tree** between components, and then **executes the tasks** that download, configure, compile and package every component to produce the final images.
- **Recipes** (files ending `.bb`) — "a **list of settings and tasks (instructions) for building packages**." A recipe describes *one* software component: its **settings** (version, source location/URL, license, dependencies) and its **tasks** (`do_fetch`, `do_unpack`, `do_patch`, `do_configure`, `do_compile`, `do_install`, `do_package`). Recipes are stored inside **layers**.
- **OE-Core (OpenEmbedded-Core)** — the set of **base layers (recipes, classes, configuration) that are shared between all OpenEmbedded-based build systems**. It is the common, validated foundation (the core that OpenEmbedded and Poky merged into in 2010) on top of which everyone layers their board- and product-specific metadata.

**Explanation:**
Yocto does not ship a finished distro; it ships the *machinery* to build your own from text descriptions. These three terms name the engine, the per-package description, and the shared base:

- **BitBake** is the brain. You give it metadata; it reads every recipe and config file, computes what depends on what, then schedules and runs the build tasks in the right order (fetching sources, configuring, compiling, installing, assembling images). Think "a smarter `make` that understands cross-compilation and whole-image builds."
- **Recipes** are the per-component instructions BitBake consumes. Each `.bb` file is a self-contained "how to build this one package" description: where to get the code and how to turn it into an installable package, expressed as the standard sequence of `do_*` tasks. This exact phrase — "a list of settings and tasks for building packages" — is the course's definition (and is also why Q1.10's answer was *recipes*).
- **OE-Core** is the common ground everyone starts from. Rather than every project reinventing the recipes for basic system software, OpenEmbedded-Core provides a **validated base set of layers/recipes/classes** shared across all OpenEmbedded-based systems (Yocto's Poky reference distro builds on it). You then add your own layers above OE-Core for your hardware and applications.

(Related glossary you may be asked separately: **metadata** = recipes + config files + classes; **layers** = stackable repositories of metadata that can override one another; **Poky** = Yocto's reference distribution that bundles BitBake + OE-Core + a sample config.)

*Source: Lecture 07 — Build Systems (Yocto glossary).*

---

### Q3.C. Compare between NAND and NOR Flash in terms of erase block size, reliability and controller complexity.

**Answer:**

| Aspect | **NOR Flash** | **NAND Flash** |
|---|---|---|
| **Erase block size** | Larger, simple structure (~**128 KB** blocks); accessed **word-at-a-time** for read/write and is **memory-mappable**, so code can run directly from it (**XIP** — eXecute In Place). | Organised in **erase blocks of ~16–512 KB**, but read/written in small **pages (typically 2–4 KB)**; a page = data area + a spare **OOB** area. Not directly memory-mappable. |
| **Reliability** | **Very reliable**; high endurance (~**100K–1M** erase cycles). Expensive per bit. Few or no bad blocks. | **Less reliable**: ships with (and develops) **bad blocks**, and individual bits flip, so it **requires ECC** (Hamming for SLC, BCH for MLC/TLC) and bad-block management. Lower endurance (~10K–100K cycles; TLC ~1K). Cheap per bit / high density. |
| **Controller complexity** | **Simple** — essentially "no initialization, just wiring and address mapping"; behaves much like ROM/RAM on the bus. | **Complex** — needs a dedicated **hardware + software controller** to handle **page/block I/O, ECC, and bad-block handling**; this lives in the SoC's NAND controller plus bootloader/kernel drivers. |

**Explanation:**
NOR and NAND are two flash technologies with opposite trade-offs; the course always compares them on these three axes.

- **Erase block size / access.** Flash can be read freely but can only be *erased* a whole **block** at a time (you cannot rewrite a single byte in place). **NOR** is wired so the CPU can address it **word by word** and even run code straight out of it (memory-mapped, **XIP**) — convenient, used for boot code. **NAND** trades that random access for density: it is accessed in **pages** (2–4 KB) within larger erase blocks, and cannot be executed in place. Both have erase blocks on the order of tens to hundreds of KB; the *defining* difference is **word-addressable/XIP (NOR) vs page-based I/O (NAND)**.
- **Reliability.** **NOR** is robust and high-endurance but expensive, so it is used for small, critical storage (bootloaders). **NAND** is cheap and dense but **error-prone**: it has factory **bad blocks** (and accumulates more), and bits flip, so it **mandates ECC** and bad-block tracking to be usable. This unreliability is the single biggest practical difference.
- **Controller complexity.** Because NOR "just works" on the bus, its **controller is trivial** (wiring + address mapping). NAND's page model + ECC + bad-block handling demand a **much more complex controller** (hardware engine in the SoC, plus driver/bootloader support). This is the direct consequence of NAND's reliability problems.

(Bottom line: **NOR** = simple, reliable, memory-mappable, small & costly → boot code; **NAND** = complex, less reliable, page-based, large & cheap → bulk storage / rootfs.)

*Source: Lecture 09 — Flash & MTD (NAND vs NOR).*

---

### Q3.D. Why is the mount time of early versions of the JFFS2 filesystem reasonably long? Describe briefly the role of summary nodes in reducing this time.

**Answer:**
**JFFS2** is a **log-structured** flash filesystem: every change (file data, directory entries, metadata) is appended as a **node** to the flash, and newer nodes obsolete older ones. Early JFFS2 keeps **no on-flash index/table of contents** of where everything is.

- **Why mount is slow:** because there is no index, when the filesystem is mounted JFFS2 must **scan the entire flash from start to finish**, reading **every node**, to reconstruct in RAM the current directory tree and the location of each file's valid data. This linear scan costs roughly **~1 second per megabyte**, so on a large flash it can take **tens to hundreds of seconds** — a very long mount time.

- **Role of summary nodes:** a **summary node** is a special node written at the **end of each erase block, just before that block is closed/filled**. It **summarises the contents of that block** (the metadata about all the nodes in it) in one compact place. At mount time, JFFS2 can then read only the **summary node** of each block instead of scanning every individual node — it gets the same information far faster. This dramatically **reduces the mount-time scan and hence the mount time** on large flashes.

**Explanation:**
The key to this question is understanding *log-structured* storage. Instead of updating data in place (which flash cannot do byte-by-byte), JFFS2 **appends** each new version of a file/inode as a node and marks the old node obsolete. Over time the flash holds a long log of nodes, some valid and some obsolete.

- **The mount problem.** The "truth" about the filesystem (which nodes are current, how the directory tree looks, where each file's bytes live) is not stored as a single structure anywhere — it is *implicit* in the whole log. Early JFFS2 therefore has to **replay/scan the entire log at mount** to rebuild that picture in memory. The cost grows with flash size (~1 s/MB), making mount painfully slow on big devices — the reason embedded boards with large JFFS2 partitions booted slowly.
- **The summary-node fix.** Scanning every node is wasteful when most of what the scanner needs is just *metadata*. So JFFS2 was extended to write a **summary node** at the close of each erase block, packing that block's metadata into one node at the block's end. On the next mount, the code reads just the summary nodes (one quick read per block) to rebuild its in-RAM maps, skipping the exhaustive node-by-node scan. Same result, far less I/O → **much shorter mount time**. (This is purely a speed optimisation; it does not change the on-flash data, and the system still works without summaries, just slower.)

*Source: Lecture 09 — Flash & MTD (JFFS2 mount time / summary nodes).*

---

### Q3.E. The bus driver is one of the main components in the Linux device model. List the main responsibilities of the bus driver that it provides to other components.

**Answer:**
In the Linux **device model**, there is one **bus driver per bus type** (USB, PCI, I2C, …). Its main responsibilities are:

1. **Register the bus type** with the kernel — i.e. create and register the bus's `struct bus_type`.
2. **Allow registration of adapter / controller drivers** (e.g. a USB host controller, an I2C adapter) that are able to **detect the devices connected** to that bus.
3. **Allow registration of device drivers** — the drivers for the individual devices found on the bus (USB/PCI/I2C devices).
4. **Match device drivers against the devices** that the adapter drivers have detected (so the right driver binds to each device).
5. **Provide an API** that lets developers implement **both** kinds of drivers (adapter drivers and device drivers) for that bus.
6. **Define the driver- and device-specific data structures** for that bus (for example, for USB: `struct usb_driver` and `struct usb_interface`).

**Explanation:**
Linux models hardware with a unified **device model** built from three cooperating pieces: **buses**, **devices**, and **drivers**. A **bus driver** is the kernel code that represents *one kind of bus* and acts as the matchmaker/registry connecting devices on that bus to their drivers. Understanding its job explains how a USB stick "just works" when you plug it in.

Walking through the six responsibilities:
1. **Registering the bus type** establishes the bus (a `struct bus_type`) in the kernel's device hierarchy so devices and drivers can be attached to it.
2–3. The bus must accept registrations from **two categories** of drivers: **adapter/controller drivers** (which talk to the host controller and *discover* what is plugged in) and **device drivers** (which operate an individual device once found). Keeping these separate lets one controller serve many different devices.
4. The core service is **matching**: when an adapter reports a detected device, the bus driver compares it against the registered device drivers and **binds** the matching one — this is what triggers a driver's `probe()` when you hot-plug a device.
5. To make writing those two driver kinds practical, the bus driver exposes an **API** (registration calls, helper functions) specific to the bus.
6. Finally it **defines the bus-specific structures** (e.g. `struct usb_driver`, `struct usb_interface`) that device/adapter drivers fill in, giving each bus its own tailored programming interface.

Together these let the kernel dynamically discover devices and hand each to the correct driver — the foundation of hot-plug and modular driver support.

*Source: Lecture 08 — Device Drivers (bus driver responsibilities in the Linux device model).*

---

<a id="exam-final"></a>

## Final Exam (`Final Exam.pdf`) — CSY441 Final (10/6/2023, Dr. Karim Emara) — Ain Shams FCIS

Total questions: 22

> **Note — duplicate paper.** `Final Exam.pdf` is identical, word-for-word, to `[Linux_23] Final Exam NewBylaw_.pdf` (same date 10/6/2023, same examiner Dr. Karim Emara, same four questions). The answers and explanations below are therefore the same as the **Linux_23** section; this section is reproduced in full so every source file in the folder has its own complete solution.

*Exam header: FCIS – Ain Shams University; CSY441: Recent Topics in Computer Systems; Final 10/6/2023; Level 3rd & 4th; Examiner: Dr. Karim Emara; Academic year 2nd term 2022–2023; duration 2 hours. "Answer the following 4 questions."*

---

### Q1.1. Choose the correct answer [7 marks, 10 min]. The ___ is the part of the Linux system that is responsible for loading kernel into memory after system reset and firmware initialization. a) Init program  b) Kernel program  c) Bootloader  d) Board firmware

**Answer:** c) Bootloader

**Explanation:**
When you power on an embedded board, the hardware cannot just "run Linux" — there is a fixed chain of hand-offs, each stage bringing more of the system to life:

1. **Board firmware / ROM code** — tiny code baked into the SoC. It runs first, initializes the bare minimum (clocks, a little on-chip SRAM), and fetches the next stage. It does *not* know what Linux is.
2. **Bootloader** (e.g. U-Boot) — this is the stage that "knows about the kernel." Its job: bring the hardware to a known state, set up RAM, place the kernel parameters (device tree + command line) in memory, then **load the kernel image into RAM and jump to it**.
3. **Kernel** — once loaded and running, it mounts the root filesystem and starts the init program.
4. **Init program** (PID 1) — the first user-space process; it launches all the system daemons and services.

So the component whose specific responsibility is *loading the kernel into memory after reset and firmware init* is the **bootloader**.

- a) Init program — wrong: init is started *by* the kernel, long after the kernel is already in memory. It cannot load the thing that starts it.
- b) Kernel program — wrong: the kernel is the *thing being loaded*; it cannot load itself into memory.
- d) Board firmware — wrong: firmware runs first and only initializes the SoC and loads the *next bootstrap stage*; it is not the component that loads the Linux kernel.

*Source: Lecture 03 — Bootloader (role: init HW, place kernel parameters, load kernel & jump).*

---

### Q1.2. The root filesystem is mounted to the system through the ____. a) mount command  b) root= kernel option  c) bootloader ROM Code  d) init program

**Answer:** b) root= kernel option

**Explanation:**
The **root filesystem (rootfs)** is the top-level "/" directory tree that holds everything user space needs: programs, libraries, configuration, and device nodes. There is a chicken-and-egg problem: the normal way to attach a filesystem is the `mount` command — but `mount` is itself a program that lives *inside* the root filesystem. You cannot run a program from a filesystem that is not mounted yet.

Linux solves this by having the **kernel mount the very first filesystem itself**, before any user-space program runs. The kernel needs to be told *which* device/partition holds the rootfs, and that is exactly what the **`root=` kernel command-line option** does, e.g. `root=/dev/mmcblk0p2`. If `root=` points at the wrong place (or nothing), the kernel cannot find an init program and **panics**.

- a) mount command — wrong: `mount` lives inside the rootfs, so it cannot be used to mount the rootfs itself (it is only used for *additional* filesystems afterward).
- c) bootloader ROM Code — wrong: ROM code runs far earlier and only bootstraps the next stage; it has no concept of a Linux filesystem.
- d) init program — wrong: init is started *after* the rootfs is already mounted; mounting rootfs is a precondition for running init.

*Source: Lecture 05 — Root filesystem (rootfs mounted directly by the kernel per the `root=` option).*

---

### Q1.3. In the filesystem hierarchy standard, the _____ directory stores the devices nodes defined for the hardware attached to the system. a) /sys  b) /driver  c) /bin  d) /dev

**Answer:** d) /dev

**Explanation:**
The **Filesystem Hierarchy Standard (FHS)** is the agreed-upon convention for what each top-level directory in a Linux system contains, so software knows where to find things. Key ones:

- **/dev** — **device nodes**. A device node is a *special file* that represents a piece of hardware (or a virtual device). Opening/reading/writing that file is how user space talks to the driver. Each node carries a type (`c` = character, `b` = block) plus a **major** number (which driver) and **minor** number (which instance/partition). Example: `/dev/ttyS0`, `/dev/null`, `/dev/mmcblk0`.
- /bin — essential user programs (ls, cp, sh).
- /etc — configuration files.
- /lib — shared libraries.
- /sys — exposes the kernel's *driver model* (sysfs), not the device nodes themselves.
- /proc — exposes *process* and kernel info.

So the directory holding the **device nodes** is **/dev**.

- a) /sys — wrong: sysfs exposes kernel/driver-model *information* (attributes, hierarchy), not the actual device-node special files used for I/O.
- b) /driver — wrong: there is no such standard FHS directory.
- c) /bin — wrong: /bin holds executable programs, not device nodes.

*Source: Lecture 05 — FHS / device nodes (/dev).*

---

### Q1.4. The system V init program begins reading the ______ file which defines rules to start programs at boot up and stop them at shutdown. a) /conf/init.d  b) /sbin/init  c) /sbin/boot  d) /etc/inittab

**Answer:** d) /etc/inittab

**Explanation:**
**System V init** (and the BusyBox init used on embedded systems) is the first user-space process, PID 1. To know *what* to start and stop, it reads a configuration file at boot: **`/etc/inittab`**.

Each line in `/etc/inittab` has the format `<id>:<runlevels>:<action>:<process>`, telling init which program to run and *how*:
- `sysinit` — run this before everything else (typically `::sysinit:/etc/init.d/rcS`, which runs the boot-time shell commands).
- `respawn` — keep this running; restart it if it dies (e.g. a getty login prompt).
- `wait` / `once` — run once.

So init **begins by reading `/etc/inittab`**, which defines the rules for starting programs at boot and stopping them at shutdown.

Note the classic trap: `/sbin/init` is the init *executable itself*, not its config file. And the *shell commands* to run at boot go in `/etc/init.d/rcS`, which is *invoked from* inittab — but the file init **reads first** is inittab.

- a) /conf/init.d — wrong: not a standard path; the real script directory is `/etc/init.d/`.
- b) /sbin/init — wrong: that is the init *binary*, not the rules file it reads.
- c) /sbin/boot — wrong: no such standard file.

*Source: Lecture 06 — Init (reads /etc/inittab; entry format `id:runlevels:action:process`).*

---

### Q1.5. In block devices, the minor number identifies the ______ of the device. a) partition  b) device driver  c) interface  d) none of the above

**Answer:** a) partition

**Explanation:**
Every device node carries two numbers:
- **Major number** — identifies the *driver / device class* (which kernel driver handles this device). Same for all instances served by that driver.
- **Minor number** — identifies *which specific thing* within that driver's domain.

What the minor number "means" depends on the **type** of device:
- **Block devices** (mass storage: SD cards, disks): the minor number identifies the **partition**. Example: `mmcblk0` (major 179) → minor 0 = the whole device, and the following minors = `mmcblk0p1`, `mmcblk0p2`, … the individual partitions.
- **Character devices**: the minor number identifies the *instance/interface* (e.g. which serial port).

The question specifically says *block devices*, so the minor number = **partition**.

- b) device driver — wrong: that is what the **major** number identifies, not the minor.
- c) interface — wrong: "interface/instance" is the meaning of the minor number for **character** devices, not block devices (this is the deliberate distractor — note the 2025 paper flipped the question to the character case).
- d) none of the above — wrong: "partition" is exactly correct for block devices.

*Source: Lecture 08 — Major/minor numbers (block minor = partition).*

---

### Q1.6. If the system has no MMU, the best C library can be used is _____. a) musl libc  b) libm  c) uClibc-ng  d) glibc

**Answer:** c) uClibc-ng

**Explanation:**
The **C library** is the layer between your program and the kernel's system calls (it provides `printf`, `malloc`, `open`, etc.). On embedded systems you pick the C library to match the hardware and size constraints.

An **MMU (Memory Management Unit)** is the hardware that provides virtual memory — separate address spaces per process, `fork()` with copy-on-write, etc. Some small microcontrollers/SoCs have **no MMU**; these run a variant of Linux historically called **uClinux**. Without an MMU you need a C library designed to work in that constrained, no-virtual-memory environment.

The decision flowchart taught in the course:
- **No MMU (uClinux)** → **uClibc-ng** ← (this question)
- Has MMU but very small RAM (< ~32 MiB) → **musl**
- Otherwise (general purpose, full features) → **glibc**

So for a **no-MMU** system, the best fit is **uClibc-ng**.

- a) musl libc — wrong: musl is the pick for *small-RAM but MMU-equipped* systems, not the no-MMU case.
- b) libm — wrong: `libm` is just the *math* library (sin, cos, sqrt) that accompanies a C library; it is not a full C library and is the obvious distractor.
- d) glibc — wrong: glibc is large and feature-rich, intended for general systems *with* an MMU; it is the worst fit for a tiny no-MMU target.

*Source: Lecture 02 — Toolchain / C library selection (no-MMU → uClibc-ng).*

---

### Q1.7. In JFFS2 filesystem, the ____ block contains only valid nodes. a) Free  b) Erased  c) Clean  d) Ful

**Answer:** c) Clean

**Explanation:**
**JFFS2** is a **log-structured** flash filesystem. Instead of overwriting data in place (which raw flash cannot do — flash must be erased a whole block at a time before rewriting), JFFS2 **appends** every change as a new "node" to the log. When a file is updated, the *old* node becomes **obsolete** and a new node is written elsewhere.

Because of this, JFFS2 classifies each erase block by what it currently holds:
- **Free** — empty, contains **no nodes** at all; ready to be written.
- **Clean** — contains **only valid (still-in-use) nodes**, no obsolete ones. ← this question
- **Dirty** — contains **at least one obsolete node** (wasted space that garbage collection can reclaim).
- **Open block** — the single block currently being appended to (receiving new writes).

When the filesystem runs low on free space, a kernel garbage-collection thread copies the valid nodes out of *dirty* blocks into the open block, then erases the dirty block to make it free again (this also provides simple wear leveling).

So the block that contains **only valid nodes** is the **Clean** block.

- a) Free — wrong: a free block contains *no* nodes, not "only valid" ones.
- b) Erased — wrong: not a JFFS2 block category; an erased block is effectively a free block (no nodes).
- d) Ful (Full) — wrong: not a JFFS2 block state; fullness alone says nothing about whether nodes are valid or obsolete.

*Source: Lecture 09 — JFFS2 blocks (Free / Clean / Dirty + open block).*

---

### Q2.1. Complete the following sentences by a suitable word/phrase [12 marks, 15 min]. The Linux command that removes all files under a directory dir1 is ____ .

**Answer:** `rm -rf dir1` (equivalently `rm -r -f dir1`)

**Explanation:**
`rm` is the Linux command to *remove* (delete) files. By itself, `rm` only deletes plain files and refuses to touch directories. To delete an entire directory and everything inside it, you combine two flags:

- **`-r`** (recursive) — descend into the directory and delete its contents (subdirectories and files), then the directory itself. This is what makes it remove *all files under* `dir1`.
- **`-f`** (force) — do not prompt for confirmation, and do not error out on missing/write-protected files. This makes the deletion non-interactive.

So `rm -rf dir1` wipes `dir1` and its whole subtree in one shot. (Order of flags does not matter; `-rf`, `-fr`, or `-r -f` are all equivalent.)

Caution worth teaching: `rm -rf` is irreversible — there is no recycle bin. Running it on the wrong path (especially `/`) is the classic catastrophic mistake.

*Source: Lecture 01 — Basic shell commands (`rm -r -f`).*

---

### Q2.2. The bootloader part that loads another small chunk of code from preprogrammed locations into SRAM is called ____ while the part that actually loads the kernel into the DRAM is called ___.

**Answer:** Blank 1 = **ROM code** ; Blank 2 = **TPL** (the full/third-stage bootloader, i.e. full U-Boot).

`[KEY vs LECTURE]` The AI-generated solution wrongly put **SPL** in blank 1. Per the lecture, the stage that "loads a small chunk of code from preprogrammed locations into **SRAM**" is the **ROM code** — the SPL is what *gets loaded into SRAM*, it is not the loader described here.

**Explanation:**
Boot on a real SoC is a multi-stage relay because, right after reset, the large external **DRAM** is not yet usable (its controller has not been configured). Only a small on-chip **SRAM** is available. So the boot proceeds in stages, each one setting up enough hardware to load the next:

1. **ROM code** — permanently burned into the SoC's mask ROM; the very first code to run. DRAM is *not* available yet, so it loads a **small chunk of code from preprogrammed locations into on-chip SRAM**. That small chunk is the SPL. → **blank 1**
2. **SPL** (Secondary Program Loader) — runs from SRAM; its main job is to **set up the DRAM/memory controller** so DRAM becomes usable, then load the next, larger stage into DRAM.
3. **TPL / full U-Boot** (Tertiary Program Loader) — now running from DRAM, this is the full-featured bootloader; it **loads the kernel image (plus the flattened device tree) into DRAM** and jumps to it. → **blank 2**

Disambiguator to remember: identify each stage by *who does the loading and into which memory*. "Into SRAM" ⇒ ROM code did it. "Loads the kernel into DRAM" ⇒ the full bootloader (TPL).

*Source: Lecture 03 — Boot phases (ROM code → SPL → TPL).*

---

### Q2.3. In device trees, the ____ property is an empty property that declares a node as a device that receives interrupt signals.

**Answer:** **interrupt-controller**

`[KEY vs LECTURE]` The AI-generated solution wrongly answered **interrupt-parent**. That is incorrect: `interrupt-parent` is *not* empty — it takes a **phandle** value pointing at the controller. The property that is an **empty** flag declaring a node *receives* interrupts (i.e. it *is* an interrupt controller) is **`interrupt-controller`**.

**Explanation:**
A **device tree (DT)** is a data structure that describes the hardware to the kernel (since non-discoverable hardware like memory-mapped peripherals can't be probed automatically). Interrupt wiring is described with a small family of properties:

- **`interrupt-controller`** — an **empty** (boolean/flag) property. Placing it in a node declares "this node is an interrupt controller — it *receives* interrupt signals from other devices." Because it is just a flag, it has no value (written simply as `interrupt-controller;`). ← this question
- **`#interrupt-cells`** — how many 32-bit cells make up one interrupt specifier for this controller.
- **`interrupt-parent`** — a **phandle** (a reference) pointing to the controller that a device's interrupts are routed to. (Has a value → not empty.)
- **`interrupts`** — on a device node, the list of interrupt specifiers describing which line(s) it uses.

So the *empty* property that marks a node as a receiver of interrupts is **`interrupt-controller`**.

*Source: Lecture 03 — Device tree interrupt properties.*

---

### Q2.4. The ___ feature in the flash translation layer maximizes the lifespan of a chip by making each block is erased roughly the same number of times.

**Answer:** **wear leveling**

**Explanation:**
Flash memory cells **wear out**: each erase block can only survive a limited number of erase/program cycles (e.g. tens of thousands) before it fails. If some blocks were erased far more often than others, those "hot" blocks would die quickly while the rest of the chip is still fresh — wasting most of the chip's life.

**Wear leveling** is the **Flash Translation Layer (FTL)** feature that spreads erase/write activity **evenly across all blocks**, so every block is erased roughly the same number of times. This maximizes the overall lifespan of the chip.

It works by decoupling the *logical* address the OS uses from the *physical* block actually written: the FTL remaps writes to less-used blocks and migrates data around. (Other FTL features — distractors — include sub-allocation, garbage collection, bad-block handling, and robustness against power loss.)

*Source: Lecture 09 — Flash Translation Layer (wear leveling).*

---

### Q2.5. In root filesystem, device nodes can be created manually using ______ or automatically on demand using ______.

**Answer:** Manually = **`mknod`** ; Automatically on demand = **udev** (or its embedded equivalents **mdev** / **devtmpfs**).

**Explanation:**
A **device node** is the special file in `/dev` that maps a name to a `(type, major, minor)` triple so user space can reach a driver. There are two ways these nodes come into existence:

- **Manually — `mknod`**: you create each node by hand, e.g. `mknod /dev/null c 1 3` (type `c` = character, major 1, minor 3). This is static: you must know every device in advance and create it explicitly. Fine for a tiny fixed embedded rootfs (which needs at least `console` and `null`).
- **Automatically on demand — udev / mdev / devtmpfs**: the kernel announces hardware as it appears (and disappears), and a manager **creates and removes the matching `/dev` nodes dynamically**. **udev** is the full desktop solution; **mdev** is BusyBox's lightweight version; **devtmpfs** is a kernel-populated `/dev` filesystem. These handle hot-plug (e.g. plugging in a USB stick) without you pre-creating anything.

So: manual = **mknod**, automatic-on-demand = **udev/mdev/devtmpfs**.

*Source: Lecture 05 — Device nodes (manual `mknod` vs automatic udev/mdev/devtmpfs).*

---

### Q2.6. In yocto project, BitBake is the task scheduler that parses the ______ and _____ to create a dependency tree and execute the building process.

**Answer:** **recipes** and **configuration files** (together these make up the *metadata*).

**Explanation:**
The **Yocto Project** builds a complete custom Linux distribution from source. Its build engine is **BitBake** — think of it as a much more powerful `make`. BitBake itself doesn't hard-code how to build anything; instead it reads **metadata** that you and the layers provide:

- **Recipes** (`.bb` files) — each recipe is "a list of settings and tasks (instructions) for building **one package/component**": where to fetch the source, how to configure, compile, and install it (`do_fetch`, `do_unpack`, `do_configure`, `do_compile`, `do_install`, …).
- **Configuration files** (`.conf`) — global/user settings and hardware (machine) configuration: which packages to include, target architecture, build options, etc.

BitBake **parses the recipes and configuration files**, works out all the inter-package dependencies into a **dependency tree**, and then **executes the tasks in order** to download, configure, build, and package everything into the final images.

So the two things BitBake parses are **recipes** and **configuration files**.

*Source: Lecture 07 — Yocto / BitBake (parses recipes + configuration files → dependency tree).*

---

### Q2.7. In Kconfig language, to define a config option that can be compiled as a kernel module, it should be defined as ____ option type.

**Answer:** **tristate**

**Explanation:**
The Linux kernel is configured through **Kconfig**, the language behind `make menuconfig`. Each configurable feature is a *config option* with a **type** that determines what values it can take:

- **bool** — two states: **y** (built **into** the kernel) or **n** (left out). On/off only.
- **tristate** — **three** states: **y** (built-in), **m** (compiled as a loadable **kernel module**, a separate `.ko` file you can insert/remove at runtime), or **n** (not built). ← this question
- Other types: `string`, `int`, `hex` for textual/numeric values.

Because building a feature as a **module** is a distinct third choice (`m`), any option that *can* be compiled as a module **must** be declared **tristate** — `bool` does not offer the `m` option.

*Source: Lecture 04 — Kconfig (tristate = y / m / n).*

---

### Q2.8. The purpose of the ___ and ____ pseudo filesystems is to expose information about processes and kernel driver model to user space, respectively.

**Answer:** **/proc** and **/sys** (sysfs) — respectively.

**Explanation:**
These are **pseudo (virtual) filesystems**: they contain no data on disk. Instead, the kernel generates their "files" on the fly, so reading/writing them is actually a window into kernel internals. The word **"respectively"** in the question fixes the order:

- **/proc** (procfs) — exposes information about **processes** (and assorted kernel/system info). Each running process appears as `/proc/<PID>/` with details about it; files like `/proc/cpuinfo`, `/proc/meminfo` give system info. → matches "information about **processes**".
- **/sys** (sysfs) — exposes the kernel's **driver/device model**: the hierarchy of devices, buses, drivers, and their attributes (e.g. the GPIO interface lives under `/sys/class/gpio`). → matches "kernel **driver model**".

So, respectively: **/proc** (processes) and **/sys** (kernel driver model).

*Source: Lecture 04 — /proc vs /sys (processes vs kernel driver model, "respectively").*

---

### Q3.A.1. [20 marks, 60 min] For the following output of the `ls -l /bin/tracert` command — `-rwxr-xr-x 1 root root 35712 Feb 6 09:15 /bin/tracert` — answer: Is it allowed for the user john (not a root group member) to execute /bin/tracert? If so, what will be the effective UID of the process when john runs it? [5 marks]

**Answer:** **Yes**, john can execute it. The **effective UID of the process is `john`** (the invoking user) — there is no setuid bit, so the program simply runs with the privileges of whoever launched it.

**Explanation:**
A long listing's first field is the file type + **permission triad**:

```
-  rwx  r-x  r-x   1 root root ...
^   |    |    |
|  owner group others
type
```

Read it in three groups of three (`r`=read/4, `w`=write/2, `x`=execute/1):
- **owner** (`root`) → `rwx` — read, write, execute.
- **group** (`root`) → `r-x` — read, execute (no write).
- **others** (everyone else) → `r-x` — read, execute (no write).

Now classify **john**: he is *not* the owner (owner is root) and *not* in the file's group (the group is `root`, and the question says john is not a root-group member). So john falls into the **"others"** category, whose permission is `r-x` → **execute is allowed**. Hence **john can run /bin/tracert**.

**Effective UID:** the *effective UID* is the identity the kernel uses for permission checks while the process runs. Normally a program runs with the **effective UID of the user who started it**. Here there is **no setuid ("s") bit** on the owner triad (it shows plain `rwx`, not `rws`), so no privilege elevation happens. Therefore when john runs it, the process's effective UID = **john**.

*Source: Lecture 05 — File permissions / effective UID.*

---

### Q3.A.2. What will be the changes on the effective UID after the root execute the following command: `chmod 4755 /bin/tracert`?

**Answer:** `chmod 4755` sets the **setuid (SUID) bit**, so the listing becomes **`-rwsr-xr-x`**. From then on, **any** user who executes the file runs it with **effective UID = root (the file owner, UID 0)** instead of their own UID. (So when john runs it now, his process's effective UID becomes **root**, not john.)

**Explanation:**
`chmod 4755` is an octal mode with **four** digits:
- The leading **`4`** is the **special-permissions** digit, and `4` = the **setuid (SUID)** bit.
- The remaining **`755`** are the normal permissions: owner `rwx` (7), group `r-x` (5), others `r-x` (5).

When the SUID bit is set on an **executable owned by root**, the owner's execute slot is displayed as **`s`** instead of `x`:

```
-rwsr-xr-x   ← the 's' shows SUID is on
```

The meaning of SUID: when the program is launched, the kernel sets the process's **effective UID to the file's owner**, *regardless of who actually started it*. So the program runs with the **owner's** privileges. Since the owner here is **root (UID 0)**, every user — john included — who runs `/bin/tracert` now executes it with **effective UID root**.

(This is exactly how tools like `ping` and `traceroute` work: they need root-level access to raw network sockets, so they are installed setuid-root so ordinary users can run them. It is also why SUID-root binaries are a security-sensitive item.)

So the change: effective UID goes from "the invoking user" to **root (the owner)** for anyone who runs it.

*Source: Lecture 05 — SUID bit / effective UID elevation.*

---

### Q3.B.1. [15 marks] A push button is connected to a GPIO pin (gpio 48) in an embedded board and pulled down to ground. Write the shell commands required to make this pin an input and enable the rising-edge interrupt.

**Answer:**
```sh
echo 48 > /sys/class/gpio/export
echo in > /sys/class/gpio/gpio48/direction
echo rising > /sys/class/gpio/gpio48/edge
```

**Explanation:**
Linux exposes GPIO pins to user space through the **sysfs gpiolib interface** under `/sys/class/gpio`. You control a pin by writing plain text to files. The three steps:

1. **Export the pin** — `echo 48 > /sys/class/gpio/export`. This asks the kernel to hand control of GPIO number 48 to user space. The kernel responds by creating a directory `/sys/class/gpio/gpio48/` containing the control files (`direction`, `value`, `edge`, `active_low`).

2. **Set the direction to input** — `echo in > /sys/class/gpio/gpio48/direction`. Writing `in` configures the pin as an input so we can *read* the button state. (Inputs are actually the default after export, but it is good practice — and what the question asks — to set it explicitly. Writing `out` would make it an output.)

3. **Arm the rising-edge interrupt** — `echo rising > /sys/class/gpio/gpio48/edge`. The `edge` file controls *interrupt* (edge-detection) behavior; valid values are `none`, `rising`, `falling`, or `both`. Writing **`rising`** tells the kernel to flag the pin whenever its level transitions from **0 → 1**. This is what lets a `poll()` call in a program block efficiently and wake up exactly when the edge occurs (instead of busy-looping on `value`).

Context for *why rising* here: the button is **pulled down to ground**, so the line idles at logic **0**; pressing the button drives it to **1**. That press is a **rising edge (0 → 1)**, which is why we enable the **rising** edge.

*Source: Lecture 08 — GPIO via sysfs (export / direction / edge).*

---

### Q3.B.2. The following program should wait for the logic level on the button and print a message when the button is pressed, but parts [1]–[5] are missing. Write the lines that contain the missing parts with their line numbers.

Given program:
```c
1: int main(int argc, char *argv[])
2: {
3:   int f;
4:   struct pollfd poll_fds[1];
5:   int ret;
6:   char value[4];
7:   int n;
8:   f = ..[1]..;
9:   if (f == -1) { perror("Can't open gpio48"); return 1; }
10:  poll_fds[0].fd = f;
11:  ..[2]...;
12:  while (1) {
13:    printf("Waiting\n");
14:    ret = ..[3]..(poll_fds, 1, -1);
15:    if (ret > 0) {
16:      n = ..[4]..;
17:      printf("Button pressed: %d bytes, value=%c\n", n, value[0]);
18:      ..[5]..;
19:    }
20:  }
21:  return 0;
22: }
```

**Answer:**
```c
8:   f = open("/sys/class/gpio/gpio48/value", O_RDONLY);   /* [1] */
11:  poll_fds[0].events = POLLPRI | POLLERR;                /* [2] */
14:  ret = poll(poll_fds, 1, -1);                           /* [3] */
16:  n = read(f, &value, sizeof(value));                    /* [4] */
18:  lseek(f, 0, SEEK_SET);                                 /* [5] */
```

**Explanation:**
This is the standard pattern for **waiting on a GPIO edge with `poll()`** instead of busy-polling. Going line by line through the blanks:

- **[1] (line 8) — `f = open("/sys/class/gpio/gpio48/value", O_RDONLY);`**
  We open the pin's **`value`** attribute file (read-only) to get a file descriptor `f`. This is the file whose readable content is the pin's current logic level ('0' or '1'), and — crucially — the file `poll()` will watch. The check on line 9 (`if (f == -1)`) confirms blank [1] must be an `open()` that returns a descriptor or `-1`.

- **[2] (line 11) — `poll_fds[0].events = POLLPRI | POLLERR;`**
  `poll()` watches an array of `struct pollfd`; each entry needs `.fd` (set on line 10) and **`.events`** — the events we want to wait for. For a GPIO sysfs edge, the kernel signals readiness via **`POLLPRI`** ("there is exceptional/priority data" — this is *how a sysfs GPIO edge is reported*, not normal `POLLIN`), and we also watch **`POLLERR`**. So `.events = POLLPRI | POLLERR;`.

- **[3] (line 14) — `poll(poll_fds, 1, -1)`**
  The call itself is **`poll`**. Arguments: the array, `1` (one descriptor), and a timeout of **`-1`** which means **block forever** until an edge arrives. It returns `> 0` when the watched event fires (handled by the `if (ret > 0)` on line 15).

- **[4] (line 16) — `n = read(f, &value, sizeof(value));`**
  After `poll()` wakes us, we **`read`** the current level from the `value` file into the `value[]` buffer; `n` is the number of bytes read, and `value[0]` is the level character printed on line 17.

- **[5] (line 18) — `lseek(f, 0, SEEK_SET);`**
  This **rewinds** the file offset back to the beginning. A sysfs attribute is re-read from the start each time; after a `read()` the offset sits at end-of-file, so without `lseek(f, 0, SEEK_SET)` the *next* `read()` in the loop would return 0 bytes (and `poll()` would not behave correctly). Rewinding makes the next loop iteration re-read the fresh value.

Two concepts the question is really testing: **`POLLPRI`** is the mechanism by which sysfs reports a GPIO edge (so it goes in `.events`), and a **timeout of `-1`** makes `poll()` block indefinitely. (Note: although the prompt says the level should "fall from 1 to 0," Q3.B.1 armed the **rising** edge per its own wording; the C-program blanks above are the canonical lecture answers regardless.)

*Source: Lecture 08 — GPIO button poll() program (slide ~50).*

---

### Q4.A. [11 marks, 35 min] When building BusyBox, it generates a single executable file. Describe briefly how it can offer multiple system tools using this single file.

**Answer:**
BusyBox is **one binary that contains the code of many small tools ("applets")** — `ls`, `cp`, `cat`, `mount`, `sh`, etc. Each applet is implemented as a function named like `ls_main`, `cp_main`, and so on, all linked into the single `busybox` executable. When BusyBox is installed, it creates **symbolic links** in the system directories — `/bin/ls`, `/bin/cat`, `/sbin/mount`, … — that all **point to the one `busybox` binary**. At runtime, when you invoke (say) `ls`, the program starts and inspects **`argv[0]`** — the name it was called by. It looks that name up in its internal **applet table**, finds the matching `*_main` function, and dispatches to it. So the *same* file behaves as whichever tool its invocation name says, letting one executable provide dozens of commands.

**Explanation:**
The motivation is **size**. On a tiny embedded system, shipping the full GNU coreutils/util-linux as dozens of separate programs — each carrying its own copy of common code and linking overhead — wastes precious flash and RAM. BusyBox instead compiles a streamlined version of each tool into a **single shared executable**, so common code is written once and there is only one program image.

The mechanism that makes "one file = many tools" work rests on two facts:

1. **`argv[0]` carries the invocation name.** When the shell runs a command, the first element of the program's argument vector (`argv[0]`) is the name used to launch it. If you call the binary as `ls`, `argv[0]` is `"ls"`; if you call it as `cp`, `argv[0]` is `"cp"`.

2. **Symlinks make every tool name resolve to BusyBox.** `busybox --install` (or the build) populates `/bin`, `/sbin`, `/usr/bin`, … with symlinks such as `ls -> /bin/busybox`. So no matter which command name the user types, the kernel actually executes the *same* `busybox` file — but with `argv[0]` set to the name that was typed.

Putting them together: BusyBox's `main()` reads `argv[0]`, strips the directory path to get the base name, searches its **applet table** for that name, and calls the corresponding `<applet>_main()` function. (If you run `busybox ls -l` directly, it uses `argv[1]` as the applet name instead.) That table-driven dispatch on the invocation name is how a single executable masquerades as many independent system tools.

*Source: Lecture 05 — BusyBox (applets + symlinks + argv[0] dispatch).*

---

### Q4.B. System V init supports the concept of runlevels. List the 8 run levels showing when each level is executed.

**Answer:**
A **runlevel** is a numbered system state that defines *which set of services is running*. System V init defines 8:

| Runlevel | When it is executed / what it means |
|---|---|
| **S** (or **s**) | **Startup / single-user** — the first state entered at boot; runs the system's startup/initialization tasks. |
| **0** | **Halt** — shut the system down / power off. |
| **1** | **Single-user mode** — minimal, one user, for maintenance/recovery (no networking, no daemons). |
| **2** | **Multi-user mode without networking** — multiple users, but network services not started. |
| **3** | **Multi-user mode with networking** — full multi-user text mode with network services (typical server default). |
| **4** | **Unused / user-definable** — not assigned by default; free for custom configuration. |
| **5** | **Multi-user mode with networking + graphical (GUI)** — like 3 plus the display manager / graphical login. |
| **6** | **Reboot** — restart the system. |

**Explanation:**
The idea behind runlevels is to bundle "what should be running" into named modes you can switch between. init reads `/etc/inittab` to know which programs belong to each runlevel, then starts/stops services accordingly.

How to read the list:
- **S** is special: it is the **startup** state entered first, where the one-time initialization (mounting filesystems, running `rcS`) happens. It is also the single-user bring-up state.
- **0 and 6 are the "transition" levels**: entering **0** halts the machine and entering **6** reboots it (you never "stay" in them).
- **1 → 2 → 3 → 5** is a ladder of increasing capability: single-user maintenance (1) → multi-user but no network (2) → multi-user with network (3) → all that plus a graphical desktop (5).
- **4** is intentionally left **unused/user-defined** so administrators can craft their own state.

When the system **switches** runlevels, init runs the **K** ("kill") scripts to stop services not wanted in the new level, then the **S** ("start") scripts to launch the ones that are. (BusyBox init understands the same concepts but ignores the runlevel field in inittab.)

*Source: Lecture 06 — Runlevels (S, 0–6).*

---

### Q4.C. Compare between NAND and NOR Flash in terms of erase block size, reliability and controller complexity.

**Answer:**

| Aspect | **NOR Flash** | **NAND Flash** |
|---|---|---|
| **Erase block size** | Relatively small erase blocks (~**128 KB**). Accessed at fine granularity — **word/byte-addressable**, read word-by-word; can be **memory-mapped and executed in place (XIP)**. | Larger erase blocks, organized as **16 KB–512 KB** blocks made of **pages**; I/O is **page-based (2–4 KB pages**, e.g. 2048 B data + 64 B spare/OOB). Not directly random-addressable like RAM. |
| **Reliability** | **Highly reliable**; high endurance, roughly **100K–1M** erase cycles. Few/no bad blocks. | **Less reliable**: ships with (and develops) **bad blocks**, and suffers **bit flips**, so it **requires error-correcting codes (ECC)** and bad-block management. Lower endurance. |
| **Controller complexity** | **Simple** — essentially "no initialization, just wiring and address mapping"; behaves like memory. | **Complex hardware + software controller** — must handle **bad-block management, ECC, and a flash translation layer**; needs dedicated NAND controller plus bootloader/kernel driver support. |

`[KEY vs LECTURE]` The AI-generated solution printed **reversed/different size ranges** (e.g. "NAND 64 KB–2 MB vs NOR 64–128 KB"). Use the **lecture's** numbers above (NOR ≈128 KB word-addressable; NAND 16 KB–512 KB page-based). The marks really hinge on the *word-vs-page access*, *reliability/ECC*, and *controller-complexity* rows rather than the exact KB figures.

**Explanation:**
Both NOR and NAND are non-volatile flash, but they are built and used very differently — and the exam wants the trade-off along three axes.

- **Why the access/erase difference matters:** **NOR** is wired so the CPU can read any address directly, like ROM/RAM. That makes it **memory-mappable** and able to run code straight out of flash — **eXecute-In-Place (XIP)** — which is why bootloaders often live in NOR. Its erase blocks are modest in size. **NAND** trades random access for **density and low cost**: you read/write whole **pages** and erase whole **blocks**, so it is great for *bulk storage* but cannot be executed in place.

- **Why reliability differs:** NAND's tight, high-density cells are inherently noisier — it is **shipped with some bad blocks**, develops more over time, and individual bits can flip. So NAND **mandates ECC** (e.g. Hamming for SLC, BCH for MLC/TLC) and **bad-block tracking**. NOR is more robust with much higher endurance and effectively no bad-block problem.

- **Why controller complexity differs:** Because NOR "just works" like addressable memory, its controller is **trivial** (wiring + address decode). NAND needs a **substantial controller** — in hardware and in the OS — to manage bad blocks, compute/verify ECC on every page, and provide a flash translation layer (wear leveling, mapping). That is the price of NAND's density.

In short: **NOR = simple, reliable, memory-mappable (XIP), smaller capacity** → good for *boot code*; **NAND = dense, cheaper, page-based, needs ECC + complex controller** → good for *mass data storage*.

*Source: Lecture 09 — NAND vs NOR flash (erase block size, reliability, controller complexity).*

---

<a id="exam-sample-1"></a>

## Sample_1 — Embedded Linux Final Exam Sample (FCIS, Spring 2023)

Total questions: 21

*Exam header: FCIS – Spring 2023; Embedded Linux Final Exam Sample. Four questions; Q1 MCQ [8 marks], Q2 fill-in [5 marks], Q3 [12 marks], Q4 [15 marks]. No official answer key exists — answers below are verified against the CSY441 lecture slides (Lectures 02–09).*

---

### Q1.1. The ___ is the part of the Linux system that is responsible for loading kernel into memory after system reset and firmware initialization. a) Init program  b) Bootloader  c) Kernel loader  d) Board firmware

**Answer:** b) Bootloader

**Explanation:**
When an embedded board powers on it cannot simply "run Linux." There is a fixed chain of hand-offs, each stage waking up a little more of the system before passing control to the next:

1. **Board firmware / ROM code** — tiny code baked permanently into the SoC. Runs first, brings up the bare minimum (a clock, a small on-chip SRAM), and fetches the next stage. It has no idea what Linux is.
2. **Bootloader** (e.g. U-Boot) — the stage that *knows about the kernel*. Its job is to (a) put the hardware in a known state and initialise DRAM, (b) place the kernel's parameters (device tree + kernel command line) into memory, then (c) **load the kernel image into RAM and jump to it**.
3. **Kernel** — once running, it mounts the root filesystem and starts the init program.
4. **Init program** (PID 1) — the first user-space process; launches the system daemons.

So the component whose specific responsibility is *loading the kernel into memory after reset and firmware initialisation* is the **bootloader**.

- a) Init program — wrong: init is started *by* the kernel, long after the kernel is already in memory; it cannot load the thing that starts it.
- c) Kernel loader — wrong: not a real component in this chain; the loading is done by the bootloader, and "kernel loader" is a made-up distractor.
- d) Board firmware — wrong: firmware runs first and only initialises the SoC and loads the *next bootstrap stage* (the bootloader); it is not what loads the Linux kernel.

*Source: Lecture 03 — Bootloader (role: init HW to a known state, place kernel parameters, load kernel & jump).*

---

### Q1.2. In ______ , both user services and kernel services are kept in the same address space. a) microkernel  b) hybird kernel  c) Monolithic kernel  d) Device drivers

**Answer:** c) Monolithic kernel

**Explanation:**
Operating-system kernels differ in *where* they run their services and *how* those services are isolated from each other:

- **Monolithic kernel** (Linux's design): the entire kernel — scheduler, memory manager, filesystems, device drivers, network stack — runs together in **one single (kernel) address space**, in privileged mode. "User services and kernel services are kept in the same address space" describes exactly this: there is no memory boundary between the kernel subsystems, so they call each other as fast, direct function calls. The trade-off is that a bug in any driver can corrupt the whole kernel.
- **Microkernel**: only the absolute minimum (scheduling, basic IPC, low-level memory) runs in kernel space; drivers, filesystems, and other services run as *separate user-space processes*, each in its **own** address space, talking via message passing. More robust and modular, but slower due to the messaging overhead.

The phrase "same address space" is the signature of the **monolithic** model.

- a) microkernel — wrong: it deliberately pushes services *out* into separate user-space address spaces; that is the opposite of sharing one space.
- b) hybird [hybrid] kernel — wrong: a hybrid mixes the two (some services in kernel space, some out); it is not defined by everything sharing one address space.
- d) Device drivers — wrong: device drivers are a *kind of service*, not a kernel architecture; the question asks for the kernel type.

*Source: Lecture 04 — Kernel architectures (monolithic = user + kernel services share one address space; vs microkernel).*

---

### Q1.3. In the filesystem hierarchy standard, the _____ directory stores the devices nodes defined for the hardware attached to the system. a) /sys  b) /bin  c) /driver  d) /dev

**Answer:** d) /dev

**Explanation:**
The **Filesystem Hierarchy Standard (FHS)** is the agreed convention for what each top-level directory contains, so software always knows where to look. The relevant ones here:

- **/dev** — holds **device nodes**. A device node is a *special file* that represents a piece of hardware (or a virtual device). Opening/reading/writing that file is how user space talks to the corresponding driver. Each node carries a type (`c` = character, `b` = block) plus a **major** number (which driver) and **minor** number (which instance/partition). Examples: `/dev/ttyS0`, `/dev/null`, `/dev/mmcblk0`.
- **/sys** — exposes the kernel's *driver model* (the sysfs virtual filesystem): a tree of objects/attributes describing devices, buses and classes. It is *about* devices but does not hold the device nodes themselves.
- **/bin** — essential user programs (ls, cp, sh).
- **/driver** — not part of the FHS at all (a distractor).

So device nodes live in **/dev**.

- a) /sys — wrong: sysfs shows the kernel's *view/model* of devices (attributes, relationships), not the openable device-node special files.
- b) /bin — wrong: that is for essential executable programs.
- c) /driver — wrong: no such standard directory exists.

*Source: Lecture 05 — Root filesystem / FHS (device nodes live in /dev).*

---

### Q1.4. The system V init program begins reading the ______ file which defines rules to start programs at boot up and stop them at shutdown. a) /etc/init.d  b) /sbin/boot  c) /sbin/init  d) /etc/inittab

**Answer:** d) /etc/inittab

**Explanation:**
**System V init** (and BusyBox's compatible init) is the first user-space process, PID 1. When it starts it needs a "script of rules" telling it *what to launch at boot and what to do at shutdown*. That configuration file is **/etc/inittab**.

Each line of inittab has the form `<id>:<runlevels>:<action>:<process>` — for example `::sysinit:/etc/init.d/rcS` runs the boot-time shell script, `::respawn:/sbin/getty 115200 ttyS0` keeps a login prompt alive, and shutdown entries say what to stop. (BusyBox init ignores the runlevels field but uses the same format.)

Note the careful wording: init *reads* **/etc/inittab**; the boot-time *shell commands* themselves are usually placed in **/etc/init.d/rcS**, which inittab invokes via its `sysinit` entry. The question asks which file init *begins reading* → inittab.

- a) /etc/init.d — wrong: that is a *directory* of service scripts (e.g. rcS); init does not "begin by reading" a directory, and the rules live in inittab.
- b) /sbin/boot — wrong: not a real init file; a distractor.
- c) /sbin/init — wrong: that is the init *program (binary)* itself, not the configuration file it reads.

*Source: Lecture 06 — Init & inittab (System V init begins by reading /etc/inittab; rcS holds boot-time commands).*

---

### Q1.5. In block devices, the minor number identifies the ______ of the device. a) partition  b) interface  c) device driver  d) none of the above

**Answer:** a) partition

**Explanation:**
Every device node carries two numbers:

- The **major number** says *which driver* (device class) handles the node.
- The **minor number** distinguishes *which specific thing* that one driver is managing.

For **block devices** (mass storage: SD cards, eMMC, disks), the driver typically manages a whole disk that is then split into **partitions**. So the convention is: the major selects the storage driver (e.g. `mmcblk` = major 179), and the **minor identifies the partition** — minor 0 is the whole device `mmcblk0`, minor 1 is `mmcblk0p1`, minor 2 is `mmcblk0p2`, and so on.

(Contrast with *character* devices, where the minor instead identifies the *instance/interface* — e.g. which serial port. The exam is specifically asking about block devices, so the answer is partition.)

- b) interface — wrong: "interface/instance" is the meaning of the minor for *character* devices, not block.
- c) device driver — wrong: that is what the *major* number selects, not the minor.
- d) none of the above — wrong: partition is correct, so this is not it.

*Source: Lecture 08 — Device drivers / major & minor numbers (block minor = partition).*

---

### Q1.6. When writing a custom character driver, the supported functions are filled in a file_operations struct and passed to the ______ function. a) device_create  b) register_chrdev  c) class_create  d) module_init

**Answer:** b) register_chrdev

**Explanation:**
A **character driver** exposes a device as a byte/stream you can `open`, `read`, `write`, and `close`. The kernel needs to know *which of the driver's functions* to call for each of those operations. The driver advertises them by filling a **`struct file_operations`** table:

```c
static struct file_operations fops = {
    .owner   = THIS_MODULE,
    .open    = my_open,
    .release = my_release,
    .read    = my_read,
    .write   = my_write,
};
```

It then hands this table to the kernel by calling **`register_chrdev(major, name, &fops)`**, which registers the driver under a major number and tells the kernel: "for this device class, route open/read/write/close to these functions." (Afterwards the driver typically calls `class_create` + `device_create` so a `/dev` node appears automatically, and the whole thing is wired up by `module_init`/`module_exit`.)

- a) device_create — wrong: that creates the `/dev` node *after* registration; it does not receive the file_operations table.
- c) class_create — wrong: that creates a device *class* in sysfs; again it takes no file_operations.
- d) module_init — wrong: that just marks which function runs when the module loads; the registration call (register_chrdev) is made *inside* it.

*Source: Lecture 08 — Writing a character driver (fill file_operations → register_chrdev).*

---

### Q1.7. The _____ is a build system oriented towards building firmware for wireless routers. a) OpenEmbedded  b) Buildroot  c) OpenWrt  d) Yocto project

**Answer:** c) OpenWrt

**Explanation:**
There are several embedded-Linux **build systems** — tools that compile a cross-toolchain, kernel, bootloader and root filesystem into a flashable image. They differ in their target audience:

- **OpenWrt** — specialised for **wireless-router / networking firmware**. It ships router-centric packages (Wi-Fi stacks, firewall, network config via UCI) and is the de-facto replacement firmware for consumer routers. This is the one "oriented towards building firmware for wireless routers."
- **Buildroot** — a simple, general-purpose build system (Makefile/Kconfig based) for any small embedded device.
- **Yocto Project / OpenEmbedded** — a powerful, layered, recipe-based system (BitBake) for building custom Linux distributions; general-purpose, not router-specific.

- a) OpenEmbedded — wrong: it is the general metadata/recipe framework underlying Yocto, not a router-focused system.
- b) Buildroot — wrong: general-purpose for small embedded systems, with no special router orientation.
- d) Yocto project — wrong: a general distribution-building framework; not aimed specifically at router firmware.

*Source: Lecture 07 — Build systems (OpenWrt = build system for wireless-router firmware).*

---

### Q1.8. In MTD, we have to use ____ function to read nand flash and skip bad blocks. a) nanddump  b) cp  c) nandread  d) flashread

**Answer:** a) nanddump

**Explanation:**
**NAND flash is unreliable by nature:** some blocks are *bad* (unusable) from the factory or wear out over time, and the chip records them with a bad-block marker. Any tool that reads or writes NAND must **skip those bad blocks**, or it will read garbage / corrupt data.

The MTD (Memory Technology Devices) user-space tools are built exactly for this:
- **`nanddump`** — reads the contents of a NAND device to a file, **skipping bad blocks** (and optionally dumping the OOB/ECC area).
- **`nandwrite`** — the write counterpart, programs a file into NAND, also skipping bad blocks.

A plain **`cp /dev/mtdX file`** is the classic trap: it treats the flash like a normal file and **fails at the first bad block**, because it has no notion of skipping them.

- b) cp — wrong: it does not understand bad blocks and breaks at the first one.
- c) nandread — wrong: not a real MTD tool (the reader is `nanddump`).
- d) flashread — wrong: also not a real MTD tool; a distractor.

*Source: Lecture 09 — Flash & MTD tools (nanddump reads NAND skipping bad blocks; nandwrite writes).*

---

### Q2.1. The _______ toolchain runs on the same type of system as the programs it generates, while the _____ toolchain produces programs for a target system different from the host.

**Answer:** **native** (first blank) and **cross(-compiled)** (second blank).

**Explanation:**
A **toolchain** is the set of tools that turn source code into runnable binaries (assembler/linker `binutils`, the C compiler `gcc`, the C library, and `gdb`). Toolchains are classified by *what kind of machine they run on* versus *what kind of machine the output binaries run on*:

- **Native toolchain** — runs on the **same** type of system as the programs it produces. Example: a compiler running on an x86 PC producing x86 programs (this is the everyday desktop case).
- **Cross(-compiled) toolchain** — runs on one type of machine (the **host**, e.g. a powerful x86 build PC) but produces programs for a **different** target architecture (e.g. an ARM board). This is the normal situation in embedded Linux: the target is too small/slow to compile on, so we cross-compile on the host. Such tools are named with a prefix like `arm-linux-gnueabihf-gcc`.

So: same-type → **native**; different-type → **cross-compiled**.

*Source: Lecture 02 — Toolchain (native = same system type as its output; cross-compiled = different target type than host).*

---

### Q2.2. In device trees, the _______ property lists tuples of the base address and length of a device node.

**Answer:** **reg**

**Explanation:**
A **device tree** is a data structure that describes the hardware to the kernel (since, unlike a PC, an embedded SoC cannot be auto-discovered). Each hardware block is a *node*, and its properties tell the kernel where and how to reach it.

The **`reg`** property gives a node's **register region(s)**: it is a list of `<base-address length>` tuples — i.e. *where* the device's registers start in the address map and *how many* bytes they span. Example:

```dts
serial@48020000 {
    compatible = "arm,pl011";
    reg = <0x48020000 0x1000>;   /* base 0x48020000, length 4 KB */
};
```

How many cells each address and each length take is set by the parent's `#address-cells` and `#size-cells`. The phrase "tuples of the base address and length" is the textbook definition of **`reg`**.

*Source: Lecture 03 — Device tree (the reg property lists tuples of base address + length).*

---

### Q2.3. In Linux Kernel build system, _______ files define each config option and its attributes, while _______ files store each the selected values of the config symbols.

**Answer:** **Kconfig** files (first blank) and **.config** file (second blank).

**Explanation:**
The Linux kernel is *configurable* — thousands of features can be turned on, off, or built as modules. The build system splits this into two clearly separated roles:

- **Kconfig files** — scattered through the source tree, they **declare** each configuration option: its symbol name, its type (`bool`, `tristate`, `string`, `int`), its prompt text, help, dependencies (`depends on`), and default. In other words, Kconfig defines *which options exist and their attributes*. Tools like `make menuconfig` read these to draw the menus.
- **.config** file — a single generated file at the top of the build tree that **stores the chosen values** of every symbol (e.g. `CONFIG_NET=y`, `CONFIG_USB=m`). Editing the menu writes here; the build then reads `.config` to decide what to compile.

Mnemonic: Kconfig = the *questions/menu definitions*; .config = the *answers*.

*Source: Lecture 04 — Kernel build system / Kconfig (Kconfig defines options & attributes; .config stores selected values).*

---

### Q2.4. The three possibilities to transfer the root filesystem are ________, _______ and ______.

**Answer:** (1) a **RAM disk** loaded into memory (initramfs / ramdisk); (2) a **disk image on block storage** (e.g. ext4 on an SD card, or JFFS2/UBIFS on raw flash); and (3) a **network filesystem (NFS)**.

**Explanation:**
The **root filesystem (rootfs)** must somehow get onto the target so the kernel can mount it as `/`. There are three standard delivery methods, each suited to a different stage or device:

1. **RAM disk (initramfs / ramdisk):** the rootfs is packed into an archive that the bootloader loads into RAM alongside the kernel; the kernel mounts that in-memory image. Fast and self-contained, but volatile (changes are lost on reboot) and limited by RAM size. Great for early boot or recovery.
2. **Disk image on block storage:** the rootfs lives on a real storage medium — e.g. an **ext4** image on an **SD card / eMMC partition**, or a flash-friendly filesystem like **JFFS2 / UBIFS** on raw **NAND/NOR flash**. This is the usual persistent production setup; the kernel mounts it via the `root=` option.
3. **Network filesystem (NFS):** the target mounts its `/` from a server over the network. The board needs no local storage for the rootfs — ideal during **development**, because you edit files on the host and they appear instantly on the target.

*Source: Lecture 05 — Root filesystem transfer methods (RAM disk / disk image on block storage / network filesystem).*

---

### Q2.5. The ______ feature in the flash translation layer maximizes the lifespan of a chip by making each block is erased roughly the same number of times.

**Answer:** **wear leveling**

**Explanation:**
Flash memory can only endure a **limited number of erase/program cycles per block** (e.g. ~10K–100K for NAND). If the system kept rewriting the *same* few blocks (say, a frequently updated log or config), those blocks would wear out and fail while the rest of the chip is still fresh — killing the device prematurely.

**Wear leveling** is the flash-translation-layer (FTL) feature that prevents this. It spreads writes/erases **evenly across all blocks**, so that every block is erased *roughly the same number of times*. It does this by remapping logical addresses to different physical blocks over time, deliberately steering new writes toward the least-worn blocks. The result is the **maximum overall lifespan** of the chip.

(Other FTL features — distractors — include garbage collection, bad-block handling, sub-allocation, and robustness; but the one defined by "each block erased roughly the same number of times" is wear leveling.)

*Source: Lecture 09 — Flash translation layer (wear leveling = even erase distribution to maximise chip lifespan).*

---

### Q3.A.1. A button is connected to P9.15 (gpio 48) on a BeagleBone Black, pulled down to ground. Write the shell commands required to make this pin an input and enable the falling edge interrupt. [2 marks]

**Answer:**
```sh
echo 48 > /sys/class/gpio/export
echo in > /sys/class/gpio/gpio48/direction
echo falling > /sys/class/gpio/gpio48/edge
```

**Explanation:**
Linux exposes GPIO pins to user space through the **sysfs gpiolib** interface under `/sys/class/gpio`. Configuring a pin is done purely by writing text into these special files:

1. **`echo 48 > /sys/class/gpio/export`** — "export" GPIO number 48 from kernel control to user space. This makes a new directory `/sys/class/gpio/gpio48/` appear, containing the control files `direction`, `value`, `edge`, and `active_low`.
2. **`echo in > .../gpio48/direction`** — set the pin as an **input** (so we can read the button). (`in` is also the power-on default, but we set it explicitly.)
3. **`echo falling > .../gpio48/edge`** — arm the **interrupt** on the **falling edge**. The `edge` file accepts `none | rising | falling | both`. Because the button is *pulled down to ground*, pressing it drives the line from logic **1 → 0**, which is a **falling** edge — exactly the transition we want to detect. Setting this lets a program later block on `poll()` until the press happens.

*Source: Lecture 08 — GPIO via sysfs (export → direction → edge to arm the interrupt).*

---

### Q3.A.2. The following program should wait for the button level to fall from 1 to 0 and print a message when pressed, but it contains mistakes. Mention the lines of errors and correct them. [5 marks]

```c
1:  int main(int argc, char *argv[])
2:  {
3:    int f;
4:    struct pollfd poll_fds[1];
5:    int ret;
6:    char value[4];
7:    int n;
8:    f = open("/proc/gpio/gpio48/value", O_RDWR);
9:    if (f == -1) { perror("Can't open gpio48"); return 1; }
10:   poll_fds[0].fd = f;
11:   poll_fds[0].polls = POLLPRI | POLLERR;
12:   while (1) {
13:     printf("Waiting\n");
14:     ret = read(poll_fds, 1, -1);
15:     if (ret > 0) {
16:       n = write(f, &value, sizeof(value));
17:       printf("Button pressed: %d bytes, value=%c\n", n, value[0]);
18:       lseek(f, 0, SEEK_SET);
19:     }
20:   }
21:   return 0;
22: }
```

**Answer:** There are **four** error lines (8, 11, 14, 16). Line 18's `lseek` is **correct** — do not change it. The corrected lines are:

- **Line 8** → wrong path and mode. GPIO is exposed under **sysfs**, not procfs:
  ```c
  f = open("/sys/class/gpio/gpio48/value", O_RDONLY);
  ```
- **Line 11** → the `struct pollfd` field is **`.events`**, not `.polls`:
  ```c
  poll_fds[0].events = POLLPRI | POLLERR;
  ```
- **Line 14** → you wait on the edge with **`poll()`**, not `read()`:
  ```c
  ret = poll(poll_fds, 1, -1);
  ```
- **Line 16** → to get the pin's level you **read** it, not write it:
  ```c
  n = read(f, &value, sizeof(value));
  ```

**Explanation:**
This is the canonical "wait for a GPIO edge with `poll()`" program. The logic is: open the pin's `value` file, register it with poll, then loop — block in `poll()` until the kernel signals an edge, read the new value, rewind, repeat. Walking through the bugs teaches the mechanism:

- **Path/mode (line 8):** the gpiolib value file lives at `/sys/class/gpio/gpio48/value` — under **/sys** (sysfs is the kernel's device-model filesystem), never under `/proc`. We only *read* the level, so `O_RDONLY` is the canonical mode.
- **`.events` (line 11):** `struct pollfd` has exactly three fields — `fd`, **`events`** (what to wait for), and `revents` (what happened). There is no `.polls` member, so the code would not even compile. We request **`POLLPRI`** because a GPIO edge on a sysfs attribute is delivered as *priority/exceptional* data (`POLLPRI`), plus `POLLERR`.
- **`poll()` (line 14):** the call that *blocks until an event* is **`poll(fds, nfds, timeout)`**. Here `poll(poll_fds, 1, -1)` watches 1 descriptor with a **timeout of -1 = block forever** until the falling edge fires. Using `read()` there is simply the wrong system call (and wrong arguments).
- **`read()` (line 16):** once poll returns, the actual pin level is obtained by **reading** the value file into `value`. Writing would try to *drive* the pin, which is meaningless for an input and is the opposite of what we want.
- **`lseek` (line 18) is correct:** sysfs attribute files do not auto-rewind. After each read you must **`lseek(f, 0, SEEK_SET)`** to return to the start, or the next read returns 0 bytes. Keep this line.

*Source: Lecture 08 — GPIO button poll() program (sysfs value path, pollfd.events, poll(), read the value, lseek to rewind).*

---

### Q3.B. Fill the missing parts [A]–[E] of the device tree so it matches the kernel command line `mtdparts=<id>:512k(SPL)ro, 780k(U-Boot)ro, 128k(UBootEnv), 4m(Kernel), -(Filesystem)`. [5 marks]

```dts
nand@0,0 {
    ... [A] ...
    #size-cells = ... [B] ...;
    partition@0 {
        label = "SPL";
        reg = ... [C] ...;
    };
    ... [D] ... {
        label = "U-Boot";
        reg = <0x80000 0xc3000>;
    };
    partition@143000 {
        label = "U-BootEnv";
        reg = <0x143000 0x20000>;
    };
    partition@163000 {
        label = "Kernel";
        reg = <0x163000 ... [E] ... >;
    };
    partition@563000 {
        label = "Filesystem";
        reg = <0x563000 0x7a9d000>;
    };
};
```

**Answer:**
- **[A]** `#address-cells = <1>;`
- **[B]** `<1>`  (so the line reads `#size-cells = <1>;`)
- **[C]** `<0 0x80000>`  (SPL: offset 0, size 512k)
- **[D]** `partition@80000`  (U-Boot node name; its unit-address = its offset 0x80000)
- **[E]** `0x400000`  (Kernel size = 4m)

Completed tree:
```dts
nand@0,0 {
    #address-cells = <1>;
    #size-cells = <1>;
    partition@0 {
        label = "SPL";
        reg = <0 0x80000>;
    };
    partition@80000 {
        label = "U-Boot";
        reg = <0x80000 0xc3000>;
    };
    partition@143000 {
        label = "U-BootEnv";
        reg = <0x143000 0x20000>;
    };
    partition@163000 {
        label = "Kernel";
        reg = <0x163000 0x400000>;
    };
    partition@563000 {
        label = "Filesystem";
        reg = <0x563000 0x7a9d000>;
    };
};
```

**Explanation:**
An **MTD partition layout** can be given either as a `mtdparts=` string on the kernel command line or, equivalently, as `partition@<offset>` child nodes in the device tree. Each partition node uses **`reg = <offset size>`** — the byte offset where it starts and its byte length. The job here is to translate sizes to hex and accumulate offsets.

**Unit conversions** (1k = 0x400, 1m = 0x100000):
- 512k = 512 × 0x400 = **0x80000**
- 780k = 780 × 0x400 = **0xC3000**
- 128k = 128 × 0x400 = **0x20000**
- 4m = 4 × 0x100000 = **0x400000**

**Offsets accumulate** (each partition starts where the previous one ended):
- SPL: starts at **0**, size 0x80000 → ends at 0x80000.
- U-Boot: starts at **0x80000**, size 0xC3000 → ends at 0x143000.
- U-BootEnv: starts at **0x143000**, size 0x20000 → ends at 0x163000.
- Kernel: starts at **0x163000**, size 0x400000 → ends at 0x563000.
- Filesystem: starts at **0x563000**, size 0x7A9D000 (the `-` in mtdparts means "use all the remaining space").

Now the blanks:
- **[A] `#address-cells = <1>;`** and **[B] `<1>`** — these declare that each partition's address (offset) and each size are written with **one 32-bit cell**. They must both be 1 so that `reg = <offset size>` is interpreted as a single offset followed by a single length.
- **[C] `<0 0x80000>`** — SPL: offset 0, size 512k (= 0x80000).
- **[D] `partition@80000`** — by device-tree convention a node's name is `name@unit-address`, where the unit-address equals its `reg` offset. The U-Boot partition starts at **0x80000**, so the node is `partition@80000` (matching its `reg = <0x80000 0xc3000>`).
- **[E] `0x400000`** — the Kernel partition's size is 4m = **0x400000** (it starts at 0x163000 and runs to 0x563000, which is exactly where the Filesystem partition begins — confirming the arithmetic).

*Source: Lecture 09 — MTD partitions (mtdparts ↔ device-tree partition nodes; reg = <offset size>; offsets accumulate).*

---

### Q4.A. Describe briefly the following Yocto project terminology: BitBake, recipes and OE-Core.

**Answer:**
- **BitBake** — the Yocto **build engine / task scheduler** (conceptually like `make`). It parses the *metadata* (recipes + configuration files), works out the dependency tree between components, and then executes each component's tasks (fetch the source, configure, compile, install, package) to ultimately produce the packages and the final images.
- **Recipes** (`.bb` files) — *"a list of settings and tasks (instructions) for building packages."* A recipe describes how to build **one** software component: where to fetch its source, which version, patches, configure/compile/install steps. The standard tasks are `do_fetch`, `do_unpack`, `do_patch`, `do_configure`, `do_compile`, `do_install`, `do_package`. Recipes are stored inside layers.
- **OE-Core (OpenEmbedded-Core)** — the set of **validated base layers** (core recipes, classes and metadata) that are **shared between all OpenEmbedded-based systems** (including Yocto/Poky). It is the common foundation created when OpenEmbedded and Poky merged their core in 2010; everyone builds their own layers *on top of* OE-Core.

**Explanation:**
The Yocto Project builds a *custom Linux distribution* from source, and its vocabulary maps cleanly onto three layers of the problem:

- The **engine that does the work** is **BitBake** — it does not itself know how to build anything; it reads instructions and orchestrates them, scheduling tasks in dependency order much like `make` does for a single project, but across the whole distribution.
- The **per-package instructions** are the **recipes**. Think of a recipe as the "build manual" for one piece of software: it answers *where do I get it, how do I configure it, how do I compile and install it*. Because every recipe exposes the same task names, BitBake can drive any package uniformly.
- The **shared, trusted starting point** is **OE-Core**. Rather than every project reinventing core recipes (libc, busybox, base classes), Yocto reuses OE-Core's validated base layers and stacks project-specific layers above them. (Closely related terms: **metadata** = recipes + config files; **layers** = stackable, hierarchical sets of metadata that can override earlier ones; **Poky** = Yocto's reference distribution that bundles BitBake + OE-Core + a sample distro.)

*Source: Lecture 07 — Yocto glossary (BitBake = build engine/scheduler; recipes = settings+tasks per package; OE-Core = shared validated base layers).*

---

### Q4.B. Compare between NAND and NOR Flash in terms of erase block size, reliability and controller complexity. Describe the role of the out-of-band area in the NAND flash.

**Answer:**

| Axis | **NOR flash** | **NAND flash** |
|---|---|---|
| **Erase block size** | Larger access granularity per *word* but conceptually simple; cells are individually addressable, so it is **readable word-by-word and memory-mappable (eXecute-In-Place / XIP)**; erase blocks ~128 KB. | Organised in **pages** (typically 2–4 KB; a page = data area + spare OOB) grouped into **blocks (~16–512 KB)**; read/write happen a page at a time, erase a whole block at a time — **not** byte-addressable for execution. |
| **Reliability** | **More reliable** — few/no bad blocks, long endurance (≈100K–1M erase cycles); data integrity is essentially guaranteed. | **Less reliable** — ships with (and develops) **bad blocks**, and individual bits flip, so it **requires ECC** (error-correcting codes) and bad-block management; lower endurance (≈10K–100K cycles). |
| **Controller complexity** | **Simple** — *"no initialisation, only wiring and address mapping"*; behaves like memory. | **Complex** — needs a dedicated hardware + software controller to handle **paging, ECC, and bad-block skipping**; the NAND controller sits in the SoC and is driven by bootloader/kernel drivers. |

**Out-of-band (OOB) area:** each NAND **page** has extra **spare bytes** alongside the main data area (e.g. 64 B of OOB for a 2 KB page). The OOB stores the *housekeeping information the chip and filesystem need but that isn't user data*: principally the **ECC** bytes (so bit-flips in the page can be detected/corrected), the manufacturer's **bad-block marker** (a flag identifying unusable blocks so tools skip them), and **filesystem metadata** (e.g. JFFS2 clean markers).

**Explanation:**
NOR and NAND are two flash technologies optimised for opposite goals. **NOR** is built for *random read access and reliability*: because every cell is individually addressable, you can run code straight out of it (XIP), and its controller is trivial — just wires and an address map. The price is cost and small capacity, so it is used for boot code. **NAND** is built for *density and capacity*: cells are packed and accessed in pages, giving cheap gigabytes, but at the cost of *reliability* — bad blocks and bit-flips are normal, so it cannot be trusted without **ECC** and **bad-block handling**, which is exactly why it needs a far more **complex controller** and driver stack (and why tools like `nanddump`/`nandwrite` exist to skip bad blocks).

The **OOB area** is the mechanism that makes unreliable NAND usable: by attaching per-page spare bytes for ECC, the system can correct the inevitable bit errors; by reserving a bad-block marker, blocks that fail are permanently avoided; and by leaving room for FS metadata, log-structured filesystems can record their bookkeeping inline with each page.

*Source: Lecture 09 — NAND vs NOR flash and the role of the OOB (spare bytes holding ECC, bad-block marker, FS metadata).*

---

### Q4.D. The JFFS2 filesystem uses MTD as interface to flash memory. Describe how it minimizes memory corruption due to an unexpected system reset and the role of its garbage collector.

**Answer:**
**Robustness against unexpected resets:** JFFS2 is a **log-structured** filesystem. Every change (file data, metadata) is written as a **node appended sequentially** to the *single currently-open* erase block — JFFS2 never overwrites data in place; updates obsolete old nodes and append new ones. Therefore an unexpected power-loss or reset can damage *at most the last write* to that one open block; everything already committed earlier in the log remains intact and consistent. This bounds the worst-case corruption to a single partial node rather than the whole filesystem.

**Garbage collector (GC):** because nothing is overwritten, blocks gradually fill with **obsolete (dirty)** nodes and free space runs low. The GC is a **kernel thread** that, once the number of free blocks drops below a threshold, **copies the still-valid nodes out of a dirty block into the open block**, and then **erases the reclaimed block** so it becomes free again. As a side-effect, by continually moving data around and erasing different blocks, the GC also spreads erases fairly evenly — i.e. it provides **simple wear leveling**.

**Explanation:**
The core insight is *append-only, log-structured* design. In a traditional filesystem you modify a block by erasing/rewriting it in place — if power fails mid-update, that block (which may hold critical metadata) is left half-written and corrupt. JFFS2 sidesteps this entirely: writes only ever *append* a new node to the end of the open block, and an interrupted append can only spoil that final node. On the next mount JFFS2 replays/scans the log and simply ignores the incomplete tail, so the filesystem comes up consistent.

The cost of never overwriting is that obsolete nodes accumulate and consume space, so a reclamation mechanism is mandatory — that is the **garbage collector**. It behaves like a copying compactor: pick a dirty block, relocate its few remaining valid nodes into the open block, erase the now-fully-obsolete block, and return it to the free pool. Running this across many blocks over time naturally distributes erase cycles, doubling as basic wear leveling on top of its primary space-reclamation job.

*Source: Lecture 09 — JFFS2 (log-structured robustness against power loss; GC kernel thread reclaims dirty blocks and provides wear leveling).*

---

### Q4.E. Describe briefly the role and format of the device table file in creating a filesystem disk image.

**Answer:**
**Role:** A **device table** is a plain **text file** read by the image-creation tool (e.g. `genext2fs`) that lets it create **root-owned files and device nodes inside the target image *without* requiring root privileges** on the build host. This is needed because the staging rootfs built by an ordinary user cannot itself contain proper `/dev` device nodes or root-owned files; the device table supplies that information so the tool stamps the correct ownership, permissions and special nodes into the image as it is assembled.

**Format:** one entry per line, **10 fields**:

```
<name> <type> <mode> <uid> <gid> <major> <minor> <start> <inc> <count>
```

where `<type>` is one of **`f` (regular file), `d` (directory), `c` (character device), `b` (block device), `p` (named pipe/FIFO)**; `<mode>` is the permission bits; `<uid>`/`<gid>` are owner/group; `<major>`/`<minor>` are the device numbers (for `c`/`b`); and `<start> <inc> <count>` describe how to generate a *range* of numbered nodes in one line (start value, increment, how many). Example lines:

```
/dev/null    c 666 0 0 1 3 0 0 -
/dev/console c 600 0 0 5 1 0 0 -
```

**Explanation:**
When you build a root filesystem as a normal (non-root) user, two things are impossible directly: you cannot `mknod` real device nodes, and you cannot make files owned by `root` — both need superuser rights. The **device table** solves this declaratively: instead of *being* root, you *describe* the privileged entries you want, and the unprivileged image tool encodes exactly those entries (with the right type, mode, ownership and device numbers) into the output image. The optional `start/inc/count` fields are a convenience for creating many similar nodes (e.g. a series of ttys) from a single line. The result is a correct, bootable image — with a proper `/dev` and root-owned files — produced entirely without root on the host.

*Source: Lecture 06 — Device table file (text file enabling root-owned files & device nodes without root; 10 fields per line, types f/d/c/b/p).*

---

### Q4.F. Differentiate among the following actions of the System V init program: sysinit, respawn, askfirst, once and wait.

**Answer:**
- **sysinit** — run **before all other actions**. Used for the very first boot-time setup (mount filesystems, run `/etc/init.d/rcS`) and is processed ahead of everything else.
- **respawn** — **restart the process whenever it terminates**. If the program exits or is killed, init immediately starts it again (used for things that must always be running, e.g. a getty login prompt or `syslogd`).
- **askfirst** — like `respawn`, **but** init first prints **"Please press Enter to activate this console"** and **waits** for the user to press Enter before actually starting the process; after that it behaves like respawn. (Typically used for spare consoles.)
- **once** — run the process **exactly once**: init does **not** wait for it to finish and does **not** restart it if it dies.
- **wait** — run the process **once** and **block**: init does not proceed to the next inittab entry until this process has **completed**.

**Explanation:**
In `/etc/inittab`, every line is `<id>:<runlevels>:<action>:<process>`, and the **action** field tells init *how* to run the given process. The five actions here differ along two axes — **does init wait for it?** and **does init restart it?**:

- **sysinit** is special only in *ordering*: it is guaranteed to execute first, which is why the boot script (rcS) is wired to it.
- **respawn** vs **once**: both can start a service, but respawn *keeps it alive* (auto-restart on exit) whereas once is *fire-and-forget* (no wait, no restart). Respawn is for permanent daemons/consoles; once is for a one-shot background job.
- **askfirst** is respawn with a **gate**: it avoids spawning a login shell on a console nobody is using by waiting for an Enter keypress first, then respawning as normal — handy for secondary serial consoles.
- **wait** is the *blocking* one-shot: init **pauses** the boot sequence until the process finishes, which you use when later steps depend on this one completing (whereas once would let boot continue in parallel).

*Source: Lecture 06 — System V/BusyBox init actions (sysinit, respawn, askfirst, once, wait).*

---

<a id="exam-sample-2"></a>

## Sample_2 — Embedded Linux Final Exam Sample 2 (FCIS, Spring 2023)

Total questions: 20

*Exam header: FCIS – Spring 2023; Embedded Linux Final Exam Sample 2. Q1: 7 MCQ [8 marks] · Q2: 6 fill-in-the-blank [7 marks] · Q3: A/B/C [15 marks] · Q4: A/B/C/D [10 marks]. No official answer key exists for this sample; answers below are verified against the lecture slides and the course study notes.*

---

### Q1.1. The ____ is a compressed version of the Linux kernel image that is self-extracting, while ____ is an image file that has a U-Boot wrapper. a) zImage, uimage  b) uImage, zImage  c) uImage, cpio  d) zImage, cpio

**Answer:** a) zImage, uimage

**Explanation:**
When the kernel finishes building it produces a raw binary called `Image` — the kernel code as-is. That file is large, so two further forms exist, and the exam tests the difference:

- **zImage** = a *compressed* kernel with a tiny **self-extracting** stub glued to the front. At boot the stub decompresses the real kernel into RAM and jumps to it. "Self-extracting compressed kernel" is exactly zImage. It does **not** know anything about a specific bootloader.
- **uImage** = a zImage (or Image) that has had a **U-Boot wrapper** added on top by the `mkimage` tool. That 64-byte header records the load address, entry point, image type, and a CRC, so the U-Boot bootloader can verify and place the image correctly. So "image file that has a U-Boot wrapper" = uImage.

The slide deck capitalises them `zImage` / `uImage`; option (a) writes the second one as the lowercase "uimage", but it is the only option that puts **zImage first (self-extracting)** and the **U-Boot-wrapped image second**, which is what the sentence asks for.

- b) uImage, zImage — wrong: reversed. The U-Boot-wrapped image is uImage, not the self-extracting one.
- c) uImage, cpio — wrong: `cpio` is the archive format used for an *initramfs* root filesystem, not a kernel image, and uImage is not self-extracting.
- d) zImage, cpio — wrong: first blank (zImage) is right, but a `cpio` archive is a rootfs image, not a U-Boot-wrapped kernel.

*Source: Lecture 04 — Kernel (zImage = self-extracting compressed kernel; uImage = +U-Boot mkimage wrapper).*

---

### Q1.2. The root filesystem is mounted to the system through the ____. a) mount command  b) bootloader ROM Code  c) root= kernel option  d) init program

**Answer:** c) root= kernel option

**Explanation:**
The **root filesystem (rootfs)** is the top-level `/` tree holding everything user space needs: programs, shared libraries, configuration files, and device nodes. There is a chicken-and-egg problem: the normal tool for attaching a filesystem is the `mount` command — but `mount` is itself a program that lives *inside* the rootfs. You cannot run a program from a filesystem that has not been mounted yet.

Linux solves this by making the **kernel mount the very first filesystem itself**, before any user-space program runs. The kernel must be told *which* device/partition holds the rootfs, and that is exactly what the **`root=` kernel command-line option** does — e.g. `root=/dev/mmcblk0p2`. If `root=` is missing or wrong, the kernel cannot find an init program to run and **panics**.

- a) mount command — wrong: `mount` lives inside the rootfs, so it cannot mount the rootfs itself; it is only used for *additional* filesystems afterwards.
- b) bootloader ROM Code — wrong: ROM code runs far earlier and only bootstraps the next boot stage; it has no concept of a Linux filesystem.
- d) init program — wrong: init is started *after* the rootfs is already mounted; a mounted rootfs is a precondition for running init.

*Source: Lecture 05 — Root filesystem (rootfs mounted directly by the kernel per the `root=` option).*

---

### Q1.3. If the mode of an executable owned by root is -rwsr-xr-x and user1 runs it, the executable will run with the privilege of the ____ . a) user1's group  b) root's group  c) user1  d) root

**Answer:** d) root

**Explanation:**
Read the mode string `-rwsr-xr-x` in triads after the leading file-type character:

- `-` = regular file.
- owner triad `rws` — note the **`s`** where the owner's execute bit `x` would normally be. That `s` is the **SETUID (SUID) bit**.
- group triad `r-x`, others triad `r-x`.

Normally a program runs with the **effective UID of whoever launched it** (so if user1 runs it, it runs as user1). The **SUID bit changes that rule**: it sets the process's **effective UID to the UID of the file's owner**. Here the owner is **root**, so no matter who starts the program — user1 included — it executes with **root** privileges. (The classic example is `ping`, which needs raw-socket access; it is owned by root and SUID so any user can run it.)

- a) user1's group — wrong: SUID affects the *user* identity, not the group; and it switches to the owner, not the caller.
- b) root's group — wrong: that would require the **SGID** bit (the `s` in the *group* triad). Here the `s` is in the *owner* triad → SUID, so it's root the user, not root's group.
- c) user1 — wrong: that is what would happen *without* the `s` bit (effective UID = caller). The SUID bit specifically overrides this.

*Source: Lecture 05 — Permissions / SUID (the `s` bit sets effective UID = file owner = root).*

---

### Q1.4. BusyBox init begins by reading ____ file which defines rules to start programs with their corresponding actions and runlevels, one per line. a) /etc/init  b) /etc/inittab  c) /etc/init.d/rcS  d) /etc/startup

**Answer:** b) /etc/inittab

**Explanation:**
After the kernel mounts the rootfs it launches **init** (PID 1), the first user-space process. BusyBox's init (a small System-V-style init) **begins by reading the `/etc/inittab` file**. Each line is one rule in the format `<id>:<runlevels>:<action>:<process>` — for example `::sysinit:/etc/init.d/rcS`. The `action` field (sysinit, respawn, askfirst, once, wait…) tells init *how* to run that program, and the runlevels field says *when*. That matches the question's wording exactly: "rules to start programs with their corresponding actions and runlevels, one per line."

The trap on this family of questions is `/etc/init.d/rcS`: that is the **script** init *runs* (the place to put boot-time shell commands), not the file init *reads first* to learn its rules.

- a) /etc/init — wrong: not a standard init configuration file in BusyBox; init reads `/etc/inittab`.
- c) /etc/init.d/rcS — wrong: this is the startup *script* listed as a sysinit action *inside* inittab, where you put boot-time shell commands. It is not the rule file init parses first.
- d) /etc/startup — wrong: not a real BusyBox/System-V init file.

*Source: Lecture 06 — Init & inittab (init begins by reading /etc/inittab; rcS holds boot-time shell commands).*

---

### Q1.5. The Yocto project component that contains settings and tasks for building packages is called ___ . a) recipes  b) layers  c) configuration Files  d) metadata

**Answer:** a) recipes

**Explanation:**
The Yocto Project builds an entire Linux distribution from text descriptions. Its vocabulary is examined repeatedly, so pin down each term:

- **Recipes** (`.bb` files) — the slide defines a recipe as "**a list of settings and tasks (instructions) for building packages**." Each recipe describes how to fetch, configure, compile and install **one** software component, via standard tasks `do_fetch`, `do_unpack`, `do_patch`, `do_configure`, `do_compile`, `do_install`, `do_package`. This is the verbatim match for the question.
- **Layers** — directories that *group* recipes/config together and can override earlier layers; they organise recipes, they are not the per-package settings-and-tasks file.
- **Configuration files** — global/user variables and hardware configuration, not per-package build instructions.
- **Metadata** — the umbrella term for *everything* (recipes + configuration files + build-control commands), so it is broader than what the question describes.

- b) layers — wrong: a layer is a collection/override mechanism for recipes, not the settings-and-tasks file for one package.
- c) configuration Files — wrong: these hold global/machine variables, not the build instructions for a package.
- d) metadata — wrong: too broad — metadata = recipes + config files + commands; the precise component "settings and tasks for building packages" is the recipe.

*Source: Lecture 07 — Build Systems / Yocto glossary (recipe = "settings and tasks for building packages").*

---

### Q1.6. In JFFS2 filesystem, there is one block receiving updates at a time which is called the ___ block. a) Update  b) Write  c) Active  d) Open

**Answer:** d) Open

**Explanation:**
**JFFS2** is a **log-structured** filesystem for raw flash: instead of overwriting data in place (flash can't do that without erasing a whole block first), it **appends** new "nodes" sequentially through the erase blocks. The slide classifies erase blocks by state:

- **Free** — contains no nodes yet (erased, ready to use).
- **Clean** — contains only valid (current) nodes.
- **Dirty** — contains at least one obsolete (superseded) node.
- **Open** — the single block **currently receiving updates**; new nodes are written here until it fills, then it closes and a fresh free block becomes the next open block.

So "one block receiving updates at a time" is the **open** block. A bonus consequence: if power is lost mid-write, only the last write to the *open* block is at risk; everything already committed is safe.

- a) Update — wrong: not a JFFS2 block state; the term is "open."
- b) Write — wrong: not the JFFS2 terminology either.
- c) Active — wrong: plausible-sounding but not the slide's name; the block taking writes is the "open" block.

*Source: Lecture 09 — Flash & MTD / JFFS2 (Free / Clean / Dirty / one Open block receives updates).*

---

### Q1.7. To write to NAND flash, the _____ tool should be used. a) cp  b) nandwrite  c) nanddump  d) nandcp

**Answer:** b) nandwrite

**Explanation:**
NAND flash is special: some erase blocks are **bad** (defective from the factory or worn out), and NAND must be **erased before it can be written**. Ordinary file tools don't understand any of that. The `mtd-utils` package provides NAND-aware tools:

- **`nandwrite`** — programs NAND from a file and **automatically skips bad blocks**. This is the correct tool to *write* NAND.
- **`nanddump`** — *reads* NAND out to a file, also skipping bad blocks (the read counterpart).
- Workflow reminder: erase first with `flash_erase /dev/mtdX 0 0`, then `nandwrite`.

The standing trap is `cp`: copying directly to a NAND device fails at the first bad block, because `cp` has no bad-block handling.

- a) cp — wrong: a generic copy with no bad-block awareness; it fails at the first bad block.
- c) nanddump — wrong: that is the *read* tool (dumps NAND to a file), not the writer.
- d) nandcp — wrong: no such standard mtd-utils tool exists.

*Source: Lecture 09 — Flash & MTD (nandwrite writes NAND skipping bad blocks; nanddump reads; cp = trap).*

---

### Q2.1. The toolchain utility that discovers the library dependencies of an executable is called ____, while the tool that removes symbol tables information from libraries is called ___ .

**Answer:** **readelf** (toolchain utility that lists an executable's shared-library dependencies; `ldd` is the on-target alternative) / **strip** (removes symbol-table information from binaries and libraries to shrink them).

**Explanation:**
A dynamically linked program does not contain the library code it uses — it only records *which* shared libraries and *which* dynamic loader it needs, so those must be discovered and copied into the target rootfs.

- **readelf** — a binutils utility that parses the ELF headers. `arm-...-readelf -a busybox | grep "interpreter"` prints the **program interpreter** (the dynamic linker, e.g. `/lib/ld-linux.so.3`); `... | grep "Shared"` prints the `(NEEDED)` **shared libraries** (e.g. `libm.so.6`, `libc.so.6`). This tells you exactly which loader and libraries to include on the target. (`ldd` does the same job but runs *on the target*; readelf is the cross/host-side toolchain tool — which is what "toolchain utility" asks for.)
- **strip** — removes the **symbol table and debugging information** from a binary or library. Those tables are only needed for debugging; stripping them makes the file much smaller, which matters on space-constrained embedded flash.

*Source: Lecture 05 — Root filesystem (readelf for dependency discovery) / Lecture 02 — Toolchain (strip removes symbol tables).*

---

### Q2.2. The bootloader must pass two pointers to the kernel: ____ and ____ .

**Answer:** the **device tree blob (dtb) base address** and the **kernel command line** (the two pointers the bootloader hands to the kernel as it jumps to it).

**Explanation:**
When the bootloader (e.g. U-Boot) finishes loading the kernel into RAM and jumps to its entry point, the kernel needs two pieces of information that the bootloader has prepared in memory, passed as pointers in registers:

1. **The device tree (dtb / FDT) base address** — a pointer to the **flattened device tree** in RAM. The device tree describes the hardware (CPUs, memory size, buses, peripherals, interrupts) that the kernel cannot auto-discover on an embedded SoC, so the kernel reads it to know what hardware exists.
2. **The kernel command line** — the boot arguments string, most importantly **`root=`** (which device holds the rootfs), plus things like `console=`. This tells the kernel how to come up and where to find user space.

With these two pointers the kernel can initialise the described hardware and then mount the root filesystem named on the command line.

*Source: Lecture 03 — Bootloader & Device Tree (bootloader passes two pointers: device tree + kernel command line).*

---

### Q2.3. In Kconfig language, to define a config option that can be compiled as a kernel module, it should be defined as ____ option type.

**Answer:** **tristate**

**Explanation:**
The kernel's configuration system (**Kconfig**) describes every build option, and each option has a **type** that controls what values it may take:

- **bool** — two states: `y` (built **into** the kernel image) or *not set* (excluded). It **cannot** be a module.
- **tristate** — **three** states: `y` (built-in), `m` (built as a **loadable kernel module**, an external `.ko` file loaded at runtime), or *not set* (excluded). The `m` state is the whole point of the question: only a **tristate** option can be "compiled as a kernel module."
- Other types (`string`, `int`, `hex`) hold values, not build inclusion.

So any feature that can optionally be a module — most drivers — is declared `tristate`.

*Source: Lecture 04 — Kernel & Kconfig (tristate: y built-in / m module / not set off).*

---

### Q2.4. The purpose of the ___ and ____ pseudo filesystems is to expose information about processes and kernel driver model to user space, respectively.

**Answer:** **/proc** (exposes information about **processes**) and **/sys** (exposes the kernel **driver model**), respectively.

**Explanation:**
Both are **pseudo (virtual) filesystems**: they hold no real data on disk. Instead, reading or writing their files runs kernel code that returns live information — a clean, file-based window into the kernel for user space.

- **/proc** (procfs) — originally and primarily a view of **processes**: `/proc/<pid>/` directories expose each process's status, memory map, open files, etc. (it also carries some general kernel info like `/proc/cmdline`, `/proc/mtd`). The keyword "processes" maps to /proc.
- **/sys** (sysfs) — a structured view of the **kernel driver/device model**: the devices discovered since boot, the buses they hang off, the drivers bound to them, and their attributes (e.g. `/sys/class/gpio/...`). The keyword "kernel driver model" maps to /sys.

The word "respectively" fixes the order: processes → /proc, driver model → /sys.

*Source: Lecture 04 / Lecture 08 — /proc = processes, /sys = kernel driver model.*

---

### Q2.5. After building buildroot, the kernel and root filesystem images will be located at the _____ directory.

**Answer:** **output/images** (commonly referred to as the `images/` directory).

**Explanation:**
A Buildroot build populates an **`output/`** tree with several sub-directories, and the exam tests which one holds the final deliverables:

- **output/images** — the **build results you actually deploy**: the bootloader, the **kernel image** (zImage/uImage + device-tree blobs), and the **root-filesystem images** (e.g. `rootfs.ext2`, `rootfs.cpio`). This is the answer.
- output/staging — a symlink to the toolchain's **sysroot** (headers + libraries).
- output/target — the rootfs *tree* as it will appear on the target, but **not directly bootable** (no device nodes, host ownership); a device table fixes ownership/permissions when the image in `images/` is created.
- output/build — per-package build directories; output/host — host tools including the cross-toolchain.

So after `make`, the kernel and rootfs images live under **output/images**.

*Source: Lecture 07 — Buildroot output/ tree (images = the deployable kernel + rootfs build results).*

---

### Q2.6. Device drivers are identified in user space by a special file called a _____.

**Answer:** **device node** (a special file in `/dev`).

**Explanation:**
User-space programs cannot call kernel driver functions directly. Instead, a driver is represented to user space by a **device node** — a *special file*, conventionally living in **/dev**, that acts as the handle to the driver. The node carries three pieces of identity:

- a **type**: `c` (character device) or `b` (block device);
- a **major number** — which **driver/device class** the node belongs to;
- a **minor number** — which **specific instance/partition** within that driver.

When a program does `open("/dev/ttyS0", ...)` / `read` / `write`, the kernel routes those calls to the driver identified by the node's major number. Nodes are created manually with **`mknod <name> <c|b> <major> <minor>`**, or automatically on demand by **udev / mdev / devtmpfs**. Example minimal nodes: `/dev/console` (`c 5 1`) and `/dev/null` (`c 1 3`).

*Source: Lecture 05 / Lecture 08 — Device nodes (special file in /dev mapping name → type, major, minor).*

---

### Q3.A. For the given device tree, answer the following questions. (Tree: an `external-bus` node with `#address-cells = <2>`, `#size-cells = <1>`, `ranges = <1 0 0x10160000 0x10000>`; inside it `i2c@1,0` with `compatible = "acme,a1234-i2c-bus"`, its own `#address-cells = <1>`, `#size-cells = <0>`, `reg = <1 0 0x1000>`; and inside the i2c node `rtc@58` with `compatible = "maxim,ds1338"`, `reg = <58>`.)

```dts
/dts-v1/;
/ { …
  external-bus {
    #address-cells = <2>;
    #size-cells = <1>;
    ranges = <1 0 0x10160000 0x10000>;
    i2c@1,0 {
      compatible = "acme,a1234-i2c-bus";
      #address-cells = <1>;
      #size-cells = <0>;
      reg = <1 0 0x1000>;
      rtc@58 {
        compatible = "maxim,ds1338";
        reg = <58>;
      };
    };
  };
};
```

1) The addresses of the I2C node ranges from ______ to ______
2) When the I2C node is accessed in the external bus address space, we can use this address range: from _______ to _________
3) Complete the interrupt-controller node to be added to the top node of the tree (base address 0x40000, address length 0x1000).
4) Assume the interrupt line input that the I2C node is attached to equals 2. Specify the necessary properties that should be added to the device tree and in which nodes to enable this interrupt line. Assume any missing information (if any).

**Answer:**

**(1)** From **0x0** to **0xFFF** (on chip-select 1). The i2c node's `reg = <1 0 0x1000>` is interpreted using its **parent's** cell counts (`external-bus` has `#address-cells = <2>`, `#size-cells = <1>`), so the tuple is *(address = chip-select 1, offset 0), (length 0x1000)*. In the I2C device's own local view that is offset **0x0** through **0x0 + 0x1000 − 1 = 0xFFF**, sitting on chip-select 1.

**(2)** From **0x10160000** to **0x10160FFF**. The `external-bus` node's `ranges = <1 0 0x10160000 0x10000>` translates a child address *(child-bus = cs1, child-offset = 0)* to the **parent (CPU) base 0x10160000**, for a window of length 0x10000. So the I2C region's local 0x0–0xFFF appears in the external-bus / CPU address space at **0x10160000** through **0x10160000 + 0xFFF = 0x10160FFF**.

**(3)** Fill the two blanks as:
- **[a]** node name = `interrupt-controller@40000`
- **[b]** `reg = <0x40000 0x1000>`

```dts
interrupt-controller@40000 {
    compatible = "fsl,p1190";
    reg = <0x40000 0x1000>;
    interrupt-controller;
    #interrupt-cells = <2>;
};
```

**(4)** To wire the I2C node to interrupt line 2, add two properties **inside the `i2c@1,0` node**:

```dts
i2c@1,0 {
    /* ...existing properties... */
    interrupt-parent = <&intc>;   /* phandle to the controller node above */
    interrupts = <2 0>;           /* TWO cells: line number 2, trigger/flags 0 */
};
```

The controller node must (and already does in part 3) carry the empty property `interrupt-controller;` and `#interrupt-cells = <2>;`. Give that controller a label (e.g. `intc: interrupt-controller@40000 { ... }`) so `interrupt-parent = <&intc>` can reference it.

**Explanation:**
A device tree describes hardware to the kernel. Two ideas drive this whole question:

- **How `reg` is decoded.** A node's `reg` is a list of *(address, length)* tuples, but the **number of cells** for each address and each length is set by the **parent** node via `#address-cells` and `#size-cells`. Here the parent `external-bus` uses `#address-cells = <2>` (so an address is two 32-bit numbers: a chip-select **plus** an offset) and `#size-cells = <1>` (length is one number). That is why `reg = <1 0 0x1000>` reads as "chip-select 1, offset 0, length 0x1000." Inside the i2c node itself `#address-cells = <1>`/`#size-cells = <0>`, which is why its child `rtc@58` only needs a single-number address (`reg = <58>`, the I2C 7-bit slave address 0x58) and no length.

- **How `ranges` translates addresses.** `ranges = <child-address parent-address length>` maps a child window into the parent's address space. `<1 0 0x10160000 0x10000>` means "child (cs1, offset 0) lives at parent base **0x10160000** for **0x10000** bytes." So the device's *local* 0x0 corresponds to *CPU* 0x10160000. (If `ranges` were empty it would be an identity 1:1 mapping; if it were **absent**, the region would be reachable only by the parent, not by the CPU.) This is exactly the worked S2 example in the notes: local cs1 0x0–0xFFF → CPU 0x10160000–0x10160FFF.

- **Adding the interrupt controller (part 3).** Node names follow `name@unit-address`, where the unit-address equals the **first address in `reg`** — here 0x40000, giving `interrupt-controller@40000`. The controller sits at the **top node**, whose `#address-cells`/`#size-cells` are 1/1 (32-bit SoC), so `reg = <0x40000 0x1000>` is one address + one length. The empty property `interrupt-controller;` *declares* "this node is a device that receives interrupt signals," and `#interrupt-cells = <2>` says each interrupt specifier that targets it is **two** numbers.

- **Connecting the I2C to line 2 (part 4).** A device announces its interrupt with two properties: **`interrupt-parent`** (a *phandle* — a `<&label>` reference — to the controller it is wired to) and **`interrupts`** (the specifier). Because the controller declared `#interrupt-cells = <2>`, the specifier must be **two cells**: the first is the **line number (2)**, the second is the **trigger type / flags** (level vs edge; we assume 0 as it is unspecified). These go **in the device node** (the i2c node), *not* in the controller. (`interrupt-parent` is often set once on the root node and inherited, but stating it on the i2c node is the explicit, always-correct form.)

*Source: Lecture 03 — Bootloader & Device Tree (reg cells from parent; ranges translation; interrupt-controller / #interrupt-cells / interrupt-parent / interrupts).*

---

### Q3.B. The /sys pseudo filesystem provides detailed information about device drivers to the user space. Complete the following table either by the directory path or by the information/content that the given directory provides. (Rows given — Directory | Content: [1] | "the kernel's view of the devices discovered since boot and how they are connected to each other"; /sys/devices/system | [2]; [3] | "contains devices that are memory-based such as /dev/null, /dev/random"; /sys/devices/platform | [4]; [5] | "is a software view of the device drivers and contains a subdirectory for each class of drivers".)

**Answer:**

| Directory | Content |
|---|---|
| **[1] `/sys/devices`** | the kernel's view of the devices discovered since boot and how they are connected to each other |
| `/sys/devices/system` | **[2]** core system devices that are not on a discoverable peripheral bus — **CPUs, memory, and clocks/buses** |
| **[3] `/sys/devices/virtual`** | contains devices that are memory-based such as /dev/null, /dev/random |
| `/sys/devices/platform` | **[4]** **platform devices** — devices integrated into the SoC / sitting on no conventional enumerable bus (enumerated statically, not auto-discovered) |
| **[5] `/sys/class`** | is a software view of the device drivers and contains a subdirectory for each class of drivers |

**Explanation:**
**sysfs** (mounted at `/sys`) is a virtual filesystem that exposes the kernel's **device model**: every device the kernel knows about, the buses they connect through, and the drivers bound to them. The mapping rule is simple — a **kernel object becomes a directory**, an **attribute becomes a file**, and a **relationship becomes a symlink**. The table walks the main branches:

- **`/sys/devices`** — the **master, real hardware view**: every device discovered since boot, arranged to show **how they connect** (a device nested under the bus/controller it hangs off). All other views are alternate organisations of these same objects.
- **`/sys/devices/system`** — the **core system devices** that are *not* on a hot-pluggable/discoverable peripheral bus: **CPUs, memory, and clocks/buses**. (You will see `cpu/cpu0`, etc.)
- **`/sys/devices/virtual`** — **memory-based / software virtual devices** that have no physical hardware behind them, such as `/dev/null` and `/dev/random`.
- **`/sys/devices/platform`** — **platform devices**: peripherals **integrated into the SoC** that sit on no conventional enumerable bus (PCI/USB can self-enumerate; platform devices cannot, so they are described statically, e.g. via the device tree).
- **`/sys/class`** — a **software (functional) view** that groups drivers by *what they do* rather than where they live: one sub-directory per **class** (e.g. `gpio`, `net`, `tty`, `block`), with symlinks pointing back into `/sys/devices`. This is the friendly path user space usually uses — e.g. `/sys/class/gpio/...`.

*Source: Lecture 08 — Device Drivers / sysfs (/sys/devices, …/system, …/virtual, …/platform, /sys/class).*

---

### Q3.C. Assume the following device tree for a NAND flash memory of 128KB erase blocks. (1) Show the expected output of `cat /proc/mtd`. (2) What is the total size of this memory in MB? (Tree: `nand@0,0` with `#address-cells = <1>`, `#size-cells = <1>`, and four partitions — `partition@0` label "SPL" `reg = <0 0x80000>`; `partition@80000` label "U-Boot" `reg = <0x80000 0xc3000>`; `partition@143000` label "Kernel" `reg = <0x143000 0x400000>`; `partition@543000` label "Filesystem" `reg = <0x543000 0x7abd000>`.)

```dts
nand@0,0 {
    #address-cells = <1>;
    #size-cells = <1>;
    partition@0 {
        label = "SPL";
        reg = <0 0x80000>;
    };
    partition@80000 {
        label = "U-Boot";
        reg = <0x80000 0xc3000>;
    };
    partition@143000 {
        label = "Kernel";
        reg = <0x143000 0x400000>;
    };
    partition@543000 {
        label = "Filesystem";
        reg = <0x543000 0x7abd000>;
    };
};
```

**Answer:**

**(1)** Expected `cat /proc/mtd` output (columns: device, size, erasesize, name — all in hex; erasesize = 128 KB = 0x20000 for every partition):

```
dev:    size   erasesize  name
mtd0: 00080000 00020000 "SPL"
mtd1: 000c3000 00020000 "U-Boot"
mtd2: 00400000 00020000 "Kernel"
mtd3: 07abd000 00020000 "Filesystem"
```

**(2)** Total size = (offset of last partition) + (size of last partition) = `0x543000 + 0x7abd000` = **0x8000000 = 128 MB** (134,217,728 bytes).

**Explanation:**
The **MTD (Memory Technology Devices)** subsystem represents raw flash, and the device tree's `partition@<offset>` nodes carve the chip into named regions. Each `reg = <offset size>` tuple is *(start offset, length)* — one cell each, because the `nand@0,0` parent sets `#address-cells = <1>` and `#size-cells = <1>`.

**Reading `/proc/mtd`:** this file lists the partitions as `mtdN: <size> <erasesize> "<name>"`, all in **hexadecimal**. Three things to get right:
- The **size** column is the partition's **length** (the *second* number of `reg`), **not** its offset. So SPL → `0x80000`, U-Boot → `0xc3000`, Kernel → `0x400000`, Filesystem → `0x7abd000`. (`/proc/mtd` pads sizes to 8 hex digits, e.g. `00080000`.)
- The **erasesize** is the flash's erase-block size, given in the question as **128 KB**. 128 × 1024 = 131072 = **0x20000**, and it is the same for every partition.
- Partitions are numbered `mtd0, mtd1, …` in declaration order, with the `label` becoming the name.

**Computing the total size:** the partitions are laid out back-to-back, so the chip's total capacity is the **end of the last partition** = its **offset + its size**:
`0x543000 + 0x7abd000 = 0x8000000`. Convert: 0x8000000 = 8 × 16⁶ = 134,217,728 bytes = 134,217,728 / 1,048,576 = **128 MB**. (Sanity check on the offsets: 0 + 0x80000 = 0x80000; 0x80000 + 0xc3000 = 0x143000; 0x143000 + 0x400000 = 0x543000 — each partition's offset equals the previous offset plus the previous size, confirming a gapless layout.)

*Source: Lecture 09 — Flash & MTD partitions (DT partition@offset ↔ /proc/mtd "mtdN: size erasesize name"; total = last offset + last size = 0x8000000 = 128 MB).*

---

### Q4.A. List six components of a minimal root filesystem. [2 marks]

**Answer:** A minimal root filesystem needs:
1. **An init program** — the first user-space process (PID 1), e.g. BusyBox init, that brings the system up.
2. **A shell** — a command interpreter so the system is usable / scriptable (e.g. BusyBox `sh`).
3. **Basic daemons / utilities** — the small set of background services and applets (login, getty, syslog, the core commands) needed to operate the system.
4. **Shared libraries** — the **C library and the dynamic loader** (e.g. `libc.so`, `ld-*.so`) that the dynamically linked programs above depend on.
5. **Configuration files** — the contents of **/etc** (e.g. `inittab`, `passwd`, `init.d/rcS`) that tell the programs how to start and behave.
6. **Device nodes** — the special files in **/dev** (at minimum **console** and **null**) that let user space talk to drivers.

(Also commonly listed: the **/proc and /sys mount points**, and any **kernel modules** the system loads.)

**Explanation:**
A "root filesystem" is not just the kernel — the kernel mounts `/` and then needs a working user space inside it. The slide lists the *minimum* set of ingredients that make that user space able to boot and run. The logic chain: the kernel hands control to **(1) init**, which reads its **(5) configuration** (`/etc/inittab`, `rcS`) and launches a **(2) shell** plus the essential **(3) daemons/services**; every one of those programs is dynamically linked, so it cannot start without the **(4) shared libraries** (C library + loader); and any of them that touches hardware or special streams needs **(6) device nodes** in `/dev` (a console to print to, `/dev/null` to discard output). Drop any one of these six and the system either won't boot or won't be usable. BusyBox conveniently provides (1), (2) and (3) from a single binary.

*Source: Lecture 05 — Root filesystem (six components: init, shell, daemons, shared libraries, configuration files, device nodes; + /proc & /sys mounts, kernel modules).*

---

### Q4.B. System V init supports the concept of runlevels. List the 8 run levels showing when each level is executed. [2 marks]

**Answer:** The eight System V runlevels and when each is entered:

| Runlevel | When executed / meaning |
|---|---|
| **S** (or **s**) | **Startup / single-user boot tasks** — the level the system enters **first** at boot to run the one-time initialisation scripts. |
| **0** | **Halt** — shuts the system down and powers off / stops the CPU. |
| **1** | **Single-user mode** — minimal maintenance/administration mode, no networking, no multi-user services. |
| **2** | **Multi-user without networking** — multiple users, but network services not started. |
| **3** | **Multi-user with networking** — the normal full text-mode operating level (network up). |
| **4** | **Unused / user-definable** — reserved for custom configurations. |
| **5** | **Multi-user with graphical login (GUI)** — like 3 plus the X display manager. |
| **6** | **Reboot** — shuts down and restarts the system. |

**Explanation:**
A **runlevel** is a named operating state of the machine; each defines **which services are running**. System V init switches between them by running the start/stop scripts associated with each level (on a switch it runs the **K**ill scripts of services to stop, then the **S**tart scripts of services to begin). The set splits naturally into three groups:

- **The boot level — S:** entered first, it runs the one-time system-initialisation tasks before any normal runlevel is reached.
- **Normal operating levels — 1, 2, 3, 4, 5:** increasing capability, from single-user maintenance (1), to multi-user without network (2), to the everyday full network text mode (3), to a spare/custom level (4), to full graphical desktop (5).
- **The two transition levels — 0 and 6:** these aren't states you "stay" in; entering **0** halts the machine and entering **6** reboots it.

A handy mnemonic: **0 = off, 6 = reboot** (think "0 and 6 bracket the ends"), 1 = single user, 3 = normal/networked, 5 = GUI.

*Source: Lecture 06 — Init & Runlevels (S startup · 0 halt · 1 single-user · 2 multi-user no net · 3 multi-user + net · 4 unused · 5 graphical · 6 reboot).*

---

### Q4.C. Explain steps of creating a buildroot package for a custom user application called MyEOS. [3 marks]

**Answer:** To add a custom Buildroot package named **MyEOS**:

1. **Create the package directory:** `mkdir package/myeos`.
2. **Write `package/myeos/Config.in`** — the Kconfig entry that makes the package selectable in the menu:
   ```
   config BR2_PACKAGE_MYEOS
       bool "myeos"
       help
         The MyEOS custom user application.
   ```
3. **Register it in the menu** by adding a line to `package/Config.in`:
   ```
   source "package/myeos/Config.in"
   ```
4. **Write the `.mk` recipe `package/myeos/myeos.mk`** describing where the source is and how to build/install it:
   ```make
   MYEOS_VERSION = 1.0
   MYEOS_SITE = /path/to/myeos/source
   MYEOS_SITE_METHOD = local

   define MYEOS_BUILD_CMDS
       $(MAKE) CC="$(TARGET_CC)" -C $(@D)
   endef

   define MYEOS_INSTALL_TARGET_CMDS
       $(INSTALL) -D -m 0755 $(@D)/myeos $(TARGET_DIR)/usr/bin/myeos
   endef

   $(eval $(generic-package))
   ```
5. **Enable it:** run `make menuconfig` and turn on the new **myeos** option (under its menu).
6. **Build:** run `make` (or rebuild just this package with `make myeos-rebuild`); the resulting binary lands in `output/target/usr/bin` and in the final rootfs image.

**Explanation:**
Buildroot already builds ~2000 packages from a uniform recipe pattern; adding your own application means slotting it into that same pattern. There are two files Buildroot needs for **every** package, and the question is really "what are those two files and how do you hook them in":

- **`Config.in`** is the *menu/Kconfig* side — it defines the boolean option `BR2_PACKAGE_MYEOS` so the user can select the package in `make menuconfig`. Buildroot only "sees" your package once you **`source`** that file from the top-level `package/Config.in` (step 3) — otherwise it never appears in the menu.
- **`myeos.mk`** is the *build* side — it tells Buildroot **where** to get the source (`_SITE` + `_SITE_METHOD`; `local` means "a directory on this machine," ideal for your own in-house app), and **how** to build and install it via the `_BUILD_CMDS` and `_INSTALL_TARGET_CMDS` blocks (note the cross-compiler `$(TARGET_CC)` and the install into `$(TARGET_DIR)`, the staging rootfs). The final line **`$(eval $(generic-package))`** is the magic that expands all the standard per-package make targets (download, configure, build, install, clean, rebuild) for you.

After both files exist and the option is enabled (step 5), a normal `make` builds MyEOS along with everything else; `make myeos-rebuild` re-runs just its build/install steps during development.

*Source: Lecture 07 — Buildroot custom package (mkdir → Config.in → source in package/Config.in → <name>.mk with generic-package → menuconfig → make).*

---

### Q4.D. Compare among message-based interprocess communication techniques provided by Linux in terms of data boundary, flow direction, max message size and message priority. [3 marks]

`[DERIVED — IPC lecture not in course folder; standard Linux / Simmonds "Mastering Embedded Linux Programming" ch. 17.]`

**Answer:**

| IPC technique | Data boundary | Flow direction | Max message size | Message priority |
|---|---|---|---|---|
| **Pipe** | **None** — a continuous **byte stream** (no message boundaries) | **Unidirectional** (one read end, one write end) | No fixed per-message limit (a kernel pipe buffer, typically 64 KB, only bounds buffering) | **None** — strictly FIFO order |
| **FIFO (named pipe)** | **None** — byte stream, like a pipe | **Unidirectional** | No per-message limit (same pipe buffer) | **None** — FIFO order |
| **Unix-domain socket** | **Depends on type:** `SOCK_DGRAM` **preserves** datagram boundaries; `SOCK_STREAM` does **not** (byte stream) | **Bidirectional** (full duplex) | Bounded by the socket send buffer (configurable) | **None** (ordinary delivery order) |
| **POSIX message queue** | **Yes** — each `mq_send`/`mq_receive` transfers **one whole message** (boundaries preserved) | Effectively bidirectional via the queue (any opener can send/receive; not a single stream) | **Fixed maximum per message** (`mq_msgsize`, set at `mq_open`) | **Yes** — messages carry a **priority**; highest-priority message is delivered first |

**Key takeaway:** the **POSIX message queue is the only technique that both preserves message boundaries *and* supports message priority** (with a configured maximum message size). Pipes and FIFOs are unidirectional byte streams with no boundaries and no priority (a FIFO is just a *named* pipe usable by **unrelated** processes). A Unix-domain socket is bidirectional and can preserve boundaries (datagram mode) but has no priority.

**Explanation:**
"Message-based" IPC means moving discrete chunks of data between processes; the four axes asked about are exactly the properties that distinguish the mechanisms:

- **Data boundary** — does the receiver get back the *same units* the sender wrote? A **byte stream** (pipe/FIFO, and stream sockets) does **not**: if the sender writes 10 bytes then 20 bytes, the reader might read all 30 at once. A **message/datagram** interface (POSIX mqueue, datagram sockets) **does**: one send = one receive of the same chunk.
- **Flow direction** — **pipes and FIFOs are unidirectional** (you need two of them for two-way traffic); **sockets are bidirectional/full-duplex**; a **message queue** is a shared mailbox that any opener can write to or read from.
- **Max message size** — pipes/FIFOs have no per-message cap (only a kernel buffer that limits how much can be in flight); a **POSIX message queue has a fixed `mq_msgsize`** chosen when the queue is created (and a max number of messages, `mq_maxmsg`); sockets are bounded by their send/receive buffer sizes.
- **Message priority** — only the **POSIX message queue** lets a sender tag each message with a priority so urgent messages jump ahead; everything else delivers strictly in order.

Quick API anchors: pipe → `pipe()`; FIFO → `mkfifo()` then `open`/`read`/`write`; Unix socket → `socket(AF_UNIX, SOCK_STREAM|SOCK_DGRAM, …)`; POSIX message queue → `mq_open()`, `mq_send()`, `mq_receive()`.

*Source: Lecture ? — Processes & IPC (lecture not present in the course folder; content per standard Linux / Simmonds ch. 17). `[DERIVED — verify against the missing Processes/IPC deck.]`*

---
