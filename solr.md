# Solr

# Obtaining Solr

Go to http://lucene.apache.org/solr/ and click on Download. Download the latest version of Solr.
At the time of writing (November 2015), the latest version of Solr was 5.3.1. I downloaded the .tgz version
of the archive.

# Installing Solr

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



