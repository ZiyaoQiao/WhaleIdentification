# Whale Identification Model

## Repository Description

### A. Topic description
Nowadays, whale is really rare and protecting whale is necessary. Different species of whales have different features in their shape of tails and special markings. Thus, in many cases, scientists monitor whales’ activities by using photos of their tails. To help scientists confirm the species of different whales in huge number of photos and save their time, we aim to build a new machine learning model to do this instead of persons.<br />

### B. Data sources
Most of datas comes from [Happy Whale](https://happywhale.com) and sorted by [Kaggle](https://www.kaggle.com), a platform which already use image process algorithms to category photos collected from public and various survey.<br />

### C. Algorithms are being used and best of public kernals
Since the competition has no award and participants have no responsibility to pubilc their code, limited kernels are available. For most of public kernels, they just try to input data, resize photos and make color channels identical — even it means it may lose some information of colored photos.<br />
Some kernels made further research. For instance, some would use constructed [CNN model to finish the initial identification](https://www.kaggle.com/sunnybeta322/what-am-i-whale-let-me-tell-you). Other use self-developed [triplet model](https://www.kaggle.com/CVxTz/beating-the-baseline-keras-lb-0-38) and it performs better than general CNN model. They beat the baseline of the competition and reached 46.821% accuracy, which seems worth to make some further research. Recently, another participant shared a [traidiional cnn model](https://www.kaggle.com/gimunu/data-augmentation-with-keras-into-cnn) with 32.875% accuracy, implement the CNN model which is different from us.<br />

### D. Evaluating the success of the model
The success of the model will be evaluated based on the accuracy of the model could achieve. The host of the competition will provide one or more test set for participants to evaluate and improve the model. What we need to do is to construct, test and improve the model based on the result we get.<br />

### E. Main model of the project
1. [traditional CNN model with relative few layers](https://github.com/ZiyaoQiao/INFO7390_FinalProject/tree/master/Basic%20Model)<br />
2. [pretrained model(including InceptionV3, Resnet50, VGG16)](https://github.com/ZiyaoQiao/INFO7390_FinalProject/blob/master/Pretrained%20Model/InceptionV3.py)<br />
<br />

### F. Project Process Description -- Basic CNN Model
#### *Before use, please make sure you download the Dataset, edit the input path in the code correctly and install all necessary packages.*

#### F(1) Detect the contour of the tail
We also write a function which could figure out apparent contour in one photo, which could highlighted the shape of the tail in some cases. We would generate a brand new dataset based on this algotithm and use models to learn this dataset.

```python
# for individual picture
# originpath: Absolute path of the Data file

g = os.walk(originPath)
for path,d,filelist in g:
for filename in filelist:
if filename.endswith('jpg'):

img = cv2.imread(originPath+filename)

gray = cv2.cvtColor(img, cv2.COLOR_BGR2GRAY)

gradX = cv2.Sobel(gray, ddepth=cv2.CV_32F, dx=1, dy=0, ksize=3)
gradY = cv2.Sobel(gray, ddepth=cv2.CV_32F, dx=0, dy=1, ksize=3)

# subtract the y-gradient from the x-gradient
gradient = cv2.subtract(gradX, gradY)
gradient = cv2.convertScaleAbs(gradient)
(_, thresh) = cv2.threshold(gradient, 100, 255, cv2.THRESH_BINARY)

thresh = cv2.dilate(thresh, None, iterations=1)
thresh = cv2.dilate(thresh, None, iterations=1)
thresh = cv2.erode(thresh, None, iterations=1)
thresh = cv2.erode(thresh, None, iterations=1)

# use cv2.RETR_TREE to locate and lock the tail
image, contours, hierarchy = cv2.findContours(thresh, cv2.RETR_TREE,
cv2.CHAIN_APPROX_SIMPLE)
img = cv2.drawContours(img, contours, -1, (0, 255, 0), 5)

# return photo with highlighted edges
canny_edges = cv2.Canny(img, 300, 300)
plt.imshow(canny_edges)
cv2.imwrite(targetPath + filename, img, [int(cv2.IMWRITE_JPEG_QUALITY), 95])
```

#### F(2) Make photos Black and White, resize the photo
Since some of photos in the datast is gray and White as well as others are colored, in the basic CNN model, we use an existing function and make all the photos in the dataset Black-and-White, try to decrease the noises generated by colors, even it also indicates we will lose some useful information in some cases.

```python
def ImportImage(filename):
img = Image.open(filename).convert("LA").resize((SIZE,SIZE))
return np.array(img)[:,:,0]
train_img = np.array([ImportImage(img) for img in train_images])
```

#### F(3) Data preprocessing and augmentation

```python
#use of an image generator for preprocessing and data augmentation
x = x.reshape((-1,SIZE,SIZE,1))
input_shape = x[0].shape
x_train = x.astype("float32")
y_train = y_cat

image_gen = ImageDataGenerator(
    featurewise_center=True,
    featurewise_std_normalization=True,
    rotation_range=15,
    width_shift_range=.15,
    height_shift_range=.15,
    horizontal_flip=True)
```


#### F(4) Label Hot Encoder
Since the dataset is not sequential, we need to set label in every pictures for the model to enact deep learning. 

```python
class LabelOneHotEncoder():
    def __init__(self):
        self.ohe = OneHotEncoder()
        self.le = LabelEncoder()
    def fit_transform(self, x):
        features = self.le.fit_transform( x)
        return self.ohe.fit_transform( features.reshape(-1,1))
    def transform( self, x):
        return self.ohe.transform( self.la.transform( x.reshape(-1,1)))
    def inverse_tranform( self, x):
        return self.le.inverse_transform( self.ohe.inverse_tranform( x))
    def inverse_labels( self, x):
        return self.le.inverse_transform(x)

```

#### F(5) Plot Images 
A function for us in developing, to watch the dataset again before we train.

```python
def plotImages(images_arr, n_images=4):
    fig, axes = plt.subplots(n_images, n_images, figsize=(12,12))
    axes = axes.flatten()
    for img, ax in zip( images_arr, axes):
        if img.ndim != 2:
            img = img.reshape((SIZE,SIZE))
        ax.imshow( img, cmap="Greys_r")
        ax.set_xticks(())
        ax.set_yticks(())
    plt.tight_layout()

```

#### F(6) Assist functions 
We also set some assistant function for our model, set up the training set, visualize image if necessary and construct the class weights. In this way, we could set class label of image as index and we could also devide photos in each step equally.

```python
#constructing class weights
WeightFunction = lambda x : 1./x**0.75
ClassLabel2Index = lambda x : lohe.le.inverse_tranform( [[x]])
CountDict = dict(train_df["Id"].value_counts())
class_weight_dic = {lohe.le.transform([image_name])[0]: WeightFunction(count) for image_name, count in CountDict.items()}

#training the image preprocessing
image_gen.fit(x_train, augment=True)

#visualization of some images out of the preprocessing
augmented_images, _ = next( image_gen.flow( x_train, y_train.toarray(), batch_size=4*4))

```

#### F(7) Convolution Nerual Network Model
After many different try, we finally worked out a CNN model with highest cost-interest ratio. This model have 2 convolutional layer after 1 input layer, with several dropout layer, flatten layer and dense layer to reshape the data and catch features. We set batchsize as 128 and step value is total number of the training set devided by batchsize, in this way we could make sure every photo in the dataset could be iterated once in a single epoch. Initially, the model run in 9 epochs to run in relative low costs. However, more epochs could be added if someone have better machine.

```python
batch_size = 128
num_classes = len(y_cat.toarray()[0])
epochs = 9

model = Sequential()
model.add(Conv2D(48, kernel_size=(3, 3),
                 activation='relu',
                 input_shape=input_shape))
model.add(Conv2D(48, (3, 3), activation='sigmoid'))
model.add(MaxPooling2D(pool_size=(2, 2)))
model.add(Conv2D(48, (5, 5), activation='sigmoid'))
model.add(MaxPooling2D(pool_size=(3, 3)))
model.add(Dropout(0.33))
model.add(Flatten())
model.add(Dense(36, activation='sigmoid'))
model.add(Dropout(0.33))
model.add(Dense(36, activation='sigmoid'))
model.add(Dense(num_classes, activation='softmax'))

model.compile(loss=keras.losses.categorical_crossentropy,
              optimizer=keras.optimizers.Adadelta(),
              metrics=['accuracy'])
model.summary()
model.fit_generator(image_gen.flow(x_train, y_train.toarray(), batch_size=batch_size),
          steps_per_epoch=  x_train.shape[0]//batch_size,
          epochs=epochs,
          verbose=1,
          class_weight=class_weight_dic)
```

### G. Project Process Description -- Pretrained Model

#### *Before use, please make sure you download the Dataset, edit the input path in the code correctly and install all necessary packages.*


#### G(1) Train without "New_whale" class
Since the category "New_Whale" is an ambiguous category, contain a lot of photos(more than 800) with various features, which would lead a large amount of noise, so in some of our model, we train the model without this category.

```python

```


### H. Results

| Method        | Accuracy in test set                         | Epochs                         | Average time (s/per epoch)    |
| ------------- | ---------------------------- | ----                  |------------------------------------------------ |
| ResNet50 without 'NewWhale'            | 32.482%                  | 60                 | 12478s|
| ResNet50 with 'NewWhale'            | 32.631%                  | 60                   | 12600s|
| VGG19 with 'NewWhale'            | 32.999%                  | 20                   | 2270s|
| inceptionV3 without 'NewWhale'            | 32.481%                  | 10                   | 4703s|
| CNN Model with contour detected            | 32.763%                  | 69                   | 55s|
| CNN Model without contour detected            | 32.875%                  | 10                   | 54s|



### I. References
1. https://www.coursera.org/learn/neural-networks-deep-learning
2. https://papers.nips.cc/paper/4824-imagenet-classification-with-deep-convolutional-neural-networks.pdf
3. https://arxiv.org/abs/1512.03385
4. https://en.wikipedia.org/wiki/Humpback_whale
5. https://cs231n.github.io<br />

### J.License (MIT)
These tutorials and source-code are published under the [MIT License](https://github.com/ZiyaoQiao/INFO7390_FinalProject/blob/master/LICENSE) which allows very broad use for both academic and commercial purposes.<br />

A few of the images used for demonstration purposes may be under copyright. These images are included under the "fair usage" laws.<br />

You are very welcome to modify these tutorials and use them in your own projects, but please keep a link to the [original repository](https://github.com/ZiyaoQiao/INFO7390_FinalProject).<br />
