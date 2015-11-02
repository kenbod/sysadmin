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

# Configuring our core

The default configuration represents a great start. I'm going to change it to serve my own needs. YMMV.

First: head to the core's configuration directory. On my machine that directory is:

`/srv/hdfs-0/data/server/solr/fixr/conf`

Second: There is a file in this directory called `managed-schema`. Rename it `schema.xml`.

Third: Make the following edits.

1. Find the two lines that look like this:

  `<field name="_text_" type="text_general" indexed="true" stored="false" multiValued="true"/>`

  `<copyField source="*" dest="_text_"/>`
  
  Delete them.
  
2. Find the line that looks like this:

  `<dynamicField name="*_s"  type="string"  indexed="true"  stored="true" />`
  
  Add the following line, right under it:
  
  `<dynamicField name="*_sni"  type="string"  indexed="false"  stored="true" />`
  
  This allows us to store text fields in our documents that are not analyzed or indexed but are stored so that they can later be retrieved. We only want these fields so we can display them as the result of a query but we are not going to be searching on their content.

3. There is currently NO step 3.

# Details about our Core

Given the above configuration, we now have a variety of options for storing/indexing data. The processing that Solr applies to a particular field depends on the suffix we give to that field's name. Here are the relevant suffixes that we'll be using:

|Suffix|Meaning|
|:----:|:------|
|_sni|Store the text of this field but do not process or index it.|
|_tdt|This field must contain a date formatted in the following way: 1995-12-31T23:59:59Z. It allows us to specify range queries on date/time values. Solr uses a trie to index these values; as such, queries on these values typically execute with blinding speed|
|_t|This field contains textual data that we want indexed and stored. We use these fields to search our corpus for text-based stings. Solr applies its standard tokenizer on this field, removes stop words (by default, no stop words are specified), and it applies a lowercase filter to the produced tokens. Thus `addDocument` and `adddocument` are treated as the same query. It also uses any information from `synonyms.txt` in your core's `conf` directory at query time. (That file by default has a few meaningless synonyms specified; you can delete all of the non-commented text from that file if you want.)|

There are many more field types defined in the default schema (check them out) but the three fields above are all I need for my project.

# Launching Solr for Long-Term Use

Solr launches in the background automatically and will continue to run after the user logs out. So, to launch solr for long-term use, you use the same two commands that you did for the first invocation:

* `su - solr`
* `solr start -d /srv/hdfs-0/data/server -s /srv/hdfs-0/data/server/solr`

Log out and verify that you can still access solr's web-based interface.








