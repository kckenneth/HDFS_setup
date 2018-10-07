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

# System Setup
Install 
```
# yum install -y rsync net-tools java-1.8.0-openjdk-devel ftp://fr2.rpmfind.net/linux/Mandriva/devel/cooker/x86_64/media/contrib/release/nmon-14g-1-mdv2012.0.x86_64.rpm
```
Setting the user and password. `passwd` will ask you to set the password for the user `hadoop`.
```
# adduser hadoop
# passwd hadoop
```

Install HDFS and extract in `/usr/local`
```
# cd /usr/local
# curl http://apache.claz.org/hadoop/core/hadoop-2.7.6/hadoop-2.7.6.tar.gz | tar -zx -C /usr/local --show-transformed --transform='s,/*[^/]*,hadoop,'
```

To allow permission on each directory. So far we have worked on `/data` and `/usr/local` directory. 
```
# chown -R hadoop.hadoop /data
# chown -R hadoop.hadoop /usr/local/hadoop
```

### Setting the passwordless between 3 nodes

`ssh-keygen` in hdfs1 will generate `id_rsa` (private) and `id_rsa.pub` (public) keys. You need to copy to other 2 nodes. When asked for password in each node, use the password you set earlier in adduser creation.
```
# ssh-keygen
# for i in hdfs1 hdfs2 hdfs3; do ssh-copy-id $i; done
```
If the passwordless functions as expected, you should be able to ssh between each node without password. Test in each node. `Ctrl+d` to log out from the node. 
```
# ssh hdfs1
Ctrl+d
# ssh hdfs2
Ctrl+d
# ssh hdfs3
Ctrl+d
```

You now need to define the hadoop path in `~/.bash_profile`. For java path, since we want to export the working path from the system, it's different from other paths export method. You could either do `|grep` line or just copy and paste the path I already retrieved. You can just `cat` all other paths or copy and paste in `~/.bash_profile` from the 2nd export method. 

##### 1. Export Method
```
# echo "export JAVA_HOME=\"$(readlink -f $(which javac) | grep -oP '.*(?=/bin)')\"" >> ~/.bash_profile
# cat <<\EOF >> ~/.bash_profile
export HADOOP_HOME=/usr/local/hadoop
export HADOOP_MAPRED_HOME=$HADOOP_HOME
export HADOOP_HDFS_HOME=$HADOOP_HOME
export YARN_HOME=$HADOOP_HOME
export PATH=$PATH:$HADOOP_HOME/bin:$HADOOP_HOME/sbin
EOF
```

##### 2. Export Method
```
# vi ~/.bash_profile

export JAVA_HOME="/usr/lib/jvm/java-1.8.0-openjdk-1.8.0.181-3.b13.el7_5.x86_64"
export HADOOP_HOME=/usr/local/hadoop
export HADOOP_MAPRED_HOME=$HADOOP_HOME
export HADOOP_HDFS_HOME=$HADOOP_HOME
export YARN_HOME=$HADOOP_HOME
export PATH=$PATH:$HADOOP_HOME/bin:$HADOOP_HOME/sbin
```

Activate the bash_profile
```
# source ~/.bash_profile
```
Check the java version
```
$ $JAVA_HOME/bin/java -version

openjdk version "1.8.0_181"
OpenJDK Runtime Environment (build 1.8.0_181-b13)
OpenJDK 64-Bit Server VM (build 25.181-b13, mixed mode)
```

## Hadoop Configuration Setup

```
# cd $HADOOP_HOME/etc/hadoop
# echo "export JAVA_HOME=\"$JAVA_HOME\"" > ./hadoop-env.sh
```
There are 4 xml files we will be updating.  
- core-site.xml 
- yarn-site.xml 
- mapred-site.xml.template 
- hdfs-site.xml 

After all those configurations setup, we will copy them into other 2 hdfs nodes. 

#### 1. core-site.xml
```
# vi core-site.xml

  <?xml version="1.0"?>
  <configuration>
    <property>
      <name>fs.defaultFS</name>
      <value>hdfs://master/</value>
    </property>
  </configuration>
```

