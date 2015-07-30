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

## Installing HDFS in Cluster Mode

**WARNING:** As you will see, installing HDFS is NOT for the faint-hearted.

**NOTE:** These instructions are adapted from the information that appears in Chapter 10 of [Hadoop: The Definitive Guide, 4th Edition](http://shop.oreilly.com/product/0636920033448.do). The text in that chapter is written at a high level of abstraction. My instructions below contain way more details. :smiley:

### Overview

We will be configuring multiple machines to run as a HDFS cluster. In my case, I created five VMs: one `master` node and four `worker` nodes: `worker1`, `worker2`, etc. **NOTE:** Unless otherwise indicated, you need to follow the instructions below for each node in your cluster.

### Configuring `/etc/hosts`

Edit the `/etc/hosts` file on **each node in your cluster** so you can refer to each machine by their name and not by their IP address. Edit the file (`sudo vi /etc/hosts`) to look something like this:

```
123.1.1.25 master
123.1.1.26 worker01
123.1.1.27 worker02
123.1.1.28 worker03
123.1.1.29 worker04
```

**Note:** You will need to change the IP addresses listed above to match your set-up.

Once you have saved the file, you can test that the changes worked with the ping command:

```
ping master
ping worker01
ping ...
```

### Installing Java

Make sure that your machines have Java installed. See [my instructions](https://github.com/kenbod/sysadmin/blob/master/java.md) or use the ~1.5B pages on the Internet that also discuss how to do that.

### Create User Accounts

We need to create user accounts for `hadoop` and `hdfs`. You will need to create passwords for each account and keep track of them separately.

1. `sudo addgroup hadoop`
2. `sudo adduser --ingroup hadoop hadoop`
3. `sudo adduser --ingroup hadoop hdfs`

### Installing Hadoop

Let's place the previously downloaded Hadoop archive into a known location. I decided to use `/usr/local/src` for that. You can use a different directory if you want. 

1. cd to the directory that contains the Hadoop archive.
2. `sudo mv <archive> /usr/local/src`
3. `cd /usr/local/src`
4. `sudo tar xzf <archive>`
5. `sudo rm <archive>` (Optional)
6. `sudo chown -R hadoop:hadoop hadoop-2.7.1` (change to match the version you downloaded)

### Configuring the Global Environment

We need to make sure that the accounts on each machine have access to Hadoop (and by extension, HDFS) and its programs.

1. Add the following line to `/etc/environment`: `sudo vi /etc/environment`

        HADOOP_HOME="/usr/local/src/hadoop-2.7.1"

2. Add the following line to `/etc/bash.bashrc`

        export PATH=$PATH:$HADOOP_HOME/bin:$HADOOP_HOME/sbin

3. Logout and login back in and check to see that you have `hadoop` in your path

Note: Be sure your version of HADOOP_HOME matches the location where you unpacked the Hadoop archive. Recall that I used `/usr/local/src` but you are free to use other directories. I won't mention this issue again moving forward but be aware of it.

