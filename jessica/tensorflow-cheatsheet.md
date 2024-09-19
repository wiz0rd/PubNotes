# TensorFlow Cheat Sheet

## Importing TensorFlow
```python
import tensorflow as tf
```

## Creating Tensors
```python
# From NumPy array
import numpy as np
np_array = np.array([1, 2, 3])
tensor = tf.convert_to_tensor(np_array)

# Directly
tensor = tf.constant([1, 2, 3])

# Zeros and Ones
zeros = tf.zeros([3, 3])
ones = tf.ones([3, 3])

# Random
random = tf.random.normal([3, 3])
```

## Basic Operations
```python
a = tf.constant([1, 2, 3])
b = tf.constant([4, 5, 6])

# Addition
c = tf.add(a, b)  # or a + b

# Multiplication
d = tf.multiply(a, b)  # or a * b

# Matrix multiplication
e = tf.matmul(a, b)
```

## Variables
```python
var = tf.Variable([1, 2, 3])
var.assign([4, 5, 6])
```

## Gradients
```python
with tf.GradientTape() as tape:
    y = 2 * x + 3
gradients = tape.gradient(y, x)
```

## Building Models
```python
model = tf.keras.Sequential([
    tf.keras.layers.Dense(64, activation='relu'),
    tf.keras.layers.Dense(10, activation='softmax')
])
```

## Training
```python
model.compile(optimizer='adam',
              loss='sparse_categorical_crossentropy',
              metrics=['accuracy'])

model.fit(x_train, y_train, epochs=5)
```

## Saving and Loading Models
```python
# Save
model.save('model.h5')

# Load
loaded_model = tf.keras.models.load_model('model.h5')
```

For more detailed information, refer to the [official TensorFlow documentation](https://www.tensorflow.org/api_docs/python/tf).
