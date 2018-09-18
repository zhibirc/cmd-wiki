## Table of Contents

1. [Samba Setup](#samba-setup)
1. [Samba Share Access (unrestricted)](#samba-share-access-(unrestricted))
1. [Samba Share Access (restricted)](#samba-share-access-(restricted))
1. [Network goodies](#network-goodies)


## Samba Setup ##

Actual for **Ubuntu 16.04**

```bash
sudo apt-get install -y samba samba-common python-glade2 system-config-samba
sudo mv /etc/samba/smb.conf /etc/samba/smb.conf.bak
```

```bash
sudo vi /etc/samba/smb.conf
```

```
[global]
workgroup = WORKGROUP
server string = Samba Server %v
netbios name = srvr1
security = user
map to guest = bad user
name resolve order = bcast host
wins support = no
dns proxy = no
```


## Samba Share Access (unrestricted) ##

```bash
sudo mkdir -p /samba/share
cd /samba
sudo chmod -R 0755 share
sudo chown -R nobody:nogroup share/
```

```
[share]
path = /samba/share
browsable = yes
writable = yes
guest ok = yes
read only = no
```

```bash
sudo service smbd restart
```


## Samba Share Access (restricted) ##

```bash
sudo mkdir -p /samba/share/secured
sudo addgroup securedgroup
cd /samba/share
sudo chown -R zhibirc:securedgroup secured
sudo chmod -R 0770 secured/
sudo usermod -a -G securedgroup zhibirc
sudo smbpasswd -a zhibirc
```

```bash
sudo vi /etc/samba/smb.conf
```

```
[secured]
path = /samba/share/secured
# valid users = zhibirc
valid users = @securedgroup
guest ok = no
writable = yes
browsable = yes
```

```bash
sudo service smbd restart
```


## Network goodies ##

Retrieve list of Samba master browser(s):

```bash
nmblookup -M -- -
```

Show NFS exports, like the ```showmount -e``` command:

```bash
nmap -sV --script=nfs-showmount 127.0.0.1
```

Mapping processes to system ports they listen for:

```bash
sudo netstat -tpln
```

Serve folder:

```bash
python -m SimpleHTTPServer 8080
# or
python3 -m http.server 8080
# or
sudo npm install http-server -g
http-server
```