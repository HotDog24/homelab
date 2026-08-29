# Homelab — Month 1

## Goal
Build practical IT support/sysadmin skills through hands-on labs, targeting entry-level helpdesk roles.

## Environment
- Host: MacBook M1
- Virtualization: UTM (native ARM virtualization)
- VM 1: Ubuntu Server 24.04 ARM64
- Cloud: Microsoft 365 Developer tenant (Entra ID, Intune)

## Plan
- Week 1: Environment setup
- Week 2: Networking fundamentals (static IP, DNS, firewall rules)
- Week 3: Cloud identity basics (Entra ID users/groups, Intune enrollment)
- Week 4: Simulated helpdesk ticketing (osTicket in Docker)

## Log

### Week 1 — Setup & Initialization

**Aug 29, 2026 — VM creation and boot loop**
- Created Ubuntu Server 24.04 LTS ARM64 VM in UTM (Virtualize mode, 2 CPU / 4GB RAM / 25GB disk).
- Hit a boot loop after install completed: VM kept booting back into the installer instead of the installed OS.
- Root cause: UTM's boot order checked the virtual CD/ISO drive before the disk, so it kept re-launching the installer on every restart.
- Fix: ejected the ISO from VM settings (Drives → CD/DVD → Clear), then rebooted — VM loaded the installed OS correctly.
- Next time: check/set boot order to disk-first *before* first boot, to skip this entirely.

**Aug 29, 2026 — Networking and SSH**
- Confirmed network adapter `enp0s1` got a DHCP-assigned IP: `192.168.64.3`.
- Installed OpenSSH server during setup; verified access from Mac host.
- Noted: IP is DHCP-assigned, so it may change on reboot — planning to set a static IP or DHCP reservation before relying on this for daily work.

**Aug 29, 2026 — Remote development setup**
- Installed the Remote-SSH extension in VS Code on the Mac host, connected to `192.168.64.3` using the SSH config below, and confirmed I can browse/edit files on the VM directly from the Mac.
