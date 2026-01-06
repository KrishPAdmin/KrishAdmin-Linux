# KrishAdmin-Linux

KrishAdmin Linux is a minimal, Debian-based desktop live ISO project built using Debian live-build. The long-term goal is to create a stable, lightweight desktop distribution with custom branding, a controlled package set, and eventually a custom Linux kernel as part of a learning-focused workflow (kernel development, debugging, and feature work).

This repo is built and tested from a Debian VM running on a Proxmox node, and the VM acts as the build factory for generating reproducible ISO images.

---

## Goals

### Primary goals
- Produce a minimal desktop ISO with no unnecessary default apps.
- Keep the image stable and Debian-based (Debian stable release line).
- Apply consistent KrishAdmin branding:
  - Custom wallpaper for both login screen and desktop
  - Predictable ISO naming (not the default live-image-amd64.*)
- Make the build process reproducible and safe to iterate on.

### Learning goals (next phase)
- Build and ship a custom Linux kernel in the ISO.
- Learn kernel development workflow:
  - Configuration, compilation, packaging
  - Boot testing in VMs
  - Debugging and tracing

---

## Current Status

### Working
- live-build repo initialized and committed
- ISO successfully builds (Debian live ISO hybrid format)
- Custom wallpaper is embedded and shows up properly in the live environment
- Git repo created with initial live-build configuration tracked

### In progress
- Fixing default live login user and password so login works reliably
- Renaming output artifacts from live-image-amd64.* to KrishAdmin-Linux-2026.*

---

## Build Outputs

After a successful build, live-build generates multiple artifacts. The primary one you boot in Proxmox is:

- KrishAdmin-Linux-2026-amd64.hybrid.iso (or the default live-image-amd64.hybrid.iso before renaming)

The .hybrid.iso format is useful because it works both as:
- A normal ISO attached to a VM
- An image that can be written to a USB drive

---

## Repo Structure (high-level)

Key folders that define what ends up inside the ISO:

- config/
  - live-build configuration
- config/package-lists/
  - Controls which packages are included in the image
- config/includes.chroot/
  - Files copied directly into the live filesystem (branding, wallpapers, config files)
- config/hooks/ (future use)
  - Scripts that run during build stages

Build artifacts (should not be committed):
- binary/, chroot/, cache/, .build/, *.iso, and various live-image-* lists/manifests

---

## How We Build

Typical clean rebuild loop:

    sudo lb clean --purge
    lb config
    sudo lb build 2>&1 | tee build.log

The ISO output is created in the repo root directory.

---

## Branding Implemented

### Wallpaper
The wallpaper is embedded into the ISO under:

- /usr/share/backgrounds/krishadmin/

It is applied in two places:
1) Login background (LightDM GTK Greeter)
2) Desktop background (XFCE default config)

This ensures a consistent look from boot to desktop.

---

## Login and User Setup (Current Roadblock)

### Problem
Booting the ISO sometimes results in a login screen where expected default credentials do not work. This blocks testing and slows iteration.

### Root causes we identified
- The live username is created at boot by live-config. If live-config is missing or boot parameters are not correctly applied, the user may not be created.
- Password must be set after the user exists (during boot), typically via a live-config hook.
- A major build blocker occurred due to auto/config recursion:
  - lb config automatically runs auto/config
  - If auto/config calls lb config without noauto, it loops forever

### Fix approach
1) Ensure auto/config uses lb config noauto ... so the script does not recursively call itself.
2) Ensure boot parameters are passed correctly:
   - Set username to krishadmin via bootappend-live (username=krishadmin and live-config.username=krishadmin)
3) Add a live-config boot hook to set password after user creation:
   - Set password for krishadmin to KrishAdmin@2003 (temporary for development)
4) Optionally enable LightDM autologin as a fallback while debugging login flow.

Important security note:
- Hardcoding a real password in a public repo is not recommended. This should be considered temporary while stabilizing the build.

---

## ISO Naming (Current Roadblock)

### Goal
Rename artifacts from the default:
- live-image-amd64.*

to:
- KrishAdmin-Linux-2026.*

### Approach
- Use live-build --image-name KrishAdmin-Linux-2026 during lb config
- Keep iso-hybrid output (recommended for VM + USB)
- Optionally copy the resulting ISO to a friendlier name:
  - KrishAdmin-Linux-2026.iso

---

## Testing in Proxmox

1) Upload the generated ISO to Proxmox ISO storage.
2) Attach it to a VM CD/DVD drive.
3) Set boot order to CD/DVD first.
4) Boot and confirm:
   - Wallpaper appears on login and desktop
   - Live user exists and can log in
   - Minimal packages only

---

## Next Steps

### Immediate
- Stabilize live login:
  - Ensure user creation and password setting works every boot
  - Confirm boot params are present via /proc/cmdline
- Finalize ISO naming changes and confirm new filenames are produced consistently

### Near-term
- Add a lightweight first boot experience:
  - Show local install/readme guide on first login
  - Keep minimal, no extra bloat

### Longer-term (Kernel Phase)
- Build custom kernel packages (.deb)
- Include the custom kernel in the live-build image
- Add VM-based kernel debugging workflow (QEMU/KVM)

---

## Notes
This project is being developed iteratively. Each change is intended to be reproducible by placing it in the live-build config under config/ rather than manually modifying the build VM.
