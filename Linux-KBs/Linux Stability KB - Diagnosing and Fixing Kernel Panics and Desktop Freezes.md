# Linux Stability KB: Diagnosing and Fixing Kernel Panics & Desktop Freezes

**Hardware:** HP EliteBook 840 G2 | **OS:** Ubuntu 24.04 LTS | **Date:** February 2026  
**Author:** JoelKwihangana | **Status:** Resolved ✅

---

## What Happened (The Problem)

Two separate but related issues occurred:

1. **Desktop freeze** — The screen locked up completely. Mouse and keyboard stopped responding. Could not click anything or switch windows.
2. **Kernel panic** — The screen went black and showed:
   ```
   KERNEL PANIC!
   Please reboot your computer.
   Fatal exception in interrupt
   ```

These are different failure modes. Understanding the difference matters:

- A **desktop freeze** means the Linux kernel is still alive, but the graphical session (GNOME/GDM) is deadlocked. The kernel can sometimes be reached via keyboard shortcuts.
- A **kernel panic** means the kernel itself hit an unrecoverable error and halted. Nothing works. Hard reboot is the only option. This is Linux's equivalent of Windows BSOD.

---

## Mental Model: How Linux Boots and Who Controls What

```
BIOS/UEFI → GRUB (bootloader) → Linux Kernel → systemd → Desktop (GNOME/GDM)
```

- **GRUB** is the boot menu. It lets you choose which kernel to boot.
- **The kernel** is the core of the OS. It talks to hardware directly (CPU, GPU, RAM, storage).
- **systemd** is the init system — it starts all services after the kernel loads.
- **GNOME/GDM** is the graphical desktop. It runs on top of all of this.

When something breaks, the layer that broke determines your recovery options.

---

## Issue 1: Desktop Freeze (Bluetooth Suspend Deadlock)

### Root Cause

When the laptop tried to **suspend (sleep)**, it sent a signal to the Bluetooth daemon asking it to pause. The Bluetooth daemon never responded. The kernel waited indefinitely. The entire desktop locked up.

Evidence from logs:
```
Feb 18 19:00:38 bluetoothd: profiles/audio/avdtp.c:handle_unanswered_req() No reply to Suspend request
```
This line says: "I sent a Suspend request to a Bluetooth audio profile, and it never replied."

### Emergency Recovery: Magic SysRq

If this happens again and the desktop is frozen, use the kernel's emergency keyboard shortcut:

```
Hold: Alt + PrintScreen
Then type one at a time: R  E  I  S  U  B
```

Remember as: **"Reboot Even If System Utterly Broken"**

Each key directly commands the kernel, bypassing the frozen desktop:
- `R` — Takes keyboard control back from X11 (the display server)
- `E` — Sends SIGTERM to all processes (graceful shutdown signal)
- `I` — Sends SIGKILL to all processes (force kill)
- `S` — Syncs all filesystems to disk (prevents data corruption)
- `U` — Remounts filesystems as read-only (safe state)
- `B` — Reboots

---

### How to Read the Logs

#### Command 1: Read errors from the previous boot
```bash
journalctl -b -1 --priority=3
```

**Breaking this down:**
- `journalctl` — The tool that reads systemd's journal (log database)
- `-b -1` — "boot minus 1" = the boot session before the current one. `-b 0` = current boot, `-b -1` = last boot, `-b -2` = two boots ago
- `--priority=3` — Only show messages at severity level 3 (errors) or worse. Linux uses levels 0-7: 0=emergency, 3=error, 7=debug

#### Command 2: Read kernel ring buffer
```bash
sudo dmesg | tail -50
# dmesg without sudo is blocked on modern Ubuntu for security reasons
```

**Breaking this down:**
- `dmesg` — Reads the kernel ring buffer. This is a circular memory buffer the kernel writes hardware and driver messages to during and after boot
- `sudo` — Required because Ubuntu 24.04 restricts raw kernel buffer access to root (security hardening added in kernel 4.15+)
- `| tail -50` — Pipe the output to `tail`, which shows only the last 50 lines

---

