# Installing Java 8 on an Ubuntu Machine

To install Java on my machines, I had to do it manually. I couldn't get the various `apt-get` methods to work.
(The automated attempt to download the JDK from Oracle's website would fail each time).

## 32-bit libraries (optional)

The 32-bit binary versions of the JDK require that various 32-bit libraries are present on the virtual machine. If you're virtual machine is a 64-bit operating system, those libraries may not be present. If you decide to use the 32-bit version of the JDK, then to make sure that the required libraries are present, run the following command:

```sudo apt-get install libc6-i386```

## Download the JDK

1. Head to &lt;http://www.oracle.com/technetwork/index.html&gt;
2. Click on **Software Downloads**
3. Click on **Jave SE**
4. Click on the **Download** button for the JDK
5. Click on the button that has you <q>agree</q> to the license agreement.
6. Click on the link for Linux x86.
7. After it downloads, use scp to copy it to your Ubuntu VM

The version I downloaded in July 2015 was: jdk-8u51-linux-x64.tar.gz (64 bit) and jdk-8u51-linux-i586.tar.gz (32 bit). Although I downloaded both, I ended up using just the 64 bit version.

## Install the JDK (manually)

These manual instructions were taken from this [useful post on Ask Ubuntu](http://askubuntu.com/questions/56104/how-can-i-install-sun-oracles-proprietary-java-jdk-6-7-8-or-jre).

1. Unpack the archive in a user directory: `tar -xvf <jdk-archive.tar.gz>`
2. Move the new directory to `/usr/lib`

        sudo mkdir -p /usr/lib/jvm
        sudo mv ./<jdk-archive> /usr/lib/jvm/

3. Create a generic symlink for JDK 1.8

        cd /usr/lib/jvm
        sudo ln -s <jdk-directory> jdk1.8.0

4. Let Ubuntu know about the newly installed commands

        sudo update-alternatives --install "/usr/bin/java" "java" "/usr/lib/jvm/jdk1.8.0/bin/java" 1
        sudo update-alternatives --install "/usr/bin/javac" "javac" "/usr/lib/jvm/jdk1.8.0/bin/javac" 1
        sudo update-alternatives --install "/usr/bin/javaws" "javaws" "/usr/lib/jvm/jdk1.8.0/bin/javaws" 1
        sudo update-alternatives --install "/usr/bin/jps" "jps" "/usr/lib/jvm/jdk1.8.0/bin/jps" 1

5. Mark the files as exectuable and make sure that `root` owns the installation

        sudo chmod a+x /usr/bin/java
        sudo chmod a+x /usr/bin/javac
        sudo chmod a+x /usr/bin/javaws
        sudo chmod a+x /usr/bin/jps
        sudo chown -R root:root /usr/lib/jvm/jdk1.8.0

6. If you have only one version of the JDK installed, skip this step. Otherwise, pick the JDK you want to use

        sudo update-alternatives --config java
        sudo update-alternatives --config javac
        sudo update-alternatives --config javaws
        sudo update-alternatives --config jps

7. Make sure that JAVA_HOME is defined in `/etc/environment`. Add this line:

        JAVA_HOME="/usr/lib/jvm/jdk1.8.0"

8. Test that everything is working

        java -version
        javac -version

9. There is no step 9.


