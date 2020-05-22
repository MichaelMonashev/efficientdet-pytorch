# EfficientDet (PyTorch)

This is a work in progress PyTorch implementation of EfficientDet. 

It is based on the
* official Tensorflow implementation by [Mingxing Tan and the Google Brain team](https://github.com/google/automl)
* paper by Mingxing Tan, Ruoming Pang, Quoc V. Le [EfficientDet: Scalable and Efficient Object Detection](https://arxiv.org/abs/1911.09070) 

I am aware there are other PyTorch implementations. Their approach didn't fit well with my aim to replicate the Tensorflow models closely enough to allow weight ports while still maintaining a PyTorch feel and a high degree of flexibility for future additions. So, this is built from scratch and leverages my previous EfficientNet work.

## Updates / Tasks

### 2020-05-22
A bunch of changes:
* COCO eval per epoch for better selection of checkpoints while training, works with distributed
* optimizations to both train and inference that should see small throughput gains
* doing the above, attempted to torchscript the full training loss + anchor labeler but ran into problems so had to back out part way due messy hacks or weird AMP issues causing silent bad results. Hopefully in PyTorch 1.6 there will be less TS issues.
* updated results after clipping boxes, now pretty much exact match to official, even slightly better on a few models

### 2020-05-04
Initial D1 training results in -- close but not quite there. Definitely in reach and better than any other non-official EfficientDet impl I've seen.

Biggest missing element is proper per-epoch mAP validation for better checkpoint selection (than loss based). I was resisting doing full COCO eval because it's so slow, but may throw that in for now...

D1:
```
 Average Precision  (AP) @[ IoU=0.50:0.95 | area=   all | maxDets=100 ] = 0.382
 Average Precision  (AP) @[ IoU=0.50      | area=   all | maxDets=100 ] = 0.577
 Average Precision  (AP) @[ IoU=0.75      | area=   all | maxDets=100 ] = 0.407
 Average Precision  (AP) @[ IoU=0.50:0.95 | area= small | maxDets=100 ] = 0.190
 Average Precision  (AP) @[ IoU=0.50:0.95 | area=medium | maxDets=100 ] = 0.437
 Average Precision  (AP) @[ IoU=0.50:0.95 | area= large | maxDets=100 ] = 0.552
 Average Recall     (AR) @[ IoU=0.50:0.95 | area=   all | maxDets=  1 ] = 0.314
 Average Recall     (AR) @[ IoU=0.50:0.95 | area=   all | maxDets= 10 ] = 0.489
 Average Recall     (AR) @[ IoU=0.50:0.95 | area=   all | maxDets=100 ] = 0.520
 Average Recall     (AR) @[ IoU=0.50:0.95 | area= small | maxDets=100 ] = 0.286
 Average Recall     (AR) @[ IoU=0.50:0.95 | area=medium | maxDets=100 ] = 0.591
 Average Recall     (AR) @[ IoU=0.50:0.95 | area= large | maxDets=100 ] = 0.713
```

Previous D0 result:
```
 Average Precision  (AP) @[ IoU=0.50:0.95 | area=   all | maxDets=100 ] = 0.324
 Average Precision  (AP) @[ IoU=0.50      | area=   all | maxDets=100 ] = 0.513
 Average Precision  (AP) @[ IoU=0.75      | area=   all | maxDets=100 ] = 0.342
 Average Precision  (AP) @[ IoU=0.50:0.95 | area= small | maxDets=100 ] = 0.121
 Average Precision  (AP) @[ IoU=0.50:0.95 | area=medium | maxDets=100 ] = 0.383
 Average Precision  (AP) @[ IoU=0.50:0.95 | area= large | maxDets=100 ] = 0.499
 Average Recall     (AR) @[ IoU=0.50:0.95 | area=   all | maxDets=  1 ] = 0.280
 Average Recall     (AR) @[ IoU=0.50:0.95 | area=   all | maxDets= 10 ] = 0.426
 Average Recall     (AR) @[ IoU=0.50:0.95 | area=   all | maxDets=100 ] = 0.452
 Average Recall     (AR) @[ IoU=0.50:0.95 | area= small | maxDets=100 ] = 0.188
 Average Recall     (AR) @[ IoU=0.50:0.95 | area=medium | maxDets=100 ] = 0.532
 Average Recall     (AR) @[ IoU=0.50:0.95 | area= large | maxDets=100 ] = 0.668
```

### 2020-05-02
First decent MSCOCO training results (from scratch, w/ pretrained classification backbone weights as starting point). 32.4 mAP for D0. Working on improvements and D1 trials still running.

### 2020-04-15
Taking a pause on training, some high priority things came up. There are signs of life on the training branch, was working the basic augs before priority switch, loss fn appeared to be doing something sane with distributed training working, no proper eval yet, init not correct yet. I will get to it, with SOTA training config and good performance as the end goal (as with my EfficientNet work). 

### 2020-04-11
Cleanup post-processing. Less code and a five-fold throughput increase on the smaller models. D0 running > 130 img/s on a single 2080Ti, D1 > 130 img/s on dual 2080Ti up to D7 @ 8.5 img/s.

### 2020-04-10
Replace `generate_detections` with PyTorch impl using torchvision batched_nms. Significant performance increase with minor (+/-.001 mAP) score differences. Quite a bit faster than original TF impl on a GPU now.

### 2020-04-09
Initial code with working validation posted. Yes, it's a little slow, but I think faster than the official impl on a GPU if you leave AMP enabled. Post processing needs some love. 

### Core Tasks
- [x] Feature extraction from my EfficientNet implementations (https://github.com/rwightman/gen-efficientnet-pytorch or https://github.com/rwightman/pytorch-image-models)
- [x] Low level blocks / helpers (SeparableConv, create_pool2d (same padding), etc)
- [x] PyTorch implementation of BiFPN, BoxNet, ClassNet modules and related submodules
- [x] Port Tensorflow checkpoints to PyTorch -- initial D1 checkpoint converted, state_dict loaded, on to validation....
- [x] Basic MS COCO validation script
  - [x] Temporary (hacky) COCO dataset and transform 
  - [x] Port reference TF anchor and object detection code
  - [x] Verify model output sanity
  - [X] Integrate MSCOCO eval metric calcs
- [x] Some cleanup, testing
- [x] Submit to test-dev server, all good
- [ ] Add torch hub support and pretrained URL based weight download
- [x] Remove redundant bias layers that exist in the official impl and weights
- [ ] Add visualization support
- [x] Performance improvements, numpy TF detection code -> optimized PyTorch
- [ ] Verify/fix Torchscript and ONNX export compatibility
- [ ] Try PyTorch 1.5 w/ NHWC (channels last) order which matches TF impl

### Possible Future Tasks
- [x] Basic Training (object detection) reimplementation
- [ ] Advanced Training w/ Rand/AutoAugment, etc
- [ ] Training (semantic segmentation) experiments
- [ ] Integration with Detectron2 / MMDetection codebases
- [ ] Addition and cleanup of EfficientNet based U-Net and DeepLab segmentation models that I've used in past projects
- [ ] Addition and cleanup of OpenImages dataset/training support from a past project
- [ ] Exploration of instance segmentation possibilities...

If you are an organization is interested in sponsoring and any of this work, or prioritization of the possible future directions interests you, feel free to contact me (issue, LinkedIn, Twitter, hello at rwightman dot com). I will setup a github sponser if there is any interest.

## Models

| Variant | Download | mAP (val2017) | mAP (test-dev2017) | mAP (TF official val2017) | mAP (TF official test-dev2017) |
| --- | --- | :---: | :---: | :---: | :---: |
| D0 | [tf_efficientdet_d0.pth](https://github.com/rwightman/efficientdet-pytorch/releases/download/v0.1/efficientdet_d0-d92fd44f.pth) | 33.6 | TBD | 33.5 | 33.8 |
| D1 | [tf_efficientdet_d1.pth](https://github.com/rwightman/efficientdet-pytorch/releases/download/v0.1/efficientdet_d1-4c7ebaf2.pth) | 39.3 | TBD | 39.1 | 39.6 |
| D2 | [tf_efficientdet_d2.pth](https://github.com/rwightman/efficientdet-pytorch/releases/download/v0.1/efficientdet_d2-cb4ce77d.pth) | 42.6 | 43.1 | 42.5 | 43 |
| D3 | [tf_efficientdet_d3.pth](https://github.com/rwightman/efficientdet-pytorch/releases/download/v0.1/efficientdet_d3-b0ea2cbc.pth) | 46.0 | TBD | 45.9 | 45.8 | 
| D4 | [tf_efficientdet_d4.pth](https://github.com/rwightman/efficientdet-pytorch/releases/download/v0.1/efficientdet_d4-5b370b7a.pth) | 49.1 | TBD | 49.0 | 49.4 |
| D5 | [tf_efficientdet_d5.pth](https://github.com/rwightman/efficientdet-pytorch/releases/download/v0.1/efficientdet_d5-ef44aea8.pth) | 50.4 | TBD | 50.5 | 50.7 |
| D6 | [tf_efficientdet_d6.pth](https://github.com/rwightman/efficientdet-pytorch/releases/download/v0.1/efficientdet_d6-51cb0132.pth) | 51.2 | TBD | 51.3 | 51.7 |
| D7 | [tf_efficientdet_d7.pth](https://github.com/rwightman/efficientdet-pytorch/releases/download/v0.1/efficientdet_d7-f05bf714.pth) | 51.8 | 52.1 | 52.1 | 52.2 |


## Usage

### Environment Setup

Tested in a Python 3.7 or 3.8 conda environment in Linux with:
* PyTorch 1.4
* PyTorch Image Models (timm) 0.1.20, `pip install timm` or local install from (https://github.com/rwightman/pytorch-image-models) 
* Apex AMP master (as of 2020-04)

*NOTE* - There is a conflict/bug with Numpy 1.18+ and pycocotools, force install numpy <= 1.17.5 or the coco eval will fail,
the validation script will still save the output JSON and that can be run through eval again later. 

### Dataset Setup

MSCOCO 2017 validation data:
```
wget http://images.cocodataset.org/zips/val2017.zip
wget http://images.cocodataset.org/annotations/annotations_trainval2017.zip
unzip val2017.zip
unzip annotations_trainval2017.zip
```

MSCOCO 2017 test-dev data:
```
wget http://images.cocodataset.org/zips/test2017.zip
unzip -q test2017.zip
wget http://images.cocodataset.org/annotations/image_info_test2017.zip
unzip image_info_test2017.zip
```

### Run COCO Evaluation

Run validation (val2017 by default) with D2 model: `python validation.py /localtion/of/mscoco/ --model tf_efficientdet_d2 --checkpoint tf_efficientdet_d2.pth`

Run test-dev2017: `python validation.py /localtion/of/mscoco/ --model tf_efficientdet_d2 --checkpoint tf_efficientdet_d2.pth --anno test-dev2017`

### Run Inference

TODO: Need an inference script

### Run Training

`./distributed_train.sh 2 /mscoco --model tf_efficientdet_d0 -b 16 --amp  --lr .05 --warmup-epochs 5  --sync-bn --opt fusedmomentum --fill-color mean --model-ema`

NOTE:
* Training script currently defaults to a model that does NOT have redundant conv + BN bias layers like the official models, set correct flag when validating.
* I've only trained with img mean (`--fill-color mean`) as the background for crop/scale/aspect fill, the official repo uses black pixel (0) (`--fill-color 0`). Both likely work fine.
* The official training code uses EMA weight averaging by default, it's not clear there is a point in doing this with the cosine LR schedule, I find the non-EMA weights end up better than EMA in the last 10-20% of training epochs 
* The default h-params is a very close to unstable (exploding loss), don't try using Nesterov momentum. Try to keep the batch size up, use sync-bn.

## Results

### TEST-DEV2017

NOTE: I've only tried submitting D2 and D7 to dev server for sanity check so far

#### EfficientDet-D2

```
 Average Precision  (AP) @[ IoU=0.50:0.95 | area=   all | maxDets=100 ] = 0.431
 Average Precision  (AP) @[ IoU=0.50      | area=   all | maxDets=100 ] = 0.624
 Average Precision  (AP) @[ IoU=0.75      | area=   all | maxDets=100 ] = 0.463
 Average Precision  (AP) @[ IoU=0.50:0.95 | area= small | maxDets=100 ] = 0.226
 Average Precision  (AP) @[ IoU=0.50:0.95 | area=medium | maxDets=100 ] = 0.471
 Average Precision  (AP) @[ IoU=0.50:0.95 | area= large | maxDets=100 ] = 0.585
 Average Recall     (AR) @[ IoU=0.50:0.95 | area=   all | maxDets=  1 ] = 0.345
 Average Recall     (AR) @[ IoU=0.50:0.95 | area=   all | maxDets= 10 ] = 0.543
 Average Recall     (AR) @[ IoU=0.50:0.95 | area=   all | maxDets=100 ] = 0.575
 Average Recall     (AR) @[ IoU=0.50:0.95 | area= small | maxDets=100 ] = 0.342
 Average Recall     (AR) @[ IoU=0.50:0.95 | area=medium | maxDets=100 ] = 0.632
 Average Recall     (AR) @[ IoU=0.50:0.95 | area= large | maxDets=100 ] = 0.756
```

#### EfficientDet-D7
```
 Average Precision  (AP) @[ IoU=0.50:0.95 | area=   all | maxDets=100 ] = 0.521
 Average Precision  (AP) @[ IoU=0.50      | area=   all | maxDets=100 ] = 0.714
 Average Precision  (AP) @[ IoU=0.75      | area=   all | maxDets=100 ] = 0.563
 Average Precision  (AP) @[ IoU=0.50:0.95 | area= small | maxDets=100 ] = 0.345
 Average Precision  (AP) @[ IoU=0.50:0.95 | area=medium | maxDets=100 ] = 0.555
 Average Precision  (AP) @[ IoU=0.50:0.95 | area= large | maxDets=100 ] = 0.646
 Average Recall     (AR) @[ IoU=0.50:0.95 | area=   all | maxDets=  1 ] = 0.390
 Average Recall     (AR) @[ IoU=0.50:0.95 | area=   all | maxDets= 10 ] = 0.631
 Average Recall     (AR) @[ IoU=0.50:0.95 | area=   all | maxDets=100 ] = 0.670
 Average Recall     (AR) @[ IoU=0.50:0.95 | area= small | maxDets=100 ] = 0.497
 Average Recall     (AR) @[ IoU=0.50:0.95 | area=medium | maxDets=100 ] = 0.704
 Average Recall     (AR) @[ IoU=0.50:0.95 | area= large | maxDets=100 ] = 0.808
 ```

### VAL2017

#### EfficientDet-D0
```
 Average Precision  (AP) @[ IoU=0.50:0.95 | area=   all | maxDets=100 ] = 0.336
 Average Precision  (AP) @[ IoU=0.50      | area=   all | maxDets=100 ] = 0.516
 Average Precision  (AP) @[ IoU=0.75      | area=   all | maxDets=100 ] = 0.354
 Average Precision  (AP) @[ IoU=0.50:0.95 | area= small | maxDets=100 ] = 0.125
 Average Precision  (AP) @[ IoU=0.50:0.95 | area=medium | maxDets=100 ] = 0.387
 Average Precision  (AP) @[ IoU=0.50:0.95 | area= large | maxDets=100 ] = 0.528
 Average Recall     (AR) @[ IoU=0.50:0.95 | area=   all | maxDets=  1 ] = 0.288
 Average Recall     (AR) @[ IoU=0.50:0.95 | area=   all | maxDets= 10 ] = 0.440
 Average Recall     (AR) @[ IoU=0.50:0.95 | area=   all | maxDets=100 ] = 0.467
 Average Recall     (AR) @[ IoU=0.50:0.95 | area= small | maxDets=100 ] = 0.194
 Average Recall     (AR) @[ IoU=0.50:0.95 | area=medium | maxDets=100 ] = 0.549
 Average Recall     (AR) @[ IoU=0.50:0.95 | area= large | maxDets=100 ] = 0.686
```

#### EfficientDet-D1
```
 Average Precision  (AP) @[ IoU=0.50:0.95 | area=   all | maxDets=100 ] = 0.393
 Average Precision  (AP) @[ IoU=0.50      | area=   all | maxDets=100 ] = 0.583
 Average Precision  (AP) @[ IoU=0.75      | area=   all | maxDets=100 ] = 0.419
 Average Precision  (AP) @[ IoU=0.50:0.95 | area= small | maxDets=100 ] = 0.187
 Average Precision  (AP) @[ IoU=0.50:0.95 | area=medium | maxDets=100 ] = 0.447
 Average Precision  (AP) @[ IoU=0.50:0.95 | area= large | maxDets=100 ] = 0.572
 Average Recall     (AR) @[ IoU=0.50:0.95 | area=   all | maxDets=  1 ] = 0.323
 Average Recall     (AR) @[ IoU=0.50:0.95 | area=   all | maxDets= 10 ] = 0.501
 Average Recall     (AR) @[ IoU=0.50:0.95 | area=   all | maxDets=100 ] = 0.532
 Average Recall     (AR) @[ IoU=0.50:0.95 | area= small | maxDets=100 ] = 0.295
 Average Recall     (AR) @[ IoU=0.50:0.95 | area=medium | maxDets=100 ] = 0.599
 Average Recall     (AR) @[ IoU=0.50:0.95 | area= large | maxDets=100 ] = 0.734
```

#### EfficientDet-D2
```
 Average Precision  (AP) @[ IoU=0.50:0.95 | area=   all | maxDets=100 ] = 0.426
 Average Precision  (AP) @[ IoU=0.50      | area=   all | maxDets=100 ] = 0.618
 Average Precision  (AP) @[ IoU=0.75      | area=   all | maxDets=100 ] = 0.452
 Average Precision  (AP) @[ IoU=0.50:0.95 | area= small | maxDets=100 ] = 0.237
 Average Precision  (AP) @[ IoU=0.50:0.95 | area=medium | maxDets=100 ] = 0.481
 Average Precision  (AP) @[ IoU=0.50:0.95 | area= large | maxDets=100 ] = 0.590
 Average Recall     (AR) @[ IoU=0.50:0.95 | area=   all | maxDets=  1 ] = 0.342
 Average Recall     (AR) @[ IoU=0.50:0.95 | area=   all | maxDets= 10 ] = 0.537
 Average Recall     (AR) @[ IoU=0.50:0.95 | area=   all | maxDets=100 ] = 0.569
 Average Recall     (AR) @[ IoU=0.50:0.95 | area= small | maxDets=100 ] = 0.348
 Average Recall     (AR) @[ IoU=0.50:0.95 | area=medium | maxDets=100 ] = 0.633
 Average Recall     (AR) @[ IoU=0.50:0.95 | area= large | maxDets=100 ] = 0.748
```

#### EfficientDet-D3
```
 Average Precision  (AP) @[ IoU=0.50:0.95 | area=   all | maxDets=100 ] = 0.460
 Average Precision  (AP) @[ IoU=0.50      | area=   all | maxDets=100 ] = 0.651
 Average Precision  (AP) @[ IoU=0.75      | area=   all | maxDets=100 ] = 0.493
 Average Precision  (AP) @[ IoU=0.50:0.95 | area= small | maxDets=100 ] = 0.283
 Average Precision  (AP) @[ IoU=0.50:0.95 | area=medium | maxDets=100 ] = 0.503
 Average Precision  (AP) @[ IoU=0.50:0.95 | area= large | maxDets=100 ] = 0.618
 Average Recall     (AR) @[ IoU=0.50:0.95 | area=   all | maxDets=  1 ] = 0.360
 Average Recall     (AR) @[ IoU=0.50:0.95 | area=   all | maxDets= 10 ] = 0.570
 Average Recall     (AR) @[ IoU=0.50:0.95 | area=   all | maxDets=100 ] = 0.605
 Average Recall     (AR) @[ IoU=0.50:0.95 | area= small | maxDets=100 ] = 0.409
 Average Recall     (AR) @[ IoU=0.50:0.95 | area=medium | maxDets=100 ] = 0.655
 Average Recall     (AR) @[ IoU=0.50:0.95 | area= large | maxDets=100 ] = 0.768
```

#### EfficientDet-D4
 ```
 Average Precision  (AP) @[ IoU=0.50:0.95 | area=   all | maxDets=100 ] = 0.491
 Average Precision  (AP) @[ IoU=0.50      | area=   all | maxDets=100 ] = 0.685
 Average Precision  (AP) @[ IoU=0.75      | area=   all | maxDets=100 ] = 0.531
 Average Precision  (AP) @[ IoU=0.50:0.95 | area= small | maxDets=100 ] = 0.334
 Average Precision  (AP) @[ IoU=0.50:0.95 | area=medium | maxDets=100 ] = 0.539
 Average Precision  (AP) @[ IoU=0.50:0.95 | area= large | maxDets=100 ] = 0.641
 Average Recall     (AR) @[ IoU=0.50:0.95 | area=   all | maxDets=  1 ] = 0.375
 Average Recall     (AR) @[ IoU=0.50:0.95 | area=   all | maxDets= 10 ] = 0.598
 Average Recall     (AR) @[ IoU=0.50:0.95 | area=   all | maxDets=100 ] = 0.635
 Average Recall     (AR) @[ IoU=0.50:0.95 | area= small | maxDets=100 ] = 0.468
 Average Recall     (AR) @[ IoU=0.50:0.95 | area=medium | maxDets=100 ] = 0.683
 Average Recall     (AR) @[ IoU=0.50:0.95 | area= large | maxDets=100 ] = 0.780
```

#### EfficientDet-D5
```
 Average Precision  (AP) @[ IoU=0.50:0.95 | area=   all | maxDets=100 ] = 0.504
 Average Precision  (AP) @[ IoU=0.50      | area=   all | maxDets=100 ] = 0.700
 Average Precision  (AP) @[ IoU=0.75      | area=   all | maxDets=100 ] = 0.543
 Average Precision  (AP) @[ IoU=0.50:0.95 | area= small | maxDets=100 ] = 0.337
 Average Precision  (AP) @[ IoU=0.50:0.95 | area=medium | maxDets=100 ] = 0.549
 Average Precision  (AP) @[ IoU=0.50:0.95 | area= large | maxDets=100 ] = 0.646
 Average Recall     (AR) @[ IoU=0.50:0.95 | area=   all | maxDets=  1 ] = 0.381
 Average Recall     (AR) @[ IoU=0.50:0.95 | area=   all | maxDets= 10 ] = 0.617
 Average Recall     (AR) @[ IoU=0.50:0.95 | area=   all | maxDets=100 ] = 0.654
 Average Recall     (AR) @[ IoU=0.50:0.95 | area= small | maxDets=100 ] = 0.485
 Average Recall     (AR) @[ IoU=0.50:0.95 | area=medium | maxDets=100 ] = 0.696
 Average Recall     (AR) @[ IoU=0.50:0.95 | area= large | maxDets=100 ] = 0.791
```

#### EfficientDet-D6
```
 Average Precision  (AP) @[ IoU=0.50:0.95 | area=   all | maxDets=100 ] = 0.512
 Average Precision  (AP) @[ IoU=0.50      | area=   all | maxDets=100 ] = 0.706
 Average Precision  (AP) @[ IoU=0.75      | area=   all | maxDets=100 ] = 0.551
 Average Precision  (AP) @[ IoU=0.50:0.95 | area= small | maxDets=100 ] = 0.348
 Average Precision  (AP) @[ IoU=0.50:0.95 | area=medium | maxDets=100 ] = 0.555
 Average Precision  (AP) @[ IoU=0.50:0.95 | area= large | maxDets=100 ] = 0.654
 Average Recall     (AR) @[ IoU=0.50:0.95 | area=   all | maxDets=  1 ] = 0.386
 Average Recall     (AR) @[ IoU=0.50:0.95 | area=   all | maxDets= 10 ] = 0.623
 Average Recall     (AR) @[ IoU=0.50:0.95 | area=   all | maxDets=100 ] = 0.661
 Average Recall     (AR) @[ IoU=0.50:0.95 | area= small | maxDets=100 ] = 0.500
 Average Recall     (AR) @[ IoU=0.50:0.95 | area=medium | maxDets=100 ] = 0.701
 Average Recall     (AR) @[ IoU=0.50:0.95 | area= large | maxDets=100 ] = 0.794
```

#### EfficientDet-D7
 ```
 Average Precision  (AP) @[ IoU=0.50:0.95 | area=   all | maxDets=100 ] = 0.518
 Average Precision  (AP) @[ IoU=0.50      | area=   all | maxDets=100 ] = 0.711
 Average Precision  (AP) @[ IoU=0.75      | area=   all | maxDets=100 ] = 0.558
 Average Precision  (AP) @[ IoU=0.50:0.95 | area= small | maxDets=100 ] = 0.368
 Average Precision  (AP) @[ IoU=0.50:0.95 | area=medium | maxDets=100 ] = 0.564
 Average Precision  (AP) @[ IoU=0.50:0.95 | area= large | maxDets=100 ] = 0.655
 Average Recall     (AR) @[ IoU=0.50:0.95 | area=   all | maxDets=  1 ] = 0.386
 Average Recall     (AR) @[ IoU=0.50:0.95 | area=   all | maxDets= 10 ] = 0.627
 Average Recall     (AR) @[ IoU=0.50:0.95 | area=   all | maxDets=100 ] = 0.665
 Average Recall     (AR) @[ IoU=0.50:0.95 | area= small | maxDets=100 ] = 0.505
 Average Recall     (AR) @[ IoU=0.50:0.95 | area=medium | maxDets=100 ] = 0.704
 Average Recall     (AR) @[ IoU=0.50:0.95 | area= large | maxDets=100 ] = 0.801
 ```
