# NIDPS Lab Network Configuration
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
Reboot Metasploitable:
```bash
sudo reboot
```
Verify:
```bash
ip a
```
---

# Kali Linux Configuration

Kali uses two interfaces.
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
# Server Configuration
The server uses **Netplan**.
Check configuration file:
Edit the file:
```bash
sudo nano /etc/netplan/<file-name>.yaml
```
Example:
```yaml
network:
  version: 2
  ethernets:
    eth0:
       dhcp4: false
       addresses: [10.10.10.12/24]
       nameservers:
         addresses: [8.8.8.8, 8.8.4.4]
    eth1:
      dhcp4: true
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
# Connectivity Test From Kali and watchserver
Internal network:
```bash
ping 10.10.10.10
ping 10.10.10.12
ping 8.8.8.8
ping google.com
```
---
# Hyper-V Monitoring Setup
To allow the sensor to inspect traffic, enable **Port Mirroring**.
### Source (Kali / Metasploitable)
```
VM Settings → Network Adapter(dubble click) → Advanced Features
Port Mirroring → Source
```
### Destination (watchserver)
```
VM Settings → Network Adapter → Advanced Features
Port Mirroring → Destination
```
Enable packet capture support: Powershell >> Run as Administratior 
```powershell
Get-VM -Name "Sensor-VM" | Get-VMNetworkAdapter | Set-VMNetworkAdapter -MacAddressSpoofing On
```
---
# Verify Packet Capture
Check mirrored traffic on the watchserver:
```bash
sudo tcpdump -i eth0
```
## Other Virtualization Platforms : Check Online 
---
