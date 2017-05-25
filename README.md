# BehavioralCloning
Tutorial for building a model that generates steering angle based on image input. A brief summary of the workflow. 
## 1. Getting around the repo
* Model_Preprocessing.ipynb is the scratchpad that was used to try and experiment with building the model.It helps us extract and preprocess information and combine it. 
* Model.py is the Keras Model that contains 
    * Model 
    * Image Aug Generators
* It obtains data from complete_data.csv which contains the sum total of all images , steering angles and their paths. 
* The data is obatined by running the simulator / stock data given by udacity itself. 
### Data/File Structure 
* The data is of the format ** Drive_Log.csv** which contains the path information where the images are sampled from the video and the actual images are stored in **IMG**. Each image is timestamped. 
## 2. Workflow 
* Simulator - Generate Data in Training Mode
* Analyze, Augment and PreProcess Data offline
* Get more Data if required.
* Upload Data to Cloud - AWS Machine Learning system . 
* Run the Model Training. 
* Download Model and run simulator in autonomous mode. 
* Repeat process 
## Summary of Steps
### Exploratory Visualization of Dataset
* Pandas + SeaBorn + MatPlotLib to create , load and append dataset from dataframe
* Visualization to understand the distribution and quality of data. 
* Distribution Plot to see the spread and quantity of data
* Time Series Plot to understand the quality of data. (To see noise to determine if filters are required)
<p align="center">
<img src= "Track1_SteeringAngle_Dist.png" width="750"/>
</p>

<p align="center">
<img src= "Track1vsTrack2.png" width="750"/>
</p>

### Data Collection based on Shortcomings
- Visualizing collected data from driving the simulator shows that the dataset looks entirely different for **Keyboard** and **Mouse**
- So we pass the data through a **Savitzky Golay** filter that averages out the samples but maintains the 
- Area = (Steering_angle x Time) this effectively filters out the noise without destroying the signal.
- Based on the histogram distribution plots collecting data by using certain driving styles.
  - Data from **Track 1** - Clock wise and anticlockwise (**MAC & Windows**)
  - Data from **Track 2** - Clock wise and anticlockwise (MAC & Windows)
  - Smooth Turn from both Tracks (MAC & Windows)
  - Recovery driving from both Tracks (MAC & Windows)
  - **Problem Areas** in both tracks (MAC & Windows)
  - Keyboard and Mouse
- After initial model save and testing driving and training in problem areas to improve model on subset of data.

Name           | Values
---------------|--------------------------------------------------------
Data Sources   | Mac, Windows, Linux Sim
Input Sources  | Keyboard, Touch Pad, Mouse, Joystick
Tracks         | Track-1, Track-2
Direction      | Clockwise and Anti-Clockwise
Special Cases  | Recovery Driving, Smooth Turns only, Stock Udacity Data 

<p align="center">
<img src= "Udacity_StockData.png" width="750"/>
</p>

<p align="center">
<img src= "Cw_vsACW.png" width="750"/>
</p>


### Data Augmentation
* Augmentation using **Flipping**, **Translation** from left and right camera images
* Reduce the time spent on data gathering through data augmentation techniques 

### Data Perturbation to Increase Model Robustness

- **Brightness Perturbation** : Random perturbation of brightness of the image.
- **Gaussian Noise** : Blur filter with a random normal distribution across the image.
- **Adaptive Histogram Equalization** : Can greatly help in the model learning the features quickly
- **Colospace inversion** :  RBG to BGR colorspace change 

### Sampling and Image Generators 
These steps increase the challenge and generalization capability by creating harder images for the model to train on. Below is an example of augmented and perturbed image batch that is linked with the image generator that generates images during model training on the go. Seen below are the distribution of data and images of one random sample generated by the generator.

- Since the distribution is clearly tri modal (peaks around 0 for Center Camera , + 0.25 and -0.25 for left and right cameras respectievely ) it is an unbalanced dataset.
- Although significant efforts have been taken gather more data around turns , there is just simply more data around 0 and +/-0.25
- Best possible option is to do Balancing through **DownSampling**
- The method used to do downsampling is **Weighted Random Normal Sampling** .
- Why we choose this is because , the dominant characteristic of the system is to stay around 0/.25 so we make sure we don't mess with that.
- The steering angles are **Discretized** i.e made to fall under categories/ Bins
- Counts are taken for each group and the weights are given as 1/ Counts in that group.
- These weights are then **Normalized** - When summed up they need to be equal to 1
- Then the batch size is used to sample this out of the data frame using the Sampling with weights 


<p align="center">
<img src= "Sample_Distribution.png" width="750"/>
</p>

