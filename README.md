# R-FCN in MXNet with distributed implementation and data parallelization
-------------------------------------------------------------------
R-FCN: Object Detection via Region-based Fully Convolutional Networks

This repo is forked from https://github.com/tornadomeet/mx-rcnn, and added some new features on it:
* support joint training and alternative optimization for R-FCN, ref. [train_end2end_resnext.py](train_end2end_resnext.py)
* support approximate joint end2end training, ref. [train_end2end.py](train_end2end.py), it can  get comparable result with alternate training.
* add DetectionList Class for any object detection dataset, you only need to prepare your annation list, ref. [detection_list.py](helper/dataset/detection_list.py).
* fix some bugs and typos.

---------------------------------------------------------------------

Region Proposal Network solves object detection as a regression problem
from the objectness perspective. Bounding boxes are predicted by applying
learned bounding box deltas to base boxes, namely anchor boxes across
different positions in feature maps. Training process directly learns a
mapping from raw image intensities to bounding box transformation targets.

Fast R-CNN treats general object detection as a classification problem and
bounding box prediction as a regression problem. Classifying cropped region
feature maps and predicting bounding box displacements together yields
detection results. Cropping feature maps instead of image input accelerates
computation utilizing shared convolution maps. Bounding box displacements
are simultaneously learned in the training process.

Faster R-CNN utilize an alternate optimization training process between RPN
and Fast R-CNN. Fast R-CNN weights are used to initiate RPN for training.

## Getting Started
* Install python package `easydict`, `cv2`, `matplotlib`. MXNet require `numpy`.
* Install MXNet with version no later than Commit 8a3424e, preferably the latest master.
  Follow the instructions at http://mxnet.readthedocs.io/en/latest/how_to/build.html. Install the python interface.
* Try out detection result by running `python demo.py --prefix final --epoch 0 --image myimage.jpg --gpu 0`.
  Suppose you have downloaded pretrained network and place the extracted file `final-0000.params` in this folder and there is an image named `myimage.jpg`.

## Show Result
* model with approximate joint end2end training
  ![train approximate](./result.jpg)


## Training and Testing R-FCN
* Clone this repo:  git clone --recursive https://github.com/terrychenism/mxnet.git && cd mxnet && git checkout rfcn-rebase && make -j8
* Install additional python package `scipy`.
* Download Pascal VOC data and place them to `data` folder according to `Data Folder Structure`.
  You might want to create a symbolic link to VOCdevkit folder by `ln -s /path/to/your/VOCdevkit data/VOCdevkit`.
* Download ResNeXt-101 pretrained model from http://data.mxnet.io/models/imagenet/resnext/101-layers/ , and place it in `model` folder.
  `model` folder will be used to place model checkpoints along the training process.
* Start training by running `python train_end2end_resnext.py` after VOCdevkit is ready.
  A typical command would be `python train_end2end_resnext.py --gpus 0`. This will train the network on the VOC07 trainval.
  More control of training process can be found in the argparse help accessed by `python train_end2end_resnext.py -h`.
* Start testing by run `python test.py` after completing the training process.
  A typical command would be `python test.py --has_rpn --prefix model/faster-resnext-101 --epoch 10`. This will test the network on the VOC07 test.
  Adding a `--vis` will turn on visualization and `-h` will show help as in the training process.
* An experiment yields 74.19 mAP, while VGG based Faster-RCNN yields 66.5 mAP.
  
## Training and Testing Faster-RCNN
* Install additional python package `scipy`.
* Download Pascal VOC data and place them to `data` folder according to `Data Folder Structure`.
  You might want to create a symbolic link to VOCdevkit folder by `ln -s /path/to/your/VOCdevkit data/VOCdevkit`.
* Download VGG16 pretrained model, use `mxnet/tools/caffe_converter` to convert it,
  rename to `vgg16-symbol.json` and `vgg16-0001.params` and place it in `model` folder.
  `model` folder will be used to place model checkpoints along the training process.
* Start training by running `python train_alternate.py` after VOCdevkit is ready.
  A typical command would be `python train_alternate.py --gpus 0`. This will train the network on the VOC07 trainval.
  More control of training process can be found in the argparse help accessed by `python train_alternate.py -h`.
* Start testing by run `python test.py` after completing the training process.
  A typical command would be `python test.py --has_rpn --prefix model/final --epoch 8`. This will test the network on the VOC07 test.
  Adding a `--vis` will turn on visualization and `-h` will show help as in the training process.

