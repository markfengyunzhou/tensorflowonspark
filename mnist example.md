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

