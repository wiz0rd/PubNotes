# TensorFlow Interview Quiz - Answer Key

1. TensorFlow is an open-source machine learning framework developed by Google. Its primary use is for developing and training machine learning models, particularly deep learning models.

2. You can import TensorFlow using:
   ```python
   import tensorflow as tf
   ```

3. A Tensor is the fundamental unit of data in TensorFlow. It's an n-dimensional array of data.

4. You can create a constant Tensor using:
   ```python
   tf.constant([1, 2, 3])
   ```

5. `tf.Variable` is mutable and typically used for model parameters that need to be updated during training. `tf.constant` is immutable and used for values that don't change.

6. Matrix multiplication can be performed using:
   ```python
   tf.matmul(matrix1, matrix2)
   ```

7. `GradientTape` is used for automatic differentiation. It records operations for later computing the gradient. Example:
   ```python
   with tf.GradientTape() as tape:
       y = 2 * x + 3
   gradients = tape.gradient(y, x)
   ```

8. A simple feedforward neural network can be created using:
   ```python
   model = tf.keras.Sequential([
       tf.keras.layers.Dense(64, activation='relu'),
       tf.keras.layers.Dense(10, activation='softmax')
   ])
   ```

9. The `compile` method configures the model for training. It specifies the optimizer, loss function, and metrics to track.

10. You can train a model using the `fit` method:
    ```python
    model.fit(x_train, y_train, epochs=5)
    ```

11. Eager execution is the default mode where operations are evaluated immediately. Graph execution builds a computational graph before running, which can be more efficient for complex models.

12. You can save and load models using:
    ```python
    # Save
    model.save('model.h5')
    # Load
    loaded_model = tf.keras.models.load_model('model.h5')
    ```

13. TensorFlow Hub is a library for reusable machine learning modules. It's useful for transfer learning and quickly incorporating pre-trained models into your own.

14. TensorFlow automatically uses GPUs when available. You can control GPU usage with:
    ```python
    tf.config.list_physical_devices('GPU')
    tf.config.experimental.set_memory_growth(physical_devices[0], True)
    ```

15. The `@tf.function` decorator is used to compile a function into a callable TensorFlow graph, which can improve performance.

16. Overfitting can be handled through techniques like regularization, dropout, and early stopping. For example:
    ```python
    tf.keras.layers.Dense(64, activation='relu', kernel_regularizer=tf.keras.regularizers.l2(0.01))
    tf.keras.layers.Dropout(0.5)
    ```

17. `tf.keras` is TensorFlow's implementation of the Keras API, integrated directly into TensorFlow. Standalone Keras is the original multi-backend Keras library.

18. You can visualize your model's architecture using:
    ```python
    tf.keras.utils.plot_model(model, to_file='model.png')
    ```

19. TensorFlow Serving is a flexible, high-performance serving system for machine learning models, designed for production environments.

20. Custom layers can be implemented by subclassing `tf.keras.layers.Layer`:
    ```python
    class MyLayer(tf.keras.layers.Layer):
        def __init__(self, output_dim, **kwargs):
            self.output_dim = output_dim
            super(MyLayer, self).__init__(**kwargs)

        def build(self, input_shape):
            self.kernel = self.add_weight(name='kernel', 
                                          shape=(input_shape[1], self.output_dim),
                                          initializer='uniform',
                                          trainable=True)

        def call(self, inputs):
            return tf.matmul(inputs, self.kernel)
    ```
