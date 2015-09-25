Shellcloud
==========

Create and control distributed jobs via SSH or RSH.

Overview
--------

Shellcloud helps to build distributed processes in *map-reduce* paradigm. 
Unlike other systems that implement both map and reduce, it focuses on the 
*map* part and allows users to utilize arbitrary external data storage for 
*reduce* part. 

For example, your data may be stored in Elasticsearch, Cassandra or other store.
To process the data conventionally, you would export it to Hadoop FS, implement 
Hadoop jobs in Java, run Hadoop jobs and import the data back to the store.

Many modern distributed storage systems allow for quick access to data directly,
therefore the apparent need for exporting and importing the data for processing 
does not apply there.

With Shellcloud you can process the data and output the results directly using
the store. In addition, Shellcloud is language-agnostic and lets you implement 
data processing jobs in any language.

Shellcloud is built based on standard Linux tools. There is no need to install
additional software on cluster nodes (usually the basic Linux installations are 
sufficient). Only the control host needs to have a copy of Shellcloud.

Creating jobs
-------------

Creating distributed jobs has never been easier. You simply create a folder and
a script that you want to run, that's it! The folder optionally can contain 
arbitrary data needed to run the job.

### Example:

In this example we create a trivial job that counts size of /tmp folder on a 
node and logs the size to NFS storage. In this case the NFS storage acts as the 
*reduce* part. When the job is executed on all cluster nodes, we can read the 
total /tmp size on the cluster as the sum of all NFS logs.

Content of a job folder:

    ./MyJob
    ./MyJob/MyScript.sh`

Content of MyScript.sh:

    du -hs /tmp > /nfs/tmp.log

If you need to run a job that requires an executable, a jar or else, you 
simply put that executable or jar in MyJob folder and add a command that starts 
that executable or jar to the script. The content of the job folder will be 
transferred to a cluster node and executed there.

Starting jobs
------------

By default each job is executed on a randomly selected node. This can be changed
by an option on the start command. Each job is added to a logical job id, that
later is used to identify jobs, therefore you need to specify a job id when 
starting a job.

### Example:

Start job on random cluster node via SSH (-s option). 'job1' is a logical job id, 
./MyJob is the job folder, MyScript.sh is the job script to execute. Optionally 
user can pass arbitrary sequence of options after the job script, just like for
a regular command.

    shc start -s job1 ./MyJob MyScript.sh

A job can also be started on a specific cluster node.

    shc start -s @node9.seegnify.net job1 ./MyJob MyScript.sh 

When the command is successful the output would be similar to this.

    Starting: host=node9.seegnify.net job=job1 cmd=MyScript.sh  OK

## Listing jobs

Job listing shows currently running jobs on the cluster. Shellcloud does not 
persist job information and all listings are transient.

### Example:

The following command lists jobs on the cluster via SSH.

    shc list -s -j

If there are jobs running, the output of the command could be as follows.

    Running: job=job1 host=node1.seegnify.net pid=21033 cmd=MyScript.sh
    Running: job=job1 host=node9.seegnify.net pid=9483 cmd=MyScript.sh

Stopping jobs
-------------

To stop jobs simply specify job id that was used to start it. The job id can 
also be obtained via 'shc list' command.

### Example:

Stop all jobs running under logical job id 'job1'.

    ./bin/shc stop -s job1

If there are any running jobs the output would like this:

    Stopping: job=job1 host=node1.seegnify.net pid=21033 OK
    Stopping: job=job1 host=node9.seegnify.net pid=9483 OK

Installation
------------

Installation is as easy as copying the project files to a desired location and 
adding Shellcloud 'bin' folder to the environment variable PATH.

Configuration
-------------

The configuration file shellcloud.hosts is located in the conf folder on the 
control host. It lists Shellcloud enabled nodes by IP address or DNS names (one 
node per line). Each node needs to be SSH or RSH enabled. To SSH enable a node 
copy SSH public ID form the control host to the node (usually via ssh-copy-id). 
To RSH enable a node add the control host IP address to .rhosts file on the 
node. Shellcloud assumes that commands run on the nodes under the same username 
as the account on the control host.

To get status of your configuration run the following command.

    shc stat -a

This will list all required tools and will show which nodes are enabled.

    OK - nmap present at /usr/bin/nmap
    OK - ssh present at /usr/bin/ssh
    OK - rsh present at /usr/bin/rsh
    OK - rcp present at /usr/bin/rcp
    OK - nc present at /bin/nc
    OK - awk present at /usr/bin/awk
    OK - base64 present at /usr/bin/base64
    OK - host node0.seegnify.net accessible via SSH with key authentication
    OK - host node0.seegnify.net accessible via RSH with host authentication
    OK - host node9.seegnify.net accessible via SSH with key authentication
    ERROR - host node9.seegnify.net not accessible via RSH with host authentication

Note that each node is listed twice and only one type of authentication is 
required. Therefore if you see ERROR for RSH and OK for SSH or vice-versa, 
it is still a good setup.
