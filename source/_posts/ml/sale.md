---
title: tensorflow多特征预测销量demo
date: 2018-01-31 15:23:22
tags:
- 机器学习
categories: 机器学习
keywords:
- tensorflow
---







最近一直在搞这个，终于算是跑成了，知识点掌握不全面，但坑走过一两个还是有进步。。



这个项目做得是多维度特征对于月销量的一个回归，操作主要包括两方面，训练并生成模型、 导入模型预测结果。



<!--more-->

github地址： https://github.com/ruushwei/TensorDemo



###  训练



样本数据是我自己找的70行数据，放到了csv文件中，用于训练

测试数据为根据一条样本数据编的5条测试数据，只改了其中两个特征，用于预测因为这两个特征改变导致结果的变化。

样本与测试数据在github上



步骤：

从文件读取数据 —>  tensorflow图的构建 —>  执行session进行训练  ——> 通过画图观察拟合程度



code:

```python
from __future__ import print_function
import tensorflow as tf
from sklearn.preprocessing import Normalizer
import matplotlib.pyplot as plt
import numpy as np


### 从文件获取数据
my_matrix = np.loadtxt(open("data2.csv", "rb"), dtype=np.float, delimiter=",", skiprows=1)

## 获取除第一列数据，第一列为结果值，写列号为笨方法，不知道该怎么写
tezheng = my_matrix[:,[1,2,3,4,5,6,7,8,9,10,11,12,13,14,15,16,17,18,19,20,21,22,23,24,25,26,27,28,29,30,31,32,33,34,35,36,37,38,39,40,41,42,43,44,45,46,47,48,49,50,51,52]]    

## 销量结果，用于训练
sale = my_matrix[:,0]

## 将特征进行归一化，使训练的时候不会受大小特征影响严重，注：非scale，scale是对于列的，scale再进行预测的时候即使相同一行数据，因为其它数据变了，这一列有变，scale之后得到的值也不一样，预测结果自然也不一样了。
## 下面这种是对行的归一化
X_train = Normalizer().fit_transform(tezheng)   ## 多维特征
y_train = sale.reshape((-1,1))        ## 结果，从一维转为二维


#### 开始进行图的构建

## 特征与结果的替代符，声明类型，维度 ，name是用来生成模型之后，使用模型的时候调用用的
inputX = tf.placeholder(shape=[None, X_train.shape[1]], dtype=tf.float32, name="inputX")
y_true = tf.placeholder(shape=[None,1], dtype=tf.float32, name="y_true")

## 丢弃样本比例，1是所有的样本使用，这个好像是自动回丢弃不靠谱的样本
keep_prob_s = tf.placeholder(dtype=tf.float32, name="keep_prob")

### 第一层，一个隐藏层 开始
## shape的第一维就是特征的数量，第二维是给下一层的输出个数,  底下的矩阵相乘实现的该转换
Weights1 = tf.Variable(tf.random_normal(shape=[52, 10]), name="weights1")  ## 权重
biases1 = tf.Variable(tf.zeros(shape=[1, 10]) + 0.1, name="biases1")       ## 偏置

## matmul矩阵相乘，nn.dropout 丢弃部分不靠谱数据
Wx_plus_b1 = tf.matmul(inputX, Weights1)
Wx_plus_b1 = tf.add(Wx_plus_b1, biases1)

Wx_plus_b1 = tf.nn.dropout(Wx_plus_b1, keep_prob=keep_prob_s)    

## 将结果曲线化，通常说非线性化
l1 = tf.nn.sigmoid(Wx_plus_b1, name="l1")

### 第一层结束

### 第二层开始，即输出层
## 上一层的10，转为1，即输出销售量
Weights2 = tf.Variable(tf.random_normal(shape=[10, 1]), name="weights2")   ## 权重
biases2 = tf.Variable(tf.zeros(shape=[1, 1]) + 0.1, name="biases2")        ## 偏置

## matmul矩阵相乘 ,l1 为上一层的结果
Wx_plus_b2 = tf.matmul(l1, Weights2)
prediction = tf.add(Wx_plus_b2, biases2, name="pred")      ## pred用于之后使用model时进行恢复

## 这里使用的这个方法还做了一个第一维结果行差别的求和，reduction_indices=1，实际这个例子每行只有一个结果,使用 loss = tf.reduce_sum(tf.square(y_true - prediction)) 即可
loss = tf.reduce_mean(tf.reduce_sum(tf.square(y_true - prediction), reduction_indices=[1]))

## 训练的operator，AdamOptimizer反正说是最好的训练器, 训练速率0.01
train_op = tf.train.AdamOptimizer(learning_rate=0.01).minimize(loss)


#### draw pics  这里画了一个坐标图
fig = plt.figure()
ax = fig.add_subplot(1, 1, 1)           # 第一个块里画
ax.plot(range(50), y_train[0:50], 'b')  # 先画出样本前50个数据的真实结果
ax.set_ylim([0, 30])                    # 设置纵轴的范围
plt.ion()                               # 打开交互模式，不打开不能在训练的过程中画图
plt.show()                              # 展示图片，现在是只有50个真实数据连成的1条线


### 开始执行
with tf.Session() as sess:

    saver = tf.train.Saver(tf.global_variables(), max_to_keep=15)     # 初始化saver，用于保存模型

    init = tf.global_variables_initializer()                           # 初始化全部变量
    sess.run(init)                                                     # 初始化全部变量

    ## 要给模型进行训练的数据，只有placeholder类型的需要传进去数据
    feed_dict_train = {inputX: X_train, y_true: y_train, keep_prob_s: 1}

    for i in range(40000):
        _loss, _ = sess.run([loss, train_op], feed_dict=feed_dict_train)  # 训练，注：loss没有训练，只是走到loss，返回值，走到train_op才会训练
        if i % 1000 == 0:
            print("步数:%d\tloss:%.5f" % (i, _loss))
            pred_feed_dict = {inputX: X_train, keep_prob_s: 1}      # 用来预测的数据,不需要y
            pred = sess.run(prediction, feed_dict=pred_feed_dict)   # 走到prediction即可

            ## 将预测的结果画到图像上,与真实值做对比
            try:
                ax.lines.remove(lines[0])
            except:
                pass
            lines = ax.plot(range(50), pred[0:50], 'r--')
            plt.pause(1)

    # 保存模型
    saver.save(sess=sess, save_path="nn_boston_model/nn_boston.model", global_step=40000)  

## done, 关闭程序后model就会出现在文件夹中
```











