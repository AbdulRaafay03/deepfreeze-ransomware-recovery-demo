# Ransomware Recovery with Deep Freeze: A Digital Forensics Lab Demo

> **Educational / research use only.** This project documents a controlled, fully isolated lab. It contains **no malware, no samples and no download links**. Never run ransomware outside an isolated environment you own and are authorized to use.

## Overview

This project demonstrates how **Faronics Deep Freeze** can instantly recover a Windows machine after a real ransomware infection (WannaCry), without restoring from backups, paying a ransom, or reimaging the system.

Deep Freeze keeps the system drive in a "Frozen" state. Every change made while frozen (including malicious encryption of files) is discarded at reboot, returning the machine to the exact state it was in when it was frozen.

**Result:** after WannaCry encrypted the victim machine, a single reboot restored the system and all user files to their original, readable state.

## Objectives

- Show the behavior of a ransomware infection in a safe, controlled environment.
- Demonstrate the Frozen vs. Thawed states of Deep Freeze.
- Show that a reboot is enough to recover from an active ransomware infection on a frozen drive.
- Discuss where this approach fits in a broader digital forensics and incident response (DFIR) strategy.

## Lab Architecture

| Role | Hostname / OS | IP Address |
|------|---------------|------------|
| Victim machine | Windows VM (Deep Freeze installed) | `10.10.10.2` |
| DNS / network control machine | Ubuntu VM running `dnsmasq` | `10.10.10.3` |

- **Hypervisor:** VMware (Workstation / Player)
- **Network:** VMware **Host-only** network. Only the two VMs can communicate with each other.
- **Host isolation:** the physical host's Wi-Fi / internet connection was **disabled** before the ransomware was executed, so nothing could leave the lab.

```
        +---------------------------+
        |        VMware Host        |
        |  (Wi-Fi / NIC disabled)   |
        |                           |
        |   Host-only network       |
        |   10.10.10.0/24           |
        |                           |
        |  +---------+ +---------+  |
        |  | Windows | | Ubuntu  |  |
        |  | Victim  |-| dnsmasq |  |
        |  |10.10.10.2| |10.10.10.3| |
        |  +---------+ +---------+  |
        +---------------------------+
```

### Role of the Ubuntu dnsmasq machine

The Windows VM's DNS server is set to `10.10.10.3`. Because the lab has no internet, `dnsmasq` acts as a fake DNS server so we control how the malware's domain lookups are answered. This lets the ransomware behave as if it were on a live network and proceed with its payload, while all traffic stays inside the host-only network.

Example `dnsmasq` configuration (adapt to your lab):

```conf
# /etc/dnsmasq.conf
interface=ens33
listen-address=10.10.10.3
no-resolv
log-queries
log-facility=/var/log/dnsmasq.log
# add address=/domain/ip entries here as required by your scenario
```

Logging DNS queries is also useful forensically: it records which domains the malware tried to resolve.

## Tools Used

- VMware Workstation / Player (host-only networking)
- Windows VM (victim)
- Ubuntu Linux VM with `dnsmasq`
- Faronics Deep Freeze
- WannaCry ransomware sample (handled only inside the isolated lab; **not included** in this repository)

## Demo Procedure

### 1. Prepare the victim machine
1. Install Windows in a VMware VM and attach it to the host-only network (`10.10.10.2`).
2. Set the Windows DNS server to `10.10.10.3`.
3. Install Deep Freeze and configure it to **freeze the C: drive**.

### 2. Create the test files (Thawed state)
1. Reboot the machine into the **Thawed** state.
2. Create two text files containing recognizable content. These act as visual proof of encryption.
3. Confirm the files open normally.

### 3. Return to the Frozen state
1. Reboot into the **Frozen** state. The two text files are now part of the protected baseline.

### 4. Isolate the lab
1. Disable the physical host's Wi-Fi / network adapters.
2. Confirm only the two VMs can reach each other.
3. Take a VM snapshot as an additional safety net (recommended).

### 5. Execute the ransomware
1. Run the WannaCry sample on the victim machine.
2. The ransomware encrypts files and displays its ransom note.

### 6. Evidence of encryption
1. The two test files are encrypted and unreadable.
2. The ransom screen is displayed.
3. Capture screenshots and video of this stage.

### 7. Recovery
1. Reboot the victim machine (it is still in the Frozen state).
2. On boot, Deep Freeze discards every change made since the freeze.

## Results

| Stage | State of test files |
|-------|--------------------|
| Before infection | Readable |
| After WannaCry execution | Encrypted / unreadable |
| After reboot (Frozen state) | Fully restored, readable |

The system returned to its pre-infection state with a single reboot: no decryption tool, no backup restore, no ransom payment.


## Digital Forensics Perspective

- **Volatile + disk evidence:** because Deep Freeze wipes changes on reboot, any forensic evidence on the frozen drive (malware binaries, dropped files, registry changes) is lost after restart. Capture memory, disk images and logs **before** rebooting if you need to investigate.
- **DNS logs:** the `dnsmasq` log provides a record of the malware's name-resolution behavior.
- **Snapshots:** VMware snapshots allow preserving the infected state for later analysis.
- **Network indicators:** since traffic is confined to a host-only network, packet captures (Wireshark/tcpdump on the Ubuntu VM) can be used to study the malware's network behavior safely.

## Limitations and Considerations

- Deep Freeze protects the **frozen drives only**. Data on thawed drives, external drives, mapped network shares, or exfiltrated data is not protected.
- Ransomware that steals data before encrypting (double extortion) is not mitigated by a reboot.
- If the machine is booted in the **Thawed** state during an infection, the encryption is permanent.
- Deep Freeze is a recovery control, not a replacement for patching, EDR, network segmentation or offline backups.
- Results apply to this lab setup and this ransomware sample.

## Safety Notes

- Run only in an isolated VM on a host-only network.
- Disable host network adapters and shared folders / clipboard / drag-and-drop between host and guest.
- Keep snapshots of clean VMs.
- Do not commit malware samples or ransom binaries to this repository.

## Repository Structure

```
.
├── README.md
├── docs/
│   ├── architecture.png
│   └── screenshots/
├── configs/
│   └── dnsmasq.conf
└── video/            (optional: or link to the video externally)
```

## Disclaimer

This project is for education and defensive security research. The author is not responsible for misuse. Always follow your organization's policies and local laws.

