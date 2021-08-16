# Results
We have conducted comparative experiments on 15 spark-bench applications. We compared LITE with four state-of-the-art Spark tuning methods. (1) Default: we
recorded the execution time of applications upon default configurations. (2) Manual: we hired experts to tune the applications based on his expertise and the tuning guides for maximally 12 hours. (3) BO: we used Bayesian Optimization, where Gaussian Process was the surrogate model and Expected Improvement was the acquisition function. We used 5 most similar instances in the training set to initialize Gaussian Process. Then, BO was trained for at least 2 hours. (4) DDPG: we built a reinforcement learning framework which
used Deep Deterministic Policy Gradient, where the action space was the configurations, and the state was the inner status summary of Spark. DDPG was also trained for at least 2 hours.

|          | PCA  | CC     | DT   | KM    | LP   | LR   | Logit | PR   | PO   | SP   | SCC  | SVD++ | SVM    | TS   | TC   |
|----------|------|--------|------|-------|------|------|-------|------|------|------|------|-------|--------|------|------|
| Default  | 3600 | 7200   | 5578 | 18756 | 7200 | 7141 | 2649  | 7200 | 7200 | 7200 | 7200 | 7200  | 7200   | 7200 | 7200 |
| Manual   | 408  | 304    | 498  | 2655  | 413  | 1283 | 324   | 4099 | 253  | 217  | 515  | 445   | 665    | 667  | 7200 |
| DDPG(2h) | 1396 | 98     | 523  | 3288  | 349  | 3476 | 1126  | 2553 | 395  | 7200 | 1095 | 7200  | 3600   | 1131 | 114  |
| DDPG+CNN |      |        |      |       |      |      |       |      |      |      |      |       |        |      |      |
| BO(2h)   | 339  | 84     | 737  | 2884  | 353  | 614  | 619   | 7200 | 249  | 168  | 586  | 423   | 675    | 1865 | 81   |
| LITE     | 316  | 81.933 | 449  | 2881  | 348  | 448  | 345   | 2184 | 116  | 145  | 316  | 352   | 456.97 | 325  | 65   |
| RFR      | 498  | 720    | 1380 |       |      | 588  | 480   | 7200 | 7200 | 7200 |      |       | 660    | 336  |      |
| tmin     | 316  | 81.933 | 449  | 2655  | 348  | 448  | 324   | 2184 | 116  | 145  | 316  | 352   | 456.97 | 325  | 65   |


# LITE 
## Enviroment

The project is implemented using python 3.6 and tested in Linux environment. Our system environment and cuda version as follows:

```
ubuntu 18.04
Hadoop 2.7.7
spark 2.4.7
HiBench 7.0
JAVA 1.8
SCALA  2.12.10
Python 3.6
Maven 4.15
CUDA Version: 10.1
```

We get the training data by running the workload in SparkBench，which can be installed by referring to https://github.com/CODAIT/spark-bench.

## 1. Data Generate


You can generate log files using the following command:

```
python scripts/bo_sample.py <path_to_spark_bench_folders>
```

The log file will be saved on your spark history server, you can use the following command to download it to the local.

```
hdfs dfs -get <path_to_spark_history_server_folders_on_hdfs> <local_folders>
```

## 2. Process Original Log

**scripts/history_by_stage.py** is used to parse the original log file generated by spark-bench, which get the running time of each stage, the amount of data read and write, etc.

After the above steps, the generated data is still based on workload, and you need to use **scripts/build_dataset.py** to divide each row into multiple stages. Then merge the data of all stages of all workloads into a complete data set and store it in csv format. The data set can be divided into training set and test set as needed.

## 3.Data Preprocessing

Obtain the stage code characteristics: enter the instrumentation folder, mark maven into a jar package, and add it to the spark-submit command.

```sh
--conf "spark.driver.extraJavaOptions=-javaagent:/pathtoyourjar/preMain-1.0.jar"
```

Our model is saved in prediction_nn,

You should use **data_process_text.py、dag2data.py、dataset_process.py** to get dictionary information、Process the edge and node information of the graph, and convert the data features into graph features.

## 4.Model Training

Then use **fast_train.py** to train model.

Use **trans_learn.py** finetune the model.

## 5.Model Testing

**nn_pred_8.py** can test the model，we use predict_first_cold.py to predict the best combination of parameters and check its actual operation.

