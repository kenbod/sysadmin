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
