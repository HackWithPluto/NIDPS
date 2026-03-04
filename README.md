# NIDPS Lab Network Configuration

This document describes the network setup for the **NIDPS (Network Intrusion Detection and Prevention System) lab**.

The lab uses three virtual machines:

| VM | Purpose |
|----|--------|
| Kali Linux | Attacker machine |
| Metasploitable | Vulnerable target |
| NIDPS Server | Monitoring server running Snort/Zeek/ELK |

Internal network range:

```
10.10.10.0/24
```

---

# Metasploitable Configuration

### Change Adapter
```
Settings → Network → Adapter Type → Legacy Network Adapter
```

### Configure Network

Edit:

```bash
sudo nano /etc/network/interfaces
```

```
auto eth0
iface eth0 inet static
    address 10.10.10.10
    netmask 255.255.255.0
```

Restart networking:

```bash
sudo service networking restart
```

Verify:

```bash
ifconfig
```

---

# Kali Linux Configuration

Kali uses two interfaces.

| Interface | Purpose |
|-----------|--------|
| eth0 | Internal network |
| eth1 | Internet (DHCP) |

Edit:

```bash
sudo nano /etc/network/interfaces
```

```
auto eth0
iface eth0 inet static
    address 10.10.10.11
    netmask 255.255.255.0

auto eth1
iface eth1 inet dhcp
```

Restart networking:

```bash
sudo systemctl restart networking
```

Verify:

```bash
ip a
```

---

# NIDPS Server Configuration

The server uses **Netplan**.

Check configuration file:

```bash
ls /etc/netplan/
```

Edit the file:

```bash
sudo nano /etc/netplan/<file-name>.yaml
```

Example:

```yaml
network:
  version: 2
  renderer: networkd
  ethernets:
    eth0:
      addresses:
        - 10.10.10.12/24
      dhcp4: no
    eth1:
      dhcp4: yes
```

Apply configuration:

```bash
sudo netplan apply
```

Verify:

```bash
ip a
```

---

# Network Addressing

| Machine | IP |
|--------|----|
| Kali | 10.10.10.11 |
| Metasploitable | 10.10.10.10 |
| NIDPS Server | 10.10.10.12 |

---

# Connectivity Test

Internal network:

```bash
ping 10.10.10.10
ping 10.10.10.12
```

Internet check:

```bash
ping 8.8.8.8
ping google.com
```

---

# Hyper-V Monitoring Setup

To allow the sensor to inspect traffic, enable **Port Mirroring**.

### Source (Kali / Metasploitable)

```
VM Settings → Network Adapter → Advanced Features
Port Mirroring → Source
```

### Destination (NIDPS Server)

```
VM Settings → Network Adapter → Advanced Features
Port Mirroring → Destination
```

Enable packet capture support:

```powershell
Get-VM -Name "Sensor-VM" | Get-VMNetworkAdapter | Set-VMNetworkAdapter -MacAddressSpoofing On
```

---

# Verify Packet Capture

Check mirrored traffic on the sensor:

```bash
sudo tcpdump -i eth0
```

or

```bash
sudo snort -i eth0
```

---

# Other Virtualization Platforms

**VMware**

```
Promiscuous Mode → Accept
MAC Changes → Accept
Forged Transmits → Accept
```

**VirtualBox**

```
Network → Adapter → Promiscuous Mode → Allow All
```

**KVM / Proxmox**

```bash
ip link set eth0 promisc on
```

---

This setup allows the NIDPS server to monitor all traffic between the attacker and the target machine.
