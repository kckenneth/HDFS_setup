# Terasort

On HDFS1 node, this will generate a 10GB set and ingest it into the distributed filesystem. The input to teragen is the number of 100 byte rows. There are three commands you need to be familiar with tera.  
1. teragen 
2. terasort 
3. teravalidate 

```
# cd /usr/local/hadoop/share/hadoop/mapreduce
# hadoop jar $(ls hadoop-mapreduce-examples-2*.jar) teragen 100000000 /terasort/in
```
#### Let's sort
This will take minutes. 
```
# hadoop jar $(ls hadoop-mapreduce-examples-2*.jar) terasort /terasort/in /terasort/out
```

#### Let's validate
```
# hadoop jar $(ls hadoop-mapreduce-examples-2*.jar) teravalidate /terasort/out /terasort/val
```
