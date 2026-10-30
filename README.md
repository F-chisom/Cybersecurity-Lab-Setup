# Week 1: Cybersecurity Lab Setup (VirtualBox)

This document records how the cybersecurity lab was set up in Oracle VirtualBox. It covers the shared NAT network, the Kali Linux VM, the Windows 10 VM and the Android VM. Screenshots are stored in the `images/` folder and shown under each step (the original evidence is in `Evidence_task_1_networkwalks.pdf`).

## Lab Overview

| VM | Purpose | Network | State in evidence |
|---|---|---|---|
| Kali Linux | Main attacker/testing machine | NAT Network (`NatNetwork`), static IP `10.0.0.2` | Running |
| Windows 10 | Target/test machine | NAT Network (`NatNetwork`) | Powered off (Snapshot 1) |
| Android 9 (android-x86) | Mobile testing target | Display configured | Powered off |
| Kali Linux Purple, Ubuntu | Pre-existing VMs | n/a | Powered off |

## Prerequisites

- Oracle VirtualBox installed on the host machine
- Internet access to download the Windows 10 and Android-x86 images
- Kali Linux VM already installed and running in VirtualBox

---

## 1. Lab Setup: VirtualBox Network

All lab VMs share one NAT Network so they can talk to each other and reach the internet.

### 1.1 Create the NAT Network

1. Open **Oracle VirtualBox Manager**.
2. Go to **File → Tools → Network Manager** (or **File → Preferences → Network** in older versions) and open the **NAT Networks** tab.
3. Click **Create** and select the new network.
4. Under **General Options**, set:
   - **Name:** `NatNetwork`
   - **IPv4 Prefix:** `10.0.0.0/24`
   - **Enable DHCP:** checked
   - **Enable IPv6:** unchecked
5. Click **Apply**.

<p align="center">
  <img src="images/01-nat-network-manager.png" alt="NAT Network created in VirtualBox Manager (NatNetwork, 10.0.0.0/24, DHCP enabled)" width="760"><br>
  <sub><b>Figure 1:</b> NAT Network created in VirtualBox Manager (NatNetwork, 10.0.0.0/24, DHCP enabled)</sub>
</p>

### 1.2 Attach the Kali Linux VM to the NAT Network

1. Select the **kali linux** VM and click **Settings**.
2. Switch to **Expert** mode and open the **Network** section.
3. On **Adapter 1**, set:
   - **Enable Network Adapter:** checked
   - **Attached to:** NAT Network
   - **Name:** `NatNetwork`
   - **Promiscuous Mode:** Deny
   - **Cable Connected:** checked
4. Click **OK**.

<p align="center">
  <img src="images/02-kali-network-adapter.png" alt="Kali Linux VM Network settings, Adapter 1 attached to NatNetwork" width="760"><br>
  <sub><b>Figure 2:</b> Kali Linux VM Network settings, Adapter 1 attached to NatNetwork</sub>
</p>

---

## 2. IP Address Configuration and Internet Setup (Kali Linux)

### 2.1 Set a static IPv4 address in the GUI

1. Start the Kali VM.
2. Open the network settings and edit **Wired connection 1**.
3. Open the **IPv4 Settings** tab and set:
   - **Method:** Manual
   - **Address:** `10.0.0.2`
   - **Netmask:** `24`
   - **Gateway:** `10.0.0.1`
   - **DNS servers:** `8.8.8.8`
4. Click **Save**.

<p align="center">
  <img src="images/03-kali-ipv4-manual.png" alt="Editing Wired connection 1 with a manual IPv4 address in Kali" width="760"><br>
  <sub><b>Figure 3:</b> Editing Wired connection 1 with a manual IPv4 address in Kali</sub>
</p>

### 2.2 Apply the changes from the terminal

Run these commands in a Kali terminal:

```bash
sudo nmcli connection modify "Wired connection 1" ipv4.dad-timeout 0
sudo nmcli connection modify "Wired connection 1"
sudo nmcli connection down "Wired connection 1"
sudo nmcli connection up "Wired connection 1"
```

A successful run prints `Connection successfully activated`, which confirms the new settings are live.

<p align="center">
  <img src="images/04-kali-nmcli-terminal.png" alt="nmcli commands run in the Kali terminal, connection reactivated successfully" width="760"><br>
  <sub><b>Figure 4:</b> nmcli commands run in the Kali terminal, connection reactivated successfully</sub>
</p>

### 2.3 Verify

```bash
ip a                # confirm 10.0.0.2/24 is assigned
ping -c 4 10.0.0.1  # gateway reachable
ping -c 4 8.8.8.8   # internet reachable
```

---

## 3. VirtualBox Settings for Kali Linux VM

### 3.1 System: Motherboard

1. Open **Settings → System → Motherboard**.
2. Configuration used:
   - **Base Memory:** about 4510 MB
   - **Boot Device Order:** Floppy and Optical enabled (Hard Disk and Network were unchecked in the evidence screenshot)
   - **Chipset:** PIIX3
   - **TPM Version:** None
   - **Pointing Device:** USB Tablet
   - **Features:** I/O APIC and Hardware Clock in UTC enabled

