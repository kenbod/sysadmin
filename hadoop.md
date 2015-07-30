# Instructions for Installing Hadoop

My instructions for installing Hadoop assume that you have already followed my instructions for [installing HDFS](https://github.com/kenbod/sysadmin/blob/master/hdfs.md). Go follow those instructions and then return here.

With HDFS installed, we now need to configure Hadoop's YARN and MapReduce services.

## Create User Accounts

We need to create additional user accounts for `mapred` and `yarn`. You will need to create passwords for each account and keep track of them separately.

1. `sudo adduser --ingroup hadoop mapred`
2. `sudo adduser --ingroup hadoop yarn`

### Configuring SSH Access

We need to create an RSA key pair for the `yarn` account. The private and public key are stored on the master node. The public key for this account must then be copied to the corresponding locations on each of the worker nodes.

#### Creating the Key Pair

On the master node, run the following commands:

1. Switch to the yarn user: `su - yarn` 
2. `ssh-keygen -t rsa -f ~/.ssh/id_rsa`
3. Enter a passphrase for the private key and record it in a safe location.
4. `cat ~/.ssh/id_rsa.pub >> ~/.ssh/authorized_keys`

#### Copying the public keys

We need to copy the public key from the master to each of the worker nodes. We will use `ssh-copy-id` for this and simply enter the password that you created for the `yarn` account earlier.

On the master node, run the following commands **FOR EACH WORKER NODE:**

1. `su - yarn`
2. `ssh-copy-id yarn@worker01`
3. Say 'yes' if you are asked to accept the fingerprint of the worker01 machine
4. Enter the password for the yarn account on worker01 if asked
5. Test the ability to login to the worker01 machine: `ssh worker01`
6. You will be asked for your passphrase. Type it and you will be logged in on the remote machine.

#### Configuring Keychain for the `yarn` account

Login to the `yarn` account and edit the .profile to make use of keychain

1. `su - yarn`
2. `vi .profile` # Add the following lines to end of .profile file

        /usr/bin/keychain $HOME/.ssh/id_rsa
        source $HOME/.keychain/$HOSTNAME-sh

3. `exit`
4. `su - yarn`
5. Keychain will ask you to enter your passphrase
6. Now if you type `ssh worker01`, you should be logged in without having to type your passphase
7. You will no longer need to enter your passphrase until your virtual machine is rebooted

### Configuring the Cluster

#### Create directories for Hadoop and HDFS

We are now going to create three directories in the `/home/hadoop/` user account to store files for Hadoop's operation. I don't think that all three of these directories need to be created on all nodes in the cluster but, just to be safe, perform these commands on the master and all of the workers.

Create a directory for Hadoop to store temporary files. This may only be necessary for the master node but, to be safe, perform these instructions on the master and all of the workers

1. `su - hdfs`
2. `mkdir tmp` # Create a directory for Hadoop to store temporary files
3. `mkdir namenode` # Create a directory for Hadoop's NameNode files
4. `mkdir datanode` # Create a directory for Hadoop's DataNode files
5. `mkdir /usr/local/src/hadoop-2.7.1/logs` # Create directory for Hadoop's log files

Verify with the `ls -l` command that the permissions for these directories are: `drwxrwxrwx`

#### Edit `core-site.xml`

Edit the file `/usr/local/src/hadoop-2.7.1/etc/hadoop/core-site.xml` on all nodes of the cluster such that the `configuration` tag now looks like this:

```xml
<configuration>
  <property>
    <name>hadoop.tmp.dir</name>
    <value>/home/hdfs/tmp</value>
    <description>Hadoop Directory for Temporary Files</description>
  </property>

  <property>
    <name>fs.defaultFS</name>
    <value>hdfs://master:54310</value>
    <description>Use HDFS as file storage engine</description>
  </property>
</configuration>
```

This tells Hadoop where the temp directory is located and that we are going to be using HDFS to store our files.

#### Edit `hdfs-site.xml`

Edit the file `/usr/local/src/hadoop-2.7.1/etc/hadoop/hdfs-site.xml` on all nodes of the cluster such that the `configuration` tag now looks like this:

```xml
<configuration>
  <property>
    <name>dfs.replication</name>
    <value>2</value>
  </property>
  <property>
    <name>dfs.namenode.name.dir</name>
    <value>file:///home/hdfs/namenode</value>
  </property>
  <property>
    <name>dfs.datanode.data.dir</name>
    <value>file:///home/hdfs/datanode</value>
  </property>
</configuration>
```

This tells HDFS to store two copies of each file and it tells HDFS where to store its namenode files and its datanode files. I will need to update these instructions to place these directories on disk partitions that have a lot of disk space. For now, I'm just trying to get a simple multi-node configuration up and running.

#### Edit `yarn-site.xml`

Edit the file `/usr/local/src/hadoop-2.7.1/etc/hadoop/yarn-site.xml` on all nodes of the cluster such that the `configuration` tag now looks like this:

```xml
<configuration>
  <property>
    <name>yarn.nodemanager.aux-services</name>
    <value>mapreduce_shuffle</value>
  </property>
  <property>
    <name>yarn.resourcemanager.scheduler.address</name>
    <value>master:8030</value>
  </property>
  <property>
    <name>yarn.resourcemanager.address</name>
    <value>master:8032</value>
  </property>
  <property>
    <name>yarn.resourcemanager.webapp.address</name>
    <value>master:8088</value>
  </property>
  <property>
    <name>yarn.resourcemanager.resource-tracker.address</name>
    <value>master:8031</value>
  </property>
  <property>
    <name>yarn.resourcemanager.admin.address</name>
    <value>master:8033</value>
  </property>
</configuration>
```

This set-ups the ports for various parts of the YARN infrastructure. All of these components will run on the master node.

#### Edit `mapred-site.xml`

Edit the file `/usr/local/src/hadoop-2.7.1/etc/hadoop/mapred-site.xml` on the master node such that the `configuration` tag now looks like the file below. Note: first, you have to create this file...

Execute these commands in the `/usr/local/src/hadoop-2.7.1/etc/hadoop` directory:

1. `sudo cp mapred-site.xml.template mapred-site.xml`.
2. `sudo chown hadoop:hadoop mapred-site.xml`
3. `sudo vi mapred-site.xml`

```xml
<configuration>
  <property>
    <name>mapreduce.framework.name</name>
    <value>yarn</value>
  </property>
</configuration>
```

This tells map reduce jobs to run using the cluster; the default, otherwise, is to run locally on the client machine which defeats the purpose of Hadoop.

#### Edit the `slaves` file.

On the master node, edit the slaves files to look like this:

```
master
worker01
```

If you have more worker nodes, then list them below `worker01` above.

#### Format the name node directory

On the master node, perform these commands to format the namenode directory

1. `su - hdfs`
2. `hdfs namenode -format`

If all goes correctly, you should see the string `/home/hdfs/namenode has been successfully formatted.` somewhere in the massive amount of output generated by the second command. You can also cd to the namenode directory and see that it is no longer empty.

#### Start the HDFS daemons

On the master node, execute these commands:

1. `su - hdfs`
2. `start-dfs.sh`

On the master, run the command `jps` as the `hdfs` user, you should see something similar to:

```
26791 DataNode
26983 SecondaryNameNode
27098 Jps
26635 NameNode
```

This indicates that on the master, there is a name node, a secondary name node, and a data node running.

On one of the worker nodes, run the command `jps` as the `hdfs` user, you should see something similar to:

```
15073 Jps
14978 DataNode
```
This indicates that on this worker, a data node was successfully launched.

#### Start the YARN daemons

On the master node, execute these commands:

1. `su - yarn`
2. `start-yarn.sh`

This command should launch a resource manager and node manager on the master node and a node manager on each worker node. To test this, run the `jps` command as the `yarn` user on each node in your cluster to verify.

#### Start the MapReduce job history daemon.

MapReduce has one server process that needs to be started on the master node.

First, specify a new location for the history daemon's log files

1. Edit `/usr/local/src/hadoop-2.7.1/etc/hadoop/mapred-env.sh`
2. Add this line: `export HADOOP_MAPRED_LOG_DIR="/home/mapred/logs"`
3. Make sure to create that directory: `su - mapred`; `mkdir logs`; `chmod 777 logs`

Second, make sure that the `mapred` account is part of the HDFS `supergroup`:

1. `sudo addgroup supergroup`
2. `sudo usermod -a -G supergroup mapred`

Finally, run these commands to launch the history daemon.

1. `su - mapred`
2. `mr-jobhistory-daemon.sh start historyserver`



