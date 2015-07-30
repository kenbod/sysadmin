# Instructions for Installing Spark

We will be installing the latest version of Spark. In July 2015, that was Spark 1.4.1. We will be configuring it to make use of Hadoop's cluster manager, YARN. See my instructions for [installing Hadoop](https://github.com/kenbod/sysadmin/blob/master/hadoop.md). A core requirement of Spark is that a JDK is installed. Be sure to use my instructions for [installing Java](https://github.com/kenbod/sysadmin/blob/master/java.md) if necessary.

## Download Spark

1. Head to &lt;http://spark.apache.org/downloads.html&gt;
2. Choose a binary version of Spark built for Hadoop 2.6 and later.
3. In July 2015, the generated download link was for `spark-1.4.1-bin-hadoop2.6.tgz`.
4. scp the archive to the VM where you intend to submit jobs (for us, that is our Hadoop master node).

## Installing Spark to work with YARN

We will only be installing Spark on what was called <q>the master node</q> in my [Hadoop installation instructions](https://github.com/kenbod/sysadmin/blob/master/hadoop.md).

1. Take the Spark archive and unpack it in `/usr/local/src`
2. Rename it: `sudo mv spark-1.4.1-bin-hadoop2.6 spark-1.4.1`
2. Create a spark group: `sudo addgroup spark`
3. Create a spark user: `sudo adduser --ingroup spark spark`
4. Change the owner for the spark distribution: `sudo chown -R spark:spark spark-1.4.1`

### Add Spark's bin directories to the path

1. Add the following line to `/etc/environment`

        SPARK_HOME="/usr/local/src/spark-1.4.1"

2. Add the following line to `/etc/bash.bashrc`

        export PATH=$PATH:$SPARK_HOME/bin:$SPARK_HOME/sbin

3. Logout and login back in and check to see that you have `hadoop` in your path

### Configure Spark so that it knows about the Hadoop Installation

1. `cd /usr/local/src/spark-1.4.1/conf`
2. `sudo cp spark-env.sh.template spark-env.sh`
3. `sudo chown spark:spark spark-env.sh`

Now edit the spark-env.sh file. Uncomment the line that defines the HADOOP_CONF_DIR environment variable. Make it look like this:

    HADOOP_CONF_DIR="/usr/local/src/hadoop-2.7.1/etc/hadoop"
