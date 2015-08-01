# Instructions for Installing Spark

We will be installing the latest version of Spark. In July 2015, that was Spark 1.4.1. We will be configuring it to run in <q>standalone mode</q>. That just means that Spark will use its own cluster manager rather than a 3rd party cluster manager like YARN or Mesos. We will, however, run our Spark cluster on a set of machines that has HDFS installed and running. See my instructions for [installing HDFS](https://github.com/kenbod/sysadmin/blob/master/hdfs.md) for more details. A core requirement of Spark is that a JDK is installed. Be sure to use my instructions for [installing Java](https://github.com/kenbod/sysadmin/blob/master/java.md) if necessary. Finally, the best API to Spark is via Scala, so it is also good to have Scala installed. See my instructions for [installing Scala](https://github.com/kenbod/sysadmin/blob/master/scala.md) on Ubuntu machines for how to do that.

## Download Spark

1. Head to &lt;http://spark.apache.org/downloads.html&gt;
2. Choose a binary version of Spark built for Hadoop 2.6 and later. (Even though we will not be using YARN, downloading this distribution allows you to switch to YARN at a later point if needed.)
3. In July 2015, the generated download link was for `spark-1.4.1-bin-hadoop2.6.tgz`.
4. scp the archive to the VM where you intend to submit jobs (for us, that is our HDFS master node).

## Installing Spark

We will only be installing Spark on what was called <q>the master node</q> in my [HDFS installation instructions](https://github.com/kenbod/sysadmin/blob/master/hdfs.md).

1. Take the Spark archive and unpack it in `/usr/local/src`
2. Rename it: `sudo mv spark-1.4.1-bin-hadoop2.6 spark-1.4.1`
2. Create a spark group: `sudo addgroup spark`
3. Create a spark user: `sudo adduser --ingroup spark spark`
4. Change the owner for the spark distribution: `sudo chown -R spark:spark spark-1.4.1`

### Add Spark's bin directories to the path

1. Add the following line to `/etc/environment`

        SPARK_HOME="/usr/local/src/spark-1.4.1"

2. Add the following line to `/etc/bash.bashrc`. Note, on the master node, make sure this line appears **BEFORE** the similar line that we added for HDFS/Hadoop. The reason: you want the start-all.sh and stop-all.sh scripts to refer to the Spark versions and not the Hadoop versions.

        export PATH=$PATH:$SPARK_HOME/bin:$SPARK_HOME/sbin

3. Logout and login back in and check to see that you have `spark-shell` in your path

### Configure Spark so that it knows about the Hadoop Installation

We want our version of Spark to be able to access HDFS. In order to do that, it needs to know where HDFS's configuration files are located. That directory exists inside of the Hadoop installation.

1. `cd /usr/local/src/spark-1.4.1/conf`
2. `sudo cp spark-env.sh.template spark-env.sh`
3. `sudo chown spark:spark spark-env.sh`

Now edit the spark-env.sh file. Uncomment the line that defines the HADOOP_CONF_DIR environment variable. Make it look like this:

    HADOOP_CONF_DIR="/usr/local/src/hadoop-2.7.1/etc/hadoop"

### Configure Spark to be Less Chatty. (Optional)

If you run the `spark-shell` command, you will see a huge amount of output before the prompt appears. We can make Spark be less chatty by changing the log4j properties. To do that, follow these instructions:

1. `su - spark`
2. `cd /usr/local/src/spark-1.4.1/conf`
3. `cp log4j.properties.template log4j.properties`
4. Edit the `log4j.properties` file to look like this:

```
# set global logging severity to INFO
log4j.rootCategory=INFO, console, file

# console config (restrict only to ERROR and FATAL)
log4j.appender.console=org.apache.log4j.ConsoleAppender
log4j.appender.console.target=System.err
log4j.appender.console.threshold=ERROR
log4j.appender.console.layout=org.apache.log4j.PatternLayout
log4j.appender.console.layout.ConversionPattern=%d{yy/MM/dd HH:mm:ss} %p %c{1}: %m%n

# file config
log4j.appender.file=org.apache.log4j.RollingFileAppender
log4j.appender.file.File=/usr/local/src/spark-1.4.1/logs/info.log
log4j.appender.file.MaxFileSize=5MB
log4j.appender.file.MaxBackupIndex=10
log4j.appender.file.layout=org.apache.log4j.PatternLayout
log4j.appender.file.layout.ConversionPattern=%d{yy/MM/dd HH:mm:ss} %p %c{1}: %m%n

# Settings to quiet third party logs that are too verbose
log4j.logger.org.spark-project.jetty=WARN
log4j.logger.org.spark-project.jetty.util.component.AbstractLifeCycle=ERROR
log4j.logger.org.apache.spark.repl.SparkIMain$exprTyper=INFO
log4j.logger.org.apache.spark.repl.SparkILoop$SparkILoopInterpreter=INFO
```

**NOTE**: This particular configuration was shamelessly stolen from [Spark in Action](http://www.manning.com/bonaci/). Don't thank me, thank them!

**NOTE**: If you ever need to see the debug output of `spark-shell` or `spark-submit`, then edit this file and change the word `ERROR` with either `WARN` or `INFO`. Then, change it back to `ERROR` when you're done.

* Run `spark-shell` to see that most of the start-up logging has been eliminated.

### Configuring SSH Access

We need to create an RSA key pair for the `spark` account. The private and public key are stored on the master node. The public key for this account must then be copied to the corresponding locations on each of the worker nodes.

#### Creating the Key Pair

On the master node, run the following commands:

1. Switch to the spark user: `su - spark` 
2. `ssh-keygen -t rsa -f ~/.ssh/id_rsa`
3. Enter a passphrase for the private key and record it in a safe location.
4. `cat ~/.ssh/id_rsa.pub >> ~/.ssh/authorized_keys`

#### Copying the public key

We need to copy the public key from the master to each of the worker nodes. We will use `ssh-copy-id` for this and simply enter the password that you created for the `spark` account earlier.

On the master node, run the following commands **FOR EACH WORKER NODE:**

1. `su - spark`
2. `ssh-copy-id spark@worker01`
3. Say 'yes' if you are asked to accept the fingerprint of the worker01 machine
4. Enter the password for the spark account on worker01 if asked
5. Test the ability to login to the worker01 machine: `ssh worker01`
6. You will be asked for your passphrase. Type it and you will be logged in on the remote machine.

#### Configuring Keychain for the `spark` account

Login to the `spark` account and edit the .profile to make use of keychain

1. `su - spark`
2. `vi .profile` # Add the following lines to end of .profile file

        /usr/bin/keychain $HOME/.ssh/id_rsa
        source $HOME/.keychain/$HOSTNAME-sh

3. `exit`
4. `su - spark`
5. Keychain will ask you to enter your passphrase
6. Now if you type `ssh worker01`, you should be logged in without having to type your passphase
7. You will no longer need to enter your passphrase until your virtual machine is rebooted

### Create the unfortunately named `slaves` file

On the master node, we need to create a file called `slaves` to tell Spark where the worker nodes should live.

1. `su - spark`
2. `cd /usr/local/src/spark-1.4.1/conf`
3. `cp slaves.template slaves`
3. `vi slaves` # List all of your worker nodes in this file, one per line

        worker01
        worker02
        ...

### Configure Master Node Hostname

On the master node, we need to edit the `spark-env.sh` file to tell Spark the hostname of the master node. This is the same hostname that you used for the master node when setting up HDFS. For my configuration, it is simply `master`.

1. `su - spark`
2. `cd /usr/local/src/spark-1.4.1/conf`
3. `vi spark-env.sh`
4. Uncomment the variable `SPARK_MASTER_IP` and make it look like this:

        SPARK_MASTER_IP=master

Make sure to use whatever hostname you assigned your master node.

## Launching the Cluster

On the master node, to launch the cluster, execute the following steps:

1. `su - spark`

    Verify that the command `start-all.sh` is the Spark version and not the Hadoop version. If it is the Hadoop version, then update your path so that the Spark version of the command takes precendence over the Hadoop version of the command.

2. `start-all.sh`

## Testing the Cluster

1. Access Spark's web user interface by navigating in a web browser to:

        <http://master:8080/>

2. Verify that all workers are listed in this interface and that you can navigate to their web interface via the links.

3. Next, try connecting the spark-shell to this cluster. On the master node, execute:

        spark-shell --master spark://master:7077

## Note on Memory

By default, a Spark worker node will use all cores on its machine and will grab all available memory on the machine minus 1 GB. These defaults should be good for most situations. However, be aware that `spark worker` != `spark executor` and it is the `spark executor` that executes application code. Thus, a Spark worker node that has four cores will typically launch four executors when it is asked to do something and have those four executors split the work for that node between them.

How much memory does an executor get? 512MB by default. Even if the worker node has more memory available. So, in my initial set-up, my worker nodes have 4 cores and 8 GBs of memory. The worker would grab 7 GB of memory and launch four executors with 512MB of memory each when I submitted a task to the cluster, consuming only 2 GB of the 7GB of memory available.

How do you give the executors more memory? You do it programmatically when creating your `SparkConf` object by explicitly setting the `spark.executor.memory` property.

```scala
val conf = new SparkConf().setAppName("Shortest Path").set("spark.executor.memory", "1024m")
```

This tells Spark to use 1GB of memory per executor on each worker node. For my set-up, that allows 4 GB of the 7GB available to be consumed. Note: If I specify 2GB for this parameter, the job still runs but I think it would lead to out-of-memory errors or performance problems if my job actually required each executor to use 2GB of memory each. The worker has 7 GB of memory total so if all 4 workers try to grab 2GB each either one of them dies with an out-of-memory error or virtual memory kicks in slowing the execution of all the executors on that node.
(1024*1024*1024*7)/4/1000/1000 ~= 1879m
(1048576*7)/4/1000
