|Title |  GPFS Installation |
|-----------|----------------------------------|
|Author | Kenneth Chen |
|Utility | IBM, Softlayer, HDFS |
|Date | 9/10/2018 |

# HDFS Setup

Setup 3 virtual servers in the cloud. 
```
$ slcli vs create --datacenter=sjc01 --hostname=hdfs1 --domain=mids.com --billing=hourly --cpu=2 --memory=4096 --disk=25 --disk=100 --network=1000 --os=CENTOS_LATEST_64
$ slcli vs create --datacenter=sjc01 --hostname=hdfs2 --domain=mids.com --billing=hourly --cpu=2 --memory=4096 --disk=25 --disk=100 --network=1000 --os=CENTOS_LATEST_64
$ slcli vs create --datacenter=sjc01 --hostname=hdfs3 --domain=mids.com --billing=hourly --cpu=2 --memory=4096 --disk=25 --disk=100 --network=1000 --os=CENTOS_LATEST_64
```

```
$ slcli vs list
:..........:..........:...............:..............:............:........:
:    id    : hostname :   primary_ip  :  backend_ip  : datacenter : action :
:..........:..........:...............:..............:............:........:
: 63000755 :  hdfs1   : 198.23.88.164 : 10.91.105.18 :   sjc01    :   -    :
: 63000769 :  hdfs2   :  198.23.82.40 : 10.91.105.41 :   sjc01    :   -    :
: 63000775 :  hdfs3   : 198.23.88.165 : 10.91.105.22 :   sjc01    :   -    :
:..........:..........:...............:..............:............:........:
```
Open 3 separate terminals to ssh into 3 vs. Make sure you got the passwords for each vs by calling 
```
$ slcli vs credentials 63000755

:..........:..........:
: username : password :
:..........:..........:
:   root   :          :
:..........:..........:

$ ssh root@198.23.88.164
```

#### Change the DNS in each node

```
# vi /etc/hosts

127.0.0.1     localhost.localdomain localhost
198.23.88.164 hdfs1.hadoop.mids.lulz.bz hdfs1
198.23.82.40  hdfs2.hadoop.mids.lulz.bz hdfs2
198.23.88.165 hdfs3.hadoop.mids.lulz.bz hdfs3
```

Check disks and which is a mounted disk
```
# fdisk -l | grep Disk | grep GB
Disk /dev/xvdc: 107.4 GB, 107374182400 bytes, 209715200 sectors
Disk /dev/xvda: 26.8 GB, 26843545600 bytes, 52428800 sectors

# mount | grep ' \/ '
/dev/xvda2 on / type ext3 (rw,noatime,seclabel,data=ordered)
```

Making the file allocation (inode)
```
# mkdir /data
# mkfs.ext4 /dev/xvdc

mke2fs 1.42.9 (28-Dec-2013)
Filesystem label=
OS type: Linux
Block size=4096 (log=2)
Fragment size=4096 (log=2)
Stride=0 blocks, Stripe width=0 blocks
6553600 inodes, 26214400 blocks
1310720 blocks (5.00%) reserved for the super user
First data block=0
Maximum filesystem blocks=2174746624
800 block groups
32768 blocks per group, 32768 fragments per group
8192 inodes per group
Superblock backups stored on blocks: 
	32768, 98304, 163840, 229376, 294912, 819200, 884736, 1605632, 2654208, 
	4096000, 7962624, 11239424, 20480000, 23887872

Allocating group tables: done                            
Writing inode tables: done                            
Creating journal (32768 blocks): done
Writing superblocks and filesystem accounting information: done   
```

#### Defining the disk path

```
# vi /etc/fstab

/dev/xvdc /data             ext4  defaults,noatime        0  0
```

It will look like this
```
UUID=afb76a75-314e-4fb8-9cca-79c61c4a10d8 /                       ext3    defaults,noatime,noatime        1 1
UUID=deb85407-6f2a-475d-9018-9dfeeeb8f3fd /boot                   ext3    defaults,noatime,noatime        1 2
LABEL=SWAP-xvdb1 swap swap    defaults        0 0

/dev/xvdc /data             ext4  defaults,noatime        0  0
```
#### Mount the disk 
```
# mount /data
# chmod 1777 /data
```


