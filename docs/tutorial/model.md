
# Model

To define a model (i.e. the computation graph) that will be used for training,
you'll need to subclass `ModelDesc` and implement several methods:

```python
class MyModel(ModelDesc):
	def _get_inputs(self):
		return [InputDesc(...), InputDesc(...)]

	def _build_graph(self, inputs):
		# build the graph

	def _get_optimizer(self):
	  return tf.train.GradientDescentOptimizer(0.1)
```

Basically, `_get_inputs` should define the metainfo of all the possible placeholders your graph may need.
`_build_graph` should add tensors/operations to the graph, where
the argument `input_tensors` is the list of input tensors matching `_get_inputs`.

You can use any symbolic functions in `_build_graph`, including TensorFlow core library
functions, TensorFlow slim layers, or functions in other packages such as tflean, tensorlayer.

tensorpack also contains a small collection of common model primitives,
such as conv/deconv, fc, batch normalization, pooling layers, and some custom loss functions.
Using the tensorpack implementations, you can also benefit from `argscope` and `LinearWrap` to
simplify the code.

## argscope and LinearWrap
`argscope` gives you a context with default arguments.
`LinearWrap` allows you to simplify "linear structure" models by
adding the layers one by one.

The following code:
```python
with argscope(Conv2D, out_channel=32, kernel_shape=3, nl=tf.nn.relu):
	l = (LinearWrap(image)  # the starting brace is only for line-breaking
			 .Conv2D('conv0')
			 .MaxPooling('pool0', 2)
			 .Conv2D('conv1', padding='SAME')
			 .Conv2D('conv2', kernel_shape=5)
			 .FullyConnected('fc0', 512, nl=tf.nn.relu)
			 .Dropout('dropout', 0.5)
			 .tf.multiply(0.5)
			 .apply(func, *args, **kwargs)
			 .FullyConnected('fc1', out_dim=10, nl=tf.identity)())
```
is equivalent to:
```
l = Conv2D('conv0', image, 32, 3, nl=tf.nn.relu)
l = MaxPooling('pool0', l, 2)
l = Conv2D('conv1', l, 32, 3, padding='SAME', nl=tf.nn.relu)
l = Conv2D('conv2', l, 32, 5, nl=tf.nn.relu)
l = FullyConnected('fc0', l, 512, nl=tf.nn.relu)
l = Dropout('dropout', l, 0.5)
l = tf.multiply(l, 0.5)
l = func(l, *args, **kwargs)
l = FullyConnected('fc1', l, 10, nl=tf.identity)
```
