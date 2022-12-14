# ML-Group-Project-8: Pothole Detection on the Jetson Nano

## Introduction
With the new era of autonomous vehicles, there is an ever growing necessity for autonomous road safety and road defect detection. Our objective in this project is to develop machine learning models which detect potholes in roads, a small step towards making autonomous driving safer. The data we are currently planning to use comes from the Brazilian National Department of Transport Infrastructure, and consists of 2235 images of highways in the states of Espírito Santo, Rio Grande do Sul and the Federal District from 2014 to 2017. The resolution of the images is at least 1280x729 with a 16:9 aspect ratio. We hope to train deep convolutional neural networks and also an SVM image classification/object detection to consistently identify potholes in new road images, with the end-goal of real time video inference using a Jetson Nano. We will also consider many data preprocessing methods such as image masking/transforming, hog extraction, negative image extractiongreyscale extraction, and scaling in our pipeline to further optimize our models. Our hypothesis is that a sufficiently large deep convolutional neural network is capable of accurately classifying road defects, and we hope to optimize its performance with what we have learned in class and previous work in the field.

## Methods

### Data Creation
The first step is to create the dataset with the images, their labels, and the parameters for the pothole bounding boxes. We will use keras to load and convert images to numpy arrays, and cv2 to detect the potholes and label them. We store the dataset as a csv file. This is what a sample/mask looks like with its corresponding bounding box

The table organizes the different attributes of the images into columns. The img column has the gray scale representation of the images that were pulled from git for more a more efficient calculations. The smaller number decreases the complexity of the convolving process. The pothole being the target of the project is encoded as 1 and 0 for true and false of whether certain images contain potholes or not.