<p align="center">
<img src= "Sample_PreProcessed_Image_Batch.png" width="1500"/>
</p>

### Define model architecture
#### Data Pre-processing steps
 * Normalization through feature scaling 
 * Cropping region of interest
 * Resize image to increase model performance
  
#### Salient Features of Model
 * Batch Normalization before every activation
 * Overfitting prevention Dropouts and batch norm
 * Dropouts are implemented before the flatten layer and before the output layer with a 50% probability
 * Tried by adding multiple dropouts but it did need seem to have an effect on improving validation losses.
 * Switched from ReLU to ELU for activations after reading this paper-https://arxiv.org/abs/1511.07289
 * NVIDIA End to End Model architecture and train from scratch
 
Layer Name                   |  Size                  | Number of Parameters
-----------------------------|------------------------|--------------------
cropping2d_1 (Cropping2D)    |  (None, 90, 320, 3)    |    0         
lambda_1 (Lambda)            |  (None, 66, 200, 3)    |    0         
lambda_2 (Lambda)            |  (None, 66, 200, 3)    |    0         
conv2d_1 (Conv2D)            |  (None, 31, 98, 24)    |    1824      
batch_normalization_1 (Batch |  (None, 31, 98, 24)    |    96        
elu_1 (ELU)                  |  (None, 31, 98, 24)    |    0         
conv2d_2 (Conv2D)            |  (None, 14, 47, 36)    |    21636     
batch_normalization_2 (Batch |  (None, 14, 47, 36)    |    144       
elu_2 (ELU)                  |  (None, 14, 47, 36)    |    0         
conv2d_3 (Conv2D)            |  (None, 5, 22, 48)     |    43248     
batch_normalization_3 (Batch |  (None, 5, 22, 48)     |    192       
elu_3 (ELU)                  |  (None, 5, 22, 48)     |    0         
conv2d_4 (Conv2D)            |  (None, 3, 20, 64)     |    27712     
batch_normalization_4 (Batch |  (None, 3, 20, 64)     |    256       
elu_4 (ELU)                  |  (None, 3, 20, 64)     |    0         
conv2d_5 (Conv2D)            |  (None, 1, 18, 64)     |    36928     
batch_normalization_5 (Batch |  (None, 1, 18, 64)     |    256       
elu_5 (ELU)                  |  (None, 1, 18, 64)     |    0         
flatten_1 (Flatten)          |  (None, 1152)          |    0         
dropout_1 (Dropout)          |  (None, 1152)          |    0         
dense_1 (Dense)              |  (None, 1164)          |    1342092   
batch_normalization_6 (Batch |  (None, 1164)          |    4656      
elu_6 (ELU)                  |  (None, 1164)          |    0         
dense_2 (Dense)              |  (None, 100)           |    116500    
batch_normalization_7 (Batch |  (None, 100)           |    400       
elu_7 (ELU)                  |  (None, 100)           |    0         
dense_3 (Dense)              |  (None, 50)            |    5050      
batch_normalization_8 (Batch |  (None, 50)            |    200       
elu_8 (ELU)                  |  (None, 50)            |    0         
dense_4 (Dense)              |  (None, 10)            |    510       
batch_normalization_9 (Batch |  (None, 10)            |    40        
elu_9 (ELU)                  |  (None, 10)            |    0         
dropout_2 (Dropout)          |  (None, 10)            |    0         
dense_5 (Dense)              |  (None, 1)             |    11        

<p align="center">
<img src= "EndToEnd_NVIDIA.png" width="1500"/>
</p>

### Setup Model Training Pipeline
- **Hyperparameters**: **Epochs** , **Steps per Epoch** and **Learning Rate** decided based on search epochs on subset of data
- **Greedy best save** and **checkpoint** implementation.
- **Metrics** is a purely **loss** based. Since the label(Steering angle) here is numeric and non-categorical , RMS Loss is used as the loss type. 

Hyperparameter Name  | Value                   |     Comments     
---------------------|-------------------------|--------------------------------------------
Epochs               | 10                      | Additional Epochs for special problem areas
Learning Rate        | 1e-4                    | Default Learning rate of 1e-2 unsuitable results
Batch Size           | 32                      | Chosen due to best trade off between CPU & GPU performance
Metric               | Loss                    | Accuracy is unsuitable as exact steering angle prediction is not what matters
Loss Type            | Root Mean Squared Error | As loss is non-categorical closeness to predicted angle is what matters
Optimizer Type       | Adam                    | Chosen from http://sebastianruder.com/optimizing-gradient-descent/index.html


### Save and Deploy Model
* Save using **json**, **hdf5** model.

<p align="center">
<img src= "run1.mp4" width="1500"/>
</p>
