# Vision-based Transfer Learning - Inference

The workflow is to perform transfer learning on images in order to accomplish different classification tasks that range from binary classification to multiclass classification, giving the best possible performance on Intel Hardware utilizing the available optimizations. This is the inference part after transfer learning for different classification tasks.

Check out more workflow examples and reference implementations in the [Developer Catalog](https://developer.intel.com/aireferenceimplementations).

## Overview

This workflow demonstrates how users can run a reference end-to-end pipeline for transfer learning with Docker container. For more detailed information, please visit the [Intel® vision based transfer learning workflow](https://github.com/intel/vision-based-transfer-learning-and-inference.git) GitHub repository.


## Hardware Requirements

The hardware below is recommended for use with this reference implementation.   


| Recommended Hardware         | Precision  |
| ---------------------------- | ---------- |
| Intel® 1st, 2nd, 3rd, and 4th Gen Xeon® Scalable Performance processors| FP32/BF16 |

### Operating Systems

| Name | Version | 
| ------ | ------ |
| RHEL | 8.2 or higher |
| CentOS | 8.2 or higher |
| Ubuntu | 18.04<br>20.04 |

### Software Dependencies

| Name | Version | 
| ------ | ------ |
| numactl | N/A |
| scikit-learn | 1.1.2 |
| tensorflow-datasets | 4.6.0 |
| tensorflow-hub | 0.12.0|
| tensorflow | 2.9.0 |
| numpy | 1.23.2 |
| matplotlib | 3.5.2 |
|tensorflow | 2.10.0|


## How it Works

The pipeline showcases how transfer learning enabled by Intel optimized TensorFlow could be used for image classification in three domains: sports , medical imaging, and remote sensing. The workflow showcases AMX(Advanced Matrix Extensions)  BF16 in SPR（Sapphire Rapids） which speeds up the training time significantly, without loss in accuracy.

Please infer [Download the dataset](#download-the-datasets) to get information about the medical imaging dataset and remote sensing dataset. The sports dataset should be already in the project repo.

While the following diagram shows the architecture for both training and inference, this specific workflow is focused on the inference portion. You must run the [Intel® transfer learning workflow - Training](https://github.com/intel/ai-workflows/blob/main/transfer_learning/tensorflow/resnet50/training/DEVCATALOG.md) workflow first or provide your own trained model.

<br><img src="https://user-images.githubusercontent.com/52259352/202562899-d2867491-f08b-4393-be27-d8db28931bd6.png"><br>
<br><img src="https://user-images.githubusercontent.com/52259352/202562891-5b065c21-9ea5-427d-b555-8cc3419c8a39.png"><br>

## Get Started

### Download the Workflow Repository
Clone [Intel® vision based transfer learning workflow](https://github.com/intel/vision-based-transfer-learning-and-inference.git) repository.

```
git clone https://github.com/intel/vision-based-transfer-learning-and-inference.git
cd vision-based-transfer-learning-and-inference
git checkout v1.0.1
```

### Download the Datasets
The Medical Imaging dataset is downloaded from TensorFlow website when the code is run for the first time. The dataset used for this domain is `colorectal_histology`. More details can be found at [Tensorflow Datasets](https://www.tensorflow.org/datasets/catalog/colorectal_histology). 

The Remote Sensing dataset used for this domain is [resisc45](https://www.tensorflow.org/datasets/catalog/resisc45).  
[Download](https://onedrive.live.com/?authkey=%21AHHNaHIlzp%5FIXjs&cid=5C5E061130630A68&id=5C5E061130630A68%21107&parId=5C5E061130630A68%21112&action=locate ) the dataset and unzip the folder. To use this dataset, it should be split into `validation` and `train` directories. Use the script [resisc_dataset.py](https://github.com/intel/vision-based-transfer-learning-and-inference/blob/v1.0.1/resisc_dataset.py). Follow the example below:
```
pip install -r requirements.txt
export INPUT_DIR=<path_to_NWPU-RESISC45_dir>
export OUTPUT_DIR=<path_to_split_dataset>
python3 resisc_dataset.py --INDIR=${INPUT_DIR} --OUTDIR=${OUTPUT_DIR}
mv ${OUTPUT_DIR}/val ${OUTPUT_DIR}/validation
```

## Run Using Docker
Below setup and how-to-run sessions are for users who want to use the provided docker image.  
For bare metal environment, please go to [bare metal session](#bare-metal).

### Set Up Docker Engine
You'll need to install Docker Engine on your development system.
Note that while **Docker Engine** is free to use, **Docker Desktop** may require
you to purchase a license.  See the [Docker Engine Server installation
instructions](https://docs.docker.com/engine/install/#server) for details.


### Set Up Docker Image

Pull the provided docker image.
```
docker pull intel/ai-workflows:vision-transfer-learning-inference 
```

If your environment requires a proxy to access the internet, export your
development system's proxy settings to the docker environment:
```
export DOCKER_RUN_ENVS="-e ftp_proxy=${ftp_proxy} \
  -e FTP_PROXY=${FTP_PROXY} -e http_proxy=${http_proxy} \
  -e HTTP_PROXY=${HTTP_PROXY} -e https_proxy=${https_proxy} \
  -e HTTPS_PROXY=${HTTPS_PROXY} -e no_proxy=${no_proxy} \
  -e NO_PROXY=${NO_PROXY} -e socks_proxy=${socks_proxy} \
  -e SOCKS_PROXY=${SOCKS_PROXY}"
```

### Run Docker Image

Run the workflow using the ``docker run`` command, and you may change the following option `PLATFORM=None`, `PRECISION=FP32` and `SCRIPT=colorectal`. On Intel® 4th Gen Xeon® Scalable Performance processors with BF16 precision, it should take about two hours to run this reference implementation example using Docker or bare metal. Time will be shorter or longer depending on how many cores your system has. 

```
export CHECKPOINT_DIR=$(pwd)/output/colorectal
export DATASET_DIR=$(pwd)/data
export OUTPUT_DIR=$(pwd)/output
export PLATFORM=None
export PRECISION=FP32
export SCRIPT=colorectal
docker run \
  -a stdout $DOCKER_RUN_ENVS \
  --env DATASET_DIR=/workspace/data \
  --env OUTPUT_DIR=${OUTPUT_DIR}/${SCRIPT} \
  --env PLATFORM=${PLATFORM} \
  --env PRECISION=${PRECISION} \
  --volume /${CHECKPOINT_DIR}:/workspace/checkpoint \
  --volume /${DATASET_DIR}:/workspace/data \
  --volume ${OUTPUT_DIR}:${OUTPUT_DIR} \
  --volume $(pwd):/workspace \
  --workdir /workspace \
  --workdir /workspace \
  --privileged --init -it \
  intel/ai-workflows:vision-transfer-learning-inference \
  conda run --no-capture-output -n transfer_learning ./${SCRIPT}.sh --inference -cp "/workspace/checkpoint"
```



## Run Using Bare Metal
Below setup and how-to-run sessions are for users who want to use bare metal environment.  
For docker environment, please go to [docker session](#docker).

### Set Up System Software
Our examples use the ``conda`` package and enviroment on your local computer.
If you don't already have ``conda`` installed, see the [Conda Linux installation
instructions](https://docs.conda.io/projects/conda/en/stable/user-guide/install/linux.html).

Install conda following the steps.
```
wget https://repo.anaconda.com/miniconda/Miniconda3-latest-Linux-x86_64.sh
sh Miniconda3-latest-Linux-x86_64.sh
```


### Set Up Workflow

Create a new conda environment.
```
conda create -n transfer_learning python=3.8 --yes
conda activate transfer_learning
```

Install TCMalloc
```
conda install -c conda-forge gperftools -y
Set conda path and LD_PRELOAD path
eg :
CONDA_PREFIX=/home/sdp/miniconda3/envs/inc/
export LD_PRELOAD="${CONDA_PREFIX}/lib/libtcmalloc.so"
```

Install Required packages
```
pip install -r requirements.txt
```


### Run Workflow

Command Line Arguments
```
--PRECISION - whether to use Mixed_Precision or FP32 precision Options : [FP32(default),Mixed Precision]"
              For Mixed Precion , BF16 is used if supported by hardware , if FP16 is supported it is chosen, if none is supported falls back to FP32
--PLATFORM - To optimize for SPR : [None(default),SPR]"
--inference - whether to run only inference"
--cp  - Specify checkpoint directory for inference"
--OUTPUT_DIR  - Specify output Directory where training checkpoints. graphs need to be saved"
--DATASET_DIR  - Specify dataset Directory; if using custom dataset please have train,val,test folders in dataset directory. 
                 If test dataset is not present validation dataset is used"
 --BATCH_SIZE - Batch Size for training[32(default)]"
 --NUM_EPOCHS  - Num epochs for training[100(default)]"

These options can also be set via export variable

ex : export OUTPUT_DIR="logs/fit/trail" 
```


1. Remote Sensing Dataset Inference run on SPR
```
    a) Inference FP32: bash resisc.sh --inference -cp "logs/fit/resiscFP32" --PLATFORM SPR --DATASET_DIR datasets/resisc45
    b) Inference BF16: bash resisc.sh --PRECISION Mixed_Precision --inference -cp "logs/fit/resiscBF16" --PLATFORM SPR --DATASET_DIR datasets/resisc45
```
   
2. Medical Imaging Dataset Inference run on SPR
```
    a) Inference FP32: bash colorectal.sh --inference -cp "logs/fit/colorectalFP32" --PRECISION FP32 --OUTPUT_DIR "logs/fit/colorectalFP32/" --DATASET_DIR datasets/colorectal --PLATFORM SPR
    b) Inference BF16: bash colorectal.sh --inference -cp "logs/fit/colorectalBF16" --PRECISION Mixed_Precision --OUTPUT_DIR "logs/fit/colorectalBF16/" --DATASET_DIR datasets/colorectal --PLATFORM SPR
```


## Expected Output

This is the expected output.

```
version information available (required by /usr/bin/bash)
inference-vision-transfer-learning-1  | INFERENCE Default value is zero
inference-vision-transfer-learning-1  | Inference option is : 1
inference-vision-transfer-learning-1  | Checkpoint File is : /workspace/checkpoint
inference-vision-transfer-learning-1  | Platform is SPR
inference-vision-transfer-learning-1  | 2022-08-25 17:59:38.778284: I tensorflow/core/platform/cpu_feature_guard.cc:193] This TensorFlow binary is optimized with oneAPI Deep Neural Network Library (oneDNN) to use the following CPU instructions in performance-critical operations:  AVX512_VNNI AVX512_BF16 AVX_VNNI AMX_TILE AMX_INT8 AMX_BF16
```
...
```
inference-vision-transfer-learning-1  | OMP: Info #255: KMP_AFFINITY: pid 22 tid 550 thread 100 bound to OS proc set 44
inference-vision-transfer-learning-1  | OMP: Info #255: KMP_AFFINITY: pid 22 tid 551 thread 101 bound to OS proc set 45
inference-vision-transfer-learning-1  | OMP: Info #255: KMP_AFFINITY: pid 22 tid 552 thread 102 bound to OS proc set 46
inference-vision-transfer-learning-1  | OMP: Info #255: KMP_AFFINITY: pid 22 tid 553 thread 103 bound to OS proc set 47
inference-vision-transfer-learning-1  | OMP: Info #255: KMP_AFFINITY: pid 22 tid 554 thread 104 bound to OS proc set 48
inference-vision-transfer-learning-1  | OMP: Info #255: KMP_AFFINITY: pid 22 tid 555 thread 105 bound to OS proc set 49
inference-vision-transfer-learning-1  | OMP: Info #255: KMP_AFFINITY: pid 22 tid 556 thread 106 bound to OS proc set 50
inference-vision-transfer-learning-1  | OMP: Info #255: KMP_AFFINITY: pid 22 tid 557 thread 107 bound to OS proc set 51
inference-vision-transfer-learning-1  | OMP: Info #255: KMP_AFFINITY: pid 22 tid 558 thread 108 bound to OS proc set 52
inference-vision-transfer-learning-1  | OMP: Info #255: KMP_AFFINITY: pid 22 tid 559 thread 109 bound to OS proc set 53
inference-vision-transfer-learning-1  | OMP: Info #255: KMP_AFFINITY: pid 22 tid 560 thread 110 bound to OS proc set 54
inference-vision-transfer-learning-1  | OMP: Info #255: KMP_AFFINITY: pid 22 tid 561 thread 111 bound to OS proc set 55
inference-vision-transfer-learning-1 exited with code 0
```



## Summary and Next Steps

In this workflow, you can choose a Docker environment or a bare metal environment and performed inference on a TensorFlow Resnet base model using Intel® Xeon® Scalable Processors. The GitHub repository also contains workflows for transfer learning training on Intel® Xeon® Scalable Processors.

## Learn More
For more information or to read about other relevant workflow
examples, see these guides and software resources:

- [Intel® AI Analytics Toolkit (AI Kit)](https://www.intel.com/content/www/us/en/developer/tools/oneapi/ai-analytics-toolkit.html)


## Troubleshooting
Issues, problems, and their workarounds if possible, will be listed here.

## Support
If you have questions or issues about this workflow, please report to [Github Issues](https://github.com/intel/vision-based-transfer-learning-and-inference/issues).
