# APT/DKMS Repair - Broadcom STA on Linux 6.17

Date: 2026-06-23
Node: `intel-mbp` / Linux Mint 22.3 `zena` / Ubuntu base `noble`
Hardware: Broadcom BCM4331 Wi-Fi `[14e4:4331]` on Intel MacBook Pro
Packages: `broadcom-sta-dkms 6.30.223.271-23ubuntu1.1`, `bcmwl-kernel-source 6.30.223.271+bdcom-23ubuntu1.1`, `linux-generic-hwe-24.04 6.17.0-35.35~24.04.1`

## Symptom

`apt upgrade` left six kernel packages half-configured:

- `linux-headers-6.17.0-29-generic`
- `linux-headers-6.17.0-35-generic`
- `linux-headers-generic-hwe-24.04`
- `linux-generic-hwe-24.04`
- `linux-image-6.17.0-29-generic`
- `linux-image-6.17.0-35-generic`

The actual failure was DKMS, not APT dependency solving. `broadcom-sta-dkms` could not build `wl.ko` for Linux `6.17.0-*`, causing kernel postinst hooks to fail.

Initial DKMS error:

```text
src/shared/linux_osl.c:23:10: fatal error: typedefs.h: No such file or directory
```

After restoring include propagation, additional Linux 6.17 compatibility errors appeared:

```text
implicit declaration of function 'from_timer'
implicit declaration of function 'del_timer'
incompatible pointer type for cfg80211_ops.set_wiphy_params
incompatible pointer type for cfg80211_ops.set_tx_power
incompatible pointer type for cfg80211_ops.get_tx_power
modpost unresolved Broadcom blob symbols
```

## Local Patch Applied

Patched the DKMS source tree at:

- `/usr/src/broadcom-sta-6.30.223.271/Makefile`
- `/usr/src/broadcom-sta-6.30.223.271/src/wl/sys/wl_linux.c`
- `/usr/src/broadcom-sta-6.30.223.271/src/wl/sys/wl_cfg80211_hybrid.c`

Backups were left beside the patched files as `*.codex-backup-*`.

Patch classes:

1. Add current kbuild flag bridges so nested objects receive Broadcom include flags:

```make
ccflags-y          += $(EXTRA_CFLAGS)
subdir-ccflags-y   += $(EXTRA_CFLAGS)
```

2. Add current kbuild linker flag for the proprietary Broadcom blob:

```make
ldflags-y           += $(src)/lib/$(SHIPPED)
```

3. Add Linux `>= 6.17` timer compatibility in `wl_linux.c`:

```c
#define from_timer(var, callback_timer, timer_fieldname) timer_container_of(var, callback_timer, timer_fieldname)
#define del_timer timer_delete
```

4. Add Linux `>= 6.17` `cfg80211_ops` callback signatures for the new `radio_idx` / `link_id` parameters.

## Repair Commands Run

```bash
dkms build -m broadcom-sta -v 6.30.223.271 -k 6.17.0-35-generic
dkms install -m broadcom-sta -v 6.30.223.271 -k 6.17.0-35-generic

dkms build -m broadcom-sta -v 6.30.223.271 -k 6.17.0-29-generic
dkms install -m broadcom-sta -v 6.30.223.271 -k 6.17.0-29-generic

dpkg --configure -a
apt-get -f install -y
apt-get upgrade -y
```

## Verification

Post-repair checks:

```bash
apt update
apt-get -s upgrade
dpkg --audit
apt-get check
dkms status -m broadcom-sta -v 6.30.223.271
```

Current state:

- `apt update` exits 0.
- `apt-get -s upgrade` shows no DKMS/kernel failures; only phased `kpartx` deferrals.
- `dpkg --audit` is empty.
- `apt-get check` is clean.
- `broadcom-sta` DKMS is installed for `6.17.0-29-generic` and `6.17.0-35-generic`.

Expected DKMS status includes:

```text
broadcom-sta/6.30.223.271, 6.17.0-29-generic, x86_64: installed
broadcom-sta/6.30.223.271, 6.17.0-35-generic, x86_64: installed
```

## Maintenance / Patch Continuum

`apt update` is safe: it only refreshes package metadata and currently exits cleanly.

`apt upgrade` is safe for the current package set: the broken kernel packages are configured, and the Broadcom module builds for the installed Linux `6.17.0-29` and `6.17.0-35` kernels.