### The Fix: A systemd Service to Block Bluetooth Before Sleep

The solution is to tell the system: **before you go to sleep, shut off Bluetooth first**. We do this with a custom systemd service unit.

#### What is a systemd service unit?

A service unit is a configuration file that tells systemd:
- When to run something (before sleep? at boot? on a timer?)
- What to run (which command/binary)
- What to do on success or failure

Service files live in `/etc/systemd/system/` for custom/admin-created services.

#### Creating the service

```bash
sudo nano /etc/systemd/system/bluetooth-suspend.service
```

**Why this path?** `/etc/systemd/system/` is where system administrators put custom service units. Units here override anything in `/lib/systemd/system/` (where packages install their units). It survives package updates.

Contents of the file:

```ini
[Unit]
Description=Disable Bluetooth before suspend
Before=sleep.target

[Service]
Type=oneshot
ExecStart=/usr/sbin/rfkill block bluetooth
ExecStop=/usr/sbin/rfkill unblock bluetooth
RemainAfterExit=yes

[Install]
WantedBy=sleep.target
```

**Explaining every line:**

`[Unit]` section — metadata and ordering:
- `Description=` — Human-readable label. Shows up in `systemctl status` output.
- `Before=sleep.target` — This is a **dependency ordering directive**. It tells systemd: "run this service BEFORE the sleep.target unit activates." `sleep.target` is what systemd activates when the system suspends.

`[Service]` section — what to actually run:
- `Type=oneshot` — The service runs a command, it finishes, and that's it. Unlike `Type=simple` (which expects a long-running process), `oneshot` is for commands that run and exit. Perfect for a one-time action like blocking Bluetooth.
- `ExecStart=/usr/sbin/rfkill block bluetooth` — The command to run when the service **starts** (i.e., before sleep). `rfkill` is the kernel's radio frequency kill switch tool. `block bluetooth` tells the kernel to disable the Bluetooth radio.
- `ExecStop=/usr/sbin/rfkill unblock bluetooth` — The command to run when the service **stops** (i.e., after waking from sleep). This re-enables Bluetooth on wake.
- `RemainAfterExit=yes` — After `ExecStart` finishes, keep the service in "active" state. This is required so that `ExecStop` gets called when the system wakes up. Without this, systemd would consider the service "dead" after the command exits and never call `ExecStop`.

`[Install]` section — how to enable it:
- `WantedBy=sleep.target` — When you run `systemctl enable`, this creates a symlink in `sleep.target.wants/` pointing to this service. That's how systemd knows to activate it before sleeping.

#### Why /usr/sbin/rfkill and not /usr/bin/rfkill?

On this system, `rfkill` lives at `/usr/sbin/rfkill`. You can always find the correct path with:
```bash
which rfkill
```

`/usr/sbin/` traditionally holds system administration binaries (meant for root). `/usr/bin/` holds general user binaries. Modern Linux blurs this distinction but the physical location still matters when hardcoding paths in service files.

#### Enabling and starting the service

```bash
# Tell systemd to load and register this service
sudo systemctl daemon-reload
# Why: systemd caches unit files in memory. After creating/editing a unit file,
# you must reload the cache or systemd won't see your changes.

# Enable the service (creates the symlink so it runs automatically)
sudo systemctl enable bluetooth-suspend.service
# Why: "enable" ≠ "start". Enable means "activate this automatically at the right time."
# Start means "run it right now."

# Start it manually right now to test it
sudo systemctl start bluetooth-suspend.service

# Verify it worked
sudo systemctl status bluetooth-suspend.service
```

**What success looks like:**
```
Active: active (exited)
Process: ExecStart=/usr/sbin/rfkill block bluetooth (code=exited, status=0/SUCCESS)
```

`active (exited)` is correct for `Type=oneshot` — it ran, succeeded, and exited. `status=0` means the command returned exit code 0, which is Unix convention for "success."

**What failure looks like:**
```
Active: failed (Result: exit-code)
Process: ExecStart=/usr/bin/rfkill block bluetooth (code=exited, status=203/EXEC)
```

