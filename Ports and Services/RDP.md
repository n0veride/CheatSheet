
### Mounting a Linux Folder Using rdesktop
```bash
rdesktop [Victim_IP] -d [Domain] -u [User] -p [Password] -r disk:[Disk_Name_Shown_on_Victim]='[Shared_Path]'
```

### Mounting a Linux Folder Using xfreerdp
```bash
xfreerdp /v:[Victim_IP] /d:[Domain] /u:[User] /p:[Password] /drive:[Disk_Name_Shown_on_Victim],[Shared_Path]
```