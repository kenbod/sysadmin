# Using HDFS

When you first set-up an HDFS cluster, the HDFS file system is <q>empty</q>.
You need to do a few things to get it set-up so that your users can store files in it
and reference those files in their Spark jobs.

**NOTE:** The commands below assume that you followed my instructions for
[installing HDFS](https://github.com/kenbod/sysadmin/blob/master/hdfs.md).

# Create the `/user` directory

HDFS assumes that a user's home directory is located in `/user/<username>` but the /user directory is not
created by default. So, let's get that created.

1. `su - hdfs`
2. `hadoop fs -mkdir /user`

# Create the home directory for a new user

1. `su - hdfs`
2. `hadoop fs -mkdir /user/<username>`
3. `hadoop fs -chown <username>:<username> /user/<username>`

Note: you should use the same usernames that you use <q>outside</q> HDFS.
HDFS looks to the user account on the machine to determine certain access rights.

# Solving permission problems with supergroup

If you run a Spark or MapReduce job and it fails because the job tried to write some data outside of your user directory on HDFS, you may have to add that user to the `supergroup` to allow that data to be created.

To do this, create the supergroup on the master node and then add the user to that group

1. `sudo addgroup supergroup` (only run once)
2. `sudo usermod -a -G supergroup <username>`

If you know this is going to be an issue for all users who try to run jobs on your cluster, you can modify the `chown` command above to read: `hadoop fs -chown <username>:supergroup /user/<username>` to ensure the user's own directories belong to that group.

# Putting files into HDFS

To copy a file from the local file system to HDFS, use the put command:

* `hadoop fs -put <file1> <file2> ... <hdfs_dir>`

Example: `hadoop fs -put edges.txt /user/kena`

# Getting files out of HDFS

To copy a file out of HDFS to the local file system, use the get command:

* `hadoop fs -get <hdfs_file> <local_dir>`

# Working with Hadoop PART files

When you create an RDD and have Spark save it uwith the `RDD.saveAsTextFile(".../path")` command, it creates a **directory** that contains each partition of the RDD stored in a separate `part-XXXXX` file, i.e. `part-00000`, `part-00001`, etc. Each of those files is just a simple text file (hence the name: `saveAsTextFile`).

It would be a pain if you have to combine all of those part files yourself each time you run a job... fortunately, you can have Hadoop do that for you (!):

* `hadoop fs -getmerge /user/kena/counts ./counts.txt`

or more generally

* `hadoop fs -getmerge /path/to/hdfs/dir ./path/to/local/combined/text/file.txt`


