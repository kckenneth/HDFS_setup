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

127.0.0.1 localhost.localdomain localhost
198.23.88.164 hdfs1.hadoop.mids.lulz.bz hdfs1
198.23.82.40  hdfs2.hadoop.mids.lulz.bz hdfs2
198.23.88.165 hdfs3.hadoop.mids.lulz.bz hdfs3
```