![sample](https://user-images.githubusercontent.com/38708456/206112718-8283b796-b305-4bf7-b2a5-9e56d6d8c95f.png)


After creating the csv dataset, it looks like this:

![table](https://user-images.githubusercontent.com/38708456/206113022-aa12b6f7-11e6-439c-9e2a-145bee95f621.JPG)



### Data Exploration
We are working with a dataset that contains **2235** samples (images). The target classes are **0** and **1**, which correspond to the given road containing a pothole (**1**) or not (**0**). As we can see in the target class distribution, the data is a bit imbalanced. We have **564** samples with potholes, and **1671** samples without. This can lead to oversampling where the data chosen for training is skewed towards one class. Therefore, it's important that we shuffle and split our data carefully.


![classes](https://user-images.githubusercontent.com/38708456/206113157-e7d53728-9381-4c36-b089-71328b6f14ae.png)


What each of the target classes look like:
<br>

![example](https://user-images.githubusercontent.com/38708456/206113332-98492211-5084-4e75-80e6-aac4add0f029.png)


The distributions of bounding box widths & heights are very right skewed, as one would expect.


![box_w](https://user-images.githubusercontent.com/38708456/206113501-3fed7fd9-30e3-4071-8dea-2594cba84a27.png)


![box_h](https://user-images.githubusercontent.com/38708456/206113611-518e8cc9-d565-4b72-a467-79e6e4d3f901.png)


### Data Preprocessing

* Our dataset consists of 2236 pairs of images. Each image is either 630 by 1024 or 640 by 1024.
In order to standardize, we scale each image down to 600 by 600. This also makes the training easier by decreasing the dimensions we input into our model [1].
* Once the image is loaded into our python environment as a PIL object, we convert to grayscale. This is actually only needed for the original images as they are in color and the pothole masks are already black and white.
* We then normalize the image by turning it into a numpy array and dividing by 255.
* Our result is 2236 image pairs all of which are (600, 600) numpy arrays with float values ranging from 0 to 1.

[1] The main reason is that as long as the rescaling doesn't significantly distort the relevant features of an image, shrinking it down allows us to build a deeper model and reduce the computational load on our models. Reducing the costs allows us to test on even more data. We will test it out but most likely we are going to end up shrinking the images even further (to 250 by 250) later on.


### [Model 1: Simple Model](https://github.com/sachinmloecher/ML-Group-Project-8/blob/510102881861f54d81c0cf5a7ddb4ef11e09b0b3/Notebooks/Simple_Model.ipynb)

Our first model is a Convolutional Neural Network with the layers:
<br>


![model](https://user-images.githubusercontent.com/38708456/206114677-ab0eb225-a785-4e9b-ba5c-62f31d3b909c.png)


This simple model has **4** convolutional layers and **1** Dense layer with 62 nodes. We used 15 epochs, with a batch size of 2, the Adam optimizer with a learning rate of **0.0001**, and $MSE$ as the loss function. This model has 5 outputs: the bounding box coordinates as well as the class.

### [Model 2: YOLO Model](https://github.com/sachinmloecher/ML-Group-Project-8/blob/be75b3f8a578206bc3725a50a11a00f088a59c75/Notebooks/Yolo_Model.ipynb)

Our second model is similar to the original yolo v1 object detection CNN, with the layers:
<br>


![yolomodel](https://user-images.githubusercontent.com/38708456/206114763-a41ee164-b0a5-473a-b617-0926c7ed2818.png)



This model has **20** convolutional layers and **5** Dense layers, with Batch Normalization and Leaky ReLu like in the original yolo v1 paper. We used 15 epochs, with a batch size of 2, and the Adam optimizer with a learning rate of **0.001**. We decided to make this model a regression only model, meaning it only outputs the bounding box predictions and not the class.

### [Model 3: VGG16 Model](https://github.com/sachinmloecher/ML-Group-Project-8/blob/be75b3f8a578206bc3725a50a11a00f088a59c75/Notebooks/VGG16_Model.ipynb)

Our third model extends an already existing network with set initial weights. We altered the VGG16 Network Head with our own trainable Dense layers to output the predicted bounding box coordinates:


![vgg_model](https://user-images.githubusercontent.com/38708456/206114814-8a349052-b376-46e9-871b-0bff2543313f.png)


This model has **13** convolutional layers and **4** Dense layers, with Max Pooling in between layers. We used 10 epochs, with a batch size of 2, and the Adam optimizer with a learning rate of **0.0001*. This model is also a regression only model, meaning it has 4 outputs corresponding to the bounding box coordinate predictions.


### [Model 4: Branched VGG16](https://github.com/sachinmloecher/ML-Group-Project-8/blob/3778621c6875979136f026b8aa47ec1d08dd1d1e/Notebooks/Branched%20VGG16_Model.ipynb) 

Our fourth model is a work in progress. It is similar to model 3, as it extends an existing network with set initial weights. We altered the VGG16 Network Head with two of our own trainable Dense layers to output the predicted bounding box coordinated and the predicticted class labels.


![branchedmodel](https://user-images.githubusercontent.com/38708456/206114870-a3bc6473-e40a-482a-814a-cb868f9e8d46.png)


This model has **13** convolutional layers and **4** Dense layers for each branch, with Max Pooling in between layers. We used 15 epochs, with a batch size of 2, and the Adam optimizer with a learning rate of **0.0001*. This model has 5 outputs, corresponding to the 4 bounding box coordinate predictions and the binary pothole classification.

### [Model 5: Binary Classification using CNNs with dropout](https://github.com/sachinmloecher/ML-Group-Project-8/blob/main/Notebooks/pothole_binary_model.ipynb))

Due to the nature of limited computational resources and a fairly small dataset in the grand scheme of things, we thought it would be best to make a model with more modest aims. I.e. instead of training a model which finds bounding boxes, why not just make a model that tells us whether a pothole is present in the image or not? This is binary classification, and is a task more suitable for the methods we learned in class (basic CNNs).

First thing’s first, we need to label our dataset. Our dataset is technically labeled with masks. Each mask is a black-and-white image with white pixels in the regions where potholes are. So for preprocessing, we can just look at the numpy array representation of a mask and if there are any values of 255 in the array then we classify the corresponding image as one with a pothole. We do this in the notebook dataset_org.ipynb and then rearrange the dataset into two folders: one for images with potholes and one for those without. We then download this new version of the dataset from colab and upload it to a public google drive zip file for easy access. The reason that we put the images into folders according to classification is that it allows us to turn the dataset into a tf.data.Dataset object which is a very efficient and convenient way to load the dataset when working in keras. It basically allows us to only load the images into memory when a batch is needed, where it then gets the images from disk and applies preprocessing methods. Now let’s get into the actual model:

![bi_model](https://github.com/sachinmloecher/ML-Group-Project-8/blob/6c71b28ad666765dd150091a0003fba4c5785fba/Images/bi_model_overview.PNG)


### [Model 6: SVM (Support Vector Machine) Model for image classification and pothole detection](https://github.com/sachinmloecher/ML-Group-Project-8/blob/main/Notebooks/SVM_Model.ipynb)

We introduce an SVM Object detection model from kaggle that was inspired by Mehmet Tekman which classified cars. We modify the code and use it to classify images and even further detect potholes via the SVM bounds. After data preprocessing we augment the images via hogs, negative images, and gray scale. The gray scale images (gray colors), hog (Histogram of Oriented Gradients) images (Images that highlight contours and distinct images like potholes), and negative images (inverse colors). The main decision also uses the heatmap, and the heatmap/image predictions are displayed in results. We also resize the image in order to simplify the amount of detail the model needs to use to study the image.

Negative Image Example:
<img width="1024" alt="NegativeImage" src="https://user-images.githubusercontent.com/38708456/206398347-02540e38-5095-40b6-9307-ff0e6a38e737.png">

We see the inverted colors make it easier for SVM to potentially detect the pothole

Hog Image Example:

<img width="681" alt="HogImageComparision" src="https://user-images.githubusercontent.com/38708456/206398295-b1066c48-8348-4524-bb8d-29a81342a805.png">

As we can see in the image, Hog (Histogram of Oriented Gradients), counts the number of occurences of a gradient in a certain rotation in one part of the image. We use this to extract edges and features from the image, hopefully the ones from the pothole specifically. The reason why we use hog over the image is because images often have certain variations in terms of occlusion, color, light, etc. This noise is reduced by the hog and serves as a representation of the image without the noise explained earlier. 






## Results

We are using IOU as an accuracy metric for the bounding boxes. Intersection over Union (IOU) is defined as the area of overlap divided by the area of union of the predicted and true bounding boxes. Typically, an IOU > 0.5 is very good.

### Model 1: Simple Model
This is how this simple model performed:
<br>


![simplemodel](https://user-images.githubusercontent.com/38708456/206113672-36a2645e-8ccf-4df5-b2d9-3c5829358257.png)


As we can see, this model is far too simple to have an IOU (accuracy in the graph) of **0.015** or higher. We can see signs of overfitting very early on.
We can see this simple model did not perform very well, but there is lots of room for improvement. Here are 2 example predictions (green: true, red:prediction):
<br>



![simplemodelpred](https://user-images.githubusercontent.com/38708456/206115090-c3897f92-70ea-4566-a845-8d4ff800dc43.png)


### Model 2: YOLO Model
This is how the Yolo model performed:
<br>


![yolo_model](https://user-images.githubusercontent.com/38708456/206112238-6f0ad27c-ac2f-4ad2-904c-cc6be4fd3576.png)


As we can see, this model also did not perform very well, resulting in an test IOU similar to that of the simple model.


### Model 3: VGG16 Model
This is how the VGG16 model performed:
<br>


![vggmodel](https://user-images.githubusercontent.com/38708456/206113892-ce97a84c-d596-4738-ae79-da170cd7ebae.png)


As we can see, this model did significantly better than the others, resulting in a training IOU of 21% and a testing IOU of almost 10%. Although there is still lots of room for improvement, this is a good start. Here are 2 example predictions made by the VGG16 model:


![vggexample1](https://user-images.githubusercontent.com/38708456/206115264-f6474ac4-d088-4ae5-b80c-589d012f439a.png)

![vggexample2](https://user-images.githubusercontent.com/38708456/206115719-f48a88a4-25c0-4307-98f5-3ccdd94ec6b9.png)


### Model 4: Branched VGG16 Model
This is how the Branched VGG16 model performed:
<br>


![branched_model](https://user-images.githubusercontent.com/38708456/206113943-0af32101-9294-4004-a5ea-867897cd4d3a.png)


We can immediately see that this model did not train well. The validation loss stays the same through the epochs, though the training loss decreases. It might seem like the bounding box accuracy is high at **70%**, but this is not good. This model uses both images with and without potholes, because it it predicting the bounding boxes and the class label. The problem with this is that the majority of the data are 'no pothole', with bounding box coordinates [0,0,0,0]. This means that using this prior, the model can achieve a bounding box accuracy of **70%** by predicting all boxes to be essentially [0,0,0,0]. Here is an example showing exactly the problem of the prior in this branched model:


![branched_example](https://user-images.githubusercontent.com/38708456/206115403-68f78b53-b820-49d3-891f-ea7d3dc127e7.png)

As we can see, the model predicted the bounding box coordinates as [0,0,0,0], when there was indeed a pothole.



### Model 5: Binary Classification using CNNs with dropout
This is how our boutique CNN binary classification model performed:
<br>
![binary_example](https://github.com/sachinmloecher/ML-Group-Project-8/blob/main/Images/bi_plot.png?raw=true)


### Model 6: SVM Model
Unfortunately, while the accuracy reaches **70** percent for strict images classification, the bounds used to detect the pothole specifically were way off. This might cause an issue with pothole detection but it was good that there was a **70** percent accuaracy in terms of image classfication. The image detection result looks like this:

SVM Prediction via heatmap:


<img width="376" alt="SVMpredictionheatmap" src="https://user-images.githubusercontent.com/38708456/206396860-848e49e5-dcfb-4c69-9c8f-c4c218d2f788.png">

The bounds clearly do not detect the pothole and use SVM like detection. In the future, we can try to improve the heatmap

SVM prediction:



<img width="379" alt="SVMprediction" src="https://user-images.githubusercontent.com/38708456/206398312-7e77ac9b-7d20-4549-a9db-096235bbe528.png">

Same for SVM prediction heatmap


## Discussion
### Model 1: Simple Model
We thought this model was a good place to start because it is not very complicated, and was trainable in a decent amount of time. Initially we had a custom loss function which was the $IOU$ measure, but we quickly realized that $IOU$ does not make a good loss function for the following reasons:
- If the predicted box entirely contains the target box, the gradients with respect to the box positions will be 0
- If the target box entirely contains the predicted box, the gradients with respect to position will be 0.
- If the predicted box and target box are completely disjoint, all the gradients will be 0, which impedes training
So we switched the loss to the classic $MSE$ loss. We think that this model was simply not complex enough to learn the pothole patterns in the images, as seen by the early overfitting. This model was also trained on data with and without potholes, and training it on only potholes like we did with the other may have increased its accuracy.


### Model 2: YOLO Model
This model performed surprisingly poorly given its complexity. At first we trained it on data with and without potholes, but saw better results when focusing on pothole images for the box regression. 


### Model 3: VGG16 Model
This model did significantly better than the others in terms of IOU. Although it is still far from perfect, the complex dense layers in combination with the VGG16 weights seem to have recognized petterns the Yolo model simply did not see.



### Model 4: Branched VGG16 Model
This model still needs to be significantly tweaked in order to see real results, but I thought the effect of the prior was very interesting.





### Model 5: Binary Classification using CNNs with dropout
This model had the highest accuracy of all our models, but it was limited to binary classification. Potentially it could improve with more epochs, but it appears that 80% accuracy might be the limit for the architecture. 

### Model 6: SVM Model
We used this model because the model in kaggle was able to cleanly detect cars from its features solelely using SVM window slicing and other preprocessing steps like hogs. We did the same thing with potholes but since they are smaller and more harder to detect than cars the pothole detection was not as great. In the future, we can extract more feature to determine if there are potholes

### Jetson Nano
After fully developing our models we would have liked to test the models using the jetson nano with live video input. In setting up the jetson nano we encountered many issues with the different version of images to install on the sd card. 4.5 was used in the end since it was the only one that worked while the newer 4.6 version did not boot for the setup. Also the ssh functionality did not work, and libraries would not install(scipy). We would have liked to test the model by bringing the jetson with us in a car to capture live stream of the potholes on the road and indicating them with bounding boxes. 

We figured out how to run the camera on the jetson nano and was able to capture video and outputed a continuous flow of frames to feed the model. However due to the library download issue we could never create the model on the jetson nano to test with the camera feed as input.


For all the models, we chose the batchsize and epochs based on what Colab could handle, and whether or not there were signs of overfitting during training. The box regression performed much better when trained on only data with potholes, and the VGG16 Model had the best accuracy so far.


## Conclusion
To conclude, the VGG16 Model performed the best of the models we created, although it still has a long ways to go. I think that pothole recognition is no simple task, and will require much more of my time to further improve the accuracy. I plan on continuing the development of these and new models regardless of if I receive credit for them. I would like to implement the Yolo v5 network next, as it has an incredible reported accuracy and speed. There is still lots of work to be done on these models, but this was a good start. 


## Collaboration
**Sachin Loecher**
- Wrote the abstract/introduction
- Did the dataset creation and preprocessing
- Did the first preprocessing milestone and write up
- Built the first model
- Did the first model milestone and write up
- Built the second model
- Built the third model
- Built the fourth model
- Wrote the Methods section
- Wrote the Results section
- Wrote the Discussion section
- Wrote the Conclusion Section

**Nicholas Chan, Dennis Li**
- Setup the Jetson Nano
- Wrote Python code for the Jetson Nano
- debugged pathing bug in model
- Proofreading and adding to README

**Gordon Feliz**
- Organized github files
- Created outline for research abstract
- Helped keep README links up to date
- Split PotholeDetection(1) into two files, created Data Creation and Processing notebook and Branched VGG16_Model
- Cleaned up & bug fixed Data Creation and Processing + Branched VGG16_Model

**Tarun Devesetti**
- Added SVM Model to get 70 percent image classificiation accuracy and attempt to detect potholes
- Added SVM Model in README: introduction, Results, Discussion, Models
- Added all images to the README
- Wrote Python Code for SVM Model (Inspired by Kaggle Website (Support Vector Machine Object Detection), Link: https://www.kaggle.com/code/mehmetlaudatekman/support-vector-machine-object-detection/notebook
- Updated Model SVM 6 with data preprocessing steps that were not used in the neural networks
- Used Preprocessing steps like MiniMax Scaling and data augmentation methods such as hog extraction, negative image extraction, and gray scale extraction for SVM model

