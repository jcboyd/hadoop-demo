# hadoop-demo

[Apache Hadoop](http://hadoop.apache.org/) is a framework for distributed storage with the Hadoop distributed file system (HDFS) and MapReduce data processing with job scheduler YARN (Yet Another Resource Negotiator). This repository gives the basic configuration steps and a demo for usage on OS X.

## Configuration

The easiest way to install Hadoop on OS X is to use [Homebrew](http://brew.sh/), a popular package manager for OS X. To get this execute

```sh
$ /usr/bin/ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"
```

Hadoop can then be installed by running

```sh
$ brew install hadoop
```

This installs Hadoop in **/usr/local/Cellar**, the directory used by Homebrew. We now make some adjustments to Hadoop's configuration files, beginning with the script **/usr/local/Cellar/hadoop/_<X.Y.Z_>/libexec/etc/hadoop/hadoop-env.sh** (where _<X.Y.Z_> is the installation version), which sets various environment variables, replacing the line

```sh
export HADOOP_OPTS="$HADOOP_OPTS -Djava.net.preferIPv4Stack=true"
```

with

```sh
export HADOOP_OPTS="$HADOOP_OPTS -Djava.net.preferIPv4Stack=true -Djava.security.krb5.realm= -Djava.security.krb5.kdc="
```

Next, we edit the file **/usr/local/Cellar/hadoop/_<X.Y.Z_>/libexec/etc/hadoop/core-site.xml**. This contains parameters for configuring the Hadoop daemons. Insert the following XML snippet

```xml
<property>
    <name>hadoop.tmp.dir</name>
	<value>/usr/local/Cellar/hadoop/hdfs/tmp</value>
    <description>A base for other temporary directories.</description>
</property>
<property>
    <name>fs.default.name</name>                                     
    <value>hdfs://localhost:9000</value>                             
</property>
```

Next, we set the number of block replications (default 3) i.e. the number of times data is replicated across the cluster in the **/usr/local/Cellar/hadoop/_<X.Y.Z_>/libexec/etc/hadoop/hdfs-site.xml** file

```xml
<configuration>
	<property>
  		<name>dfs.replication</name>
  		<value>1</value>
 	</property>
</configuration>
```

Finally, we insert the following into **/usr/local/Cellar/hadoop/_<X.Y.Z_>/libexec/etc/hadoop/mapred-site.xml** to specify the IP address and port number of the *job tracker*, the process that distributes map reduce tasks throughout the cluster

```xml
<configuration>
	<property>
  		<name>mapred.job.tracker</name>
  		<value>localhost:9010</value>
 	</property>
</configuration>
```

To allow the job tracker to operate on localhost, remote login must be enabled under **System Preferences -> Sharing**. Also, cryptographic keys must be generated (if they do not already exist) and added to the authorised keys with

```sh
$ ssh-keygen -t rsa
$ cat ~/.ssh/id_rsa.pub >> ~/.ssh/authorized_keys
```

To confirm this gives access to the job tracker we can manually log in with

```sh
$ ssh localhost
```

Finally, we format the hdfs with

```sh
$ hdfs namenode -format
```

## Usage

To test out the configuration we take the **WordCount.java** source from the Apache [MapReduce tutorial](https://hadoop.apache.org/docs/current/hadoop-mapreduce-client/hadoop-mapreduce-client-core/MapReduceTutorial.html). To start Hadoop, we execute the pair of scripts

```sh
$ /usr/local/Cellar/hadoop/_<X.Y.Z_>/sbin/start-dfs.sh;/usr/local/Cellar/hadoop/_<X.Y.Z_>/sbin/start-yarn.sh
```

Hadoop will now run in the background until we stop it with

```sh
$ /usr/local/Cellar/hadoop/_<X.Y.Z_>/sbin/stop-yarn.sh;/usr/local/Cellar/hadoop/_<X.Y.Z_>/sbin/stop-dfs.sh
```

Note it is useful to give each of these commands an alias to make repeated use easier. The Hadoop web interfaces can be accessed at **http://localhost:50070** for the resource manager and **http://localhost:8088/** for the job tracker. For the full set of dfs commands we may run

```sh
$ hdfs dfs -help
```

 We now create a directory on the hdfs with

```sh
$ hdfs dfs -mkdir /input
```

to which we add a text file **input.txt** containing some (any) text with

```sh
$ hdfs dfs -put /input/input.txt
```

Compiling our source code

```sh
$ javac WordCount.java -cp $(hadoop classpath)
$ jar cf wc.jar WordCount*.class
```

we can run our program with

```sh
$ hadoop jar wc.jar WordCount input.txt output.txt 
```

Once complete, we can view program output with

```sh
$ hdfs dfs -cat /output/part-r-00000
```

## Credits

This demo was modelled after the tutorial given on [dtflaneur.wordpress.com](https://dtflaneur.wordpress.com/2015/10/02/installing-hadoop-on-mac-osx-el-capitan/).











