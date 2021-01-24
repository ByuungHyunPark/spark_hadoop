# Hadoop3.1.4 Single mode install
- __참고 링크__ : https://data-flair.training/blogs/installation-of-hadoop-3-on-ubuntu/)
-  [jdk 설치 (java SE 8)](https://www.oracle.com/java/technologies/javase-jdk11-downloads.html)

---

__Step 01 :__ install ssh on your system using the below command:
```
$ sudo apt-get install ssh
```  

<br>

__Step 02 :__ Install pdsh on your system using the below command
```
$ sudo apt-get install pdsh
```

<br>

__Step 03:__ Open the .bashrc file in the nano editor using the following command:
```
$ nano .bashrc

export PDSH_RCMD_TYPE=ssh  # 맨 아래에 추가 , ctrl+X로 저장
```

<br>

__Step 04:__ Now configure ssh. To do so, create a new key with the help of the following command:
```
$ ssh-keygen -t rsa -P ""
``` 
![sshKey](/img/ssh_key.png)

__Step 05:__ Copy the content of the public key to authorized_keys.
```
$ cat ~/.ssh/id_rsa.pub >> ~/.ssh/authorized_keys
```
![copyKey](/img/copy_key.png)

<br>

__Step06 :__ Now examine the SSH setup by connecting to the localhost

- "__yes__"를 입력하여, connection 진행

<br>

__Step07__ : Update the source lists.
```
$ sudo apt-get update
```

<br>

__Step08__ : Noew install Java 8 using the following command:
```
$ sudo apt-get install openjdk-8-jdk
```
<br>

__Step09__ : To cross-check wheter you have successfully installed Java on your machine or not, run the below command:
```
$ java -version
$ javac -version
```
<br>

__Step 10__ : Now locate the Hadoop tar file in your system.
- 직접, apache hadoop 홈페이지에서도 설치 가능
```
$ wget https://downloads.apache.org/hadoop/common/hadoop-3.1.4/hadoop-3.1.4.tar.gz
```
<br>

__Step 11__ : Extract the __Hadoop-3.1.4.tar.gz__ file using the below command
```
$ tar xzf hadoop-3.1.4.tar.gz 
```
![wget](/img/wget.png)

<br>

__Step 12 :__ Rename __hadoop-3.1.2.tar.gz__ as __hadoop__ for ease of use.
```
$ mv hadoop-3.1.4 hadoop
```
_Any doubts in the process to install Hadoop 3.1.4 till now? Share them in the comment section._

<br>

__Step 14 :__ Now check the Java home path which we will set up in Hadoop env.sh


```
$ ls /usr/lib/jvm/java-8-openjdk-amd64/
```
![ls](/img/ls.png)
<br>

__Step 15 :__ Open the __hadoop-env.sh__ file in the nano editor. This file is located in ~/__hadoop/etc/hadoop__ configuration directory.
```
$ nano hadoop-env  # 링크에는 이렇게나왔지만, 아래가 맞는듯?
$ nano hadoop-env.sh
```
![nano1](/img/nano1.png)
##### Set JAVA_HOME
```
#export JAVA_HOME=<path-to-the-root-of-your-Java-installation> (eg: /usr/lib/jvm/java-8-openjdk-amd64/)

export JAVA_HOME=/usr/lib/jvm/java-8-openjdk-amd64/
```
![export](/img/step15.png)



To Save the changes you've made, press __Ctrl+O__, To exit the nano editor, press __Ctrl+X__ and then pree __'Y'__ to exit the editor.

<br>

__Step 16 :__ Open the __core-site.xml__ file in the nano deitor. This file is also located in the ~/__hadoop/etc/hadoop__ (Hadoop configuration directory).

```
$ nano core-site.xml
```
And the Following configuration properties:
```
<configuration>
    <property>
        <name>fs.defaultFS</name>
        <value>hdfs://localhost:9000</value>
    </property>
    <property>
        <name>hadoop.tmp.dir</name>
        <value>/home/dataflair/hdata</value>
    </property>
</configuration>
```
![step16_2](/img/step16_2.png)

<br><br>

__Step 17 :__ Open the __hdfs-site.xml__ file in the nano editor. This file is also located in ~/__hadoop/etc/hadoop__ (Hadoop configuration directory):
```
$ nano hdfs-site.xml
```
Add the following entries in core-site.xml:
```
<configuration>
    <property>
        <name>dfs.replication</name>
        <value>1</value>
    </property>
</configuration>
```

<br><br>

__Step 18 :__ Open the __mapred-site.xml__ file in the nano editor. This file is also located in ~/__hadoop/etc/hadoop__ (Hadoop configuration directory).

```
$ nano mapred-site.xml
```
Add the following entries in core-site.xml:
```
<configuration>
    <property>
        <name>mapreduce.framework.name</name>
        <value>yarn</value>
    </property>
<property>
 <name>yarn.app.mapreduce.am.env</name>
 <value>HADOOP_MAPRED_HOME=/home/dataflair/hadoop</value>
</property>
<property>
 <name>mapreduce.map.env</name>
 <value>HADOOP_MAPRED_HOME=/home/dataflair/hadoop</value>
</property>
<property>
 <name>mapreduce.reduce.env</name>
 <value>HADOOP_MAPRED_HOME=/home/dataflair/hadoop</value>
</property>
</configuration>
```
![step18](/img/step18.png)

<br><br>

__Step19 :__
 Open the __yarn-site.xml__ file in the nano editor. This file is also located in ~/__hadoop/etc/hadoop__ (Hadoop configuration directory).

```
$ nano yarn-site.xml
```
Add the following entries in the core-site.xml:
```
<configuration>
    <property>
        <name>yarn.nodemanager.aux-services</name>
        <value>mapreduce_shuffle</value>
    </property>
    <property>
        <name>yarn.nodemanager.aux-services.mapreduce.shuffle.class</name>
        <value>org.apache.hadoop.mapred.ShuffleHandler</value>
    </property> 
</configuration>
```

![img19](/img/step19.png)

__Step 20 :__ Open the bashrc files in the nano editor using the following command :
```
$ nano .bashrc
```

![step20_1](/img/step20_1.png)
Edit .bashrc file located in the user's home directory and add the following parameters:
```
export HADOOP_HOME="/home/dataflair/hadoop"
export PATH=$PATH:$HADOOP_HOME/bin
export PATH=$PATH:$HADOOP_HOME/sbin 
export HADOOP_MAPRED_HOME=${HADOOP_HOME}
export HADOOP_COMMON_HOME=${HADOOP_HOME}
export HADOOP_HDFS_HOME=${HADOOP_HOME}
export YARN_HOME=${HADOOP_HOME}
```
![step20_2](/img/step20_2.png)
To save the changes you've made, press __Ctrl+O__. To exit the nano editor, press __Ctrl+X__ and then press __'Y'__ to exit the editor.

<br><br>

__Step 21:__ Before starting Hadoop, we need to format HDFS, which can be done using the below command:
```
~/hadoop $ bin/hdfs namenode -format
```
![step21_1](/img/step21_1.png)


- 아래 과정에서 , __HADOOP_HOME__ 제대로 설정하지 않아서  위 코드가 실행되지 않았음.
- 상당히 고생했지만 , hadoop home 제대로 설정해줘서 해결

```
$ echo $HADOOP_HOME  
# 하둡 설치한 폴더에 제대로 설정 되있는지 확인
```
![step21_2](/img/21_2.png)


__Step 22 :__ Start the HDFS services:
```
$ sbin/start-dfs.sh
```
![step22](/img/step22.png)

__Step 23 :__ Open the HDFS Web console:
```
localhost:9870
```
웹 접속이 안됨,, namenode 관련해서 추가공부 해서 더 올릴예정
