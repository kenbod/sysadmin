# Installing Java 8 on an Ubuntu Machine

To install Java on my machines, I had to do it manually. I couldn't get the various `apt-get` methods to work.
(The automated attempt to download the JDK from Oracle's website would fail each time).

## 32-bit libraries

The 64-bit binary versions of the JDK still require that various 32-bit libraries are present on the machine.
To make sure they are present, various pages on the Internet recommend the following command:

```sudo apt-get install libc6-i386```

## Download the JDK

1. Head to &lt;http://www.oracle.com/technetwork/index.html&gt;
2. Click on **Software Downloads**
3. Click on **Jave SE**
4. Click on the **Download** button for the JDK
5. Click on the button that has you <q>agree</q> to the license agreement.
6. Click on the link for Linux x86.
7. After it downloads, use scp to copy it to your Ubuntu VM

The version I downloaded in July 2015 was: jdk-8u51-linux-i586.tar.gz

## Install the JDK (manually)

These manual instructions were taken from this [useful post on Ask Ubuntu](http://askubuntu.com/questions/56104/how-can-i-install-sun-oracles-proprietary-java-jdk-6-7-8-or-jre).

* Unpack the archive in a user directory: `tar -xvf <jdk-archive.tar.gz>`
* Move the new directory to `/usr/lib`

```
sudo mkdir -p /usr/lib/jvm
sudo mv ./<jdk-archive> /usr/lib/jvm/
```

* Create a generic symlink for JDK 1.8

```
cd /usr/lib/jvm
sudo ln -s <jdk-directory> jdk1.8.0
```

* Let Ubuntu know about the newly installed commands

```
sudo update-alternatives --install "/usr/bin/java" "java" "/usr/lib/jvm/jdk1.8.0/bin/java" 1
sudo update-alternatives --install "/usr/bin/javac" "javac" "/usr/lib/jvm/jdk1.8.0/bin/javac" 1
sudo update-alternatives --install "/usr/bin/javaws" "javaws" "/usr/lib/jvm/jdk1.8.0/bin/javaws" 1
```

* Mark the files as exectuable and make sure that `root` owns the installation

```
sudo chmod a+x /usr/bin/java
sudo chmod a+x /usr/bin/javac
sudo chmod a+x /usr/bin/javaws
sudo chown -R root:root /usr/lib/jvm/jdk1.8.0
```

* If you have only one version of the JDK installed, skip this step. Otherwise, pick the JDK you want to use

```
sudo update-alternatives --config java
sudo update-alternatives --config javac
sudo update-alternatives --config javaws
```

* Make sure that JAVA_HOME is defined in `/etc/environment`. Add this line:

```
JAVA_HOME="/usr/lib/jvm/jdk1.8.0"
```

* Test that everything is working

```
java -version
javac -version
```


