
# Network Info
## Interfaces, IP Addresses, DNS Info
- View network interfaces, both physical and virtual. Can show dual-homed endpoints.
```powershell
ipconfig /all
```

## ARP Table
- View other hosts the host recently comm'd with.  Could be good indication of which hosts admins are connecting via RDP or WinRM from this host.
```powershell
arp -a
```

## Routing Table
- View Info about local network and networks around it. Can gather info about the local domain & IP addresses of DCs
```powershell
route print
```

# Protections (AV/EDR)
