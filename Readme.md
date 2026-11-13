<div align="center">

# 🔥 BLAZE GKI Kernel Builder

### Custom **android12-5.10** GKI kernel, built in GitHub Actions

**KernelSU-Next · SUSFS · NoMount · Baseband Guard · BBRv3 · MGLRU · NTSync · Droidspaces**

![Kernel](https://img.shields.io/badge/kernel-android12--5.10%20(GKI%20LTS)-3DDC84?style=for-the-badge&logo=android&logoColor=white)
![Root](https://img.shields.io/badge/root-KernelSU--Next-8A2BE2?style=for-the-badge)
![SUSFS](https://img.shields.io/badge/SUSFS-enabled-orange?style=for-the-badge)
![CI](https://img.shields.io/badge/CI-GitHub%20Actions-2088FF?style=for-the-badge&logo=githubactions&logoColor=white)
![Output](https://img.shields.io/badge/output-AnyKernel3-informational?style=for-the-badge)

*Run one workflow, get one flashable kernel zip.*

</div>

---

## 📑 Table of Contents

- [Overview](#-overview)
- [Features](#-features)
- [Quick Start](#-quick-start)
- [Workflow Inputs](#-workflow-inputs)
- [Build Pipeline](#-build-pipeline)
- [Build Output](#-build-output)
- [Flashing Guide](#-flashing-guide)
- [Customization](#-customization)
- [Troubleshooting](#-troubleshooting)
- [Limitations](#-limitations)
- [Credits](#-credits)
- [Disclaimer](#-disclaimer)

---

## 🧭 Overview

This repository holds one GitHub Actions workflow, `build.yml`. It syncs the official AOSP **android12-5.10** GKI source, applies root, hiding, networking and container patches, compiles the kernel with **LTO=thin**, and packages the result as a flashable **AnyKernel3** zip.

Nothing needs to be installed locally. The build runs on an `ubuntu-22.04` runner with a 150-minute timeout and uses **ccache** to speed up repeat builds.

---

## ✨ Features

| Category | Feature | Notes |
|---|---|---|
| 🔐 **Root** | **KernelSU-Next** | Branch selectable, default `dev-susfs`, with an automatic fix patch for `selinux_hide.c` |
| 🕶️ **Hiding** | **SUSFS** | Path, mount, kstat, uname and cmdline spoofing, open redirect, map hiding, symbol hiding |
| 📂 **Mounts** | **NoMount** (built-in) | `CONFIG_NOMOUNT=y`, toggleable, verified in the final `.config` |
| 🛡️ **Security** | **Baseband Guard (BBG)** | Registered as an LSM, toggleable |
| 🚀 **Network** | **BBRv3** backport | Falls back to BBR v1 automatically if the patch cannot be fixed |
| 🌐 **Qdiscs** | FQ, FQ-CoDel, CAKE | Plus CUBIC, BIC, Westwood and H-TCP; default congestion control is `bbr` |
| 🧠 **Memory** | **MGLRU** | `CONFIG_LRU_GEN=y`, enabled by default |
| ⚡ **Tuning** | Daily-use optimizations | Faster freeze timeout, F2FS urgent GC, ext4/jbd2 commit interval, lower swappiness, lower VFS cache pressure |
| 🔧 **Compat** | **NTSync** | Kernel patches for Wine/Proton sync primitives |
| 🐧 **Containers** | **Droidspaces** | kABI-safe SysV IPC and POSIX mqueue patches, plus namespace, cgroup and netfilter configs |
| 🔬 **Tracing** | **eBPF** + KProbes | BPF JIT, cgroup BPF, BPF events, classifier and action support |
| 🔤 **Filesystems** | Unicode and NLS | UTF-8, CP437, ISO8859-1 |
| 📦 **Packaging** | **AnyKernel3** zip | Name includes the SUSFS commit for traceability |

---

## 🚀 Quick Start

1. **Fork** this repository, or copy `build.yml` into `.github/workflows/`.
2. Open the **Actions** tab and choose **Build Custom GKI Kernel (KernelSU-Next + SUSFS + BBRv3)**.
3. Click **Run workflow**, set the [inputs](#-workflow-inputs), and start the run.
4. When it finishes, download the artifact from the run summary page.
5. Flash it by following the [Flashing Guide](#-flashing-guide).

> 💡 The first run is the slowest. Later runs reuse the ccache.

---

## ⚙️ Workflow Inputs

| Input | Description | Default |
|---|---|---|
| `os_patch_level` | Kernel branch tag. `lts` tracks the rolling LTS branch | `lts` |
| `susfs_commit` | SUSFS commit hash. Empty means the latest on `gki-android12-5.10` | *(empty)* |
| `ksun_branch` | KernelSU-Next branch passed to `setup.sh` | `dev-susfs` |
| `device_name` | Label used in the zip and artifact name | `gki-custom` |
| `enable_bbg` | Enable Baseband Guard | `true` |
| `enable_nomount` | Build NoMount into the kernel | `true` |
| `nomount_branch` | `maxsteeel/nomount` branch used by its `setup.sh` | `dev` |

---

## 🏗️ Build Pipeline

```text
 ┌──────────────────────────────────────────────────────────────┐
 │ 1. Prepare     checkout · free disk · deps · repo · ccache   │
 │ 2. Sync        GKI android12-5.10 · detect SUBLEVEL          │
 │ 3. SUSFS       clone · compat fixes · patch · cleanup        │
 │ 4. Root        KernelSU-Next setup · static.patch fix        │
 │ 5. Features    NoMount · Baseband Guard · BBRv3 · TCP configs│
 │ 6. Tuning      daily-use optimizations · MGLRU               │
 │ 7. Extras      NTSync · Droidspaces patches and configs      │
 │ 8. Finalize    clean dirty flag · branding · disable checks  │
 │ 9. Build       build/build.sh with LTO=thin and ccache       │
 │10. Verify      NoMount present in final .config              │
 │11. Package     AnyKernel3 zip · upload artifact              │
 └──────────────────────────────────────────────────────────────┘
```

Where a safe fallback exists, a failing patch step degrades gracefully instead of killing the build. BBRv3 rolls back to BBR v1, and the NoMount integration falls back to a manual copy if its setup script fails. Critical steps such as SUSFS and the KernelSU-Next wiring fail loudly.

---

## 📦 Build Output

| Artifact | Purpose |
|---|---|
| `<device_name>-KSUNext-android12-5.10-susfs<commit>.zip` | Flashable **AnyKernel3** kernel package |

Artifacts are kept for **14 days**. Download them or attach them to a release before they expire.

---

## 📲 Flashing Guide

**Requirements:** unlocked bootloader, a custom recovery or a kernel flasher with AnyKernel3 support, and a **backup of your current boot image**.

1. Download the zip from the workflow artifacts and extract it if GitHub wrapped it.
2. Flash it with your recovery or kernel flasher.
3. Reboot.
4. Install the **KernelSU-Next manager** app and grant root as needed.

> ⚠️ Flash only a build that matches your **device's GKI generation and kernel version**. A mismatch can bootloop the device.

---

## 🛠️ Customization

- **Branding:** edit the *Apply custom kernel branding* step. The current suffix is `-BLAZE-KSUN-STABLE`.
- **Kernel options:** the config steps append `CONFIG_*` lines to `gki_defconfig`. Add or remove lines there.
- **Disable a feature:** use `enable_bbg` or `enable_nomount`, or delete the related step.
- **Reproducible builds:** pin `susfs_commit` and use a fixed branch instead of `lts`.
- **Tuning values:** the *Apply safe daily-use optimizations* step lists each default it changes. Edit the replacement values to taste.

> Config lines are dropped silently when the symbol does not exist or a dependency is missing. Check the final `.config` after adding options. The NoMount verification step does this for `CONFIG_NOMOUNT`.

---

## 🩺 Troubleshooting

| Symptom | Likely cause and fix |
|---|---|
| Steps pane looks black on GitHub mobile | UI glitch while the run starts. Refresh, use the desktop site, or open the raw logs |
| SUSFS patch fails to apply | Upstream or kernel sublevel changed. Pin `susfs_commit` to a known-good commit |
| `CONFIG_NOMOUNT=y` missing in final config | A Kconfig dependency dropped it. Check the verification step output |
| BBRv3 warning in logs | The patch had rejects. The workflow tries to repair it, then rolls back to BBR v1 |
| Droidspaces SysV IPC configs not enabled | The kABI patches did not apply, so the workflow skips those options on purpose |
| Bootloop after flashing | Wrong GKI generation for your device. Restore your boot backup |

---

## ⚠️ Limitations

- Targets **android12-5.10 GKI** only.
- Several steps pull patches from external repositories. If an upstream moves or changes, a step can break. Pin versions where you can.
- This workflow builds the kernel only. It does not include a root manager app, modules or userspace tools.

---

## 🙏 Credits

- [Android Open Source Project](https://android.googlesource.com/kernel/common): GKI common kernel
- [KernelSU-Next (pershoot fork)](https://github.com/pershoot/KernelSU-Next): root framework
- [simonpunk / susfs4ksu](https://gitlab.com/simonpunk/susfs4ksu): SUSFS
- [maxsteeel / nomount](https://github.com/maxsteeel/nomount): NoMount
- [vc-teahouse / Baseband-guard](https://github.com/vc-teahouse/Baseband-guard): Baseband Guard
- [WildKernels](https://github.com/WildKernels): BBRv3 and NTSync patches, AnyKernel3 template
- [ravindu644 / Droidspaces-OSS](https://github.com/ravindu644/Droidspaces-OSS): Droidspaces kABI patches
- [AnyKernel3](https://github.com/osm0sis/AnyKernel3): flashable packaging

---

## 📜 Disclaimer

This project is provided **as is**, with no warranty. Flashing a custom kernel can brick your device, void your warranty or cause data loss. You are responsible for what you flash.

The Linux kernel is licensed under **GPLv2**. If you distribute built binaries, make the corresponding source and your patches available.

---

<div align="center">

**Built with 🔥 by the BLAZE project**

*If this helped you, give the repo a ⭐*

</div>
