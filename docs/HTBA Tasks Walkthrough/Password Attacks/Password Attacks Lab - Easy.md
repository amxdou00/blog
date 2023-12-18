**QUESTION 1**: Examine the first target and submit the root password as the answer.

Since we are only given an IP, let's do an nmap scan

``` bash
sudo nmap -A -T4 <IP> -Pn
```

![](images/Password%20Attacks%20Lab%20-%20Easy/nmap.png)

We can try to brute force the ssh service with hydra since it is faster than ssh
> This might take a few minutes

```
hydra -L username.list -P password.list ftp://<IP> -t 64
```
![](images/Password%20Attacks%20Lab%20-%20Easy/ftp_bruteforce.png)

We found the user `mike` and the very secure password `7777777`.
We can now connect to ftp with these credentials.

![](images/Password%20Attacks%20Lab%20-%20Easy/ftp_login_ls.png)

Listing the content of the ftp server, we find very interesting files.
Let's get the id_rsa file and use it to connect via ssh.
> Don't forget to change the permissions of the id_rsa file otherwise it won't work

```
ssh mike@<IP> -i id_rsa
```
Trying to connect like this, it will ask us for a passphrase for the id_rsa key.
Let's try the password we found earlier: `7777777`

![](images/Password%20Attacks%20Lab%20-%20Easy/ssh_connected.png)

And it worked! We are now connected as mike.
From here we need to privesc to root.

Listing the home directory of `mike` does not show anything interesting.
Let's look at his command history

![](images/Password%20Attacks%20Lab%20-%20Easy/root_password.png)

We can see something that looks like a password for root.
Let's try to ssh with it to see if it is valid

![](images/Password%20Attacks%20Lab%20-%20Easy/root_ssh.png)

We are successfully connected as root.

Answer: **dgb6fzm0ynk@AME9pqu**