`status=203/EXEC` means systemd couldn't execute the binary — wrong path. Fix with:
```bash
sudo sed -i 's|/usr/bin/rfkill|/usr/sbin/rfkill|g' /etc/systemd/system/bluetooth-suspend.service
# sed -i = in-place edit (modify the file directly)
# 's|old|new|g' = substitute old with new, globally (all occurrences)
# The | delimiter instead of / avoids conflicts with paths that contain /
```

---

## Issue 2: Kernel Panic (Kernel 6.17 + Old Hardware)

### Root Cause

The system was running **Linux kernel 6.17**, which is a very new kernel. The laptop is a 2015 machine with an **Intel HD Graphics 5500 (Broadwell architecture)**. The i915 GPU driver in kernel 6.17 had compatibility issues with this old hardware, causing a fatal interrupt exception — a crash so severe the kernel couldn't log it before dying.

Evidence:
```bash
uname -r
# 6.17.0-14-generic  ← too new for this hardware

lspci | grep VGA
# Intel Corporation HD Graphics 5500 (rev 09)  ← 2015 Broadwell GPU
```

Also suspicious:
```
[drm] ACPI BIOS requests an excessive sleep of 5000 ms, using 1500 ms instead
ACPI: [Firmware Bug]: BIOS _OSI(Linux) query ignored
MDS CPU bug present and SMT on, data leak possible.
```

These show the kernel is already fighting with the BIOS over power management. Old BIOS firmware + aggressively new kernel = instability.

### The Fix: Downgrade to Kernel 6.14

Kernel 6.14 was already installed on the system. The goal is to boot into it and lock it as the permanent default.

---

### Step 1: Make GRUB Visible

GRUB is the bootloader — it's what lets you choose which kernel to boot. By default, Ubuntu hides GRUB and boots immediately. We need to make it visible.

```bash
sudo nano /etc/default/grub
```

Change these two lines:
```
GRUB_TIMEOUT_STYLE=hidden   →   GRUB_TIMEOUT_STYLE=menu
GRUB_TIMEOUT=0              →   GRUB_TIMEOUT=30
```

**Why these settings:**
- `GRUB_TIMEOUT_STYLE=hidden` means GRUB waits silently with no menu shown. Changing to `menu` makes it display the boot options.
- `GRUB_TIMEOUT=0` means wait 0 seconds (boot immediately). `30` gives you 30 seconds to make a selection.

Then apply the changes:
```bash
sudo update-grub
# This regenerates /boot/grub/grub.cfg from your settings in /etc/default/grub
# Never edit grub.cfg directly — it gets overwritten
```

---

### Step 2: Boot into 6.14

On the next reboot, when GRUB appears:
1. Press **down arrow** immediately to stop the countdown
2. Select **"Advanced options for Ubuntu"** → Enter
3. Select **"Ubuntu, with Linux 6.14.0-37-generic"** → Enter

Verify after boot:
```bash
uname -r
# Should output: 6.14.0-37-generic
```

`uname -r` prints the running kernel's release version. `-r` = release. You can also use `uname -a` for full system info.

---

### Step 3: Hold the Kernel (Prevent Auto-Updates)

Ubuntu's package manager (`apt`) will automatically upgrade your kernel unless you tell it not to. `apt-mark hold` pins a package at its current version.

```bash
sudo apt-mark hold linux-image-6.17.0-14-generic linux-image-generic-hwe-24.04
```

**Why two packages?**
- `linux-image-6.17.0-14-generic` — the actual kernel image file
- `linux-image-generic-hwe-24.04` — the HWE (Hardware Enablement) meta-package. This is a "virtual" package that always points to the latest kernel. If you only hold the kernel image but not this, `apt` will install a new kernel image next time you run `apt upgrade`. Holding both cuts off both paths.

Verify:
```bash
apt-mark showhold
# Should list both packages
```

---

### Step 4: Set 6.14 as Permanent Default

So you don't have to manually select it from GRUB every reboot:

```bash
sudo nano /etc/default/grub
```

