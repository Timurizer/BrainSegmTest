# BrainSegmTest

## Preprocessing

Images were were provided along with #.txt files containing their angles of rotation.
The first step of preprocessing was to rotate them.  It was done using SimpleITK.
  
The results of rotation can be seen below:
<img src="https://github.com/Timurizer/BrainSegmTest/blob/main/imgs/turning.PNG" width="750">

Due to rotation, voids are filled with zeros by default,
which causes distortions in data. It can be seen at third image in rotation result as a bright corners.

It was only realized at the stage of data exploration, when examining image values at points of interest:  
<img src="https://github.com/Timurizer/BrainSegmTest/blob/main/imgs/imgValues.PNG" width="400">

The solution was to set pixel value as -1024 duiring rotation:  
<img src="https://github.com/Timurizer/BrainSegmTest/blob/main/imgs/defaultPixelValue.PNG" width="250">


In order to increase the contrast of the image, it was clipped between -200 and 300 and scaled:  
<img src="https://github.com/Timurizer/BrainSegmTest/blob/main/imgs/crop.png" width="500">

## Pipeline

Since data contains а limited number of patients with a slight difference in quantity of slices for each patient, data of one patient was immediately taken as a validation
data. Other patients were taken for train data.

I decided to use the fact that the brain is symmetrical, so only 10 classes were used to train model.
Class labels above 10 was mapped to it's analogs from another hemisphere.

## Sampling
To make sure that samples of each class are present in each epoch, a custom
Sampler was implemented. Sampler contains class - img list data mapping and 
guarantees that desired amount of samples for each class are selected for training.

## Augmentation
Several augmentation techniques were used to increase data diversity and reduce overfitting.
These include:
 * ShiftScaleRotate
 * Blur
 * IAAAdditiveGaussianNoise
 * And one of:
   * ElasticTransform
   * OpticalDistortion
   * GridDistortion
   
All taken from "Albumentations" library.

<img src="https://github.com/Timurizer/BrainSegmTest/blob/main/imgs/aug1.PNG" width="250"><img src="https://github.com/Timurizer/BrainSegmTest/blob/main/imgs/aug2.PNG" width="250">
<img src="https://github.com/Timurizer/BrainSegmTest/blob/main/imgs/aug3.PNG" width="250">
 

## Model
FPN was chosen because it has already shown its efficiency in segmentation tasks ([[1]](http://presentations.cocodataset.org/ECCV18/COCO18-Panoptic-Caribbean.pdf), [[2]](https://arxiv.org/abs/1901.02446)), and is competitive with state-of-the-art models.

I used segmentation_models_pytorch implementation of FPN along with the efficientnet-b0 encoder. 
Encoder was chosen based on the fact that it was pretrained on 512x512 data just as data at this case.
It is also the most lightweight of all efficientnet encoders provided by segmentation_models_pytorch.

As a loss function, BCE + LogDice loss was implemented.  
As a learning rate scheduler, Torch's ReduceLROnPlateau was used.

This model achieved mean IoU score of 0.79144084 on all classes .

Training visualizations:  
<img src="https://github.com/Timurizer/BrainSegmTest/blob/main/imgs/trainVisualization.PNG" width="700">



The score on each class can be seen below:
<img src="https://github.com/Timurizer/BrainSegmTest/blob/main/imgs/results.PNG" width="700">