## Training and Testing Fast R-CNN
* Download Pascal VOC data and place them to `data` folder according to `Data Folder Structure`.
  You might want to create a symbolic link to VOCdevkit folder by `ln -s /path/to/your/VOCdevkit data/VOCdevkit`.
* Download precomputed selective search data and place them to `data` folder according to `Data Folder Structure`.
* Download VGG16 pretrained model, use `mxnet/tools/caffe_converter` to convert it,
  rename to `vgg16-symbol.json` and `vgg16-0001.params` and place it in `model` folder.
  `model` folder will be used to place model checkpoints along the training process.
* Start training by running `python -m tools.train_rcnn --proposal ss` to use the selective search proposal.
* Start testing by running `python -m tools.test_rcnn --proposal ss`.

## Information
* Download link to faster-rcnn model
  Baidu Yun: http://pan.baidu.com/s/1boRhGvH (ixiw) or Dropbox: https://www.dropbox.com/s/jrr83q0ai2ckltq/final-0000.params.tar.gz?dl=0
* Download link to r-fcn-resnext101 model
  Onedrive: https://1drv.ms/u/s!Aqd-q_R495LrljchqMVhnv5ubF2O symbol：https://1drv.ms/u/s!Aqd-q_R495LrljY3o_udXofNNmiw
* Download link to Pascal VOC and precomputed selective search proposals

  ```
  Pascal VOCdevkit
  http://host.robots.ox.ac.uk/pascal/VOC/voc2007/VOCtrainval_06-Nov-2007.tar
  http://host.robots.ox.ac.uk/pascal/VOC/voc2007/VOCtest_06-Nov-2007.tar
  http://host.robots.ox.ac.uk/pascal/VOC/voc2007/VOCdevkit_08-Jun-2007.tar
  selective_search_data (by Ross Girshick)
  Download link accessible at https://github.com/rbgirshick/fast-rcnn/blob/master/data/scripts/fetch_selective_search_data.sh
  ```

* Data Folder Structure (create a `data` folder if there is none)

  ```
  VOCdevkit
  -- VOC + year (JPEG images and annotations)
  -- results (will be created by evaluation)
  ---- VOC + year
  ------ main
  -------- comp4_det_val_aeroplane.txt
  selective_search_data
  rpn_data (will be created by rpn)
  cache (will be created by imdb)
  ```

## Disclaimer
This repository used code from [MXNet](https://github.com/dmlc/mxnet),
[Fast R-CNN](https://github.com/rbgirshick/fast-rcnn),
[Faster R-CNN](https://github.com/rbgirshick/py-faster-rcnn),
[caffe](https://github.com/BVLC/caffe). Training data are from
[Pascal VOC](http://host.robots.ox.ac.uk/pascal/VOC/),
[ImageNet](http://image-net.org/). Model comes from
[VGG16](http://www.robots.ox.ac.uk/~vgg/research/very_deep/).

## References
1. Tianqi Chen, Mu Li, Yutian Li, Min Lin, Naiyan Wang, Minjie Wang, Tianjun Xiao, Bing Xu, Chiyuan Zhang, and Zheng Zhang. MXNet: A Flexible and Efficient Machine Learning Library for Heterogeneous Distributed Systems. In Neural Information Processing Systems, Workshop on Machine Learning Systems, 2015
2. Ross Girshick. "Fast R-CNN." In Proceedings of the IEEE International Conference on Computer Vision, 2015.
3. Shaoqing Ren, Kaiming He, Ross Girshick, and Jian Sun. "Faster R-CNN: Towards real-time object detection with region proposal networks." In Advances in Neural Information Processing Systems, 2015.
4. Yangqing Jia, Evan Shelhamer, Jeff Donahue, Sergey Karayev, Jonathan Long, Ross Girshick, Sergio Guadarrama, and Trevor Darrell. "Caffe: Convolutional architecture for fast feature embedding." In Proceedings of the ACM International Conference on Multimedia, 2014.
5. Mark Everingham, Luc Van Gool, Christopher KI Williams, John Winn, and Andrew Zisserman. "The pascal visual object classes (voc) challenge." International journal of computer vision 88, no. 2 (2010): 303-338.
6. Jia Deng, Wei Dong, Richard Socher, Li-Jia Li, Kai Li, and Li Fei-Fei. "ImageNet: A large-scale hierarchical image database." In Computer Vision and Pattern Recognition, IEEE Conference on, 2009.
7. Karen Simonyan, and Andrew Zisserman. "Very deep convolutional networks for large-scale image recognition." arXiv preprint arXiv:1409.1556 (2014).
