# Installing Scala

Scala is the primary way of interaction with [Spark](https://github.com/kenbod/sysadmin/blob/master/spark.md). As a result, it would be good to have Scala installed on each machine of our Spark cluster (this is not strictly necessary). One constraint we face is that the current version of Spark (1.4.1 in July 2015) is compiled against Spark 2.10.x, so we cannot install the latest version of Scala on our machines.

## Downloading Scala

With that as background, follow these instructions to download Scala:

1. Head to `<http://www.scala-lang.org/download/2.10.5.html>`
2. Click the link to download `scala-2.10.5.tgz` to your machine.
3. Use `scp` to copy the archive to the machines in your cluster.

## Installing Scala

Following the conventions of the other pages in this repo, we will unpack scala in `/usr/local/src`. You are free to use another directory (such `/opt` or `/home/scala`) if you want.

1. Move archive: `sudo mv scala-2.10.5.tgz /usr/local/src`
2. Unpack it: `sudo tar xzf scala-2.10.5.tgz`
3. Remove it: `sudo rm scala-2.10.5.tgz` (Optional)
4. Create a scala group: `sudo addgroup scala`
5. Create a scala user: `sudo adduser --ingroup scala scala`
6. Change the owner for the scala distribution: `sudo chown -R scala:scala scala-2.10.5`

## Configure Environment

1. Add the following line to `/etc/environment`

        SCALA_HOME="/usr/local/src/scala-2.10.5"

2. Add the following line to `/etc/bash.bashrc`

        export PATH=$PATH:$SCALA_HOME/bin

3. Logout and login back in and check to see that you have `scala` and `scalac` in your path.


