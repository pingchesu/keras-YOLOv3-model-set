# keras-yolo3-Mobilenet

[![license](https://img.shields.io/github/license/mashape/apistatus.svg)](LICENSE)

## Introduction

A Keras implementation of YOLOv3 (Tensorflow backend) inspired by [allanzelener/YAD2K](https://github.com/allanzelener/YAD2K).
#### And I change the backend of darknet53 into
- [x] Mobilenet
- [x] VGG16
- [x] ResNet101
- [x] ResNeXt101

## Experiment on open datasets

| Model name | InputSize | TrainSet | TestSet | mAP | Speed | Ps |
| ----- | ------ | ------ | ------ | ----- | ----- | ----- |
| YOLOv3-Mobilenet | 320x320 | VOC07 | VOC07 | 64.22% | 29fps | Keras on 1080Ti |
| YOLOv3-Mobilenet | 320x320 | VOC07+12 | VOC07 | 74.56% | 29fps | Keras on 1080Ti |
| YOLOv3-Mobilenet | 416x416 | VOC07+12 | VOC07 | 76.82% | 25fps | Keras on 1080Ti |
| [MobileNet-SSD](https://github.com/chuanqi305/MobileNet-SSD) | 300x300 | VOC07+12+coco | VOC07 | 72.7% | (unknown) ||
| [MobileNet-SSD](https://github.com/chuanqi305/MobileNet-SSD) | 300x300 | VOC07+12 | VOC07 | 68% | (unknown) ||
| [Faster RCNN, VGG-16](https://github.com/ShaoqingRen/faster_rcnn)| ~1000x600 | VOC07+12| VOC07 | 73.2% | 151ms | Caffe on Titan X |
|[SSD,VGG-16](https://github.com/pierluigiferrari/ssd_keras) | 300x300 | VOC07+12 | VOC07	| 77.5% | 39fps | Keras on Titan X |

#### PS:
1. Compared with MobileNet-SSD, YOLOv3-Mobilenet is much better on VOC2007 test, even without pre-training on Ms-COCO
2. I use the default anchor size that the author cluster on COCO with inputsize of 416\*416, whereas the anchors for VOC 320 input should be smaller. The change of anchor size could gain performance improvement.
3. Evaluation on https://github.com/Adamdad/Object-Detection-Metrics.git
4. I only use the pure model of YOLOv3-Mobilenet with no additional tricks.

# Guide of refreshed train & evaluate & demo process

### Train
1.train.py
> * Support YOLOv3 & YOLOv3-Tiny model, related param (dataset, pretrained weights, epochs num etc.) could be changed in code

2.train_multiscale.py
> * Multiscale training script for YOLOv3 & YOLOv3-Tiny model

3.train_Mobilenet.py
> * Support YOLOv3_Lite-Mobilnet & YOLOv3_Lite-Mobilnet-Tiny model (YOLOv3-Lite use Depthwise Conv in yolo head part), related param could be changed in code

4.train_Mobilenet_multiscale.py
> * Multiscale training script for YOLOv3_Lite-Mobilnet & YOLOv3_Lite-Mobilnet-Tiny model

#### Checkpoints during training could be found at logs/000/. Choose a best one as result

### Model dump
#### We need to dump out inference model from training checkpoint. Following script cmd work for that.

1.YOLOv3 & YOLOv3-Tiny model
```
python yolo_video.py --model_path=logs/000/<checkpoint>.h5 --anchors_path=model_data/yolo_anchors.txt --classes_path=model_data/voc_classes.txt --dump_model --output_model_file=test.h5
```

2.YOLOv3_Lite-Mobilnet & YOLOv3_Lite-Mobilnet-Tiny model
```
python yolo_Mobilenet.py --model_path=logs/000/<checkpoint>.h5 --anchors_path=model_data/yolo_anchors.txt --classes_path=model_data/voc_classes.txt --dump_model --output_model_file=test.h5
```
#### Change anchors file & class file for different training mode

### Evaluation
Use "compute_mAP.py" to do evaluation on the inference model. It will draw out rec/pre curve for each class and AP/mAP result chart under "result" dir, and optionally save all the detection result on evaluation dataset as images
```
python compute_mAP.py --model_path=test.h5 --anchors_path=model_data/yolo_anchors.txt --classes_path=model_data/voc_classes.txt --model_image_size=416x416 --annotation_file=2007_test.txt --save_result
```

### TFLite convert & validate
1. Use tflite_convert to generate TFLite inference model. We need to specify input node name and input shape since our inference model doesn't have input image shape. Only valid under tensorflow 1.13
```
tflite_convert [--post_training_quantize] --input_arrays=input_1 --input_shapes=1,416,416,3 --output_file=test[_quant].tflite --keras_model_file=test.h5
```
2. Run TFLite validate script
```
python validate_yolo_tflite.py --model_path=test.tflite --anchors_path=model_data/yolo_anchors.txt --classes_path=model_data/voc_classes.txt --image_file=test.jpg --loop_count=1
```
#### You can also use "compute_mAP.py" to do evaluate on the TFLite model

3. TODO item
- [] TFLite C++ inplementation of yolo head


# Origin Guide of keras-yolov3-Mobilenet

1.train_Mobilenet.py

> * **Code for training**
> * I change some of the code to read in the annotaions seperately (train.txt and val.txt), remember to change that, and the .txt file are in the same form descibed below

2.yolo3/model_Mobilenet.py

> * **Model_Mobilenet is the yolo model based on Mobilenet**
> * If you want to go through the source code,ignore the other function,please see the yolo_body
(I extract three layers from the Mobilenet to make the prediction)

3.yolo_Mobilenet.py

 > * **Testing on images**


###### Be sure that you do not load pretrained model when training because I did it on keras_applications,and the keras will load the pretrained model for you
##### if you find anything tricky, contact me as you wish
---
# Evaluation
#### Please use this repo to draw the RP curve calculate the MAP https://github.com/Adamdad/Object-Detection-Metrics.git
---
# Guide of keras-yolov3
[this is the guide for darknet53 not mobilenet]
## Quick Start

1. Download YOLOv3 weights from [YOLO website](http://pjreddie.com/darknet/yolo/).
2. Convert the Darknet YOLO model to a Keras model.
3. Run YOLO detection.

```
wget https://pjreddie.com/media/files/yolov3.weights
python convert.py yolov3.cfg yolov3.weights model_data/yolo.h5
python yolo_video.py [OPTIONS...] --image, for image detection mode, OR
python yolo_video.py [video_path] [output_path (optional)]
```

For Tiny YOLOv3, just do in a similar way, just specify model path and anchor path with `--model model_file` and `--anchors anchor_file`.

### Usage
Use --help to see usage of yolo_video.py:
```
usage: yolo_video.py [-h] [--model MODEL] [--anchors ANCHORS]
                     [--classes CLASSES] [--gpu_num GPU_NUM] [--image]
                     [--input] [--output]

positional arguments:
  --input        Video input path
  --output       Video output path

optional arguments:
  -h, --help         show this help message and exit
  --model MODEL      path to model weight file, default model_data/yolo.h5
  --anchors ANCHORS  path to anchor definitions, default
                     model_data/yolo_anchors.txt
  --classes CLASSES  path to class definitions, default
                     model_data/coco_classes.txt
  --gpu_num GPU_NUM  Number of GPU to use, default 1
  --image            Image detection mode, will ignore all positional arguments
```
---

4. MultiGPU usage: use `--gpu_num N` to use N GPUs. It is passed to the [Keras multi_gpu_model()](https://keras.io/utils/#multi_gpu_model).

## Training

1. Generate your own annotation file and class names file.
    One row for one image;
    Row format: `image_file_path box1 box2 ... boxN`;
    Box format: `x_min,y_min,x_max,y_max,class_id` (no space).
    For VOC dataset, try `python voc_annotation.py`
    Here is an example:
    ```
    path/to/img1.jpg 50,100,150,200,0 30,50,200,120,3
    path/to/img2.jpg 120,300,250,600,2
    ...
    ```

2. Make sure you have run `python convert.py -w yolov3.cfg yolov3.weights model_data/yolo_weights.h5`
    The file model_data/yolo_weights.h5 is used to load pretrained weights.

3. Modify train.py and start training.
    `python train.py`
    Use your trained weights or checkpoint weights with command line option `--model model_file` when using yolo_video.py
    Remember to modify class path or anchor path, with `--classes class_file` and `--anchors anchor_file`.

If you want to use original pretrained weights for YOLOv3:
    1. `wget https://pjreddie.com/media/files/darknet53.conv.74`
    2. rename it as darknet53.weights
    3. `python convert.py -w darknet53.cfg darknet53.weights model_data/darknet53_weights.h5`
    4. use model_data/darknet53_weights.h5 in train.py

---

## Some issues to know

1. The test environment is
    - Python 3.5.2
    - Keras 2.1.5
    - tensorflow 1.6.0

2. Default anchors are used. If you use your own anchors, probably some changes are needed.

3. The inference result is not totally the same as Darknet but the difference is small.

4. The speed is slower than Darknet. Replacing PIL with opencv may help a little.

5. Always load pretrained weights and freeze layers in the first stage of training. Or try Darknet training. It's OK if there is a mismatch warning.

6. The training strategy is for reference only. Adjust it according to your dataset and your goal. And add further strategy if needed.

7. For speeding up the training process with frozen layers train_bottleneck.py can be used. It will compute the bottleneck features of the frozen model first and then only trains the last layers. This makes training on CPU possible in a reasonable time. See [this](https://blog.keras.io/building-powerful-image-classification-models-using-very-little-data.html) for more information on bottleneck features.




# Citation
Please cite MobileNet-YOLO in your publications if it helps your research:
```
@article{MobileNet-Yolov3,
     Author = {Adam Yang},
     Year = {2018}
}
 @article{yolov3,
     title={YOLOv3: An Incremental Improvement},
     author={Redmon, Joseph and Farhadi, Ali},
     journal = {arXiv},
     year={2018}
}
@article{mobilenets,
     title={MobileNets: Efficient Convolutional Neural Networks for Mobile Vision Applications},
     author={Andrew G. Howard, Menglong Zhu, Bo Chen,Dmitry Kalenichenko,Weijun Wang, Tobias Weyand,Marco Andreetto, Hartwig Adam},
     journal = {arXiv},
     year = {2017}
}
```
