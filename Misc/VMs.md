
### Fixing 'Same UUID as an existing virtual machine' error with new machine creation

From the .vbox & .vdi directory:
- Run internal commands on the .vdi file twice
```bash
VBoxManage internalcommands sethduuid kali-linux-2023.3-virtualbox-amd64.vdi
	UUID changed to: b03b0789-6780-402e-8baa-3173d0cff886

# Run again 
VBoxManage internalcommands sethduuid kali-linux-2023.3-virtualbox-amd64.vdi
	UUID changed to: 0874ee2c-afc4-4da1-8859-31b2325b4143
```

- Change the 3 UUIDs in the .vbox file
```bash
vim kali-linux-2023.3-virtualbox-amd64.vbox

# Line 9 - First run uuid result
<Machine uuid="{b03b0789-6780-402e-8baa-3173d0cff886}" name="kali-linux-2023.3-virtualbox-amd64" OSType="Debian_64" snapshotFolder="Snapshots" lastStateChange="2024-06-28T19:21:46Z">

# Line 37 - Second run uuid result
<HardDisk uuid="{0874ee2c-afc4-4da1-8859-31b2325b4143}" location="kali-linux-2023.3-virtualbox-amd64.vdi" format="vdi" type="Normal"/>

# Line 82 - Second run uuid result
<Image uuid="{0874ee2c-afc4-4da1-8859-31b2325b4143}"/>
```




### chmod for all directories recursively

```
find . -type d -print0 | xargs -0 chmod 0755
```




### chmod for all files recursively

```
find . -type f -print0 | xargs -0 chmod 0644
```




### [Updating postgresql](https://medium.com/@gembit.soultan/how-to-upgrade-postgresql-15-to-postgresql-16-using-pg-upgradeclusters-in-ubuntu-22-04-c9f279c5d3ab)