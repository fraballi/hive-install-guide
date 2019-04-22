# Hive Installation

> ## *Step-By-Step Guide for Hive Deployment*
>
> *Hive version: 3.1.1*

---

## Create a Docker Container

```bash

docker run -it --name hive-node -h hive --cpus=1 \
--net=hadoop-cluster \
--ip="192.168.0.102" \
--memory="1024MB" --memory-swap="2048MB" \
--add-host="master:192.168.0.1" \
--add-host="ambari:192.168.0.100" \
--add-host="kafka:192.168.0.101" \
-v /var/logs/hive:/var/logs/hive \
debian:hadoop-hive

```

## Copy Hive Binaries to Docker Container

```bash

docker cp <local-folder>/<hive-binaries> hive-node:/opt/<hive-folder>

docker exec -it hive-node bash

# e.g 'apache-hive-3.1.1-bin.tar.gz'
tar -xvf apache-hive-x.y.z-bin.tar.gz  # Creates folder: /opt/apache-hive-x.y.z-bin

```

## Set Environment Variables / Entrypoint (**/start-hive.sh**)

```bash

# e.g Or run command: 'java -version', to check installed version
export JAVA_HOME=/opt/jdk1.8.0_201  

export HIVE_HOME=/opt/apache-hive-3.1.1-bin
export HIVE_CONF_DIR=$HIVE_HOME/conf
export HIVE_HCATALOG=$HIVE_HOME/hcatalog
export PATH=$PATH:$HIVE_HOME/bin

# !!!Important: Put HADOOP CONFIGURATION section here (Check 'hadoop-install-guide')
# <HADOOP environment variables>

export HADOOP_PID_DIR=/home/hadoop/hdfs/pid

```

### *[Optional] Install Local Tools*

```bash

sudo apt install telnet rsync curl nano less

```

## *Configure Passwordless SSH*

```bash

sudo apt install openssh-server openssh-client

```

### *Generate a SSH Key*

```bash

ssh-keygen -t rsa

cat ~/.ssh/id_rsa.pub >> ~/.ssh/authorized_keys

service ssh start

ssh localhost

```

## *Edit Hive settings*

```bash

# Create new settings from templates

cp $HIVE_CONF_DIR/hive-default.xml.template $HIVE_CONF_DIR/hive-site.xml

cp $HIVE_CONF_DIR/hive-env.sh.template $HIVE_CONF_DIR/hive-env.sh
```

### *Edit Hive Environment *(**/hive-env.sh**)**

```bash

export HADOOP_HOME=$HADOOP_HOME

# ...

export HEAP_SIZE=512

# ...

```

### *Edit Hive Site *(**/hive-site.xml**)**

```xml

<!-- Important:Add these ath the beginning of this file (after <configuration> ...) -->
<property>
  <name>system:java.io.tmpdir</name>
  <value>/tmp/hive/java</value>
</property>
<property>
  <name>system:user.name</name>
  <value>${user.name}</value>
</property>

<!-- Important: Modify/Add these properties below -->
<property>
  <name>javax.jdo.option.ConnectionURL</name>
  <value>jdbc:derby:;databaseName=/opt/apache-hive-3.1.1-bin/metastore_db;create=true</value>
  <description>
JDBC connect string for a JDBC metastore.
To use SSL to encrypt/authenticate the connection, provide database-specific SSL flag in the connection URL.
For example, jdbc:postgresql://myhost/db?ssl=true for postgres database.
  </description>
</property>
<property>
  <name>hive.metastore.warehouse.dir</name>
  <value>/user/hive/warehouse</value>
  <description>location of default database for the warehouse</description>
</property>
<property>
  <name>hive.metastore.uris</name>
  <value/>
  <description>Thrift URI for the remote metastore. Used by metastore client to connect to remote metastore.</description>
</property>
<property>
  <name>javax.jdo.option.ConnectionDriverName</name>
  <value>org.apache.derby.jdbc.EmbeddedDriver</value>
  <description>Driver class name for a JDBC metastore</description>
</property>
<property>
  <name>javax.jdo.PersistenceManagerFactoryClass</name>
  <value>org.datanucleus.api.jdo.JDOPersistenceManagerFactory</value>
  <description>class implementing the jdo persistence</description>
</property>

```

## *Initialize Metastore Schema*

```bash

# Hive binaries already set in Step: 'Set Environment Variables / Entrypoint (/start-hive.sh)'  

schematool -initSchema -dbType derby

# Fix Exception:
# 'Exception in thread "main" java.lang.RuntimeException: com.ctc.wstx.exc.WstxParsingException: Illegal character entity: expansion character (code 0x8' at [row,col,system-id]: [3218,96,"file:/opt/apache-hive-3.1.1-bin/conf/hive-site.xml"]

nano $HIVE_CONF_DIR/hive-site.xml

# Property: hive.txn.xlock.iow
# Remove characters: ...acquire Exclusive locks for '&#8'

```

## Create Hive HDFS Directories

```bash

# Command to execute in: 'hive-node'
hadoop dfsadmin -safemode leave

hdfs dfs -mkdir -p /user/hive/warehouse

hdfs dfs -mkdir /tmp

# Set HDFS folders permissions
hdfs dfs -chmod g+w /user/hive/warehouse
hdfs dfs -chmod g+w /tmp

```

## Launch Hive

```bash

# Enters into Hive's std-in
hive

```

## Testing a Database

```bash

# Important: Previous step is mandatory (opens Hive's std-in)
show databases;

create table employee (id string, name string, dept string) row format delimited fields terminated by '\t' stored as textfile;

# Exits from Hive's std-in
exit;

```

## Apache Hive Documentation

- ### *[Apache Hive: Installation on Ubuntu](https://www.edureka.co/blog/apache-hive-installation-on-ubuntu)*

### Passwordless SSH

- ### *[SSH: 2-Steps Passwordless Ubuntu Set-Up](https://www.linuxbabe.com/linux-server/setup-passwordless-ssh-login)*