# User Enumeration

## Kerbrute

### Installation

``` bash
sudo git clone https://github.com/ropnop/kerbrute.git
cd kerbrute
sudo make all
sudo mv kerbrute_linux_amd64 /usr/local/bin/kerbrute
```

### User Enumeration

``` bash
kerbrute userenum -d $DOMAIN --dc $DC_IP users.list -o valid_users.txt
```