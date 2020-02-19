# MFS-Config
A document summarizing the configuration of MooseFS on an oDroid HC2

Download Armbian: https://www.armbian.com/odroid-hc1/
Write OS image to SD card
Boot HC2
ssh into HC2 using root/1234
groupadd mfs
useradd -G mfs -m mfs
passwd mfs
Logout and login with new id
sudo apt update
sudo apt upgrade
armbian-config /system/dtb - Set for HC2
sudo reboot
Configure network settings:
armbian-config network
sudo apt install unzip xfsprogs gdisk net-tools dnsutils python
Update to fixed IP:
sudo nano /etc/netplan/01-netcfg.yaml

Fill in information below and replace <xx> with appropriate IPs:
# This file describes the network interfaces available on your system
# For more information, see netplan(5).
network:
  version: 2
  renderer: networkd
  ethernets:
    eth0:
     dhcp4: no
     addresses: [<ip>/24]
     gateway4: <GatewayIP>
     nameservers:
       addresses: [<dns1>,<dns2>]



If using standard config - I recommended compiling yourself, instructions below
Add key:
sudo wget -O - https://ppa.moosefs.com/moosefs.key | sudo apt-key add -

Add RPI deb: 
Armhf(HC2): deb http://ppa.moosefs.com/moosefs-3/apt/raspbian/jessie jessie main
X86: deb http://ppa.moosefs.com/moosefs-3/apt/ubuntu/bionic bionic main
To: /etc/apt/sources.list.d/moosefs.list

Install packages:
Master Server
sudo apt install moosefs-master moosefs-cgi moosefs-cgiserv moosefs-cli moosefs-chunkserver

Chunkserver/Metalogger
sudo apt install moosefs-chunkserver moosefs-metalogger

Chunkserver
sudo apt install moosefs-chunkserver

Clients needing access MooseFS
sudo apt install moosefs-client

If building yourself -- recommended
sudo apt install build-essential libpcap-dev zlib1g-dev fuse libfuse-dev pkg-config
wget https://github.com/moosefs/moosefs/archive/master.zip
unzip master.zip
Navigate into resulting folder
sudo ./linux_build.sh
sudo make install


Now we configure the disks - We assume the disk is devoted to MFS and so do not create partitions
Verify disk exists and appropriate path.  On all my HC2’s it is /dev/sda:
ls /dev/sd*

sudo mkfs.xfs -s size=4K /dev/sda

Edit /etc/fstab
sudo nano /etc/fstab

Add the text below to the bottom.
/dev/sda /mnt/mfschunks0 xfs nodev,noatime,nodiratime 0 2
sudo mount /dev/sda

sudo mkdir /mnt/mfschunks0
sudo chown mfs:mfs /mnt/mfschunks0
sudo chmod 770 /mnt/mfschunks1
sudo mount /mnt/mfschunks0

Configure MFS through cfg files:
/etc/mfs

Consideration:
MooseFS slaves like to use DNS to find mfsmaster.  If possible set the DNS name of the master as ‘mfsmaster’. (See 2.1 in the PDF linked below)

MooseFS install page:
Installation page - we have already installed the app and performed most steps.  This is more reference and there are some additional steps related to MFS config: https://moosefs.com/blog/how-install-moosefs/

MooseFS installation pdf:
https://moosefs.com/Content/Downloads/moosefs-installation.pdf