> Tip: once the OS is installed, enable **Hard Disk** in the boot order (and move it above Optical) so the VM boots from its disk.

### 3.2 System: Acceleration

1. Open the **Acceleration** tab.
2. Configuration used:
   - **Paravirtualization Interface:** Default
   - **Hardware Virtualization:** Nested Paging enabled

<table align="center">
  <tr>
    <td align="center" width="50%"><img src="images/05-kali-system-motherboard.png" alt="Motherboard tab" width="100%"><br><sub><b>Figure 5:</b> Motherboard tab</sub></td>
    <td align="center" width="50%"><img src="images/06-kali-system-acceleration.png" alt="Acceleration tab" width="100%"><br><sub><b>Figure 6:</b> Acceleration tab</sub></td>
  </tr>
</table>

### 3.3 Shared Folders and User Interface

1. Open **Settings → Shared Folders**. No machine, global or transient folders were configured for this VM.
2. Open **Settings → User Interface**. Mini ToolBar is set to **Show in Full-screen/Seamless** (not shown at top of screen).

<p align="center">
  <img src="images/07-kali-shared-folders-ui.png" alt="Kali Linux Shared Folders and User Interface settings" width="760"><br>
  <sub><b>Figure 7:</b> Kali Linux Shared Folders and User Interface settings</sub>
</p>

---

## 4. VirtualBox Settings for Windows 10 VM

### 4.1 Download Windows 10

1. Go to the Microsoft Software Download page for Windows 10 (`microsoft.com/en-us/software-download/windows10`).
2. Download the **Windows 10 2022 Update (Version 22H2)** image.

> Note: Microsoft stated that after October 14, 2025, Windows 10 no longer receives free security updates. Keep this VM isolated inside the lab network.

<p align="center">
  <img src="images/08-windows10-download.png" alt="Microsoft Windows 10 download page (Version 22H2)" width="760"><br>
  <sub><b>Figure 8:</b> Microsoft Windows 10 download page (Version 22H2)</sub>
</p>

### 4.2 Create and configure the VM

1. In VirtualBox click **New** and create a VM named **Windows 10** using the downloaded image.
2. Complete the Windows installation.

### 4.3 Take a snapshot

1. With the VM powered off, open the **Snapshots** tab and click **Take**.
2. Snapshot details:
   - **Name:** `Snapshot 1`
   - **Description:** Windows 10 virtual machine is installed and configured
3. Use **Restore** later to return the VM to this clean state.

<p align="center">
  <img src="images/09-windows10-snapshot.png" alt="Windows 10 VM with Snapshot 1 taken" width="760"><br>
  <sub><b>Figure 9:</b> Windows 10 VM with Snapshot 1 taken</sub>
</p>

### 4.4 Network settings

1. Open **Settings → Network → Adapter 1**.
2. Configuration used:
   - **Enable Network Adapter:** checked
   - **Attached to:** NAT Network
   - **Name:** `NatNetwork`
   - **Adapter Type:** Intel PRO/1000 MT Desktop (82540EM)
   - **Promiscuous Mode:** Deny
   - **MAC Address:** `080027430FF2`
   - **Cable Connected:** checked

<p align="center">
  <img src="images/10-windows10-network-adapter.png" alt="Windows 10 VM Network settings, Adapter 1 on NatNetwork" width="760"><br>
  <sub><b>Figure 10:</b> Windows 10 VM Network settings, Adapter 1 on NatNetwork</sub>
</p>

---

## 5. VirtualBox Settings for Android VM

### 5.1 Download Android-x86

1. Open the **android-x86** project on SourceForge (`sourceforge.net/projects/android-x86/`).
2. Click **Download** and save the ISO. (The project's last update was 2022-12-20.)

<p align="center">
  <img src="images/11-android-x86-download.png" alt="android-x86 project page on SourceForge" width="760"><br>
  <sub><b>Figure 11:</b> android-x86 project page on SourceForge</sub>
</p>

### 5.2 Create the VM

1. In VirtualBox click **New** and create a VM named **Android 9** using the downloaded ISO.

### 5.3 Display settings

1. Open **Settings → Display → Screen**.
2. Configuration used:
   - **Video Memory:** 16 MB
   - **Number of Virtual Monitors:** 1
   - **Scale Factor:** 100%
   - **Graphics Controller:** VMSVGA
   - **3D Acceleration:** disabled

<p align="center">
  <img src="images/12-android-display.png" alt="Android 9 VM Display settings" width="760"><br>
  <sub><b>Figure 12:</b> Android 9 VM Display settings</sub>
</p>

---

## Troubleshooting

- **No internet in Kali:** confirm the VM is on `NatNetwork`, the gateway is `10.0.0.1`, DNS is `8.8.8.8`, and re-run the `nmcli connection down/up` commands.
- **Browser shows "trouble finding that site":** usually a DNS or gateway issue; check the IPv4 settings in section 2.1.
- **VMs cannot see each other:** make sure every VM uses the same NAT Network name and has a unique IP in `10.0.0.0/24`.

## Evidence

The original screenshots are in `Evidence_task_1_networkwalks.pdf`, and the individual images are in the `images/` folder.