#### 2. yarn-site.xml
```
vi yarn-site.xml

  <?xml version="1.0"?>
  <configuration>
    <property>
      <name>yarn.resourcemanager.hostname</name>
      <value>master</value>
    </property>
    <property>
      <name>yarn.nodemanager.aux-services</name>
      <value>mapreduce_shuffle</value>
    </property>
  </configuration>
```
If you want to use priviate IP for your cluster, 
```
      <property>
       <name>yarn.resourcemanager.bind-host</name>
       <value>0.0.0.0</value>
      </property>
```
If you're using 2CPU/4G node, you add this property to yarn-site.xml as well.
```
     <property>
        <name>yarn.nodemanager.resource.cpu-vcores</name>
        <value>2</value>
    </property>
    <property>
        <name>yarn.nodemanager.resource.memory-mb</name>
        <value>4096</value>
    </property>
```
#### 3. mapred-site.xml.template

```
vi mapred-site.xml.template

  <?xml version="1.0"?>
  <configuration>
    <property>
      <name>mapreduce.framework.name</name>
      <value>yarn</value>
    </property>
  </configuration>
```

#### 4. hdfs-site.xml
```
vi hdfs-site.xml

  <?xml version="1.0"?>
  <configuration>
    <property>
        <name>dfs.datanode.data.dir</name>
        <value>file:///data/datanode</value>
    </property>

    <property>
        <name>dfs.namenode.name.dir</name>
        <value>file:///data/namenode</value>
    </property>

    <property>
        <name>dfs.namenode.checkpoint.dir</name>
        <value>file:///data/namesecondary</value>
    </property>
  </configuration>
```

##### Copy all configuration files to other hdfs nodes
```
# rsync -a /usr/local/hadoop/etc/hadoop/* hadoop@hdfs1:/usr/local/hadoop/etc/hadoop/
# rsync -a /usr/local/hadoop/etc/hadoop/* hadoop@hdfs2:/usr/local/hadoop/etc/hadoop/
```
Change the nodes information in `slaves` file. Remove anything in there. 
```
# vi slaves

hdfs1
hdfs2
hdfs3
```

# Create HDFS FileSystem

First we will format the namenode before we spin up our cluster. If you format a running cluster, you will lose everthing. 
```
# hdfs namenode -format
```
On HDFS1 node (assume as Master node)
```
# start-dfs.sh
# start-yarn.sh
```
Check the HDFS status
```
# hdfs dfsadmin -report

18/10/07 16:36:55 WARN util.NativeCodeLoader: Unable to load native-hadoop library for your platform... using builtin-java classes where applicable
Configured Capacity: 316664487936 (294.92 GB)
Present Capacity: 300318064640 (279.69 GB)
DFS Remaining: 300317990912 (279.69 GB)
DFS Used: 73728 (72 KB)
DFS Used%: 0.00%
Under replicated blocks: 0
Blocks with corrupt replicas: 0
Missing blocks: 0
Missing blocks (with replication factor 1): 0

-------------------------------------------------
Live datanodes (3):

Name: 198.23.82.40:50010 (hdfs2.hadoop.mids.lulz.bz)
Hostname: ec2-54-208-77-124.compute-1.amazonaws.com
Decommission Status : Normal
Configured Capacity: 105554829312 (98.31 GB)
DFS Used: 28672 (28 KB)
Non DFS Used: 62955520 (60.04 MB)
DFS Remaining: 100106358784 (93.23 GB)
DFS Used%: 0.00%
DFS Remaining%: 94.84%
Configured Cache Capacity: 0 (0 B)
Cache Used: 0 (0 B)
Cache Remaining: 0 (0 B)
Cache Used%: 100.00%
Cache Remaining%: 0.00%
Xceivers: 1
Last contact: Sun Oct 07 16:47:42 CDT 2018
....
....
```
Check the YARN status
```
# yarn node -list
```
# Checking the cluster
Go to your browser
To check your cluster, browse to:  
master IP = 198.23.88.164. Don't change the port. 
```
http://master-ip:50070/dfshealth.html
http://master-ip:8088/cluster
http://master-ip:19888/jobhistory (for Job History Server) [might not work unless you have job running]
```
#### dfshealth.html

<p align="center">
<img src="img/dfs.png" width="600"></p>
<p align="center">Figure 1. DFS Health</p>

#### cluster

<p align="center">
<img src="img/cluster.png" width="600"></p>
<p align="center">Figure 2. Cluster Control</p>

# Next Step to run Terasort

https://github.com/kckenneth/HDFS_setup/blob/master/Terasort.md



