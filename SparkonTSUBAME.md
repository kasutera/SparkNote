# Spark on TSUBAME
This document provides tips to set up Spark on TSUBAME.

## Overview
1. Compile Spark on your laptop
2. Upload the compiled Spark to TSUBAME
3. Set up JDK
4. Set SPARK_HOME
5. Run on Batch Node
6. Submit Spark job and get running history
7. Profile

## Reference
(Spark document)
- http://spark.apache.org/docs/latest/
- http://spark.apache.org/docs/latest/building-spark.html
- http://spark.apache.org/docs/latest/spark-standalone.html
- https://spark.apache.org/docs/latest/configuration.html

(TSUBAME document)
- http://tsubame.gsic.titech.ac.jp/docs/guides/tsubame2/html_en/overview.html#storage
- http://tsubame.gsic.titech.ac.jp/docs/guides/tsubame2/html_en/queues.html

(Others)
- http://0x0fff.com/spark-architecture/
- http://0x0fff.com/spark-architecture-shuffle/
- http://0x0fff.com/spark-memory-management/

## 1. Compile Spark on your laptop
The quick way to compile Spark project is to use self-contained maven (`build/mvn`).
The build takes long time (over 30 minutes at the first time).
```
(on your laptop)
$ git clone https://github.com/apache/spark.git
$ cd spark
$ build/mvn -DskipTests clean package
```
If you use maven in your laptop, you have to set `MAVEN_OPTS` for larger memory space.
```
$ git clone https://github.com/apache/spark.git
$ cd spark
$ export MAVEN_OPTS="-Xmx2g -XX:MaxPermSize=512M -XX:ReservedCodeCacheSize=512m"
$ mvn -DskipTests clean package
```

If you want to build with IntelliJ, see [SparkonIntelliJ.md](./SparkonIntelliJ.md).

## 2. Upload the compiled Spark to TSUBAME
Use `scp` `rsync` to upload.

## 3. Set up SDK

## 4. Set SPARK_HOME
Set `$SPARK_HOME`.
For performance, Spark project should be on GPFS(`/data0`) or Lustre(`/work0`) instead of nfs(`home`).
```
(on TSUBAME)
$ export SPARK_HOME="[Spark directory]"
```
## 5. Run on Batch Node
First, run Spark interactively using `allocnode` script.
`allocnode` is in `/work1/t2gsgraph/apps/utils/bin/`.

Log in Batch Node.
```
$ allocnode -g [TSUBAME Group Name] -n [# of Nodes] -w [Time(hour)] -q S
```
Spark needs over 4G Java heap memory. Thus you need to set up `_JAVA_OPTIONS`.
```
$ export _JAVA_OPTIONS="-Xmx45G"
```
And then launch master (current node) and worker (other allocated nodes).
```
(Start master)
$ cd $SPARK_HOME
$ ./sbin/start-master.sh

(Start workers)
$ grep -v `hostname` $PBS_NODEFILE > conf/slaves
$ ./sbin/start-slaves.sh
```
For setting up Spark configuration,  edit `conf/spark-defaults.conf` like that.
```
$ echo -e \
"spark.executor.memory  45g  \n"\
"spark.executor.cores   12   \n"\
"spark.local.dir        /scr \n"\
> ./conf/spark-defaults.conf
```
Launch Spark application
```
$ ./bin/spark-submit --depoly-mode cluster \
  --class org.apache.spark.examples.SparkPageRank \
  --master spark://`localhost`:6066 \
  ./examples/target/spark-examples_2.11-2.0.0-SNAPSHOT.jar \
  ./graphx/data/followers.txt 30
```
You can see the execution result from Web UI.
```
$ w3m http://localhost:8080
```
Before exit from Batch Node, stop master and worker.
```
$ ./sbin/stop-all.sh
```

## 6. Submit Spark job and get running history
For running on larger-scale machines, the interactive way is not suited.
Submitting job **with a job script** is a good idea.
A template of the job script is [here](./src/job_script.template).

First, you should edit the job_script.template.
```
#### Editable Parameters ####
# Spark application class including main() and its arguments.
SPARK_APP_CLASS="org.apache.spark.examples.SparkPageRank"
SPARK_APP_ARGS="$SPARK_HOME/graphx/data/followers.txt 30

# Spark executable jar file
SPARK_JAR="$SPARK_HOME/examples/target/spark-examples_2.11-2.0.0-SNAPSHOT.jar"

# Number of threads per node (spark.executor.cores)
SPARK_NUM_THREAD=12

# Memory size per node (spark.executor.memory)
SPARK_MEM_SIZE=45g

# Local scratch file directory (spark.local.dir)
# /scr is local SSD. /tmp is NFS
SPARK_LOCAL_FILE_DIR="/scr"

# Event log directory (spark.eventLog.dir)
SPARK_LOG_DIR="[path to eventLog]"

# If you want to set up other Spark parameters, edit source code below.
#############################
```
And then, make your own script (named `job_script.sh`)
Submit `job_script.sh` to PBS with `t2sub`.
For more details, see [the TSUBAME document 5.2 Submission]( http://tsubame.gsic.titech.ac.jp/docs/guides/tsubame2/html_en/queues.html#submission).
```
$ t2sub -W group_list=[TSUBAME Group] \
        -q S \
        -l select=[Num Nodes]:mpiprocs=1:ncpus=12:mem=50gb \
        -l place=scatter \
        ./job_script.sh
```

After finishing the job, you can see the execution info from Web UI (port 18080 with `firefox`(needs X forwarding) or `w3m`).
```
$ ./sbin/start-history-server.sh
(You need to log in with X forwarding like: $ ssh -X user@131.112.4.49)
$ firefox http://localhost:18080
```
After checking, you should quit the history server.
```
$ ./sbin/stop-history-server.sh
```

The history server can be launched in your laptop.
```
(In your laptop)
$ cd [Spark Home]
$ mkdir eventlog
$ scp -C [usrID]@131.112.4.49:[path to eventLog]/* ./eventlog
$ ./sbin/start-history-server eventlog
```
And then you can see `http://localhost:18080` in your laptop.
After checking, you should quit the history server.
```
$ ./sbin/stop-history-server.sh
```

## 7. Profile
**TODO**

created by Masa (Feb. 23 2016)
