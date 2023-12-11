# Hydra

## Usage
```
hydra [OPTIONS] <IP> <SERVICE>
```
## Useful Flags
- `h`: see the help menu
- `l <LOGIN>`: Pass single username/login
- `L <FILE>`: Pass multiple usernames/logins
- `p <LOGIN>`: Pass single known password
- `P <FILE>`: Pass a password list or wordlist (ex.: `rockyou.txt`)
- `s <PORT>`: Use custom port
- `f`: Exit as soon as at least one a login and a password combination is found
- `R`: Restore previous session (if crashed/aborted)

## HTTP Post Form
```
hydra -l user -P rockyou.txt <IP> http-post-form "<Login Page>:<Request Body>:<Error Message>"
```

## SSH
```
hydra -f -l user -P /usr/share/wordlists/rockyou.txt $IP -t 4 ssh
```

## MySQL
```
hydra -f -l user -P /usr/share/wordlists/rockyou.txt $IP mysql
```

## FTP
```
hydra -f -l user -P /usr/share/wordlists/rockyou.txt $IP ftp
```

## SMB
```
hydra -f -l user -P /usr/share/wordlists/rockyou.txt $IP smb
```