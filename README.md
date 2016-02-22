# Running D2F Queries in with Hawq
##Setup run-environment
Option 1: Docker

To be able to use Hawq with Docker you just have to follow the instructions on `https://hub.docker.com/r/mayjojo/hawq-devel`, which has a virtual machine version of Centos 6 / 7 with most necessary dependencies already pre-installed:

1. Install Docker and allow Docker to use your network so that it can later use your internet connection (instructions for your specific Linux distribution can be found on https://docs.docker.com/ under the Docker Engine > Install > On Linux distributions tab) 

2. Use command: `git clone https://github.com/wangzw/hawq-devel-env.git` to download the virtual machine

3. Go into the newly created folder and execute the makefile with make run

4. Afterwards, use `docker exec -it centos7-namenode bash` to enter the bash environment of your newly created virtual machine

Option 2: Installing the depencies yourself

Just install the depencies listed here https://cwiki.apache.org/confluence/display/HAWQ/Build+and+Install under "Compile Depencies"
If you are using CentOS 7.X you can use the yum install option.

Installing Hive:

1. For Option 1: download from http://www.us.apache.org/dist/hive/stable/ the newest stable release to your host-system and copy it into your virtual machine with `docker cp [OPTIONS] SRC_PATH | - CONTAINER:DEST_PATH`

2. For Option 2: Follow the instructions on the Hive-homepage (https://cwiki.apache.org/confluence/display/Hive/GettingStarted) to install through the command line 

3. Include Hive in the virtual machine's PATH-variable

Installing and generating data / running queries with D2F-Bench: 

Just follow the instructions on https://github.com/t-ivanov/D2F-Bench inside the virtual machine, as there is nothing different about the installation when using the docker environment. The only thing we have changed was in the config file later on the filetype of our generated data (we chose to use simple textfiles). Anything else is just as described on the github page.


##Installing Hawq and data preparation

For docker use the instructions on https://hub.docker.com/r/mayjojo/hawq-devel/ to install Hawq.

Else use https://cwiki.apache.org/confluence/display/HAWQ/Build+and+Install

Accessing data on HDFS:

There are multiple ways to access data on HDFS as there are multiple protocols to use with psql and external tables. PXF and gphdfs should both be able to access HDFS data in its entirety and therefore don't need a special preparation. For some undiscernable reason, both of them failed in our particular cases to work or even be recognized as an installed protocol.
If you're having issues using PXF or installing gphdfs yourself, here are the instructions on how to use gpfdist to access the generated data through your local file system (uses lots of space because the data is decompressed first):

1. Decompress .deflate-files with: `sudo -u hdfs hdfs dfs -text /hdfs_path/to/file.deflate | sudo -u hdfs hdfs dfs -put – /hdfs_path/to/decompressed_file`
2. To download the data from HDFS to your local file system, use `sudo hadoop fs -get /hdfs_path/to/decompressed_file /local_path/to/local_file`
3. Before creating an external table in psql, because you are using gpfdist, the gpfdist server program has to be running and listening to the correct port. You can ensure this by using `gpfdist -p 8081 -d /var/data/staging -l /home/gpadmin/log &`


##Creating external tables and running queries:
First you need to create external tables with the protocol you want to use. Note that some datatypes can't be ported 1:1 and have to be edited. Refer to http://hawq.docs.pivotal.io/docs-hawq/topics/HAWQDataTypes.html to see how. The tables in the Tables.txt are already edited in such manner.

PXF: Refer to http://hawq.docs.pivotal.io/docs-hawq/topics/PXFExternalTableandAPIReference.html.

You can use the `CREATE EXTERNAL TABLE ..` commands from the Tables.txt if you edit the the location path.

For gpfdist (http://hawq.docs.pivotal.io/docs-hawq/docs-hawq-shared/admin_guide/load/topics/g-gpfdist-protocol.html)

You can also use the `CREATE EXTERNAL TABLE ..` commands from the Tables.txt. Note that you also have to edit the location path.
e.g. `LOCATION ('gpfdist://centos7-namenode:8081/lineitem/datatext')`

After your created the external tables you can check with a simple `Select * from <table>` if the creation was successfull. If for some reason the creation fails try to add an empty column to every table. e.g.  `L_COMMENT TEXT, L_NOTHING TEXT)` for lineitem. 

The queries of D2F-Bench are already modified in a manner to work with psql, see the .sql files in the queries folder. If they dont work try to modify them yourself, note that the datatypes have been modified also.
To run the queries in the psql prompt use `\i /path/to/file.sql` or simply copy the queries over.

