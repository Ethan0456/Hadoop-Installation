# Hadoop Installation Steps

Source

[https://medium.com/@festusmorumbasi/installing-hadoop-on-ubuntu-20-04-4610b6e0391e](https://medium.com/@festusmorumbasi/installing-hadoop-on-ubuntu-20-04-4610b6e0391e)

Prerequisite:

- Download and Install any Debian based distribution of linux (Xubuntu: Recommended) on VMware or VirtualBox

Steps:

1. Create separate Hadoop User

`sudo adduser hadoop`

Error: (which will be faced in step 5)

> user not in sudoers’ list
> 

Solution:

`sudo usermod -aG Hadoop`

1. Install Java
- First Update packages information

`sudo apt update`

- Then Install Java

`sudo apt install openjdk-8-jdk -y`

- After installation run below command to check the java version

`java -version`

1. Install Openssh on Ubuntu

`sudo apt install openssh-server openssh-client -y`

1. Switch from current user to newly created user Hadoop

`sudo su - hadoop`

1. We need to setup password less ssh connection with the Hadoop user
- Generate public private key pair for ssh connection

`ssh-keygen -t rsa`

- (optional) If present working directory is not the home directory then run

`cd`

- Then concatenate newly generated public key to the “authorized_keys” file

`sudo cat ~/.ssh/id_rsa.pub >> ~/.ssh/authorized_keys`

- Change the permission of the “authorized_keys” file

`sudo chmod 640 ~/.ssh/authorized_keys`

> In linux you can change permission using the numerical format as well as using flags like rwx with operators like + and - to add and remove permissions
> 

> here 640 stands for the permission, in linux each file has 3 permission groups
> 
> 
> owners, groups, all users
> 
> each number in this 3 digit number is decimal format of binary permissions
> 
> - 0 = ---
> - 1 = --x
> - 2 = -w-
> - 3 = -wx
> - 4 = r-
> - 5 = r-x
> - 6 = rw-
> - 7 = rwx
> 
> 6 = 2^2+2^1+2^0 = 110 = rw-
> 
> 4 = 2^0+2^2+2^0 = 010 = r—
> 
> 0 = 2^0+2^0+2^0 = 000 = —-
> 
> For example:
> 
> - chmod 777 foldername will give read, write, and execute permissions for everyone.
> - chmod 700 foldername will give read, write, and execute permissions for the user only.
- After that run code to connect to own ssh to add machine to known hosts

`ssh localhost`

1. Install Hadoop from the official link

`wget [https://downloads.apache.org/hadoop/common/hadoop-3.3.2/hadoop-3.3.2.tar.gz](https://downloads.apache.org/hadoop/common/hadoop-3.3.1/hadoop-3.3.1.tar.gz)`

- Extract the above downloaded file

`tar -xvzf hadoop-3.3.2.tar.gz`

- Change directory name to Hadoop

`mv hadoop-3.3.2 hadoop`

1. Configure Java Environment variables for setting up Hadoop

`dirname $(dirname $(readlink -f $(which java)))`

---

Now we need to configure Hadoop

- To configure hadoop we need to edit a bunch of files like

.bashrc, [hadoop-env.sh](http://hadoop-env.sh/), core-site.xml, hdfs-site.xml, mapred-site-xml and yarn-site.xml

1. .bashrc
- It is the configuration file for bash (Bourne Again Shell)
- We need to add Paths for our hadoop to work properly

`export JAVA_HOME=/usr/lib/jvm/java-8-openjdk-amd64
export HADOOP_HOME=/home/hadoop/hadoop
export HADOOP_INSTALL=$HADOOP_HOME
export HADOOP_MAPRED_HOME=$HADOOP_HOME
export HADOOP_COMMON_HOME=$HADOOP_HOME
export HADOOP_HDFS_HOME=$HADOOP_HOME
export YARN_HOME=$HADOOP_HOME
export HADOOP_COMMON_LIB_NATIVE_DIR=$HADOOP_HOME/lib/native
export PATH=$PATH:$HADOOP_HOME/sbin:$HADOOP_HOME/bin
export HADOOP_OPTS="-Djava.library.path=$HADOOP_HOME/lib/native"`

- To Activate above changes in bash, execute

`source ~/.bashrc`

1. hadoop-env.sh
- nano is cli editor you can use any other text editor as well like vim.

`sudo nano $HADOOP_HOME/etc/hadoop/hadoop-env.sh`

- Here we need to configure JAVA_HOME variable so that Hadoop can know which java version to use.

Method 1: If you followed above method to install Java, add below line in this file

`export JAVA_HOME=/usr/lib/jvm/java-8-openjdk-amd64`

Method 2: If you installed by other method

Find where Java is installed, by executing

`which java`

- Then type below command to find the path of OpenJDK directory

`readlink -f /usr/bin/javac`

- Unix system too have concept of shortcuts, readlink gives the path of the actual file to which a shortcut directs

<img width="405" alt="Screenshot_2023-01-26_at_11 36 18_PM" src="https://user-images.githubusercontent.com/88190547/224557093-6ab5fc20-5105-4474-82fd-b58b70fd4232.png">

- In the above result the path before /bin/javac i.e. /usr/lib/jvm/java-8-openjdk-amd64 needs to be added in the hadoop-env.sh

Example : If Your output is like /usr/lib/jvm/java-11-openjdk-amd64/bin/javac then add /usr/lib/jvm/java-11-openjdk-amd64 to the file.

1. core-site.xml

`sudo vim $HADOOP_HOME/etc/hadoop/hadoop-env.sh`

- We need to edit this file to add URL for our name node as it is maintained from a web interface.
- If you have <configuration> </configuration> code present in your file then just add below code between those tags or if not then first add those configuration tags then add below code.

`<property>`

`<name>fs.defaultFS</name>`

`<value>hdfs://localhost:9000</value>`

`</property>`

1. hdfs-site.xml

`sudo nano $HADOOP_HOME/etc/hadoop/hdfs-site.xml`

- Same as above add below code between configuration tags.
- With this file we can define the locations to store node metadata, fsimage file, and edit log file.
- dfs.name.dir is assigned the location to store the name node data.
- dfs.data.dir is assigned the location to store the data node data.
- This code specifies replication value which is set to 1 as it is a single node cluster.

`<property>
<name>dfs.replication</name>
<value>1</value>
</property>        <property>
<name>dfs.name.dir</name>
<value>file:///home/hadoop/hadoopdata/hdfs/namenode</value>
</property>        <property>
<name>dfs.data.dir</name>
<value>file:///home/hadoop/hadoopdata/hdfs/datanode</value>
</property>`

1. mapred-site.xml

`sudo nano $HADOOP_HOME/etc/hadoop/mapred-site.xml`

- Below code changes the default mapreduce framework name to yarn

`<property>`

`<name>mapreduce.framework.name</name>`

`<value>yarn</value>`

`</property>`

1. *yarn-site.xml*

`sudo nano $HADOOP_HOME/etc/hadoop/yarn-site.xml`

- Add below lines of code to this file

`<property>`

`<name>yarn.nodemanager.aux-services</name>`

`<value>mapreduce_shuffle</value>`

`</property>`

1. Format NameNode

`hdfs namenode -format`

1. Start Hadoop Cluster

`start-dfs.sh` : to start distributed file system

> Error:
> 

<img width="543" alt="Screenshot_2023-01-27_at_9 31 35_AM" src="https://user-images.githubusercontent.com/88190547/224557124-fc8269f7-65cb-46d6-9c0b-83aa41c3cbd1.png">


> Solution:
> 

> `cd hadoop/etc/hadoop`
> 

> `vim hadoop-env.sh`
> 

> Paste below line in this file
> 

> `export HADOOP_SSH_OPTS="-p 22”`
> 

`start-yarn.sh` : to start resource negotiator

`jps` : Lists running Java VM on target machine

The output of above code will look like

<img width="233" alt="Screenshot_2023-01-26_at_11 59 01_PM" src="https://user-images.githubusercontent.com/88190547/224557139-980924ca-fca5-4d13-ad6d-d9ee43024d76.png">


1. Access Hadoop Web UI

After all this steps, we can monitor Hadoop from the web interface by reaching below URL from browser

`[http://localhost:9870](http://localhost:9870)`
