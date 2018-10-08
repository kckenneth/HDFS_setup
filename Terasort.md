# Terasort

On HDFS1 node, this will generate a 10GB set and ingest it into the distributed filesystem. The input to teragen is the number of 100 byte rows. There are three commands you need to be familiar with tera.  
1. teragen 
2. terasort 
3. teravalidate 

`teragen` creates sample data and places it in an output directory.  
`terasort` runs through the directory and creates the reduced output on an output directory.  
`teravalidate` ensures that terasort reduced and mapped correctly.

To check if there's any file in HDFS
```
# hadoop fs -ls /
```
#### I. Load teragen

Teragen, as the name implies, generates terabytes worth of random data which is to be sorted later in HDFS cluster. It's a kind of benchmarks to test the HDFS cluster system. The generated data will be dropped in the HDFS path you defined, `/terasort/in`. The directory shouldn’t exist in advance and teragen will automatically create the directory defined. If it already exists, teragen will throw an error.

```
# cd /usr/local/hadoop/share/hadoop/mapreduce
# hadoop jar $(ls hadoop-mapreduce-examples-2*.jar) teragen 100000000 /terasort/in
```
Note 
In shell commands, `$()` is a command. `${}` is a parameter substitution. So `$(ls ..)` is list of the jar file and run with `hadoop jar` cli command. 

#### II. Let's sort
This will take minutes. 
```
# hadoop jar $(ls hadoop-mapreduce-examples-2*.jar) terasort /terasort/in /terasort/out
```

#### III. Let's validate
```
# hadoop jar $(ls hadoop-mapreduce-examples-2*.jar) teravalidate /terasort/out /terasort/val
```

### Saving File from HDFS to local

```
# hadoop fs -get </HDFS/path> </local/path/>
```
