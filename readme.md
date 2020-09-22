# Hadoop cluster project

This project contains information on how to create a Hadoop Cluster.
I am using the instruction that is located here: `https://tecadmin.net/set-up-hadoop-multi-node-cluster-on-centos-redhat/` and here:
`https://linuxconfig.org/how-to-install-hadoop-on-redhat-8-linux` and here:
`https://www.linode.com/docs/databases/hadoop/how-to-install-and-set-up-hadoop-cluster/`.
The first link is for multi-node cluster while the second is single node.
To install Open JDK 8, I used this link `https://www.digitalocean.com/community/tutorials/how-to-install-java-on-centos-and-fedora`

## Materials

- Centos 8 with NO GUI: 4 GB Memory, 4 CPUs, 50 GB diskspace.
- Open JDK 8
- 2 Slave Nodes (`hadoop-slave-1`, `hadoop-slave-2`), 1 Master Node (`hadoop-master`)

That is it for now.

## Steps in prepration

1. Create a bare VM with Centos 8
2. Install Open JDK 8 with the latest version.
3. Establish JAVA_HOME for the Open JDK (`sudo alternatives --config java`)
4. Edit /etc/hosts to create a FQDN `hadoop-master` entry with the host IP (varied).
5. Create user account `hadoop` with password `hadoop`
6. Shutdown `hadoop-master` VM
7. Clone (`full`) this VM for `hadoop-slave-1` and `hadoop-slave-2`
8. Run `hadoop-slave-1` and `hadoop-slave-2`
9. Run `ifconfig` to find their IP addresses.
10. Edit `/etc/hosts` file and update the host names and IP addresses accordingly.
11. Shutdown the VMs

## Steps in installing Hadoop

1. Start `hadoop-master`, `hadoop-slave-1`, and `hadoop-slave-2` VMs
2. Make sure to update `/etc/hosts` of hadoop-master with appropriate hosts and ips of
`hadoop-slave-1` and `hadoop-slave-2`:

    ```shellscript
    192.168.1.23    hadoop-master
    192.168.1.132   hadoop-slave-1
    192.168.1.120   hadoop-slave-2
    ```

3. Disable firewall
    I disabled the firewalls from `hadoop-master`, `hadoop-slave-1`, and `hadoop-slave-2`.
    I could have just enable `ports` but was too lazy to do that for this experiment.

    ```shellscript
    sudo systemctl stop firewalld
    sudo systemctl disable firewalld
    ```

4. Configuring key based login. Do this on hadoop master vm. Use blank passphrase

    ```shellscript
    su - hadoop
    ssh-keygen -t rsa
    ssh-copy-id -i ~/.ssh/id_rsa.pub hadoop@hadoop-master
    ssh-copy-id -i ~/.ssh/id_rsa.pub hadoop@hadoop-slave-1
    ssh-copy-id -i ~/.ssh/id_rsa.pub hadoop@hadoop-slave-2
    chmod 0600 ~/.ssh/authorized_keys
    exit
    ```

5. Download 3.3.0 from here `http://apache.spinellicreations.com/hadoop/common/hadoop-3.3.0/hadoop-3.3.0.tar.gz`

    ```shellscript
    mkdir /opt/hadoop
    cd /opt/hadoop/
    wget http://apache.spinellicreations.com/hadoop/common/hadoop-3.3.0/hadoop-3.3.0.tar.gz
    tar -xzf hadoop-3.3.0.tar.gz
    mv hadoop-1.2.0 hadoop
    chown -R hadoop /opt/hadoop
    cd /opt/hadoop/hadoop/
    ```

6. Install hadoop as instructured here with an exception of using the latest 3.3.0

7. Edit these files:

    - /opt/hadoop/hadoop/etc/hadoop/core-site.xml
    - /opt/hadoop/hadoop/etc/hadoop/hdfs-site.xml
    - /opt/hadoop/hadoop/etc/hadoop/mapred-site.xml
    - /opt/hadoop/hadoop/etc/hadoop/hadoop-env.xml

    ```shellscript
    export JAVA_HOME=/usr/lib/jvm/java-1.8.0-openjdk-1.8.0.262.b10-0.el8_2.x86_64
    export HADOOP_OPTS=-Djava.net.preferIPv4Stack=true
    export HADOOP_CONF_DIR=/opt/hadoop/hadoop/conf
    ```

    - .bashrc

    ```shellscript
    export JAVA_HOME=/usr/lib/jvm/java-1.8.0-openjdk-1.8.0.262.b10-0.el8_2.x86_64
    export HADOOP_HOME=/opt/hadoop/hadoop
    export PATH=$PATH:$HADOOP_HOME/bin:$HADOOP_HOME/sbin
    ```

8. Format namenode

    ```shellscript
    hdfs namenode -format
    ```

9. Start services

    ```shellscript
    start-dfs.sh
    ```

    ```shellscript
    stop-dfs.sh
    ```

    ```shellscript
    start-yarn.sh
    ```

    ```shellscript
    stop-yarn.sh
    ```

    ```shellscript
    mapred --daemon start historyserver
    ```

    ```shellscript
    mapred --daemon stop historyserver
    ```

10. Accessing hadoop cluster remotely

**update @hadoop-master /opt/hadoop/hadoop/etc/hadoop/hdfs-site.xml**

```xml
<property>
  <name>dfs.permissions</name>
  <value>false</value>
</property>
```

I logged in as `student` account under my VM.

- I ssh'ed into `hadoop-master` as the hadoop cluster account `hadoop`
- I created an hdfs folder `/user/student`:

    ```shellscript
    hdfs dfs -mkdir -p /user/student
    ```

- I changed owner from `hadoop` to `student`

    ```shellscript
    hdfs dfs -chown -R student /user/student
    ```

- From my VM logged in as `student`, I access the HDFS cluster using all the 
usual commands except that I need to prefix the hat with `hdfs://hadoop-master:9000`.
For example,

    ```shellscript
    hdfs dfs -mkdir hdfs://hadoop-master:9000/user/student/shakespeare
    ```

*Notes*:

1. **hdfs-site.xml: data and name directories must be separate folders**
2. **disable firewall in order to access to the website**
3. **unless, you permanently disable the firealld, you needs to disable it from all VMs prior to start-all.sh**