Change:
```
GRUB_DEFAULT=0
```
To:
```
GRUB_DEFAULT="Advanced options for Ubuntu>Ubuntu, with Linux 6.14.0-37-generic"
```

**Why the full string?** `GRUB_DEFAULT=0` means "boot the first entry in the menu." But after a kernel update, entry 0 might change. Using the exact menu path string is deterministic — it always boots that specific kernel regardless of menu order.

Then:
```bash
sudo update-grub
```

---

## Noise vs. Signal: How to Read Logs Without Panicking

Not every error in `journalctl` is a real problem. Learn to distinguish:

| Log Message | Real Problem? | Why |
|---|---|---|
| `VMX disabled by BIOS` | No | Just says virtualization is off in BIOS. Harmless unless you need VMs. |
| `spi-nor probe failed with error -95` | No | A flash chip driver that doesn't match your hardware. Kernel skips it gracefully. |
| `bluetoothd: SAP driver failed` | No | SIM Access Profile needs root. Harmless cosmetic error on every boot. |
| `gdm3: assertion 'GDM_IS_REMOTE_DISPLAY' failed` | No | GDM display manager quirk. Doesn't affect normal use. |
| `bluetoothd: No reply to Suspend request` | **YES** | This caused the desktop freeze. |
| `Fatal exception in interrupt` | **YES** | This caused the kernel panic. |
| `rmi4_physical: Failed to read irqs, code=-6` | Mild | Touchpad briefly fails on wake. Usually self-resolves in seconds. |

**The pattern:** Look for errors that appear right before the crash timestamp. Errors that appear at boot every time and the system works fine afterward are almost always harmless driver/firmware mismatches.

---

## Debugging Methodology (The Mental Framework)

This is how you approach any Linux stability issue:

```
1. OBSERVE   → What exactly happened? (freeze? panic? slowdown?)
2. LOCATE    → Which layer failed? (kernel? service? app?)
3. READ LOGS → journalctl -b -1, dmesg, /var/log/syslog
4. HYPOTHESIZE → What could cause this specific error?
5. TEST      → Apply one fix at a time
6. VERIFY    → Confirm with logs and real usage
7. DOCUMENT  → Write it down (this KB is that step)
```

Never apply multiple fixes at once. If you change three things and the problem goes away, you don't know which one fixed it — or if one of them introduced a new problem.

---

## Quick Reference: Commands Used in This Incident

