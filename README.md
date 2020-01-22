# Piecewise Polynomial Layers for Tensorflow
Tensorflow layers using piecewise Chebyshev polynomials.  Long ago I wrote a c++ code that explored higher 
order weights in the synapse of of a standard neural network [here](https://www.researchgate.net/publication/276923198_Discontinuous_Piecewise_Polynomial_Neural_Networks) .  This is an effort to reproduce that work in Tensorflow.

## Idea

The idea is extremely simple - instead of a single weight at the synapse, use n-weights.  The n-weights describe a piecewise polynomial and each of the n-weights can be updated independently.  A Chebyshev polynomial and gauss lobatto points are used to minimize oscillations of the polynomial.

## Why

Using higher order polynomial representations might allow networks with much fewer total weights. In physics, higher order methods
can be much more efficient, (while being more complex to implement). Spectral and discontinuous galerkin methods are examples of this.  Note that a standard neural network with relu activations is piecewise linear.  Here there are no bias weights and the "non-linearity" is in the synapse. 

# Installation

```bash
pip install snovalleyai-piecewise-polynomial-layers
```

# Use

```python
import tensorflow as tf
import snovalleyai_piecewise_polynomial_layers.PolynomialLayers as poly
mnist = tf.keras.datasets.mnist

(x_train, y_train),(x_test, y_test) = mnist.load_data()
x_train, x_test = (x_train / 128.0-1.0), (x_test / 128.0-1.0)

units = 20

basis = poly.b3

model = tf.keras.models.Sequential([
  tf.keras.layers.Flatten(input_shape=(28, 28)),
  poly.Polynomial(units, 28*28, basis=basis),
  tf.keras.layers.LayerNormalization(),
  poly.Polynomial(units, units, basis=basis),
  tf.keras.layers.LayerNormalization(),
  poly.Polynomial(units, units, basis=basis),
  tf.keras.layers.LayerNormalization(),
  poly.Polynomial(units, units, basis=basis),
  tf.keras.layers.LayerNormalization(),
  tf.keras.layers.Dense(10, activation='softmax')
])

model.compile(optimizer='adam',
              loss='sparse_categorical_crossentropy',
              metrics=['accuracy'])

model.fit(x_train, y_train, epochs=20, batch_size=10)
model.evaluate(x_test, y_test)
```

# Example - Simple Polynomial

Here is the result for fitting a sin wave with no hidden layers.  There is only one input neuron and one output neuron and no neuronal non-linearity.  A 5th order polynomial is used in the synapse - there are 6 weights and only one synapse in the network.

![](polynomialSynapse.png)

# Example 2 - Piecewise Discontinuous Polynomial (2 pieces)

Same problem, but comparison between 1st 3rd and 5th order piecewise discontinuous polynomial synapses.  We all know gradient descent doesn't work with discontinuous functions so this doesn't work... Except that these are actually non-convex optimizers so it still works.  You'll find it also works in multi-layer networks.  Whether it's better or not remains to be seen - however, for problems with actual discontinuities, like fluid dynamics, it might be better.

![](sin5d.png)

# Example 3 - Piecewise Continuous Polynomial (2 pieces)

Same problem, but comparison between 1st 3rd and 5th order piecewise continuous polynomial synapses.

![](sin5c.png)

# Available polynomial orders

```python
import snovalleyai_piecewise_polynomial_layers.PolynomialLayers as poly

#Non piecewise polynomials
poly.b1 #linear chebyshev
poly.b2 #quadratic chebyshev
poly.b3 #3rd order chebyshev
boly.b4 #4th order chebyshev
poly.b5 #5th order chebyshev

## Discontinous piecewise polynomials, 2 pieces
poly.b1D #linear chebyshev (discontinuous pair)
poly.b2D #quadratic chebyshev (discontinuous pair)
poly.b5D #5th order chebyshev (discontinuous pair)

## Continuous piecewise polynomials, 2 pieces
poly.b1C #linear chebyshev (continuous pair)
poly.b2C #quadratic chebyshev (continuous pair)
poly.b5C #5th order chebyshev (continuous pair)
```
The layer inside tensorflow is then called (see mnist example above)
```
poly.Polynomial(units, input, basis=basis),
```
where units is the number of units and input is the size of the input and basis would be 'poly.b3' for example.
