
##Purpose
This library is a general purpose Artificial Neural Network implementation in MATLAB focused on research purposes. It is designed to have a large number of often experimental features that can easily be enabled/disabled without bogging the user down in details they aren't immediately concerned with.

Last Update: **11 Mar 16**
Version: **0.1.0** alpha release (stable, but not thoroughly tested)
*Requirements:* Matlab parallel processing toolkit
*Optional:* Matlab computer vision toolkit (some optional functions)

###Basic Usage:
Model parameters define all hyper parameters and settings the algorithm needs. They are stored in a struct and should be generated using `initialize_model`. Data sets are prepared in a similar data structure created using `initialize_data`. A trained model, complete with metrics, trained weights, etc. is generated by calling `train_ann` with a model or a cell array of many models, and a data set or cell array of many datasets.
```matlab
    model = initialize_model(...);
    data  = initialize_data(...);
    trained_model = train_ann(model, data);
	predictions   = predict_ann(trained_model, data);
```
`initialize_model` requires 1 parameter, everything else is optional and are defined using matlab's standard `('name',value)` parameterization. Parameters for `initialize_model` are documented in detail later in this guide.

`initialize_data` requires two parameters, the feature data in NxD format (N samples by D features), and label data in NxC (N samples by C output neurons). Cross validation feature and labels, and options to normalize the data are optional and documented later in this guide.

`train_ann` accepts one or more models and one or more data sets and provides one or more trained_models as output. If you provide multiple models or data sets to be trained (in parallel) provide them in a cell array (see example usage). Note that you may not provide multiple models AND multiple data sets, one of the two must be single.

This function returns the same model that was created with initialize_model along with a Metrics structure added to it and weights set. This is meant to allow for easy storage of the model and parameters that were used to generate the model.

`predict_ann` provides predictions based on a trained model. Note that if you provide a cross validation data set this will occur automatically and the Metrics produced will reflect the results, it is common not to use this function during experimentation.

###Example usage:
```matlab
% Initialize model parameters to a 3 layer network with 784 feature inputs, 
% a single 30 neuron hidden layer, and 10 output classes.
model = initialize_model( [784 30 10], ...
                          'cost_function', 'cross_entropy', ...
                          'regularization', 'L2' );

% Initialize a dataset of 1000 MNIST digits and 1000 cross validation digits
% This example assumes mnist_features and mnist_labels was already created
% by the user and contains a random set of features and labels.
data = initialize_data( mnist_features(1:1000,:), ...
                        mnist_labels(1:1000,:), ...
                        'cv_features', mnist_features(1001:2000,:), ...
                        'cv_labels',   mnist_labels(1001:2000,:), ...
                        'rescale_method', 'standardize' );

% Train the model using the dataset defined above
trained_model = train_ann(model, data);

% Plot the results
plot_n_runs_accuracy_cost( trained_model );
```    

###initialize_model parameter options
|Parameters|Value|Default|Description|
|----------|:-----|-------|:-----------|
|layer_sizes|[784&nbsp;30&nbsp;30&nbsp;10]|Required|Defines the network size, the example shown in the Value column creates a network of 784 inputs, two hidden layers of 30 neurons, and 10 output neurons.|
|'plot_title'|Text|None|The text used in the plot legend (and possibly elsewhere that a model name is needed).|
|'update_method'|'GD' or 'EG+-'|'GD'|The update method to use, Gradient Descent and EG+- are options (Exponentiated Gradient)|
|'U'|Numeric|40|The regularization term used by EG+-, unused if EG+- isn't chosen as the update method.|
|'learning_rate'|Numeric|depends on cost function|The 'Eta' learning rate, default value set dynamically depending on the cost function chosen. 3.0 for quadratic cost and 0.5 default for cross entropy.
|'num_epochs'|Numeric|30|Number of epochs of the training set to run through. Note that this parameter will change in future versions.|
|'mini_batch_size'|Numeric|100|Number of samples to compute in each mini batch. 1 equals stochastic, choosing a value equal to the # of epochs is equal to full batch gradient descent.|
|'cost_function'|'sum_of_squares', 'cross_entropy'|'sum_of_squares'|The cost function to use.|
|'initial_weights'|Cell array of weights|Auto initialized|The weights are initialized to random values based on the 'weight_init_method'. You can specify them manually.|
|'initial_biases'|Cell array of biases|Auto initialized|Bias units, works the same as initial_weights|
|'weight_init_method'|'1/sqrt(n)', 'gaussian-0-mean-1-std'|'1/sqrt(n)'|The method used to initialize the weights and biases. |
|'regularization'|'none', 'L1', 'L2'|'L2'|The regularization method to use. Note this is only used by Gradient Descent updates, not EG+-.|
|'lambda'|Numeric|1.0e-4|Degree of regularization to apply. Note that this is a fixed percent value, some texts, notably NeuralNetworksAndDeepLearning.com suggest to use lambda/n, but this causes an unnecessary dependence on the size of the input dataset.|
|'monitor_training_cost'|true/false|true|Enables generation of training data accuracy and cost function data Metrics.|
|'monitor_accuracy'|true/false|true|Enables generation of cross validation accuracy metrics per epoch of training.|
|'verbosity'|'none', 'epoch', 'debug'|'epoch'|The amount of console output to write during single training runs.|
|'rng_seed'|Numeric, 'shuffle'|'shuffle'|A random seed to generate deterministic results.|
|'noise_function'|@function_handle, 'none'|'none'|A function that will take feature input and transform it. The noise function must take an MxD matrix of features and return a transformed MxD matrix of features. It will be called on each epoch to transform the data before processing.|

