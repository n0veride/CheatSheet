# Windows


### List directories and files of a share
```powershell
# CMD
dir \\[IP]\[Share]\

# PS
Get-ChildItem \\[IP]\[Share]\
```

### Connect share locally and Assign to drive 'n'
```powershell
# CMD Non-Authenticated
net use n: \\[IP]\[Share]

# CMD Authenticated
net use n: \\[IP]\[Share] /user:[User] [Password]

# PS Non-Authenticated
New-PSDrive -Name "N" -Root "\\[IP]\[Share]" -PSProvider "FileSystem"

# PS Authenticated
$username = '[User]'
$password = '[Password]'
$secpassword = ConvertTo-SecureString $password -AsPlainText -Force
$cred = New-Object System.Management.Automation.PSCredential $username, $secpassword

New-PSDrive -Name "N" -Root "\\[IP]\[Share]" -PSProvider "FileSystem" -Credential $cred
```
	- With the shared folder mapped as the `n` drive, we can execute Windows commands as if this shared folder is on our local computer.

### Search for specific names in a share drive assigned to 'n'
```powershell
# CMD
dir n:\*[Keyword]* /s /b

# PS
Get-ChildItem -Recurse -Path N:\ | Select-String "[Keyword]" -List
```

### Search for specific word in file
```powershell
findstr /s /i [Keyword] n:\*.*
```


# Linux

### Service Enumeration
```bash
sudo nmap [IP] -sV -sC -p139,445
```


### Evaluate minimum SMB version
```bash
smbclient -L ip -U username -m NT1
smbclient -L ip -U username -m SMB2
smbclient -L ip -U username -m SMB3
```
	- If one or more fail, the next one that succeeds is the minimum SMB version supported.


### Display list of server shares
```bash
# smbclient
smbclient -N -L //[IP]     # Null session specified
smbclient -L //[IP] -U "[DOMAINNAME]\\[User]"
smbclient -L //[IP] -U [User] --pw-nt-hash [NTLM_Hash]

# smbmap
smbmap -H [IP]
```


### Display share directories
```bash
smbmap -H [IP] -r [Share]
```


### Gather share info
```bash
# Connect
rpcclient -U'%' [IP]
rpcclient -U '[User]' [IP] --pw-nt-hash [NTLM_Hash]

# Server Info
srvinfo

# User Info
endomusers
enumprivs

# Group Info
enumdomgroups
enumdomains
enumalsgroups builtin
enumalsgroups domain

# Domain PW Policy Info
getdompwinfo
getusrdompwinfo 1000

# Create New User
createdomuser [User]
setuserinfo2 [User] 24 '[New_PW]'

# Change PW
chgpasswd3 [User] [Old_]PW [New_PW]

# Create New Share
netshareadd "C:\Windows" "Windows" 10 "Windows Share"
```
	- Can output all to file `> info.txt`


### Upload/ Download file
```bash
smbmap -H [IP] --upload [File] "[Share]\[Output_File]"

smbmap -H [IP] --download "[Share]\[Output_File]"
```



### Connect to share
```bash
smbclient \\[IP]\\[Share]
smbclient -U "[DOMAINNAME]\[User]" //[IP]/[Share]
```


### Mount SMB share locally
```bash
sudo mkdir /mnt/[Share]
# Authenticated
sudo mount -t cifs -o username=[User],password=[Password],domain=. //[Share_IP]/[Share] /mnt/[Share]

# Using a credential file
mount -t cifs //[Share_IP]/[Share] /mnt/[Share] -o credentials=/[Path]/[Credential_File]
```
	- May need to install `cifs-utils` to connect to an SMB share folder.
##### Cred File
```txt
username=plaintext
password=Password123
domain=.
```


### Enum4Linux
```bash
./enum4linux-ng.py [IP] -A -C
```
