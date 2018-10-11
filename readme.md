[![GitHub](https://img.shields.io/github/license/mashape/apistatus.svg?style=flat-square)](license.md)
[![Maintenance](https://img.shields.io/maintenance/yes/2018.svg?style=flat-square)]()

## Table of Contents

1. [Samba Setup](#samba-setup)
1. [Samba Share Access (unrestricted)](#samba-share-access-(unrestricted))
1. [Samba Share Access (restricted)](#samba-share-access-(restricted))
1. [Network goodies](#network-goodies)
1. [Diff between files/folders](#diff-between-files/folders)
1. [Protected archives](#protected-archives)
1. [Pumping .bash_aliases](#pumping-.bash_aliases)
1. [Encrypt/decrypt a file](#encrypt/decrypt-a-file)
1. [Stress test via DoS attack](#stress-test-via-dos-attack)
1. [cURL cheatsheet](#curl-cheatsheet)
1. [Installing programs from sources](#installing-programs-from-sources)
1. [Installing Oracle Java 8 / 9](#installing-oracle-java-8-/-9)
1. [Bash-Snippets](#bash-snippets)


### Samba Setup

Actual for **Ubuntu 16.04**

```bash
sudo apt-get install -y samba samba-common python-glade2 system-config-samba
sudo mv /etc/samba/smb.conf /etc/samba/smb.conf.bak
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


### Samba Share Access (unrestricted)

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


### Samba Share Access (restricted)

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


### Network goodies

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

Find out MAC address by using IP address:

```bash
arping -I eth0 -c 2 destination_ip
```

Using `arp-scan` allows to discover all IP hosts on the local network, including those that block all IP traffic such as firewalls and systems with ingress filters.
It works on Ethernet and 802.11 wireless networks. Requires root privilege.

```bash
# "eth0" is used for example, in reality the network interface name depends on the OS, the network type and other factors
sudo arp-scan --interface=eth0 --localnet
# or
sudo arp-scan --localnet
```


### Diff between files/folders

```bash
# install "Meld", visual diff and merge tool for files, folders and VCS
sudo apt install meld
# diff between files
meld file1 file2
# diff between folders
meld dir1 dir2
```

Also it's possible and widely used to set **Meld** as a Git `difftool` and `mergetool`.


### Protected archives

Create encrypted ZIP archive (password as a plain text):

```bash
zip -P s0me_paSS -r protected.zip /home/sites/*/www/
```

Create encrypted ZIP archive (request to enter password), different choices:

```bash
zip --encrypt protected.zip file_name
zip --encrypt protected.zip file1 file2 file3
zip --encrypt -r protected.zip /home/user/folder/
zip --encrypt -r protected.zip /folder1/ /folder2/
```

ZIP supports a simple password-based symmetric encryption system, which is documented in the ZIP specification, and known to be **seriously flawed**, 
so don't use it for data with limited access.


### Pumping .bash_aliases

It's possible to put a lot of useful shortcuts in `~/.bash_aliases` which can improve work effectiveness:

```bash
# general aliases
alias df="df -h"
alias du="du -c -h"
alias mkdir="mkdir -pv"
alias ls="ls --color=auto --group-directories-first"
alias ll="ls -lA"
alias lx="ll -BX"   # sort by extension
alias lz="ll -rS"   # sort by size
alias lt="ll -rt"   # sort by date
alias l.="ll -d .*" # show only hidden files
alias ..="cd .."
alias mnt="mount | column -t"
alias pwdgen="openssl rand -base64 30"
alias ports="netstat -tulanp" # quickly list all TCP/UDP port on the server
alias ping="ping -c 5" # stop after sending count ECHO_REQUEST packets
alias wget="wget -c" # can resume downloads
alias i="ifconfig"
alias net="netstat -tunlep"

# Git
alias ga='git add'
alias gp='git push'
alias gl='git log --pretty=format:"%h %ad | %s%d [%an]" --graph --date=short'
alias gs='git status'
alias gd='git diff'
alias gm='git commit'
alias gb='git branch'
alias gc='git checkout'
alias gf='git reflog'
alias gma='git commit -am'
alias gra='git remote add'
alias grr='git remote rm'
alias gpu='git pull'
alias gcl='git clone'
alias gta='git tag -a -m'
```


### Encrypt/decrypt a file

Use the built-in **gpg** tool:

```bash
# encrypt
gpg -c important.data.txt
# decrypt
gpg important.data.txt.gpg
```


### Stress test via DoS attack

Using **ab** (Apache HTTP server benchmarking tool).
Official docs: [link](https://httpd.apache.org/docs/2.4/programs/ab.html)

```bash
ab -k -c 350 -n 20000 example.com
```

For testing multiple URL's concurrently create a shell script with multiple `ab` calls:

```bash
#!/bin/sh

ab -n 100 -c 10 example.com/login > test1.txt &
ab -n 100 -c 10 example.com/news > test2.txt &
```

Using **Siege**:

```bash
siege -d10 -c50 example.com
```


### cURL cheatsheet

Debug options `--verbose` (`-v`), `--trace`, `--trace-ascii`, `--trace-time` allow to get more details as they show EVERYTHING **curl** sends and receives.

```bash
# use "-" as filename to have the output sent to stdout
curl --trace-ascii - http://www.example.com/
```

Make GET request, only print the response headers and display the time it took:

```bash
curl -sIX GET -w "Total time: %{time_total} s\n" www.example.com
# or
curl -o /dev/null -D- www.example.com
```

Typical usage, send GET request with headers:

```bash
curl -X GET 'http://www.example.com' -H 'Accept-Language: en' -H 'Authorization: Bearer A0v7mf98JJvWQTEbpEYNTt0uw2q0yl6P' -H 'Content-Type: application/json'
```

Identify the HTTP options available on the target URL, including the various types of allowed HTTP methods:

```bash
curl -v -X OPTIONS http://www.example.com/
```


### Installing programs from sources

```bash
tar xzvf program.sources.tar.gz
cd program.sources
# configure and compile
# if README is present, read it first
./configure
make
sudo make install
# clean up any temp files, optional
make clean
```


### Installing Oracle Java 8 / 9

```bash
sudo add-apt-repository ppa:webupd8team/java
sudo apt update; sudo apt install oracle-java8-installer
# or replace oracle-java8-installer with oracle-java9-installer to install Java 9
# check the Java version
javac -version
# set Java environment variables
sudo apt install oracle-java8-set-default
```


### Bash-Snippets [![Awesome](https://cdn.rawgit.com/sindresorhus/awesome/d7305f38d29fed78fa85652e3a63e154dd8e8829/media/badge.svg)]()

A collection of small bash scripts for heavy terminal users with no dependencies: [link](https://github.com/alexanderepstein/Bash-Snippets)