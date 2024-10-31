
Creating a secure web server for file upload operations.

# Nginx - Enabling PUT
- A good alternative to Apache for transferring files because the configuration is less complicated and the module system doesn't lead to security issues.
- Ensure users can't upload web shells and execute them.

### Create a Directory to Handle Uploaded Files
```bash
sudo mkdir -p /var/www/uploads/SecretUploadDirectory
```

### Change the Owner to www-data
```bash
sudo chown -R www-data:www-data /var/www/uploads/SecretUploadDirectory
```

### Create Nginx Config File
- Saved in `/etc/nginx/sites-available/upload.conf`
```nginx
server {
	listen 9001;
	
	location /SecretUploadDirectory/ {
		root   /var/www/uploads;
		dave_methods PUT;
	}
}
```

### Symlink our Site to the sites-enabled Directory
```bash
sudo ln -s /etc/nginx/sites-available/upload.conf /etc/nginx/sites-enabled/
```

### Start Nginx
```bash
sudo systemctl restart nginx.service
```

> Check error messages at `/var/log/nginx/errror.log`

### Verifying Errors
```bash
tail -2 /var/log/nginx/error.log

2020/11/17 16:11:56 [emerg] 5679#5679: bind() to 0.0.0.0:'80' failed (98: A`dress already in use`)
2020/11/17 16:11:56 [emerg] 5679#5679: still could not bind()
```
```bash
ss -lnpt | grep 80

LISTEN 0  100   0.0.0.0:80    0.0.0.0:*   users:(("python",pid=`2811`,fd=3),("python",pid=2070,fd=3),("python",pid=1968,fd=3),("python",pid=1856,fd=3))
```
```bash
ps -ef | grep 2811

user65   2811   1856  0 16:05 ?     00:00:04 `python -m websockify 80 localhost:5901 -D`
root     6720   2226  0 16:14 pts/0 00:00:00 grep --color=auto 2811
```
	- Shown is that there's already a module listening on port 80.  To get around this, remove the default Nginx config, which binds on port 80

### Remove Nginx Default Config
```bash
sudo rm /etc/nginx/sites-enabled/default
```

### Upload File Using cURL
```bash
curl -T [Input_File] http://localhost:9001/SecretUploadDirectory/[Output_File]
```


>Good test is to ensure the directory listing is not enabled by navigating to `http://localhost/SecretUploadDirectory`.
>By default with Apache, if we hit a directory w/o an index file, it'll list all the files which is not good
>Nginx features like that aren't enabled by default
