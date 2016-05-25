This document takes you through how you can setup a hyperconverged environment on your laptop or machine. 

Note that this isn't the recommended or practical way to using a hyperconverged environment, this is merely for learning purposes. A typical hyperconverged environment is setup using a minimum of three nodes or physical machines.

####Pre-requisities: A laptop / system with CentOS installed.

We will be using CentOS as the operating system, oVirt for the virtualization manager and GlusterFS as our filesystem.

* To start with, let's update our packages using:

yum update

* Next, we have to install the oVirt 3.6 release package:

yum install http://resources.ovirt.org/pub/yum-repo/ovirt-release36.rpm

* Install oVirt hosted engine and then install vdsm package to run the hosted engine

yum install screen ovirt-hosted-engine-setup vdsm-gluster

* Start gluster service

systemctl start glusterd.service

* Open the following firewall ports (ask Ramesh about this in detail)

  firewall-cmd  --zone=public --add-service=glusterfs --permanent
  firewall-cmd --zone=public --add-port=111/tcp --permanent
  firewall-cmd --zone=public --add-port=2049/tcp --permanent
  firewall-cmd --zone=public --add-port=54321/tcp --permanent
  firewall-cmd --zone=public --add-port=5900/tcp --permanent
  firewall-cmd --zone=public --add-port=5900-6923/tcp --permanent
  firewall-cmd --zone=public --add-port=5666/tcp --permanent
  firewall-cmd --zone=public --add-port=16514/tcp --permanent
  firewall-cmd --reload 
  
* In this tutorial, we will be needing a storage volume to store virtual instances or images that we create. We won't be writing any data at this stage. We are using GlusterFS to create this volume. 
(Note that in case we were to write data, we would be in need of another volume, which would make it two volumes in all.)

Now, we need to have three bricks in order to create a Gluster volume - three bricks because we are simulating creating a Gluster volume using three hard disks on three physical machines.

We can create these bricks simply by creating three directories using the following commands:

mkdir -p /brick/b1

mkdir -p /brick/b2

mkdir -p /brick/b3

Using these commands, we have created a directory called "brick" under root, and under it, three directories b1, b2 and b3. 

* Let's create a volume using our three bricks/directories!

gluster volume create engine replica 3 <local system IP>:/brick/b1 <local system IP>:/brick/b2 <local system IP>:/brick/b3

The name of our gluster volume is "engine".

* We have to now run some commands in order to make our volume ready to store virtual instance images.

gluster volume set engine group virt

What this command does is set the volume to use options in the "virt" file. This file becomes available on your system when you install the required rpms above. The options in the file are recommended for performance reasons, when implementing virtualisation. 

Note that after using this command, it's not advisable to use the volume to store anything other than virtual instance images. 

* Adding permissions / ownership to our volume:

gluster volume set engine storage.owner-uid 36
gluster volume set engine storage.owner-gid 36

* Using the shard feature

gluster volume set engine  features.shard on
gluster volume set engine features.shard-block-size 512MB

Sharding is a method or a feature that GlusterFS uses to store large files. The above command will cut the file into blocks of 512 MB each for storing them across the volume.

* ______

gluster volume set data performance.low-prio-threads 32
gluster volume set data cluster.data-self-heal-algorithm full


* Start the volume

gluster volume start engine

####Hosted Engine Setup

yum install ovirt-engine-appliance

screen

hosted-engine --deploy

Through the course of this setup, you will come across a lot of options to choose from with a default option suggested to you in square brackets. Pressing 'Enter' without selecting an option will imply that you wish to go ahead with the default option.

We'll do that for most options. 

<insert screenshot of all commands>


For configuring IP tables however, we say 'No', this is a temporary workaround as saying 'Yes' results into gluster ports being left unopened.

<insert screenshot>

We want to configure oVirt with both Virt and Gluster mode. Automatically executing engine setup configures oVirt in Virt mode alone. So, when we're asked if we want to automatically execute engine setup, we say 'No'.

<insert screenshot>

We'll add a dummy entry to the /etc/hosts file temporarily. 

<insert screenshot>

When asked to enter the domain name to be used for the engine appliance, we'll enter this domain name and continue till the setup launches the setup and asks us to execute engine-setup

<insert screenshot>

Here, we launch our VM using:

remote-viewer vnc://10.209.64.36:5900

10.209.64.36 is the IP of our hostmachine and 5900 is the port number required for remote access to our VM.

Once the VM is launched, entering this command will help us know the IP of the VM:

ip addr


Once the IP of our VM is obtained, we correct the entry in the /etc/hosts file of our host machine. The dummy IP address will be replaced by this IP.

<insert screenshots>

At this point, we should have two entries in our /etc/hosts file - one pointing to our local host machine and the other one to our VM.

<insert screenshot>

We'll copy both these entries and paste them in the /etc/hosts file of the VM.

<insert screenshot>

We have to now run 'engine-setup' on this VM (terminal) using the command 'engine-setup' and once it's complete, we go back to the terminal on our host machine to continue with the rest of the setup.

<insert screenshot>










 
 