```bash
# Read errors from previous boot session
journalctl -b -1 --priority=3

# Read errors from current boot session
journalctl -b 0 --priority=3

# Read kernel messages (requires sudo on Ubuntu 24.04)
sudo dmesg | tail -50

# Filter kernel messages for specific keywords
sudo dmesg | grep -iE "panic|i915|drm|gpu|interrupt|oops" | tail -40

# Check currently running kernel version
uname -r

# List all installed kernel images
dpkg --list | grep linux-image

# Pin packages to prevent auto-upgrade
sudo apt-mark hold <package-name>

# Show all held packages
apt-mark showhold# Linux Stability KB: Diagnosing and Fixing Kernel Panics & Desktop Freezes

**Hardware:** HP EliteBook 840 G2 | **OS:** Ubuntu 24.04 LTS | **Date:** February 2026  
**Author:** akaijoe | **Status:** Resolved ✅

---

## What Happened (The Problem)

Two separate but related issues occurred:

1. **Desktop freeze** — The screen locked up completely. Mouse and keyboard stopped responding. Could not click anything or switch windows.
2. **Kernel panic** — The screen went black and showed:
   ```
   KERNEL PANIC!
   Please reboot your computer.
   Fatal exception in interrupt
   ```

These are different failure modes. Understanding the difference matters:

- A **desktop freeze** means the Linux kernel is still alive, but the graphical session (GNOME/GDM) is deadlocked. The kernel can sometimes be reached via keyboard shortcuts.
- A **kernel panic** means the kernel itself hit an unrecoverable error and halted. Nothing works. Hard reboot is the only option. This is Linux's equivalent of Windows BSOD.

---

## Mental Model: How Linux Boots and Who Controls What

```
BIOS/UEFI → GRUB (bootloader) → Linux Kernel → systemd → Desktop (GNOME/GDM)
```

- **GRUB** is the boot menu. It lets you choose which kernel to boot.
- **The kernel** is the core of the OS. It talks to hardware directly (CPU, GPU, RAM, storage).
- **systemd** is the init system — it starts all services after the kernel loads.
- **GNOME/GDM** is the graphical desktop. It runs on top of all of this.

When something breaks, the layer that broke determines your recovery options.

---

## Issue 1: Desktop Freeze (Bluetooth Suspend Deadlock)

### Root Cause

When the laptop tried to **suspend (sleep)**, it sent a signal to the Bluetooth daemon asking it to pause. The Bluetooth daemon never responded. The kernel waited indefinitely. The entire desktop locked up.

Evidence from logs:
```
Feb 18 19:00:38 bluetoothd: profiles/audio/avdtp.c:handle_unanswered_req() No reply to Suspend request
```
This line says: "I sent a Suspend request to a Bluetooth audio profile, and it never replied."

### Emergency Recovery: Magic SysRq

If this happens again and the desktop is frozen, use the kernel's emergency keyboard shortcut:

```
Hold: Alt + PrintScreen
Then type one at a time: R  E  I  S  U  B
```

Remember as: **"Reboot Even If System Utterly Broken"**

Each key directly commands the kernel, bypassing the frozen desktop:
- `R` — Takes keyboard control back from X11 (the display server)
- `E` — Sends SIGTERM to all processes (graceful shutdown signal)
- `I` — Sends SIGKILL to all processes (force kill)
- `S` — Syncs all filesystems to disk (prevents data corruption)
- `U` — Remounts filesystems as read-only (safe state)
- `B` — Reboots

---

### How to Read the Logs

#### Command 1: Read errors from the previous boot
```bash
journalctl -b -1 --priority=3
```

**Breaking this down:**
- `journalctl` — The tool that reads systemd's journal (log database)
- `-b -1` — "boot minus 1" = the boot session before the current one. `-b 0` = current boot, `-b -1` = last boot, `-b -2` = two boots ago
- `--priority=3` — Only show messages at severity level 3 (errors) or worse. Linux uses levels 0-7: 0=emergency, 3=error, 7=debug

#### Command 2: Read kernel ring buffer
```bash
sudo dmesg | tail -50
# dmesg without sudo is blocked on modern Ubuntu for security reasons
```

**Breaking this down:**
- `dmesg` — Reads the kernel ring buffer. This is a circular memory buffer the kernel writes hardware and driver messages to during and after boot
- `sudo` — Required because Ubuntu 24.04 restricts raw kernel buffer access to root (security hardening added in kernel 4.15+)
- `| tail -50` — Pipe the output to `tail`, which shows only the last 50 lines

---

### The Fix: A systemd Service to Block Bluetooth Before Sleep

The solution is to tell the system: **before you go to sleep, shut off Bluetooth first**. We do this with a custom systemd service unit.

#### What is a systemd service unit?

A service unit is a configuration file that tells systemd:
- When to run something (before sleep? at boot? on a timer?)
- What to run (which command/binary)
- What to do on success or failure

Service files live in `/etc/systemd/system/` for custom/admin-created services.

#### Creating the service

```bash
sudo nano /etc/systemd/system/bluetooth-suspend.service
```

**Why this path?** `/etc/systemd/system/` is where system administrators put custom service units. Units here override anything in `/lib/systemd/system/` (where packages install their units). It survives package updates.

Contents of the file:

```ini
[Unit]
Description=Disable Bluetooth before suspend
Before=sleep.target

[Service]
Type=oneshot
ExecStart=/usr/sbin/rfkill block bluetooth
ExecStop=/usr/sbin/rfkill unblock bluetooth
RemainAfterExit=yes

