# Creating/Mounting Volumes

## Create new volume in OpenStack:

1. Head to Compute => Volumes
2. Click on Create Volume
3. Give it a name: e.g. spark-worker1-data
4. Select `gfs-r620` as the type (at least for the HDFS data volumes to make them local to the machine)
5. Do not give it a source (let it be an empty volume)
6. Give it a size (I'm using 500 GB to start).
7. Select an availability zone: there's only one (for now): `nova`

## Attach it to your desired instance

Once the volume is available:

1. Select Manage Attachments
2. In the menu under `Attach to Instance` select the desired instance.
3. Click attach volume
4. There is no step 4.

## Format the volume

Login to the instance and format the volume with the command:

1. `sudo mkfs.ext3 -L hdfs-1 /dev/vdb`

## Mount the volume

1. Create a mount point for the volume: `sudo mkdir srv/hdfs-1`
2. fsent="LABEL=hdfs-1 /srv/hdfs-1 ext3 noatime 1 2"
3. echo "$fsent" | sudo tee -a /etc/fstab
4. sudo mount /srv/hdfs-1

Note: You should change the label to match the context of use. So, in the commands above `hdfs-1` is just a label.
So, for worker 2, I'm going to change that label to be hdfs-2. It is not a <q>set in stone</q> part of the command above
but something that varies as you create each volume.