###initialize_data parameter options
|Parameters|Value|Default|Description|
|----------|:-----|-------|:-----------|
|feature_data|MxD matrix|Required|An MxD matrix of feature data where M is the number of training samples and D is the number of features.|
|labels|MxC matrix|Required|The labels for the feature_data in N samples by C output neurons format|
|'cv_features'|MxD matrix|N/A|A matrix of feature data to use as an unseen cross validation dataset (not necessarily the same size as the feature data). Same format as feature_data.|
|'cv_labels'|MxC matrix|N/A|If cv_features was specified this is required, same format as labels|
|'rescale_method'|'none', 'standardize', 'rescale'|'none'|The method used to normalize the input features. Note that if you do not normalize the input features but have a high variance in the inputs you will receive a warning.|

###Plotting Results
Plot functions assume a cell array of 1 or more trained models. Plot functions available:
`plot_n_run_accuracy_cost( trained_models )` Plots training accuracy, cross validation accuracy, and traing cost for each trained model in the cell array passed to it.

Note that figure legends will use the model parameter 'plot_title' from each model.

|Parameters|Value|Default|Description|
|---------|:---------|-----------|
|trained_models|Result from 'train_ann'|Required|A cell array of trained models, this is the default returned value from 'train_ann'|
|'plot_training_accuracy'|true/false|true|Plot the accuracy on the training data as a function of the number of epochs|
|'plot_cv_accuracy'|true/false|true|Plot the accuracy on the unseen cross validation data as a function of the number of epochs on the same plot|
|'plot_training_cost'|true/false|true|Plot the training cost generated aggregated by training epoch|

**Example usage:**
```matlab
% Plot the Training and Cross validation accuracy only.
plot_n_run_accuracy_cost( trained_models, 'plot_cost_function', false );
```

###Metrics
If metrics generation wasn't disabled in the model the trained model will contain a new struct called `Metrics`. It will contain the following metrics:
 - `training_cost` per-mini batch cost function results
 - `training_accuracy` per-epoch accuracy on the training data
 - `cv_accuracy` per-epoch accuracy on the optional cross validation data

###Planned future enhancements:
 - Built in auto encoder
 - Callback capabilities to manipulate the mechanics of the network each mini batch
 - Also will be used for dynamically generating plots and interactively altering parameters such as regularization during network training for real-time analysis during training.
 - Dropout regularization
 - Support for running on the GPU (implemented but untested/nonfunctional)
 - Regression

###Known issues
 - `predict_ann` is not currently implemented, for experimental usage it is not necessary if you define a CV set in your data. When you do this your cross validation data will be predicted as part of the metrics generation and everything you need to report results is available and prepared for you.
 - Currently only classification is supported.

###Licensing and Contact
This package is maintained by David Parks, UCSC  --  Contact: dfparks@ucsc.edu

The MIT License (MIT)

Copyright (c) [2016] [David Freeman Parks - dfparks@ucsc.edu]

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all
copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
SOFTWARE.




