# [Deep Motion Modeling Tracker](https://shijies.github.io/DMMN_Page/)

We introduce the **D**eep **M**otion **M**odeling **N**etwork (**DMM-Net**) that performs implicit detection and association of the objects in an end-to-end manner. DMM-Net models comprehensive object features over multiple frames and simultaneously infers object motion parameters, categories and visibilities. These outputs are readily used to update the tracklets for efficient MOT. DMM-Net achieves PR-MOTA score of 12.80 @ **120+ fps** for jointly performing detection and tracking on the popular UA-DETRAC challenge - orders of magnitude faster than the existing methods with better performance.

------

- [Deep Motion Modeling Tracker](#deep-motion-modeling-tracker)
  - [Results](#results)
  - [RoadMap](#roadmap)
  - [Protocol](#protocol)
  - [DMM-Net](#dmm-net)
  - [DMM Tracker](#dmm-tracker)
  - [Requirement](#requirement)
  - [Preparation](#preparation)
  - [OMOTD](#omotd)
    - [Test](#test)
    - [Train](#train)
  - [UA-DETRAC](#ua-detrac)
    - [Test](#test-1)
    - [Train](#train-1)
  - [Citation](#citation)
  - [Acknowledge](#acknowledge)
  - [License](#license)

------

## Results

> DMM-Net on Omni-MOT dataset 

[![](images/omni_result.gif)](https://www.youtube.com/watch?v=eWgUHj8lNns&list=PLfYk__bSOigA_EAE4iX1p0Pi3W59jn_GD)

> DMM-Net on UA-DETRAC dataset

[![](./images/mvi_39271.gif)](https://www.youtube.com/watch?v=2RIpCtxMYEg&list=PLfYk__bSOigC-b67N_BBrCRtlJ9wW60ju&index=2&t=0s)


## RoadMap

| Date   | Event                                                        |
| ------ | ------------------------------------------------------------ |
| 201911 | Finish the papers :-)                                        |
| 201910 | Preparing papers                                             |
| 201908 | Get Result on [Omini-MOT](https://github.com/shijieS/OmniMOTDataset) dataset                                   |
| 201908 | Can Train on [Omini-MOT](https://github.com/shijieS/OmniMOTDataset) dataset                                    |
| 201907 | Can Train on MOT17 dataset                                   |
| 201906 | Can Train on ``[CVPR 2019 Tracking Challenge](<https://motchallenge.net/data/CVPR_2019_Tracking_Challenge/#download>)'' |
| 201905 | Can Train On the Whole UA-DETRAC dataset                     |
| 201905 | Design the tracker                                           |
| 201904 | Recording Five Cities Training Dataset                       |
| 201903 | Start A Plan of Create New Dataset                           |
| 201902 | Optimized this network                                       |
| 201812 | Can Do the Basic Detection                                   |
| 201811 | Design the Loss Fucntion                                     |
| 201810 | Try the UA-DETRAC dataset                                    |
| 201809 | Re-design the input and output                               |
| 201808 | Design the whole network                                     |
| 201807 | Start this idea                                              |

## Protocol

- <img src="https://latex.codecogs.com/svg.latex?N_F,N_C,N_P,N_T"/> respectively denote the number of input frames, object categories (0 for `background'), time-related motion parameters, and anchor tunnels.
- <img src="https://latex.codecogs.com/svg.latex?W,H"/> are the frame width, and frame height.
- <img src="https://latex.codecogs.com/svg.latex?\bf{I}_t"/> denotes the video frame at time <img src="https://latex.codecogs.com/svg.latex?t"/>. Subsequently, a 4-D tensor <img src="https://latex.codecogs.com/svg.latex?\bf{I}_{t_1:t_2:N_F}\in\mathbb{R}^{3\times&space;N_F&space;\times&space;W&space;\times&space;H}"/> denotes <img src="https://latex.codecogs.com/svg.latex?N_F"/> video frames from time <img src="https://latex.codecogs.com/svg.latex?t_1"/> to <img src="https://latex.codecogs.com/svg.latex?t_2-1"/>. For simplicity, we often ignore the subscript ``<img src="https://latex.codecogs.com/svg.latex?:N_F"/>''.
- <img src="https://latex.codecogs.com/svg.latex?\bf{B}_{t_1:t_2:N_F},\bf{C}_{t_1:t_2:N_F},\bf{V}_{t_1:t_2:N_F}"/> respectively denote the ground truth boxes, categories, and visibilities in the selected <img src="https://latex.codecogs.com/svg.latex?N_F"/> video frames from time <img src="https://latex.codecogs.com/svg.latex?t_1"/> to <img src="https://latex.codecogs.com/svg.latex?t_2-1"/>. The text also ignores ``<img src="https://latex.codecogs.com/svg.latex?:N_F"/>'' for these notations.
- <img src="https://latex.codecogs.com/svg.latex?\bf{O}_{M,t_1:t_2:N_F},O_{C,t_1:t_2:N_F},O_{V,t_1:t_2:N_F}"/> denote the estimated motion parameters, categories, and visibilities. With time stamps and frames clear from the context, we simplify these notations as <img src="https://latex.codecogs.com/svg.latex?O_M,O_C,O_V"/>.

## DMM-Net

Schematics of end-to-end trainable DMM-Net: <img src="https://latex.codecogs.com/svg.latex?N_F"/> frames and their time stamps <img src="https://latex.codecogs.com/svg.latex?t_1:t_2"/> are input to the network. The frame sequence is first processed with a *Feature Extractor* comprising 3D ResNet-like convolutional groups. Outputs of selected groups are processed by Motion Subnet, Classifier Subnet, and Visibility Subnet. Each sub-network uses 3D convolutions to learn features that are concatenated and used to predict motion parameters (<img src="https://latex.codecogs.com/svg.latex?O_M\in\mathbb{R}^{N_T\times&space;N_P\times&space;4}"/>), object categories (<img src="https://latex.codecogs.com/svg.latex?O_C\in\mathbb{R}^{N_T\times&space;N_C}"/>), and visibility (<img src="https://latex.codecogs.com/svg.latex?O_V\in\mathbb{R}^{N_F\times&space;N_T\times&space;2}"/>), where <img src="https://latex.codecogs.com/svg.latex?N_T"/> <img src="https://latex.codecogs.com/svg.latex?N_P"/> and <img src="https://latex.codecogs.com/svg.latex?N_C"/> denote the number of anchor tunnels, motion parameters and object categories.

![framework](./images/framework.png)

## DMM Tracker

We directly deploy the trained network into the **DMM Tracker** (**DMMT**), as shown in the following figure.  <img src="https://latex.codecogs.com/svg.latex?2N_F"/> frames are processed by the tracker, where the trained DMM-Net selects <img src="https://latex.codecogs.com/svg.latex?N_F"/> frames as its input, and outputs predicted tunnels containing all possible object's motion parameter matrice <img src="https://latex.codecogs.com/svg.latex?(O_M)"/>, category matrice <img src="https://latex.codecogs.com/svg.latex?(O_C)"/> and visibility matrice <img src="https://latex.codecogs.com/svg.latex?(O_V)"/>, which are then filtered by the Tunnel Filter. After that, the track set <img src="https://latex.codecogs.com/svg.latex?\mathcal{T}_{t_i}"/> is updated by associating the filtered tunnels by their IOU with previous track set <img src="https://latex.codecogs.com/svg.latex?\mathcal{T}_{t_{i-1}}"/>.

![demployment](images/tracker.png)

> This tracker can achieve **120+** fps for jointly performing detection and tracking.




## Requirement
| Name   | Version |
| ------ | ------- |
| Python | 3.6     |
| CUDA   | >=8.0   |

Besides, install all the python package by following command

```shell
cd <project path>
pip install -r requirement.txt
```

## Preparation
- Clone this repository

  ```shell
  git clone <repository url>
  ```

- Download the [pre-trained base net model](https://drive.google.com/open?id=1CYb-RBZpz3UTbQRM4oIRipZrWrq10iIQ), and save it into `<project path>/weights/resnext-101-64f-kinetics.pth`

## OMOTD

- Download the training dataset and testing dataset from [[baidu]](https://pan.baidu.com/s/1yqeoBd5RidsIn8mn7_m7zA) or [dropbox no space availabe :-(]

### Test

- Download the network weights file ([[dropbox]](https://www.dropbox.com/s/y5a3vutowfgp68u/ssdt67650.pth?dl=0), [[baidu]](https://pan.baidu.com/s/1HEXPriqLvLBAFkgotmbxDA))

- Modify the `<project path>/config/__init__.py` to

  ```python
  15. # configure_name = 'config_gpu4_ua.json'
  16. # configure_name = 'config_gpu4_ua_test.json'
  17. # configure_name = 'config_gpu4_ua_with_amot.json'
  18. configure_name = 'config_gpu4_amot.json'
  ```

- Modify the `<project path>/config/config_gpu4_amot.json`

  ```python
  1.    {
  2.       "dataset_name": "AMOTD",
  3.       "dataset_path": <your downloaded omotd folder>,
  4.		 "phase": "test",    
      ...
  24.			"test": {
  25.				"resume": <your downloaded weights>,
  26.			    "dataset_type": <you can use "train" or "test">,    
      ...
  30.    			"base_net_weights": null,
  31.    			"log_save_folder": <your log save folder>,
  32.			    "image_save_folder": <your image save folder>,
  33.			    "weights_save_folder": <your weights save folder>, 
  ```

- Activate your python environment, and run

  ```shell
  cd <project folder>
  python test_tracker_amotd.py
  ```

### Train

- Modify the `<project path>/config/__init__.py` to

  ```python
  15. # configure_name = 'config_gpu4_ua.json'
  16. # configure_name = 'config_gpu4_ua_test.json'
  17. # configure_name = 'config_gpu4_ua_with_amot.json'
  18. configure_name = 'config_gpu4_amot.json'
  ```

- Modify the `<project path>/config/config_gpu4_amot.json`

  ```python
  1.	{
  2.		"dataset_name": "AMOTD",
  3.	  	"dataset_path": <your downloaded omotd folder>,
      ...
  4.		"phase": "train",    
      ...
  64. 	"train": {    
  65.    		"resume": null,
  66.    		"batch_size": 8 <you can change it according to your gpu capability>,
      ...
  71.			"log_save_folder": <your log save folder>,
  72.		    "image_save_folder": <your image save folder>,
  73.		    "weights_save_folder": <your weights save folder>,    
      ...    
  ```

- Activate your python environment, and run

  ```shell
  cd <project folder>
  python train_amot.py
  ```

## UA-DETRAC

- Download the training and testing dataset from [[UA-DETRAC Official Site]](https://detrac-db.rit.albany.edu/) or [[baidu]](https://pan.baidu.com/s/1K1-o_gNKgBv4LKUWC3N_Mg)

### Test

- Download the network weights file([[dropbox]](https://www.dropbox.com/s/25ao070zfx8jq9o/ssdt193895.pth?dl=0), [[baidu]](https://pan.baidu.com/s/1Phq1FR2LpaE0UJ8FxqHmsQ))

- Modify the `<project path>/config/__init__.py` to

  ```python
  15. configure_name = 'config_gpu4_ua.json'
  16. # configure_name = 'config_gpu4_ua_test.json'
  17. # configure_name = 'config_gpu4_ua_with_amot.json'
  18. # configure_name = 'config_gpu4_amot.json'
  ```

- Modify the `<project path>/config/config_gpu4_ua.json`

  ```python
  1. {
  2.   "dataset_name": "UA-DETRAC",
  3.   "dataset_path": <the UA-DETRAC dataset folder>,
  4.   "phase": "test",
      ...
  36.  "test": {
  37.    "resume": <network weights file>,
  38.    "dataset_type": "test",    
      ...
  42.    "base_net_weights": null,
  43.    "log_save_folder": <your log save folder>,
  44.    "image_save_folder": <your image save folder>,
  45.    "weights_save_folder": <your network weights save folder>,    
  ```

- Activate your python environment, and run

  ```shell
  cd <project folder>
  python test_tracker_ua.py
  ```

### Train

- Modify the `<project path>/config/__init__.py` to

  ```python
  15. configure_name = 'config_gpu4_ua.json'
  16. # configure_name = 'config_gpu4_ua_test.json'
  17. # configure_name = 'config_gpu4_ua_with_amot.json'
  18. # configure_name = 'config_gpu4_amot.json'
  ```

- Modify the `<project path>/config/config_gpu4_ua.json`

  ```python
  1. {
  2.   "dataset_name": "UA-DETRAC",
  3.   "dataset_path": <the UA-DETRAC dataset folder>,
  4.   "phase": "train", 
      ...
  81.		"log_save_folder": <your log save folder>,
  82.     "image_save_folder": <your image save folder>,
  83.     "weights_save_folder": <your network weights save folder>,
  ```

- activate your python environment, and run

  ```shell
  cd <project folder>
  python train_ua.py
  ```

## Citation

```
@inproceedings{ShiJie20,
  author = {Shijie Sun, Naveed Aktar, XiangYu Song, Huansheng Song, Ajmal Mian, Mubarak Shah},
  title = {Simultaneous Detection and Tracking with Motion Modelling for Multiple Object Tracking},
  booktitle = {Proceedings of the European conference on computer vision (ECCV)}},
  year = {2020}
  ```

## Acknowledge

This work is based on the [Pytroch](https://github.com/pytorch/pytorch) and [3D ResNet](https://github.com/kenshohara/video-classification-3d-cnn-pytorch). It also inspired by [SSD](https://github.com/amdegroot/ssd.pytorch) and [DAN](https://www.researchgate.net/publication/334508412_Deep_Affinity_Network_for_Multiple_Object_Tracking).

## License

The methods provided on this page are published under the [Creative Commons Attribution-NonCommercial-ShareAlike 3.0 License](http://creativecommons.org/licenses/by-nc-sa/3.0/) . This means that you must attribute the work in the manner specified by the authors, you may not use this work for commercial purposes and if you alter, transform, or build upon this work, you may distribute the resulting work only under the same license. If you are interested in commercial usage you can contact us for further options.

<!--
## Issues

|   Symbol  | Meanings   |
| :-------: | :--------: |
| :hourglass_flowing_sand:      | Plan to solve         |
| :repeat:                      | try to solve it again |
| :no_entry:                    | abandoned issue       |
| :ballot_box_with_check:       | solved                |
| :black_square_button:         | unsolved              |
| :negative_squared_cross_mark: | cannot get solved     |
| :boom:                        | focusing              |
| :exclamation:                 | important             |

|   SartDate|                            Content                            | State |
| :------:  | :----------------------------------------------------------: | :---: |
| 2019/05/18 | ![1558137403985](images/progress/1558137403985.png) ![1558137434625](images/progress/1558137434625.png)![1558137448039](images/progress/1558137448039.png) | :ballot_box_with_check: Cannot detect static vehicle (because of the training dataset)|
| 2019/05/18 | ![1558137352644](images/progress/1558137352644.png) | :ballot_box_with_check: cannot detect bus （because of the training dataset) |
| 2019/05/18 | ![1558137469303](images/progress/1558137469303.png) | :ballot_box_with_check: Limit on the detection regions |
| 2019/05/18 | ![1558137499772](images/progress/1558137499772.png) | :ballot_box_with_check: Something overlapped in the bus |
| 2019/05/18 | ![1558137539934](images/progress/1558137539934.png)![1558137545991](images/progress/1558137545991.png) | :boom: totally different scene |
| 2019/05/18 | ![1558137571322](images/progress/1558137571322.png) | :ballot_box_with_check: Some weird boxes (because of the visibility)|
| 2019/05/18 | ![1558137594908](images/progress/1558137594908.png) | :ballot_box_with_check: totally different vehicle |
| 2019/05/18 | ![1558137620722](images/progress/1558137620722.png) | :ballot_box_with_check: Wrongly located boxes |
| 2019/05/15 | ![1557913738424](./images/progress/weird_rectangles3.png) | :ballot_box_with_check: solved by change the loss according the first exist box <br />:boom: Weird rectangles and incorrect classifications |
| 2019/05/07 | None-filling rectangles <br />![](./images/progress/none_filling1.png) | :ballot_box_with_check: waiting <br />:boom:2019/05/07 process |
| 2019/05/07 | Weird rectangles without label <br />![](./images/progress/weird_rectangles1.png)![](./images/progress/weird_rectangles2.png) | :ballot_box_with_check: find the reason <br />:boom:2019/05/07 process |
| 2019/05/07 | Lost some objects in other scene <br />![1557188487001](./images/progress/lost_objects3.png) ![](./images/progress/lost_objects4.png) | :ballot_box_with_check:finish by reconfigure the anchor boxes<br />:boom:2019/05/07 process |
| 2019/04/26 | Try MOT 17 | :ballot_box_with_check: Need to do (Finish) |
| 2019/04/26 | Train A-MOT Dataset | :ballot_box_with_check: Need to do (Finish) |
| 2019/04/26 | Train UA-DETRAC | :ballot_box_with_check: Training (Finish)|
| 2019/04/16 | Clean this project | :boom: Ready to do<br>:hourglass_flowing_sand: |
| 2019/04/16 | Overlap ratio of **Tunnel Anchor** too small | :ballot_box_with_check: Find the best overlap ratio​<br>:boom:<br>:hourglass_flowing_sand: |
| 2019/04/16 | Add **Random Mirror** Preprocessing | :no_entry:<br>:hourglass_flowing_sand: |
| 2019/04/16 | Add **Random Crop** Preprocessing | :no_entry:<br>:hourglass_flowing_sand: |
| 2019/04/16 | Needs Testing The Network | :ballot_box_with_check:Thoroughly testing see [result video](<https://www.dropbox.com/s/m63g9jotgs35xu5/1.avi?dl=0>)<br>:boom: 2019/04/16 processing |
| 2019/04/14 | Motion Model Needs Rewrite | :exclamation::exclamation::ballot_box_with_check: 2019/04/16 Rewriting motion model :)​<br>:boom: 2019/04/14 rewriting |
| 2019/04/13  | Lost some objects<br/> ![](./images/progress/lost_objects1.png)![](./images/progress/lost_objects2.png)<br> |  :ballot_box_with_check: set confidence and existing threshold<br> :boom:20​19/04/13 process  |
| 2019/04/13  | NMS doesn't work well <br>![](./images/progress/nms_doesnt_work_well1.png) ![](./images/progress/nms_doesnt_work_well.png)<br> | :ballot_box_with_check: ​the bad training data<br>2019/04/13 :boom: |
| 2019/04/13  | Problems of object at the edge of the frames. <br> ![](./images/progress/object_at_frame_edge.png)![](./images/progress/object_at_frame_edge1.png) |   :ballot_box_with_check: ​remove edging boxes from training data<br>  2019/04/13 :boom:   |
| 2019/04/13  | Weird detected objects.<br> ![](./images/progress/werid_detect_object.png)![](./images/progress/werid_detect_object1.png) |   :ballot_box_with_check: ​the motion model <br>2019/04/13 :boom:   |

> In our experiment, we find the missing bounding box is caused by the following code:
>
> ```python
> conf[mean_best_truth_overlap < threshold] = 0  # label as background
> ```
>
> Be careful to set this threshold.
-->