[Install]
WantedBy=sleep.target
```

**Explaining every line:**

`[Unit]` section — metadata and ordering:
- `Description=` — Human-readable label. Shows up in `systemctl status` output.
- `Before=sleep.target` — This is a **dependency ordering directive**. It tells systemd: "run this service BEFORE the sleep.target unit activates." `sleep.target` is what systemd activates when the system suspends.

`[Service]` section — what to actually run:
- `Type=oneshot` — The service runs a command, it finishes, and that's it. Unlike `Type=simple` (which expects a long-running process), `oneshot` is for commands that run and exit. Perfect for a one-time action like blocking Bluetooth.
- `ExecStart=/usr/sbin/rfkill block bluetooth` — The command to run when the service **starts** (i.e., before sleep). `rfkill` is the kernel's radio frequency kill switch tool. `block bluetooth` tells the kernel to disable the Bluetooth radio.
- `ExecStop=/usr/sbin/rfkill unblock bluetooth` — The command to run when the service **stops** (i.e., after waking from sleep). This re-enables Bluetooth on wake.
- `RemainAfterExit=yes` — After `ExecStart` finishes, keep the service in "active" state. This is required so that `ExecStop` gets called when the system wakes up. Without this, systemd would consider the service "dead" after the command exits and never call `ExecStop`.

`[Install]` section — how to enable it:
- `WantedBy=sleep.target` — When you run `systemctl enable`, this creates a symlink in `sleep.target.wants/` pointing to this service. That's how systemd knows to activate it before sleeping.

#### Why /usr/sbin/rfkill and not /usr/bin/rfkill?

On this system, `rfkill` lives at `/usr/sbin/rfkill`. You can always find the correct path with:
```bash
which rfkill
```

`/usr/sbin/` traditionally holds system administration binaries (meant for root). `/usr/bin/` holds general user binaries. Modern Linux blurs this distinction but the physical location still matters when hardcoding paths in service files.

#### Enabling and starting the service

```bash
# Tell systemd to load and register this service
sudo systemctl daemon-reload
# Why: systemd caches unit files in memory. After creating/editing a unit file,
# you must reload the cache or systemd won't see your changes.

# Enable the service (creates the symlink so it runs automatically)
sudo systemctl enable bluetooth-suspend.service
# Why: "enable" ≠ "start". Enable means "activate this automatically at the right time."
# Start means "run it right now."

# Start it manually right now to test it
sudo systemctl start bluetooth-suspend.service

# Verify it worked
sudo systemctl status bluetooth-suspend.service
```

**What success looks like:**
```
Active: active (exited)
Process: ExecStart=/usr/sbin/rfkill block bluetooth (code=exited, status=0/SUCCESS)
```

`active (exited)` is correct for `Type=oneshot` — it ran, succeeded, and exited. `status=0` means the command returned exit code 0, which is Unix convention for "success."

**What failure looks like:**
```
Active: failed (Result: exit-code)
Process: ExecStart=/usr/bin/rfkill block bluetooth (code=exited, status=203/EXEC)
```

`status=203/EXEC` means systemd couldn't execute the binary — wrong path. Fix with:
```bash
sudo sed -i 's|/usr/bin/rfkill|/usr/sbin/rfkill|g' /etc/systemd/system/bluetooth-suspend.service
# sed -i = in-place edit (modify the file directly)
# 's|old|new|g' = substitute old with new, globally (all occurrences)
# The | delimiter instead of / avoids conflicts with paths that contain /
```

---

## Issue 2: Kernel Panic (Kernel 6.17 + Old Hardware)

### Root Cause

The system was running **Linux kernel 6.17**, which is a very new kernel. The laptop is a 2015 machine with an **Intel HD Graphics 5500 (Broadwell architecture)**. The i915 GPU driver in kernel 6.17 had compatibility issues with this old hardware, causing a fatal interrupt exception — a crash so severe the kernel couldn't log it before dying.

Evidence:
```bash
uname -r
# 6.17.0-14-generic  ← too new for this hardware