###  使用model预测



很简单，大概就是获取整个图结构和所有变量信息，然后走图，到prediction，就可以预测了



```Python
import tensorflow as tf
from sklearn.preprocessing import Normalizer
import numpy as np

### 从文件获取测试数据
my_matrix2 = np.loadtxt(open("data3.csv", "rb"), dtype=np.float, delimiter=",", skiprows=1)

tezheng2 = my_matrix2[:, [1,2,3,4,5,6,7,8,9,10,11,12,13,14,15,16,17,18,19,20,21,22,23,24,25,26,27,28,29,30,31,32,33,34,35,36,37,38,39,40,41,42,43,44,45,46,47,48,49,50,51,52]]

X = Normalizer().fit_transform(tezheng2)

with tf.Session() as sess:

    # 拿到 图的元信息 saver
    saver = tf.train.import_meta_graph(meta_graph_or_file="nn_boston_model/nn_boston.model-40000.meta")          

    # 这个也很重要，虽然还不知道是做什么的
    model_file = tf.train.latest_checkpoint(checkpoint_dir="nn_boston_model")

    # 执行恢复
    saver.restore(sess=sess, save_path=model_file)

    # 此处得到的是图结构
    graph = tf.get_default_graph()

    # get placeholder from graph  拿到两个需要输入的数据节点，预测时输入
    inputX = graph.get_tensor_by_name("inputX:0")
    keep_prob_s = graph.get_tensor_by_name("keep_prob:0")

    # get operation from graph   拿到能输出结果的数据节点，预测时执行到这里，拿到预测结果值
    prediction = graph.get_tensor_by_name("pred:0")

    # 开始预测, 还是不用输入y , 丢弃我一直写的1，不丢弃
    feed_dict = {inputX: X, keep_prob_s: 1}
    y_pred = sess.run(prediction, feed_dict=feed_dict)

    # 得到预测结果
    print(y_pred)
```













目前所有知道的东西都在上面了。。整个实现还是面向过程的，有待升级。。
