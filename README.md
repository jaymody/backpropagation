# Backpropagation and Gradient Descent
### Author: Jay Mody
---
This repo is a workspace for me to develop my own gradient descent algorithims implemented in python from scratch using only numpy.

## Main NN Code
```python
#### Imports ####
import numpy as np

#### Neural Network Class ####
class MLP:
    ##### Constructor ####
    def __init__(self, n_input_nodes, hidden_nodes, n_output_nodes, lr):
        ## Network ##
        self.n_input_nodes = n_input_nodes
        self.n_output_nodes = n_output_nodes
        
        self.nodes = hidden_nodes
        self.nodes.insert(0, n_input_nodes)
        self.nodes.append(n_output_nodes)
        
        ## Weights and Biases##
        self.weights = []
        self.biases = []
        for i in range(1, len(self.nodes)):
            self.weights.append(np.random.uniform(-1.0, 1.0, (self.nodes[i-1], self.nodes[i])))
            self.biases.append(np.random.uniform(-1.0, 1.0, (1, self.nodes[i])))
        
        ## Learning Rate ##
        self.lr = lr
        
        ## Activation Functions ##
        # Linear Activation
        self.linear = lambda x: x
        self.d_linear = lambda x: np.ones(x.shape)
        
        # Relu Activation
        def relu(x):
            x[x<0] = 0
            return x
        def d_relu(out):
            out: x[x>0] = 1
            return out
        self.relu = relu
        self.d_relu = d_relu
            
        # Sigmoid Activation
        self.sigmoid = lambda x: 1 / (1 + np.exp(-x))
        self.d_sigmoid = lambda out: out * (1 - out)  # assumes out is tanh(x)
        
        # Hyperbolic Tangent Activation
        self.tanh = lambda x: np.tanh(x)
        self.d_tanh = lambda out: 1 - out**2 # assumes out is tanh(x)
        
    def getWeights(self):
        return self.weights.copy()
    def getBiases(self):
        return self.biases.copy()
    
    def setWeights(self, weights):
        self.weights = weights.copy()
    def setBiases(self, biases):
        self.biases = biases.copy()
    
    #### Feed Forward ####
    def feed_forward(self, X):
        outputs = [X]
        
        logits = np.dot(X, self.weights[0]) + self.biases[0]
        
        for i in range(1, len(self.nodes) - 1):
            out = self.sigmoid(logits)
            outputs.append(out)
            logits = np.dot(out, self.weights[i]) + self.biases[i]
        
        out = self.sigmoid(logits)
        outputs.append(out)
        
        return outputs
    
    #### Backpropagation ####
    def backpropagation(self, X, y, outputs):
        weights_gradients = []
        biases_gradients = []
        
        d1 = y - outputs[-1]
        d2 = self.d_sigmoid(outputs[-1])
        error = d1 * d2
        
        grad = outputs[-2].T * error 
        weights_gradients.append(grad)
        biases_gradients.append(error)
        
        for i in range(len(self.weights) - 2, 1, -1):
            d = self.d_sigmoid(outputs[i])
            error = np.dot(error, self.weights[i+1].T) * d
            
            grad = outputs[i-1].T * error 
            weights_gradients.append(grad)
            biases_gradients.append(error)
        
        return weights_gradients, biases_gradients
    
    #### Training ####
    def train(self, features, targets):
        # Batch Size for weight update step
        batch_size = features.shape[0]
        
        # Delta Weights Variables
        delta_weights = [np.zeros(weight.shape) for weight in self.weights]
        delta_biases = [np.zeros(bias.shape) for bias in self.biases]
        
        # For every data point, forward pass, backpropogation, store weights change
        for X, y in zip(features, targets):
            # Forward pass
            X = X.reshape(1, X.shape[0])
            outputs = self.feed_forward(X)
            
            # Back propogation
            weights_gradients, biases_gradients = self.backpropagation(X, y, outputs)
            
            for i in range(len(weights_gradients)):
                delta_weights[-(i+1)] += weights_gradients[i]
                delta_biases[-(i+1)] += biases_gradients[i]
        
        for i in range(len(delta_weights)):
            self.weights[i] += (self.lr * delta_weights[i]) / batch_size
            self.biases[i] += (self.lr * delta_biases[i]) / batch_size

    #### Testing Methods ####
    def predict(self, X):
        # Gives prediction
        return self.feed_forward(X)[-1]
    
    def test(self, features, targets):
        predictions = self.predict(features)

        n_correct = 0
        for i in range(len(predictions)):
            prediction = np.argmax(predictions[i])
            correct = np.argmax(targets[i])

            if prediction == correct:
                n_correct += 1

        return n_correct / len(targets)
 ```


**Limitations and Features:**
So far, i've been able to succesfully claissify clusters of data (multiple classes) generated using make_blobs from SKLearn. It doesn't work everytime, with my testing I found here are a few reasons:

- Loss will increase possibly due to overshooting of gradients
- Sometimes the network will get stuck in a local minumum
- Since weights are initialized randomly, some initial configurations fail despite many epochs of training (no learning, local minimum)
- Activations hugely affected my results
- The network suffers from vanishing gradient


As such, only about 70% of the time I find that the network was able to 'learn'. Although, I'm happy with the results, as this was more an excersize for me to understand backprop and gradient descent than actually implementing it for use. Here's an example:



![before](/imgs/before.png)


![training](/imgs/loss_acc.png)


![after](/imgs/after.png)



**Completed:**
- Created dataset
- Visualize data
- Implemented MLP class
- Feed Forward
- Backpropagation
- Training algorithim and update weights using GD with backprop
- Implemented SGD
- Implemented Mini-Batch GD
- Visualize outputs
- Test and Predict
- Generalize backprop and ff for any number of layers


**To Do:**
- Regularization
- Momentum
- Gradient Clipping
- Choose activations per layer

TESTING COMMIT
