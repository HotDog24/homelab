# Homelab — Month 1

## Goal
Build practical IT support/sysadmin skills through hands-on labs, targeting entry-level helpdesk roles.

## Environment
- Host: MacBook M1
- Virtualization: UTM (native ARM virtualization)
- VM 1: Ubuntu Server 24.04 ARM64
- Cloud: Microsoft 365 Business Premium 30-day trial

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

**Aug 29, 2026 — M365 tenant setup**
- Signed up for Microsoft 365 Business Premium 30-day trial (chose this after 
  the free Developer Program required a qualifying subscription I didn't have).
- Admin center accessible at admin.microsoft.com — confirmed login.

**Aug 29, 2026 — Static IP configuration**
- Replaced DHCP with a static IP via netplan to stop the address from changing on reboot (was risking breaking SSH/VS Code access each time).
- Config (`/etc/netplan/*.yaml`):
```yaml
  network:
    version: 2
    ethernets:
      enp0s1:
        dhcp4: no
        addresses:
          - 192.168.64.3/24
        routes:
          - to: default
            via: 192.168.64.1
        nameservers:
          addresses:
            - 1.1.1.1
            - 8.8.8.8
```
- Applied with `sudo netplan apply`.
- Verified persistence and connectivity after reboot:
![alt text](image.png)

## Week 1 Summary
- Working Ubuntu Server VM with static IP and SSH access, developed remotely via VS Code.
- Live M365 Business Premium tenant ready for Entra ID/Intune work in Week 3.
- Public repo established as the running record of this project.
- One open item carried into Week 2: confirm static IP/DNS survives reboot (see verification block above).

### Week 2 — Networking Fundamentals

**Aug 30, 2026 — VM2 creation and dual-VM connectivity**
- Created VM 2: Ubuntu Server 24.04 LTS ARM64 in UTM (Virtualize mode, 2 CPU / 4GB RAM / 25GB disk), OpenSSH enabled during install.
- Confirmed both VMs are on UTM's Shared Network mode, same virtual switch — checked in UTM VM settings before assuming it, rather than after hitting a problem.
- Assigned VM2 a static IP on the same subnet as VM1 via netplan:
```yaml
  network:
    version: 2
    ethernets:
      enp0s1:
        dhcp4: no
        addresses:
          - 192.168.64.4/24
        routes:
          - to: default
            via: 192.168.64.1
        nameservers:
          addresses:
            - 1.1.1.1
```
- Applied with `sudo netplan apply`, rebooted, verified IP held.
- Verified bidirectional connectivity between the two VMs:
![alt text](image-1.png)
![alt text](image-2.png)
- Result: both VMs reachable in both directions on first attempt — no UTM network-mode mismatch to troubleshoot this time.
- Next: set up VM1 as a basic DNS server (dnsmasq) and point VM2 at it.