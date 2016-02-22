# Running D2F Queries in with Hawq
##Prerequisites:
Option 1: Docker
To be able to use Hawq with Docker you just have to follow the instructions on `https://hub.docker.com/r/mayjojo/hawq-devel`, which has a virtual machine version of Centos 6 / 7 with most necessary dependencies already pre-installed:

1.: Install Docker and allow Docker to use your network so that it can later use your internet connection (instructions for your specific Linux distribution can be found on https://docs.docker.com/ under the Docker Engine > Install > On Linux distributions tab)
3.:Use command: `git clone https://github.com/wangzw/hawq-devel-env.git` to download the virtual machine
4.:Go into the newly created folder and execute the makefile with make run
5.:Afterwards, use `docker exec -it centos7-namenode bash´ to enter the bash environment of your newly created virtual machine

The next few things left are installing Hive, D2F-Bench and Hawq:

Installing Hive:

1.: Follow either the instructions on the Hive-homepage to install through the command line (https://cwiki.apache.org/confluence/display/Hive/GettingStarted), or just download from http://www.us.apache.org/dist/hive/stable/ the newest stable release to your host-system and copy it into your virtual machine with the command:  docker cp [OPTIONS] SRC_PATH | - CONTAINER:DEST_PATH 
2.: Include Hive in the virtual machine's PATH-variable


Installing and generating data / running queries with D2F-Bench: 

Just follow the instructions on https://github.com/t-ivanov/D2F-Bench inside the virtual machine, as there is nothing different about the installation when using the docker environment. The only thing we have changed was in the config file later on the filetype of our generated data (we chose to use simple textfiles). Anything else is just as described on the github page.


Installing HAWQ:

Follow the instructions on https://hub.docker.com/r/mayjojo/hawq-devel/ to install Hawq:

Inside der virt. Maschine:

1.: To download Hawq, use the command: git clone https://github.com/apache/incubator-hawq.git /data/hawq
2.: Go inside the newly created folder and execute the command: ./configure --prefix=/data/hawq-devel  
3.: Execute make and make install to install Hawq 
4.: Next, execute the following commands to make Hawq know about the name- and datanodes:

sed 's|localhost|centos7-namenode|g' -i /data/hawq-devel/etc/hawq-site.xml
echo 'centos7-datanode1' > /data/hawq-devel/etc/slaves
echo 'centos7-datanode2' >> /data/hawq-devel/etc/slaves
echo 'centos7-datanode3' >> /data/hawq-devel/etc/slaves 

5.: To be able to initialise Hawq, execute the command: source /data/hawq-devel/greenplum_path.sh      to make the console be able to find Hawq

6.: Execute the following commands to finalise initialisation of Hawq and create a database to work on: 

hawq init cluster 
createdb 

7.: You can now use psql to access Hawq 

Accessing Data on HDFS:

There are multiple ways to access data on HDFS as there are multiple protocols to use with psql and external tables. PXF and gphdfs should both be able to access HDFS data in its entirety, but for some undescernable reason, both of which failed in our particular cases to work or even be recognized as an installed protocol. As such, we had to resort to downloading our generated data off of HDFS to the local file system. If you're having issues using PXF or installing gphdfs yourself, here are the instructions on how to use gpfdist to access the generated data:

1.: If the files are still compressed, decompress  them with the command: sudo -u hdfs hdfs dfs -text /hdfs_path/to/file.deflate | sudo -u hdfs hdfs dfs -put – /hdfs_path/to/decompressed_file
2.: To download the data from HDFS to your local file system, use the command: sudo hadoop fs -get /hdfs_path/to/decompressed_file /local_path/to/local_file
3.: Before creating an External Table in psql, because we are using gpfdist, the gpfdist server program has to be running and listening to the correct port. You can ensure this by using the command: gpfdist -p 8081 -d /var/data/staging -l /home/gpadmin/log &


Creating External Tables and running queries on Hawq:
