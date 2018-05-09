## 1、安装python

cd Python-3.6.0

mkdir ../Python

./configure --prefix /home/hpe/Python --with-ssl

make all & make install

## 2、打包zip，upload用
cd ../Python

zip -r Python.zip *

pip3 install tensorflow
pip3 install tensorflowonspark

## 3、下载tensorflowonspark源码

wget https://codeload.github.com/yahoo/TensorFlowOnSpark/zip/master

## 4、mnist文件转csv
四个文件打包到zip
zip -r mnist.zip *

PYSPARK_DRIVER_PYTHON=Python/bin/python3 \
PYSPARK_PYTHON=Python/bin/python3 \
spark-2.2.0-bin-hadoop2.7/bin/spark-submit \
--master yarn \
--deploy-mode cluster \
--num-executors 4 \
--executor-memory 4G \
--archives hdfs:///data/mnist/mnist.zip#mnist,Python/Python.zip#Python /home/hpe/TensorFlowOnSpark/examples/mnist/mnist_data_setup.py \
--output mnist/csv --format csv

## 5、下载ecosystem源码

protoc --proto_path=/home/hpe/tensorflow/ --java_out=/home/hpe/ecosystem/hadoop/src/main/java/ /home/hpe/tensorflow/tensorflow/core/example/{example,feature}.prot

cd /home/hpe/tensorflow

mvn clean package


## 6、上传jar

hadoop dfs -put  /home/hpe/ecosystem/hadoop/target/tensorflow-hadoop-1.6.0.jar /lib

## 7、训练
```
必须保证所有的executor能够同时获取到资源并运行起来， executor-cores不需要设置，或者只能设置为1
```
PYSPARK_DRIVER_PYTHON=Python/bin/python3 \
PYSPARK_PYTHON=Python/bin/python3 \
spark-submit \
--master yarn \
--deploy-mode cluster \
--num-executors  3 \
--executor-memory 20G \
--driver-memory 10G \
--py-files /home/hpe/TensorFlowOnSpark/examples/mnist/spark/mnist_dist.py \
--conf spark.dynamicAllocation.enabled=false \
--conf spark.yarn.maxAppAttempts=1 \
--conf spark.executorEnv.LD_LIBRARY_PATH="${JAVA_HOME}/jre/lib/amd64/server" \
--conf spark.executorEnv.CLASSPATH="$($HADOOP_HOME/bin/hadoop classpath --glob):${CLASSPATH}" \
--jars /home/hpe/ecosystem/hadoop/target/tensorflow-hadoop-1.6.0.jar \
--archives hdfs:///lib/Python.zip#Python \
/home/hpe/TensorFlowOnSpark/examples/mnist/spark/mnist_spark.py \
--images hdfs:///user/hpe/mnist/csv/train/images \
--labels hdfs:///user/hpe/mnist/csv/train/labels \
--format csv \
--mode train \
--model mnist_model

`生成model`
```
/user/hpe/mnist_model/checkpoint
/user/hpe/mnist_model/events.out.tfevents.1525836375.hpe01
/user/hpe/mnist_model/events.out.tfevents.1525836376.hpe01
/user/hpe/mnist_model/graph.pbtxt
/user/hpe/mnist_model/model.ckpt-1.data-00000-of-00001
/user/hpe/mnist_model/model.ckpt-1.index
/user/hpe/mnist_model/model.ckpt-1.meta
/user/hpe/mnist_model/model.ckpt-601.data-00000-of-00001
/user/hpe/mnist_model/model.ckpt-601.index
/user/hpe/mnist_model/model.ckpt-601.meta
```
