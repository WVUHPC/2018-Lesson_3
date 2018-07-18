---
title: "Machine Learning: (scikit-learn, Keras, and TensorFlow)"
teaching: 45
exercises: 15
questions:
- "What is Machine Learning and all the fuzz about it?"
objectives:
- "Get a first hand experience working with Keras and TensorFlow"
keypoints:
- "Neural Networks and Deep Learning are active topics of development nowadays, TensorFlow and Keras are just two popular tools on that area."
---

## One Example

~~~
import tensorflow as tf
sess = tf.Session(config=tf.ConfigProto(log_device_placement=True))

x=tf.Variable(3, name='x')
y=tf.Variable(4, name='y')
f = x*x*y + y + 2

init = tf.global_variables_initializer()

sess = tf.InteractiveSession()
init.run()

result = f.eval()
print(result)
sess.close()
~~~
{: .source}

~~~
import numpy as np
from sklearn.datasets import fetch_california_housing
housing = fetch_california_housing()
~~~
{: .source}

~~~
m,n = housing.data.shape
housing_data_plus_bias = np.c_[np.ones((m,1)), housing.data]
housing.data
X = tf.constant(housing_data_plus_bias, dtype=tf.float32, name='X')
y = tf.constant(housing.target.reshape(-1,1),
dtype=tf.float32, name='y')
~~~
{: .source}

~~~
XT=tf.transpose(X)
theta = tf.matmul(tf.matmul(tf.matrix_inverse(tf.matmul(XT,X)), XT),y)
~~~
{: .source}

~~~
sess = tf.InteractiveSession()
theta_value = theta.eval()
theta_value
~~~
{: .source}

## MNIST

~~~
from tensorflow.examples.tutorials.mnist import input_data
mnist = input_data.read_data_sets("MNIST_data/", one_hot=True)
~~~
{: .source}

~~~
x = tf.placeholder(tf.float32, [None, 784])
W = tf.Variable(tf.zeros([784, 10]))
b = tf.Variable(tf.zeros([10]))
y = tf.nn.softmax(tf.matmul(x, W) + b)
y_ = tf.placeholder(tf.float32, [None, 10])
~~~
{: .source}

~~~
cross_entropy = tf.reduce_mean(-tf.reduce_sum(y_ * tf.log(y), reduction_indices=[1]))
train_step = tf.train.GradientDescentOptimizer(0.5).minimize(cross_entropy)
sess = tf.InteractiveSession()
tf.global_variables_initializer().run()
~~~
{: .source}

~~~
for _ in range(1000):
  batch_xs, batch_ys = mnist.train.next_batch(100)
  sess.run(train_step, feed_dict={x: batch_xs, y_: batch_ys})
~~~
{: .source}

~~~
correct_prediction = tf.equal(tf.argmax(y,1), tf.argmax(y_,1))
accuracy = tf.reduce_mean(tf.cast(correct_prediction, tf.float32))
~~~
{: .source}

~~~
print(sess.run(accuracy, feed_dict={x: mnist.test.images, y_: mnist.test.labels}))
~~~


{% include links.md %}
