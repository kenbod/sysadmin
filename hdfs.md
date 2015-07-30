# Instructions for Installing HDFS

I previously had the instructions below at the beginning of my instructions on how to
[install Hadoop](https://github.com/kenbod/sysadmin/blob/master/hadoop.md). However, I have come to realize
that you can install Hadoop and not move on to installing/configuring Hadoop's other services (YARN, MapReduce, etc.)
and so I decided to split the instructions. If you just need to install HDFS to, for instance, use it
in conjunction [with Spark](https://github.com/kenbod/sysadmin/blob/master/spark.md), read on.

## Downloading Hadoop

Even though we are just installing HDFS, that still means downloading a version of Hadoop. In July 2015, the latest
version of Hadoop was 2.7.1. So:

1. Head to &lt;http://hadoop.apache.org/releases.html&gt;
2. Download the binary version of Hadoop: `hadoop-2.7.1.tar.gz`.
3. `scp` the archive to all of the machines that you want to participate in the HDFS cluster. 
