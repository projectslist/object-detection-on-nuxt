

# Object Detection with Tensorflow.js and Coco SSD

This is a web application built with Vue.js that uses the Coco SSD object detection model from TensorFlow.js to detect objects in real-time from a webcam stream. The detected objects are tracked over time and their durations are displayed in a sorted list.

# Features
Object detection using the Coco SSD model
Real-time tracking of detected objects
Display of tracked objects in a sorted list by duration
Start and stop the webcam stream
Session storage of tracked items and longest duration item

# Getting Started
To run this application locally, follow these steps:

1 - Clone this repository:
```
git clone https://github.com/projectslist/object-detection-on-nuxt.git
```
2 - Install the dependencies:
```
cd object-detection-on-nuxt
npm install
```

3 - Run the development server:
```
npm run dev
```

# Usage
1 - Click on the "Start Camera" button to start the webcam stream.

2 - Objects detected in the webcam stream will be highlighted with a rectangle and labeled with their class name.

3 - The list of tracked objects and their durations will be displayed in the "Tracked Items" section of the page.

4 - The list of tracked items is sorted by duration in ascending order.

5 - The "Stop Camera" button can be clicked to stop the webcam stream.

# Credits
This application was built using the following libraries and resources:

Vue.js

TensorFlow.js

Coco SSD object detection model

Vuetify

Icons from Material Design Icons

# License
This project is licensed under the MIT License - see the LICENSE.md file for details.



## Build Setup For Nuxt If It is needed

```bash
# install dependencies
$ npm install

# serve with hot reload at localhost:3000
$ npm run dev

# build for production and launch server
$ npm run build
$ npm run start

# generate static project
$ npm run generate
```