There is still a patch continuum to maintain:

- Future `6.17.0-*` kernel updates should normally work because DKMS will reuse the patched `/usr/src/broadcom-sta-6.30.223.271` source.
- A future `broadcom-sta-dkms` package upgrade can overwrite `/usr/src/broadcom-sta-6.30.223.271`; re-check this runbook if DKMS fails again.
- A future kernel ABI/API change beyond 6.17 may require additional source compatibility fixes.
- The real upstream fix belongs in Ubuntu's `broadcom-sta` / `bcmwl` packaging; Mint inherits the package from Ubuntu restricted.

## Upstream State

Ubuntu already has `broadcom-sta-dkms 6.30.223.271-23ubuntu1.2` in `noble-proposed`.

Relevant changelog entry:

```text
broadcom-sta (6.30.223.271-23ubuntu1.2) noble; urgency=medium
  * Fix FTBFS against kernel 6.17 (LP: #2120508)
  * Failed to build against linux-6.17
    - 42-broadcom-wl-fix-linux-6.17.patch
```

That package is the cleaner long-term answer than carrying local edits in `/usr/src`. The Mint-side ask is not "please invent a patch"; it is "please ensure Linux Mint 22.x users on Ubuntu noble HWE 6.17 can receive the Ubuntu `broadcom-sta-dkms` 1.2 fix without enabling all of `noble-proposed`."

Until that package lands in the normal update path, keep this local patch note.

Potential one-package escape hatch, if we choose to replace the local patched source with the Ubuntu proposed package:

```bash
cd /tmp
wget https://archive.ubuntu.com/ubuntu/pool/restricted/b/broadcom-sta/broadcom-sta-dkms_6.30.223.271-23ubuntu1.2_all.deb
sudo apt install ./broadcom-sta-dkms_6.30.223.271-23ubuntu1.2_all.deb
sudo dpkg --configure -a
sudo apt-get -f install
dkms status -m broadcom-sta -v 6.30.223.271
```

Do not enable the whole `noble-proposed` pocket globally for this node unless there is a separate reason to test proposed updates.

## Mint / Ubuntu Report Draft

Title:

```text
Linux Mint 22.3 on Intel MacBook Pro: broadcom-sta-dkms 23ubuntu1.1 fails against HWE 6.17; Ubuntu noble-proposed 23ubuntu1.2 fixes it
```

Body:

```text
Linux Mint 22.3 "zena" on an Intel MacBook Pro with Broadcom BCM4331 Wi-Fi [14e4:4331] currently receives broadcom-sta-dkms 6.30.223.271-23ubuntu1.1 from noble-updates/restricted while the HWE kernel is 6.17.0-35-generic.

Result: apt upgrade leaves linux-image/linux-headers/linux-generic-hwe packages half-configured because DKMS cannot build wl.ko.

Primary make.log failure:
src/shared/linux_osl.c:23:10: fatal error: typedefs.h: No such file or directory

After fixing include propagation locally, the same source also needs Linux 6.17 compatibility for timer_container_of/timer_delete, cfg80211_ops radio_idx/link_id callback signatures, and current kbuild linker flags for the proprietary Broadcom blob.

Ubuntu already has the intended fix in noble-proposed:
broadcom-sta-dkms 6.30.223.271-23ubuntu1.2
Changelog: "Fix FTBFS against kernel 6.17 (LP: #2120508)" and patch 42-broadcom-wl-fix-linux-6.17.patch.

Request: please make this fixed Broadcom STA DKMS package available to Mint 22.x users through the normal Mint/Ubuntu update path, or document a one-package install path that does not require enabling all of noble-proposed.

Local verification after applying equivalent patches:
- apt update exits 0
- dpkg --audit is empty
- apt-get -s upgrade no longer reports DKMS/kernel failures
- dkms status includes broadcom-sta installed for 6.17.0-29-generic and 6.17.0-35-generic
```

Before rebooting into a newly installed kernel, verify:

```bash
dkms status -m broadcom-sta -v 6.30.223.271
find /lib/modules/6.17.0-35-generic/updates/dkms -name 'wl.ko*' -ls
```

After reboot, verify Wi-Fi:

```bash
uname -r
lsmod | grep '^wl'
nmcli device status
```
