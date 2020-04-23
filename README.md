# T.E.D.D. 1104
A supervised deep neural network that learns to drive in video games. The main objective of this project is to 
achieve a model that can drive in Grand Theft Auto V. The model is not expected to drive following traffic 
regulations, but imitate how humans drive in this game: Drive at full speed through the city avoiding other 
cars and occasionally humans and lampposts.
A marker will be set in the game map, the model should be able to arrive to the marker driving trough the city. 

The model is trained using human labeled data. We record the game and key inputs of humans while the play the game, this data
is used to train the model. 

While we focus on self-driving cars and the video game Grand Theft Auto V this model can be adapted to play any existing
video game. 
    <table>
    <tr>
    <td> <img src="github_images/demo.gif" alt="gotta go fast!"/> </td>
    <td> <img src="github_images/demo2.gif" alt="gotta go fast2!"/> </td>
    </tr>
    </table>

## News
**NEW 15/04/2020** First pretrained model released!!, [click here to download it](https://github.com/ikergarcia1996/Self-Driving-Car-in-Video-Games/releases/tag/0.2). For instructions on how to run the AI see the [Run the Model](#run-the-model) Section  
**NEW 7/04/2020** Let's generate a huge training dataset together!! [Click here so see how to collaborate in the project (Spanish)](https://youtu.be/utQoMGLbCFc)  

## Architecture
Most previous models that attempt to achieve self-driving in video games consists of a deep convolutional neural network 
(usually Inception or Resnet). The models take as input a single image.
 Would you be able to know what to do if I give you this image?
 
<p align="center">
  <img src="github_images/choices.png" alt="Make a choice"/>
</p>

You may think that the best choice is to brake to avoid the blue/gray car, but, 
what if both cars are stopped waiting for you to cross the street? What if your car is 
currently driving in reverse? Does your current speed and the speed of the other cars allow you to 
cross the road without hitting them? A single image does not provide enough information to successfully 
achieve a self-driving car. More information is needed, that is why our approach uses sequences of images. 
5 images are captured with an interval of 1/10 seconds between them, this approach gives the model information 
about the motion of other cars, environment and himself. 


<p align="center">
  <img src="github_images/sequence.png" alt="Sequences 4 the win"/>
</p>

T.E.D.D. 1104 consists of a Deep Convolutional Neural Network (Resnet: K He et al. 2016) followed by 
a Recurrent Neural Network (LSTM). The CNN receives as input a sequence of 5 images and generates for each one a 
vector representation. These representations are fed into the RNN that generates a unique vector representation 
for the entire sequence. Finally, a Feed-Forward Neural Network outputs the key to press in the keyboard based 
on the vector representation for the sequence.

<p align="center">
  <img src="github_images/network_architecture.png" alt="The brain!"/>
</p>

The model has been implemented using Pytorch: https://pytorch.org/

## Software and HOW-TO
This repository contains all the files need for generating the training data, training the model and use the model to 
drive in the video game. The software has been written in Python 3. This model has only been tested in Windows 10 because
is the only supported SO by most video games.  

### Requirements
```
Pytorch
Torchvision
tensorboard (only for training a model)
Nvidia Apex (only for FP16 training)
numpy
cupy (optional but highly recommended especially for training the model, 10x speed up in data preprocessing comparated with numpy)
cv2 (opencv-python)
glob
h5py
json 
win32api (PythonWin) - Should be installed by default in newest Python versions (Python 3.7 reccomended)
```

### Run the model Generate dataset 
* File: generate_data.py
* Usage example: 
```
python generate_data.py --save_dir tedd1007\training_data
```
* How-to:
  * Set your game in windowed mode
  * Set your game to 1600x900 resolution
  * Move the game window to the top left corner, there should be a blue line of 1 pixel in the left bezel of your
         screen and the window top bar should start in the top bezel of your screen.
  * Play the game! The program will capture your screen and generate the training examples. There will be saved
         as files named "training_dataX.npz" (numpy compressed array). Don't worry if you re-launch this script,
          the program will search for already existing dataset files in the directory and it won't overwrite them.
  * At any moment push Q + E to stop the program.
  
<p align="center">
  <img src="github_images/example_config.png" alt="Setup Example"/>
</p>
  


### Train the model 
* File: train.py
* Usage example: 
```
python train.py --train_new 
--train_dir tedd1007\training_data\train 
--dev_dir tedd1007\training_data\dev 
--test_dir tedd1007\training_data\test 
--output_dir tedd1007\models 
--batch_size 10 
--num_epochs 5 
--fp16
```
* How-to:
  Train a model using the default hyper parameters, to see a description of the network hyper parameters use 
  "python train.py -h" or check the "train.py" and "model.py" files. train, dev and test directories should contain
   as many files named "training_dataX.npz" as you want. The FP16 flag allows you to use Mixed Precision Training if
   you have a modern Nvidia GPU with Tensor cores (RTX 2000, RTX Titan, Titan V, Tesla V100...), 
   it uses the Nvidia Apex library: https://github.com/NVIDIA/apex.
   The model is VERY memory demanding, as a
   reference I use a batch size of 15 for a RTX 2080 (8GB VRAM) for FP16 training (half the Vram usage than FP32 training) 
   using the default parameters. 
   
 * If you want to continue training from a checkpoint use (Note: The checkpoint will automatically use the same 
 floating point precision (FP16 or FP32) used for training when it was created):
   
 ```
python train.py --continue_training
--train_dir tedd1007\training_data\train 
--dev_dir tedd1007\training_data\dev 
--test_dir tedd1007\training_data\test 
--output_dir tedd1007\models 
--batch_size 10 
--num_epochs 5 
--checkpoint_path tedd1007\checkpoint\epoch1checkpoint.pt
```
   
### Run the model
* File: run_TEDD1104.py
* Pretrained-Models: [See the releases sections](https://github.com/ikergarcia1996/Self-Driving-Car-in-Video-Games/releases/)
* Usage example: 
```
python run_TEDD1104.py --model_dir D:\GTAV-AI\models --show_current_control --fp16
```
Use the FP16 flag if you have an Nvidia GPU with tensor cores (RTX 2000, RTX Titan, Titan V...) 
for a nice speed up (~x2 speed up) and half the VRAM usage. 
Requires the Nvidia Apex library: https://github.com/NVIDIA/apex

* How-to:
  * Set your game in windowed mode
  * Set your game to 1600x900 resolution
  * Move the game window to the top left corner, there should be a blue line of 1 pixel in the left bezel of your
         screen and the window top bar should start in the top bezel of your screen.
  * Let the AI play the game!
  * Push Q + E to exit
  * Push L to see the input images
  * Push and hold J to use to use manual control
          
<p align="center">
  <img src="github_images/example_config.png" alt="Setup Example"/>
</p>


  

# Authors:
```
- Iker García
  Personal Webpage: https://ikergarcia1996.github.io/Iker-Garcia-Ferrero/
```

This repository is a greatly improved version of the model we published 2 years ago: https://github.com/ikergarcia1996/GTAV-Self-driving-car (by Eritz Yerga and Iker García)
  
