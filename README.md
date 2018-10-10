# Unsupervised Domain Adaptation for Semantic Segmentation via Class-Balanced Self-Training

By Yang Zou*, Zhiding Yu*, Vijayakumar Bhagavatula, Jinsong Wang (* indicates equal contribution).

### Update
- **2018.10.9**: code release for GTA-5 to Cityscapes and SYNTHIA to Cityscapes

### Contents
0. [Introduction](#introduction)
0. [Citation](#citation)
0. [Requirements](#requirements)
0. [Setup](#models)
0. [Usage](#usage)
0. [Results](#results)
0. [Note](#note)

### Introduction

This code heavily borrow [ResNet-38](https://github.com/itijyou/ademxapp)

### Requirements:
The code is tested in Ubuntu 16.04. It is based on [MXNet 1.3.0](https://mxnet.apache.org/install/index.html?platform=Linux&language=Python&processor=GPU) and Python 2.7.12. We use TiTan Xp. The maximum GPU memory consumption is about 7GB.

### Citation
If you finds this method or code useful, please cite:
> @InProceedings{Zou_2018_ECCV,
author = {Zou, Yang and Yu, Zhiding and Vijaya Kumar, B.V.K. and Wang, Jinsong},
title = {Unsupervised Domain Adaptation for Semantic Segmentation via Class-Balanced Self-Training},
booktitle = {The European Conference on Computer Vision (ECCV)},
month = {September},
year = {2018}
}

### Results:
1. GTA2city:

	Case|mIoU|Road|SW|Build|Wall|Fence|Pole|Traffic Light|Traffic Sign|Veg.|Terrain|Sky|Person|Rider|Car|Truck|Bus|Train|Motor|Bike
	---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---
	Source|35.4|70.0|23.7|67.8|15.4|18.1|40.2|41.9|25.3|78.8|11.7|31.4|62.9|29.8|60.1|21.5|26.8|7.7|28.1|12.0
	Adapted|46.2|88.0|56.2|77.0|27.4|22.4|40.7|47.3|40.9|82.4|21.6|60.3|50.2|20.4|83.8|35.0|51.0|15.2|20.6|37.0

2. SYNTHIA2City:

### Setup
We assume you are working in cbst-master folder.
1. Datasets:

- Download [GTA-5](https://download.visinf.tu-darmstadt.de/data/from_games/) dataset. Since GTA-5 contains images with different resolutions, we recommend resize all images to 1052x1914. 

- Download [Cityscapes](https://www.cityscapes-dataset.com/).

- Download [SYNTHIA-RAND-CITYSCAPES](http://synthia-dataset.net/download/808/).

- Put downloaded data in "data" folder.

2. Source pretrained models:
- Download [source model](https://www.dropbox.com/s/idnnk398hf6u3x9/gta_rna-a1_cls19_s8_ep-0000.params?dl=0) trained in GTA-5.

- Download [source model](https://www.dropbox.com/s/l6oxhxhovn2l38p/synthia_rna-a1_cls16_s8_ep-0000.params?dl=0) trained in SYNTHIA.

- Put source trained models in "models/" folder

3. Spatial priors 

- Download [Spatial priors](https://www.dropbox.com/s/o6xac8r3z30huxs/prior_array.mat?dl=0) from GTA-5. Spatial priors are only used in GTA2Cityscapes. Put the prior_array.mat in "spatial_prior/gta/" folder.

### Usage

Set the PYTHONPATH environment variable:
~~~~
cd cbst-master
export PYTHONPATH=PYTHONPATH:./
~~~~

SYNTHIA2City:
CBST:
~~~~
python issegm/script-self-paced-self-trained-segresnet.py --num-round 12 --test-scales 1850 --scale-rate-range 0.7,1.3 --dataset synthia --dataset-tgt cityscapes --split train --split-tgt val --data-root /home/datasets/RAND_CITYSCAPES --data-root-tgt /home/datasets/cityscapes/data_original --output spst_syn2city/cbst_eccv_min0-8 --model cityscapes_rna-a1_cls16_s8 --weights models/synthia_rna-a1_cls16_s8_ep-0000.params --batch-images 2 --crop-size 500 --origin-size 1280 --origin-size-tgt 2048 --init-tgt-port 0.2 --init-src-port 0.02 --max-src-port 0.06 --seed-int 0 --mine-port 0.8 --mine-id-number 3 --mine-thresh 0.001 --base-lr 1e-4 --to-epoch 2 --source-sample-policy cumulative --self-training-script issegm/self-paced-self-trained-segresnet-public-v1.py --kc-policy cb --prefetch-threads 2 --gpus 2 --with-prior False
~~~~

GTA2Cityscapes:
CBST-SP:
~~~~
python issegm/script-self-paced-self-trained-segresnet.py --num-round 5 --test-scales 1850 --scale-rate-range 0.7,1.3 --dataset gta --dataset-tgt cityscapes --split train --split-tgt val --data-root DATA_ROOT_GTA5 --data-root-tgt DATA_ROOT_CITYSCAPES --output spst_gta2city/cbst-sp --model cityscapes_rna-a1_cls19_s8 --weights models/gta_rna-a1_cls19_s8_ep-0000.params --batch-images 2 --crop-size 500 --origin-size-tgt 2048 --init-tgt-port 0.15 --init-src-port 0.03 --seed-int 0 --mine-port 0.8 --mine-id-number 3 --mine-thresh 0.001 --base-lr 1e-4 --to-epoch 2 --source-sample-policy cumulative --self-training-script issegm/self-paced-self-trained-segresnet.py --kc-policy cb --prefetch-threads 2 --gpus 0 --with-prior True
~~~~

For CBST, set "--kc-policy cb" and "--with-prior False". For ST, set "--kc-policy global" and "--with-prior False".
we use a small class patch mining strategy to mine the patches including small classes. To turn off small class mining, set "--mine-port 0.0".  

#evaluate
Test in Cityscapes (Initial source trained model as example)
~~~~
python issegm/evaluate-segresnet.py --data-root DATA_ROOT_CITYSCAPES --output val/cityscapes --dataset cityscapes --phase val --weights models/gta_rna-a1_cls19_s8_ep-0000.params --split val --test-scales 1850 --test-flipping --gpus 0 --no-cudnn
~~~~
Test in GTA-5
~~~~
python issegm/evaluate-segresnet.py --data-root DATA_ROOT_GTA --output val/gta --dataset gta --phase val --weights ../BAK/models/gta_rna-a1_cls19_s8_ep-0000.params --split train --test-scales 1850 --test-flipping --gpus 3 --no-cudnn
~~~~
Test in SYNTHIA
~~~~

~~~~

Contact: yzou2@andrew.cmu.edu

The model and code are available for non-commercial research purposes only.
