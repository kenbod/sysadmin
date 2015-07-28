# Instructions for Installing Hadoop

I decided to install the latest version of Hadoop. In July 2015, that was Hadoop 2.7.1.

## Downloading Hadoop

1. Head to &lt;http://hadoop.apache.org/releases.html&gt;
2. Download the binary version of Hadoop: hadoop-2.7.1.tar.gz.
3. scp the archive to the VMs that need it.

## Installing Hadoop

**WARNING:** As you will see, installing Hadoop is NOT for the faint-hearted.

**NOTE:** These instructions are adapted from the information that appears in Chapter 10 of [Hadoop: The Definitive Guide, 4th Edition](http://shop.oreilly.com/product/0636920033448.do). The text in that chapter is written at a high level of abstraction. My instructions below contain way more details. :smiley:

### Installing Java

Make sure that your machines have Java installed. See [my instructions](https://github.com/kenbod/sysadmin/java.md) for doing that.

