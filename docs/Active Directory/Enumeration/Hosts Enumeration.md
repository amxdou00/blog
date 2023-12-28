# Host Discovery

## Network Traffic Analysis

### Wireshark

``` bash
sudo -E wireshark
```

- Filtering for `ARP` packets shows us some of the hosts present on the network

- Filtering for `MDNS` packets can show the name of the host ACADEMY-EA-WEB01 for example

### tcpdump
```
sudo tcpdump -i $INTERFACE
```

### Responder in Analyze mode

```
sudo responder -I $INTERFACE -A
```

## Ping sweep

### fping

``` bash
fping -asgq 10.10.10.10/24
```

- `-a` shows the targets that are alive
- `-s` prints stats at the end of the scan
- `-g` generates a target list from CIDR netword
- `-q` not to show per-target results

### nmap

``` bash
sudo nmap -sn 10.10.10.10/24
```

- `-sn` disables port scan (just ping scan)

### bash
``` bash
for i in {1..254} ;do (ping -c 1 172.16.5.$i | grep "bytes from" &) ;done
```

### powershell
``` powershell
1..254 | % {"172.16.5.$($_): $(Test-Connection -count 1 -comp 172.15.5.$($_) -quiet)"}
```

### windows cmd
```
for /L %i in (1 1 254) do ping 172.16.5.%i -n 1 -w 100 | find "Reply"
```
____
# Host scanning

## nmap

``` bash
nmap -v -A -iL hosts.txt -oN nmap.txt
```