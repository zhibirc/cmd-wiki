[![Wiki](https://img.shields.io/badge/wiki-knowledge--base-orange)]()
[![GitHub](https://img.shields.io/github/license/mashape/apistatus.svg?style=flat-square)](LICENSE.md)
[![Maintenance](https://img.shields.io/maintenance/yes/2022.svg?style=flat-square)]()
[![Platform](https://img.shields.io/badge/OS-GNU%2FLinux-yellowgreen.svg?style=flat-square)]()
[![Platform](https://img.shields.io/badge/macOS-yellowgreen.svg?style=flat-square)]()
[![Spellcheck](https://github.com/zhibirc/wiki/actions/workflows/spellcheck.yml/badge.svg)](https://github.com/zhibirc/wiki/actions/workflows/spellcheck.yml)

## Table of Contents

1. [Git](#git)
1. [NPM helpers](#npm-helpers)
1. [Shell usefulness](#shell-usefulness)
1. [Free Up Disk Space](#free-up-disk-space)
1. [Samba setup](#samba-setup)
1. [Samba share access (unrestricted)](#samba-share-access-unrestricted)
1. [Samba share access (restricted)](#samba-share-access-restricted)
1. [Network goodies](#network-goodies)
1. [Diff between files/folders](#diff-between-filesfolders)
1. [Protected archives](#protected-archives)
1. [Pumping .bash_aliases](#pumping-bash_aliases)
1. [Encrypt/decrypt a file](#encryptdecrypt-a-file)
1. [Stress test via DoS attack](#stress-test-via-dos-attack)
1. [cURL cheatsheet](#curl-cheatsheet)
1. [Wget basics](#wget-basics)
1. [Installing programs from sources](#installing-programs-from-sources)
1. [Installing Oracle Java 8 / 9](#installing-oracle-java-8--9)
1. [FFmpeg sweets](#ffmpeg-sweets)
1. [Getting file info](#getting-file-info)
1. [Getting hardware info](#getting-hardware-info)
1. [Generate a random password](#generate-a-random-password)
1. [Amazon Web Services (AWS)](#amazon-web-services-aws)
1. [Docker](#docker)
1. [Cool cheatsheets / tutorials / helper tools](#cool-cheatsheets--tutorials--helper-tools)
1. [Bash-Snippets](#bash-snippets-)


---


### Git

[**Documentation**](https://git-scm.com/doc)

Check if merge conflicts will occur before actual merging:

```shell
git merge <branch> --no-ff --no-commit
git merge --abort
```

Discard all unstaged changes:

```shell
git checkout -- .
```

Create branch from particular commit:

```shell
git checkout -b <branch name> <commit hash>
```

Create a local branch that tracks a remote branch:

```shell
# starting to work on an existing remote branch (e.g. upstream/develop)
git checkout --track <remote branch name>
```

Add/stage (modified and deleted files only, new files are not affected) and commit in one command:

```shell
git commit -am "commit message"
```

Automate formation of release announcements:

```shell
git shortlog -s | awk -F\\t '{print $2}' > contributors.md
git shortlog --no-merges | awk -F '[[:alnum:] ]+ \\([0-9]' '{print $1}' | grep . > changelog.md
```

Get a nice list of actual different commits not shared between the branches:

```shell
git log --left-right --graph --cherry-pick --oneline master..develop
```

Housekeeping tools:

```shell
# cleans up unreachable or "orphaned" Git objects

# what is set to be pruned but not actually prune it
git prune --dry-run --verbose

# prune and display output of all objects and actions taken by it
git prune --progress
```

Ignore changes to a file that's already tracked in the repository. 
It's a common task and is very helpful in case of adding some local tokens, for example, to configuration file with boilerplate:

```shell
git update-index --assume-unchanged <file>
# tracking changes again
git update-index --no-assume-unchanged <file>
```


### NPM helpers

Get debug info (useful for reports, GitHub issues, etc.):

```shell
npx envinfo --binaries --languages --system --utilities
```


### Shell usefulness

Check which init system your platform uses (**systemd** (`systemctl` command), or older **System V** (which uses the `service` command)):

```shell
ps --no-headers -o comm 1
```

Remove multiple sub-folders:

```shell
find . -type d -name node_modules -prune -exec rm -rf '{}' \;
```

Find patterns:

```shell
# find files containing a given text
find . -type f -print0 | xargs -0 grep -l "search string"
# "l" means that only the name of each input file with matched content will print
grep -rl "search string" /

# handle multiple search patterns
grep -e hacker -e root -e admin /etc/passwd
```

Sometimes process (Apache, for example) prevents to start service on the same port (nginx, for example):

```shell
# end the conflict process
sudo fuser -k 80/tcp
```

APT

```shell
# list all installed packages
apt list --installed | less

# list all ready-to-upgrade packages
apt list --upgradeable | less

# update list of available packages
sudo apt update

# upgrade the system by installing/upgrading packages
sudo apt upgrade

# search in package descriptions
apt search <needle_name>

# show package details
apt show <package_name>

# remove automatically all unused packages
sudo apt autoremove

# all-in-one upgrade system command
sudo apt update && sudo apt upgrade -y
```

Change/setup bash custom prompt (PS1) with Git branch displaying (if exists). Specify this in `~/.bashrc` and run `source ~/.bashrc` for applying changes:

```shell
git_branch() {
  git branch 2> /dev/null | sed -e '/^[^*]/d' -e 's/* \(.*\)/[\1]/'
}

COLOR_USER='\[\e[1;32m\]'
COLOR_PATH='\[\e[00;36m\]'
COLOR_RESET='\[\e[0m\]'

PS1="${COLOR_RESET}${COLOR_USER}\u▶${COLOR_RESET} ${COLOR_PATH}\w ${COLOR_USER}\[\033[00;32m\]\$(git_branch)\[\033[00m\] $ ${COLOR_RESET}"
```

Result of the above is something like:

![PS1 example](assets/images/ps1.png)

Fast checking of PHP SSL support:

```shell
echo '<?php phpinfo(); ?>' | php 2>&1 |grep -i ssl
```

Installing and basic setup of SSH Server:

```shell
sudo apt install openssh-server

# check status
service --status-all | grep ssh
# or
systemctl list-units | grep ssh

# config in /etc/ssh/sshd_config

# help
man sshd_config

# after making changes to the /etc/ssh/sshd_config file, save the file, and restart the sshd server to effect the changes using the following command:
sudo systemctl restart sshd.service

# in case of using SSH keys don't forget to copy the id_rsa.pub file to the remote host and append it to ~/.ssh/authorized_keys, then give it the right permissions:
chmod 600 .ssh/authorized_keys
```


### Free Up Disk Space

You can check the size of interested directory with `du -sh <directory>` preliminarily.

```shell
# clean the thumbnail cache
rm -rf ~/.cache/thumbnails/*

# remove packages that were automatically installed to satisfy dependencies for other packages and are now no longer needed
sudo apt autoremove

# clean the apt cache
sudo apt-get clean

# remove old, unused kernels, except for the last two - the current and the previous
sudo purge-old-kernels
```


### Samba setup

```shell
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


### Samba share access (unrestricted)

```shell
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

```shell
sudo service smbd restart
```


### Samba share access (restricted)

```shell
sudo mkdir -p /samba/share/secured
sudo addgroup securedgroup
cd /samba/share
sudo chown -R zhibirc:securedgroup secured
sudo chmod -R 0770 secured/
sudo usermod -a -G securedgroup zhibirc
sudo smbpasswd -a zhibirc
```

```shell
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

```shell
sudo service smbd restart
```


### Network goodies

Retrieve list of Samba master browser(s):

```shell
nmblookup -M -- -
```

Show NFS exports, like the ```showmount -e``` command:

```shell
nmap -sV --script=nfs-showmount 127.0.0.1
```

Mapping processes to system ports they listen for:

```shell
sudo netstat -tpln
```

Serve folder:

```shell
python -m SimpleHTTPServer 8080
# or
python3 -m http.server 8080
# or
sudo npm install http-server -g
http-server
```

Find out MAC address by using IP address:

```shell
arping -I eth0 -c 2 destination_ip
```

Using `arp-scan` allows to discover all IP hosts on the local network, including those that block all IP traffic such as firewalls and systems with ingress filters.
It works on Ethernet and 802.11 wireless networks. Requires root privilege.

```shell
# "eth0" is used for example, in reality the network interface name depends on the OS, the network type and other factors
sudo arp-scan --interface=eth0 --localnet
# or
sudo arp-scan --localnet
```


### Diff between files/folders

```shell
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

```shell
zip -P s0me_paSS -r protected.zip /home/sites/*/www/
```

Create encrypted ZIP archive (request to enter password), different choices:

```shell
zip --encrypt protected.zip file_name
zip --encrypt protected.zip file1 file2 file3
zip --encrypt -r protected.zip /home/user/folder/
zip --encrypt -r protected.zip /folder1/ /folder2/
```

ZIP supports a simple password-based symmetric encryption system, which is documented in the ZIP specification, and known to be **seriously flawed**, 
so don't use it for data with limited access.


### Pumping .bash_aliases

It's possible to put a lot of useful shortcuts in `~/.bash_aliases` which can improve work effectiveness:

```shell
# General aliases
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

# Install NPM packages in Docker container to prevent security flaws
alias dnpm='docker run -it --rm -u=$UID:$(id -g $USER) -v "$PWD":/npm -w /npm node npm'
alias dnpx='docker run -it --rm -u=$UID:$(id -g $USER) -v "$PWD":/npm -w /npm node npx'
alias dnode='docker run -it --rm -u=$UID:$(id -g $USER) -v "$PWD":/npm -w /npm node node'
alias dyarn='docker run -it --rm -u=$UID:$(id -g $USER) -v "$PWD":/npm -w /npm node yarn'
```


### Encrypt/decrypt a file

Use the built-in **gpg** tool:

```shell
# encrypt
gpg -c important.data.txt
# decrypt
gpg important.data.txt.gpg
```


### Stress test via DoS attack

Using **ab** (Apache HTTP server benchmarking tool).
Official docs: [link](https://httpd.apache.org/docs/2.4/programs/ab.html)

```shell
ab -k -c 350 -n 20000 example.com
```

For testing multiple URL's concurrently create a shell script with multiple `ab` calls:

```shell
#!/bin/sh

ab -n 100 -c 10 example.com/login > test1.txt &
ab -n 100 -c 10 example.com/news > test2.txt &
```

Using **Siege**:

```shell
siege -d10 -c50 example.com
```


### cURL cheatsheet

Debug options `--verbose` (`-v`), `--trace`, `--trace-ascii`, `--trace-time` allow to get more details as they show EVERYTHING **curl** sends and receives.

```shell
# use "-" as filename to have the output sent to stdout
curl --trace-ascii - http://www.example.com/
```

Make GET request, only print the response headers and display the time it took:

```shell
curl -sIX GET -w "Total time: %{time_total} s\n" www.example.com
# or
curl -o /dev/null -D- www.example.com
```

Typical usage, send GET request with headers:

```shell
curl -X GET 'http://www.example.com' -H 'Accept-Language: en' -H 'Authorization: Bearer A0v7mf98JJvWQTEbpEYNTt0uw2q0yl6P' -H 'Content-Type: application/json'
```

POST request format depends on content type (`application/x-www-form-urlencoded` is the default):

```shell
# or simply -d
curl --data "param1=value1&param2=value2" -X POST https://example.com/resource.cgi
curl -d '{"key1":"value1", "key2":"value2"}' -H "Content-Type: application/json" -X POST http://www.example.com
```

Identify the HTTP options available on the target URL, including the various types of allowed HTTP methods:

```shell
curl -v -X OPTIONS http://www.example.com/
```


### Wget basics

Downloading an entire Web Site:

```shell
# download the entire Web site
# convert links so that they work locally, off-line
# download all the files that are necessary to properly display a given HTML page
# guarantee that only the files below a certain hierarchy will be downloaded
# wait the specified number of seconds between the retrievals
# cause the time between requests to vary between 0.5 and 1.5 * wait (see above) seconds
wget --recursive --convert-links --page-requisites --no-parent --wait=5 --random-wait http://www.example.com/
```

You may want to specify `--user-agent` option which allows you to change the "**User-Agent**" line.
Specifying empty user agent with `--user-agent=""` instructs Wget not to send the "**User-Agent**" header in HTTP requests.


### Installing programs from sources

```shell
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

```shell
sudo add-apt-repository ppa:webupd8team/java
sudo apt update; sudo apt install oracle-java8-installer
# or replace oracle-java8-installer with oracle-java9-installer to install Java 9
# check the Java version
javac -version
# set Java environment variables
sudo apt install oracle-java8-set-default
```


### FFmpeg sweets

Get metadata information from media file:

```shell
# work on any file FFmpeg supports
ffmpeg -i video.mp4 -hide_banner
# advanced method using FFprobe, multimedia stream analyzer
ffprobe -v error -show_format -show_streams video.mp4
```

Convert MP4 video to MP3 audio:

```shell
ffmpeg -i video.mp4 audio.mp3
# or, with additional options
ffmpeg -i video.mp4 -b:a 192k -vn audio.mp3
```

Convert RTSP stream to HLS:

```shell
ffmpeg -i rtsp://184.72.239.149/vod/mp4:BigBuckBunny_115k.mov -fflags flush_packets -max_delay 2 -flags -global_header -hls_time 2 -hls_list_size 3 -vcodec copy -y video.m3u8
```

Split a video into images:

```shell
mkdir video; ffmpeg -i video.mp4 image%d.jpg
```

Reduce the file size of MP4 file:

```shell
# get file information
ffmpeg -i video.mp4
# 497 kb/s, 30 fps, 30 tbr, 15360 tbn, 60 tbc (default)
# reduce bitrate by approximately half
ffmpeg -i video.mp4 -b 248k video.out.mp4
```

Crop video file:

```shell
ffmpeg -i video.mp4 -ss 00:00:03 -t 00:00:08 -async 1 fragment.mp4
```


### Getting file info

```shell
# display file or file system status
stat file.name
# get basic file info, recognize the type of data contained in
file file.name
# or, for getting mime type
file -i file.name
# read image metadata, ImageMagick is required
# get format and characteristics of one or more image files
identify -verbose file.name 
```

In case of media file container used by a multimedia stream use information from [FFmpeg sweets](#ffmpeg-sweets).


### Getting hardware info

Overall:

```shell
uname -a

sudo dmidecode | less

sudo lshw | less

# pretty print
sudo lshw -html > system_info.html
```

Specific:

```shell
# list USB devices
lsusb

# list all PCI devices
lspci

# CPU
cat /proc/cpuinfo
# or more precise form
lscpu

# RAM
cat /proc/meminfo
free

# SSD/HDD
sudo fdisk -l
```

Or, if you prefer some GUI tool, use `hardinfo` (`sudo apt install hardinfo`).


### Generate a random password


### Amazon Web Services (AWS)

Setup:

```shell
sudo apt install awscli
aws --version

# configuring
aws configure
AWS Access Key ID [None]: <associated with an IAM user>
AWS Secret Access Key [None]: <associated with an IAM user>
Default region name [None]: <any available region>
Default output format [None]: <json|text|table>

# or configure the same for another user
aws configure --profile <username> 
```

```shell
# enable command-completion feature

# locate the AWS Completer script, use this path in command below
which aws_completer

# in ~/.bashrc
complete -C '/usr/bin/aws_completer' aws
source ~/.bashrc 
```


### Docker

[**Documentation**](https://docs.docker.com/)

Clean-up (containers, images, networks, cache):

```shell
alias d='docker' && d stop $(d ps -q) && d rm $(d ps -qa) && d rmi $(d images -q) && d network prune -f && d builder prune -f
```

Remove exited containers:

```shell
docker rm $(docker ps -a -q -f status=exited)
```

Build image in verbose mode (auto, plain, tty modes are available):

```shell
docker build . --tag <tag_name> --no-cache --progress=plain
```


### Cool cheatsheets / tutorials / helper tools

* [Bash-hackers wiki](http://wiki.bash-hackers.org/) _(bash-hackers.org)_
* [Shell vars](http://wiki.bash-hackers.org/syntax/shellvars) _(bash-hackers.org)_
* [Learn bash in y minutes](https://learnxinyminutes.com/docs/bash/) _(learnxinyminutes.com)_
* [Bash Guide](http://mywiki.wooledge.org/BashGuide) _(mywiki.wooledge.org)_
* [ShellCheck](https://www.shellcheck.net/) _(shellcheck.net)_


### Bash-Snippets [![Awesome](https://cdn.rawgit.com/sindresorhus/awesome/d7305f38d29fed78fa85652e3a63e154dd8e8829/media/badge.svg)]()

A collection of small bash scripts for heavy terminal users with no dependencies: [link](https://github.com/alexanderepstein/Bash-Snippets)
