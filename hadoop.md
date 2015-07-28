# Instructions for Installing Hadoop

I decided to install the latest version of Hadoop. In July 2015, that was Hadoop 2.7.1.

## Downloading Hadoop

1. Head to &lt;http://hadoop.apache.org/releases.html&gt;
2. Download the binary version of Hadoop: hadoop-2.7.1.tar.gz.
3. scp the archive to the VMs that need it.

## Installing Hadoop in Cluster Mode

**WARNING:** As you will see, installing Hadoop is NOT for the faint-hearted.

**NOTE:** These instructions are adapted from the information that appears in Chapter 10 of [Hadoop: The Definitive Guide, 4th Edition](http://shop.oreilly.com/product/0636920033448.do). The text in that chapter is written at a high level of abstraction. My instructions below contain way more details. :smiley:

### Installing Java

Make sure that your machines have Java installed. See [my instructions](https://github.com/kenbod/sysadmin/blob/master/java.md) for doing that.

### Create User Accounts

We need to create user accounts for hadoop, hdfs, mapred, and yarn. You will need to create passwords for each account and keep track of them separately.

1. `sudo addgroup hadoop`
2. `sudo adduser --ingroup hadoop hadoop`
3. `sudo adduser --ingroup hadoop hdfs`
4. `sudo adduser --ingroup hadoop mapred`
5. `sudo adduser --ingroup hadoop yarn`

### Installing Hadoop

1. cd to the directory that contains the Hadoop archive that you downloaded earlier.
2. `sudo mv <archive> /usr/local/src`
3. `cd /usr/local/src`
4. `sudo tar xzf <archive>`
5. `sudo rm <archive>` (Optional)
6. `sudo chown -R hadoop:hadoop hadoop-2.7.1` (change to match the version you downloaded)

### Configuring the environment to have access to Hadoop

1. Add the following line to `/etc/environment`

        HADOOP_HOME="/usr/local/src/hadoop-2.7.1"

2. Add the following line to `/etc/bash.bashrc`

        export PATH=$PATH:$HADOOP_HOME/bin:$HADOOP_HOME/sbin

3. Logout and login back in and check to see that you have `hadoop` in your path

### Configuring SSH Access

More info soon!

