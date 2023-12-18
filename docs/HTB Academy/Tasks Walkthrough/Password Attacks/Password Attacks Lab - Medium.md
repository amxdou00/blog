**QUESTION 1**: Examine the second target and submit the contents of flag.txt in /root/ as the answer.

Let's do a quick nmap scan
```
sudo nmap -A -T4 10.129.202.221 -Pn
```

![](images/Password%20Attacks%20Lab%20-%20Medium/nmap.png)

We can see that `ssh` and `smb` are running on the target. 
Let's try to list the available shares using smbclient

```
smbclient -L \\<IP>
```

![](images/Password%20Attacks%20Lab%20-%20Medium/smbclient_list_shares.png)

We can try to brute force the service with crackmapexec

```
crackmapexec smb <IP> -u username.list -p password.list --shares
```

![](images/Password%20Attacks%20Lab%20-%20Medium/crackmapexec_creds.png)

We got some credentials: `john:123456`, let's try to use them in `smbclient`

```
smbclient \\\\<IP>\SHAREDRIVE -U john
```

![](images/Password%20Attacks%20Lab%20-%20Medium/smb_share_docszipfile.png)

We can see a zip archive called "Docs.zip". Let's download it and see its content.

![](images/Password%20Attacks%20Lab%20-%20Medium/docs_zip_password_protected.png)

As expected, the archive is password-protected. Let convert it with `zip2john` and try to crack the password

```
zip2john Docs.zip > Docs.zip.hash
john --wordlist=mut_password.list Docs.zip.hash
```
> I used the mutated wordlist from the `Password mutations` module

![](images/Password%20Attacks%20Lab%20-%20Medium/cracked_docs_zip.png)

We cracked the password of the zip file. Now we should be able to see what it contains.

![](images/Password%20Attacks%20Lab%20-%20Medium/content_zip_file.png)

We can see that the zip archive contained a Microsoft Word document.
Trying to open it, we can see that it is password-protected. We will follow the same process as with the zip archive in order to crack the docx file.
We will use `office2john` in this case

```
office2john Documentation.docx > Documentation.docx.hash
john --wordlist=mut_password.list Documentation.docx.hash
```

![](images/Password%20Attacks%20Lab%20-%20Medium/docx_password.png)

As we can see the password is `987654321`. Let's use it to open the docx file.
> I used libreoffice to open the file. You can install it on kali with `sudo apt install libreoffice`

![](images/Password%20Attacks%20Lab%20-%20Medium/root_password.png)

Looks like we have some credentials here: `jason:C4mNKjAtL2dydsYa6`.
Let's use them to connect with ssh

![](images/Password%20Attacks%20Lab%20-%20Medium/jason_ssh.png)

Now we need to elevate our privileges.
Looking around, we can see that there is another user "dennis" but there is not much we can do since we can't access this user's .ssh directory.

At the end of the word document, it was mentioned that a database should already be set up. This might be a hint.
Let's look at the open ports on the box first

```
netstat -ltu
```

* `-l`: Only shows listening ports
* `-t`: TCP ports
* `-u`: UDP ports

![](images/Password%20Attacks%20Lab%20-%20Medium/open_ports.png)

We can see that `mysql` is running. Let's connect to it with our credentials with the following command:

```
mysql -u jason -p
```
After login in and selecting the `users` database with `use users;`. We will see a table containing users and passwords, we can see at the end the password of the user `dennis` which is `7AUgWWQEiMPdqx`:

![](images/Password%20Attacks%20Lab%20-%20Medium/dennis_pass_mysql.png)

Let's try switching to dennis with the password we obtained previously.

![](images/Password%20Attacks%20Lab%20-%20Medium/su_dennis.png)

From here, the privilege escalation path is not so obvious.
After many trials and errors the only thing left is ==to try to use the id_rsa of dennis to connect as root :skull:==

![](images/Password%20Attacks%20Lab%20-%20Medium/id_rsa_dennis_passphrase.png)

We got `P@ssw0rd12020!` as the passphrase. Let's use it to connect as root

![](images/Password%20Attacks%20Lab%20-%20Medium/ssh_root.png)

Now we can get the flag

![](images/Password%20Attacks%20Lab%20-%20Medium/flag.png)

Answer: **HTB{PeopleReuse_PWsEverywhere!}**