lspci | grep VGA
# Intel Corporation HD Graphics 5500 (rev 09)  ← 2015 Broadwell GPU
```

Also suspicious:
```
[drm] ACPI BIOS requests an excessive sleep of 5000 ms, using 1500 ms instead
ACPI: [Firmware Bug]: BIOS _OSI(Linux) query ignored
MDS CPU bug present and SMT on, data leak possible.
```

These show the kernel is already fighting with the BIOS over power management. Old BIOS firmware + aggressively new kernel = instability.

### The Fix: Downgrade to Kernel 6.14

Kernel 6.14 was already installed on the system. The goal is to boot into it and lock it as the permanent default.

---

### Step 1: Make GRUB Visible

GRUB is the bootloader — it's what lets you choose which kernel to boot. By default, Ubuntu hides GRUB and boots immediately. We need to make it visible.

```bash
sudo nano /etc/default/grub
```

Change these two lines:
```
GRUB_TIMEOUT_STYLE=hidden   →   GRUB_TIMEOUT_STYLE=menu
GRUB_TIMEOUT=0              →   GRUB_TIMEOUT=30
```

**Why these settings:**
- `GRUB_TIMEOUT_STYLE=hidden` means GRUB waits silently with no menu shown. Changing to `menu` makes it display the boot options.
- `GRUB_TIMEOUT=0` means wait 0 seconds (boot immediately). `30` gives you 30 seconds to make a selection.

Then apply the changes:
```bash
sudo update-grub
# This regenerates /boot/grub/grub.cfg from your settings in /etc/default/grub
# Never edit grub.cfg directly — it gets overwritten
```

---

### Step 2: Boot into 6.14

On the next reboot, when GRUB appears:
1. Press **down arrow** immediately to stop the countdown
2. Select **"Advanced options for Ubuntu"** → Enter
3. Select **"Ubuntu, with Linux 6.14.0-37-generic"** → Enter

Verify after boot:
```bash
uname -r
# Should output: 6.14.0-37-generic
```

`uname -r` prints the running kernel's release version. `-r` = release. You can also use `uname -a` for full system info.

---

### Step 3: Hold the Kernel (Prevent Auto-Updates)

Ubuntu's package manager (`apt`) will automatically upgrade your kernel unless you tell it not to. `apt-mark hold` pins a package at its current version.

```bash
sudo apt-mark hold linux-image-6.17.0-14-generic linux-image-generic-hwe-24.04
```

**Why two packages?**
- `linux-image-6.17.0-14-generic` — the actual kernel image file
- `linux-image-generic-hwe-24.04` — the HWE (Hardware Enablement) meta-package. This is a "virtual" package that always points to the latest kernel. If you only hold the kernel image but not this, `apt` will install a new kernel image next time you run `apt upgrade`. Holding both cuts off both paths.

Verify:
```bash
apt-mark showhold
# Should list both packages
```

---

### Step 4: Set 6.14 as Permanent Default

So you don't have to manually select it from GRUB every reboot:

```bash
sudo nano /etc/default/grub
```

Change:
```
GRUB_DEFAULT=0
```
To:
```
GRUB_DEFAULT="Advanced options for Ubuntu>Ubuntu, with Linux 6.14.0-37-generic"
```

**Why the full string?** `GRUB_DEFAULT=0` means "boot the first entry in the menu." But after a kernel update, entry 0 might change. Using the exact menu path string is deterministic — it always boots that specific kernel regardless of menu order.

Then:
```bash
sudo update-grub
```

---

## Noise vs. Signal: How to Read Logs Without Panicking

Not every error in `journalctl` is a real problem. Learn to distinguish:

| Log Message | Real Problem? | Why |
|---|---|---|
| `VMX disabled by BIOS` | No | Just says virtualization is off in BIOS. Harmless unless you need VMs. |
| `spi-nor probe failed with error -95` | No | A flash chip driver that doesn't match your hardware. Kernel skips it gracefully. |
| `bluetoothd: SAP driver failed` | No | SIM Access Profile needs root. Harmless cosmetic error on every boot. |
| `gdm3: assertion 'GDM_IS_REMOTE_DISPLAY' failed` | No | GDM display manager quirk. Doesn't affect normal use. |
| `bluetoothd: No reply to Suspend request` | **YES** | This caused the desktop freeze. |
| `Fatal exception in interrupt` | **YES** | This caused the kernel panic. |
| `rmi4_physical: Failed to read irqs, code=-6` | Mild | Touchpad briefly fails on wake. Usually self-resolves in seconds. |

**The pattern:** Look for errors that appear right before the crash timestamp. Errors that appear at boot every time and the system works fine afterward are almost always harmless driver/firmware mismatches.

---

## Debugging Methodology (The Mental Framework)

This is how you approach any Linux stability issue:

```
1. OBSERVE   → What exactly happened? (freeze? panic? slowdown?)
2. LOCATE    → Which layer failed? (kernel? service? app?)
3. READ LOGS → journalctl -b -1, dmesg, /var/log/syslog
4. HYPOTHESIZE → What could cause this specific error?
5. TEST      → Apply one fix at a time
6. VERIFY    → Confirm with logs and real usage
7. DOCUMENT  → Write it down (this KB is that step)
```

Never apply multiple fixes at once. If you change three things and the problem goes away, you don't know which one fixed it — or if one of them introduced a new problem.

---

## Quick Reference: Commands Used in This Incident

```bash
# Read errors from previous boot session
journalctl -b -1 --priority=3

