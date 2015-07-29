# Instructions for Installing Hadoop

I decided to install the latest version of Hadoop. In July 2015, that was Hadoop 2.7.1.

## Downloading Hadoop

1. Head to &lt;http://hadoop.apache.org/releases.html&gt;
2. Download the binary version of Hadoop: hadoop-2.7.1.tar.gz.
3. scp the archive to the VMs that need it.

## Installing Hadoop in Cluster Mode

**WARNING:** As you will see, installing Hadoop is NOT for the faint-hearted.

**NOTE:** These instructions are adapted from the information that appears in Chapter 10 of [Hadoop: The Definitive Guide, 4th Edition](http://shop.oreilly.com/product/0636920033448.do). The text in that chapter is written at a high level of abstraction. My instructions below contain way more details. :smiley:

### Overview

We will be configuring multiple machines to run as a Hadoop cluster. In my case, I created five VMs: one `master` node and four `worker` nodes: `worker1`, `worker2`, etc. **NOTE:** Unless otherwise indicated, you need to follow the instructions below for each node in your Hadoop cluster.

### Configuring `/etc/hosts`

Edit the `/etc/hosts` file on each node in your cluster so you can refer to each machine by their name and not by their IP address. Edit the file (`sudo vi /etc/hosts`) to look something like this:

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

We need to create an RSA key pair for the hdfs and yarn accounts. The private and public keys are stored on the master node. The public keys for each of these accounts must then be copied to the corresponding locations on each of the worker nodes.

#### Creating the keypairs

On the master node, run the following commands:

1. `su - hdfs` # Switch to be the hdfs user
2. `ssh-keygen -t rsa -f ~/.ssh/id_rsa`
3. Be sure to enter a passphrase for the key pair and record it in a safe location.
4. `cat ~/.ssh/id_rsa.pub >> ~/.ssh/authorized_keys`
5. `su - yarn` # Switch to the yarn user
6. `ssh-keygen -t rsa -f ~/.ssh/id_rsa`
7. Again, enter a (different) passphrase for the key pair and record it.
8. `cat ~/.ssh/id_rsa.pub >> ~/.ssh/authorized_keys`

#### Copying the public keys

We need to copy the `authorized_keys` files from the master to each of the worker nodes. We will use scp for this and simply enter the password that you created for the hdfs and yarn accounts earlier. On the master node, run the following commands:

1. `su - hdfs`
2. `ssh-copy-id hdfs@worker01`
3. Say 'yes' if you are asked to accept the fingerprint of the worker01 machine
4. Enter the password for the hdfs account on worker01 if asked
5. Test the ability to login to the worker01 machine: `ssh worker01`
6. You will be asked for your passphrase. Type it and you will be logged in on the remote machine.

#### Configuring Keychain

As the ubuntu account on your master node, install keychain

1. `sudo apt-get update`
2. `sudo apt-get install keychain`

Then login to the hdfs account and edit the .profile to make use of keychain

1. `su - hdfs`
2. `vi .profile` # Add the following lines to end of .profile file

        /usr/bin/keychain $HOME/.ssh/id_rsa
        source $HOME/.keychain/$HOSTNAME-sh
3. `exit`
4. `su - hdfs`
5. Keychain will ask you to enter your passphrase
6. Now if you type `ssh worker01`, you should be logged in without having to type your passphase
7. You will no longer need to enter your passphrase until your virtual machine is rebooted



