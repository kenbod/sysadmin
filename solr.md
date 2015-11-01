# Solr

# Obtaining Solr

Go to http://lucene.apache.org/solr/ and click on Download. Download the latest version of Solr.
At the time of writing (November 2015), the latest version of Solr was 5.3.1. I downloaded the .tgz version
of the archive.

# Installing Solr

Note: I am configuring Solr such that it runs as its own special user. I'm doing this to make it hard for regular users to accidently delete Solr's server/data directory. They would have to type `sudo rm -r server` to make it happen and thus they would really have to mean it. If you don't need this level of protection, you can skip the steps that involve creating the `solr` user/group and setting Solr's directories to belong to the `solr` user.

## Getting Ready

* `sudo addgroup solr`
* `sudo adduser --ingroup solr solr`

## Unpacking and Installing the Archive

These instructions assume you start with the archive in your home directory.

* `gunzip solr-5.3.1.tgz`
* `sudo mv solr-5.3.1.tar /usr/local/src` # Note: you can install this wherever you want
* `cd /usr/local/src`
* `tar xvf solr-5.3.1.tar`
* `sudo tar xvf solr-5.3.1.tar`
* `sudo rm solr-5.3.1.tar`
* `sudo chown -R solr:solr solr-5.3.1`

## Putting Solr into your path

If you would like to have solr directly in your path:

* Edit `/etc/environment` and add the following line:

  `SOLR_HOME="/usr/local/src/solr-5.3.1"`

* Then, edit `/etc/bash.bashrc` and add the following line:

  `export PATH=$PATH:$SOLR_HOME/bin`
  
* Log out and log back in and test that solr is in your path:

  `which solr` # On my machine, this produces: `/usr/local/src/solr-5.3.1/bin/solr`

# Configuring Solr

My root partition is too small for Solr's `server` directory to live on it. Solr creates its indexes in a sub-directory of the server directory. As a result, it can potentially consume a large amount of disk space over time. As a result, I need to move it to a new location that has more disk space. If you're in the same situation, then do the following:

* `cd /usr/local/src/solr-5.3.1`
* `cp -r server /srv/hdfs-0/data` # change target directory to match your environment
* `cd /srv/hdfs-0/data`
* `sudo chown -R solr:solr server`

Unfortunately, this means that when we launch solr that we have to tell it which directory to use each time.

# Launching Solr for the first time

We will let Solr create our first core for us. We will then quit Solr and configure the core that it made for our own use.

To launch Solr for the first time:

* `su - solr`
* `solr start -d /srv/hdfs-0/data/server -s /srv/hdfs-0/data/server/solr`
* View your solr instance at: http://XXX.XX.XXX.XXX:8983/

Now, let's have Solr create a core that we can customize:

* `solr create -c fixr` # You can name your core whatever you want

This command will create a core named `fixr` that makes use of the default <q>data driven</q> Solr core. We will customize this core down below. After running this command, head back to Solr's web page and refresh it to confirm that it was created. Head to your file system and confirm that it was created in the correct location. For me, that location is:

`/srv/hdfs-0/data/server/solr/fixr`

Now, it's time to turn solr off and configure our newly created core:

* `solr stop`

You can also check on solr's status by typing (oddly enough):

* `solr status`