# Read errors from current boot session
journalctl -b 0 --priority=3

# Read kernel messages (requires sudo on Ubuntu 24.04)
sudo dmesg | tail -50

# Filter kernel messages for specific keywords
sudo dmesg | grep -iE "panic|i915|drm|gpu|interrupt|oops" | tail -40

# Check currently running kernel version
uname -r

# List all installed kernel images
dpkg --list | grep linux-image

# Pin packages to prevent auto-upgrade
sudo apt-mark hold <package-name>

# Show all held packages
apt-mark showhold

# Reload systemd after editing unit files
sudo systemctl daemon-reload

# Enable + start a service
sudo systemctl enable <service>
sudo systemctl start <service>
sudo systemctl status <service>

# Find the real path of a binary
which rfkill

# Regenerate GRUB config after editing /etc/default/grub
sudo update-grub
```

---

## Key Concepts to Study Next

- **systemd unit files** — Learn `Type=simple` vs `Type=oneshot` vs `Type=forking`. Understand `Wants=`, `Requires=`, `Before=`, `After=` ordering.
- **Linux kernel architecture** — How the kernel manages hardware through drivers and interrupts. What an interrupt handler is and why a fatal exception in one crashes the whole kernel.
- **GRUB internals** — How `/etc/default/grub` → `update-grub` → `/boot/grub/grub.cfg` works. What `grub-install` does vs `update-grub`.
- **rfkill** — The kernel's unified interface for managing radio devices (WiFi, Bluetooth, NFC). Read `man rfkill`.
- **journald** — How systemd's logging system works. Binary journal format, priorities, boot IDs. Compare with traditional syslog.

---

*Incident resolved February 19, 2026. System stable on kernel 6.14.0-37-generic.*

# Reload systemd after editing unit files
sudo systemctl daemon-reload

# Enable + start a service
sudo systemctl enable <service>
sudo systemctl start <service>
sudo systemctl status <service>

# Find the real path of a binary
which rfkill

# Regenerate GRUB config after editing /etc/default/grub
sudo update-grub
```

---

## Key Concepts to Study Next

- **systemd unit files** — Learn `Type=simple` vs `Type=oneshot` vs `Type=forking`. Understand `Wants=`, `Requires=`, `Before=`, `After=` ordering.
- **Linux kernel architecture** — How the kernel manages hardware through drivers and interrupts. What an interrupt handler is and why a fatal exception in one crashes the whole kernel.
- **GRUB internals** — How `/etc/default/grub` → `update-grub` → `/boot/grub/grub.cfg` works. What `grub-install` does vs `update-grub`.
- **rfkill** — The kernel's unified interface for managing radio devices (WiFi, Bluetooth, NFC). Read `man rfkill`.
- **journald** — How systemd's logging system works. Binary journal format, priorities, boot IDs. Compare with traditional syslog.

---

*Incident resolved February 19, 2026. System stable on kernel 6.14.0-37-generic.*
