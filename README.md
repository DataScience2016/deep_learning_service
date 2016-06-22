# Bitfusion Mobile Deep Learning Service

This AMI is designed to run on the following GPU enabled EC2 instances:

* g2.2xlarge
* g2.8xlarge

## GPU REST ENGINE API

This AMI leverages nvidia GPU rest engine: https://github.com/NVIDIA/gpu-rest-engine

GPUs are efficient parallel processors that can deliver low-latency response times for services responding to arbitrary incoming requests.

The NVIDIA GPU REST Engine (GRE) is a critical component for developers building low-latency web services. GRE includes a multi-threaded HTTP server that presents a RESTful web service and schedules requests efficiently across multiple NVIDIA GPUs. The overall response time depends on how much processing you need to do, but GRE itself adds very little overhead and can process null-requests in as little as 10 microseconds.


## Example Curl Call

Currently the inference api only supports JPG images

```
host="EC2 instance IP"
curl \
   --verbose \
  -X POST \
   --data-binary @images/1.jpg \
   http://${host}:8000/api/classify
```

Several trained example model are provided on the system amongst which you can select:
 * CaffeNet
 * AlextNet
 * GoogleNet
 
You can use this AMI to train your own Caffe model as well. If you have trained your own model simply update the variables below to point to your model collateral.

The GPU Rest API starts on boot and can be configured using the provided "/etc/default/gpurestengine" script:


```
# Caffe Model - Default
DEPLOY_PROTEXT="caffe/models/bvlc_reference_caffenet/deploy.prototxt"
CAFFE_MODEL="caffe/models/bvlc_reference_caffenet/bvlc_reference_caffenet.caffemodel" \
MEAN="caffe/data/ilsvrc12/imagenet_mean.binaryproto" \
SYNSET_WORDS="caffe/data/ilsvrc12/synset_words.txt"

# Alexnet Model
#DEPLOY_PROTEXT="caffe/models/bvlc_alexnet/deploy.prototxt"
#CAFFE_MODEL="caffe/models/bvlc_alexnet/bvlc_alexnet.caffemodel"
#MEAN="caffe/data/ilsvrc12/imagenet_mean.binaryproto"
#SYNSET_WORDS="caffe/data/ilsvrc12/synset_words.txt"

# Googlenet Model
#DEPLOY_PROTEXT="caffe/models/bvlc_googlenet/deploy.prototxt" \
#CAFFE_MODEL="caffe/models/bvlc_googlenet/bvlc_googlenet.caffemodel" \
#MEAN="caffe/data/ilsvrc12/imagenet_mean.binaryproto" \
#SYNSET_WORDS="caffe/data/ilsvrc12/synset_words.txt"
```

### 

```
sudo service gpurestengine start
```

## Starting the GPU Rest Engine manually via init script:

```
sudo service gpurestengine start
```


## Starting the GPU Rest Engine manually 

This would start the server with the bvlc reference caffe model trained with ilsvrc12 dataset. Server listens on port 8000

```
nvidia-docker run --name=gpurestengine --net=host --rm \
  bitfusion/gpurestengine \
    inference \
      "caffe/models/bvlc_reference_caffenet/deploy.prototxt" \
      "caffe/models/bvlc_reference_caffenet/bvlc_reference_caffenet.caffemodel" \
      "caffe/data/ilsvrc12/imagenet_mean.binaryproto" \
      "caffe/data/ilsvrc12/synset_words.txt"
```

This would start the server with the bvlc alexnet model trained with ilsvrc12 dataset. Server listens on port 8000

```
nvidia-docker run --name=gpurestengine --net=host --rm \
  bitfusion/gpurestengine \
    inference \
      "caffe/models/bvlc_alexnet/deploy.prototxt" \
      "caffe/models/bvlc_alexnet/bvlc_alexnet.caffemodel" \
      "caffe/data/ilsvrc12/imagenet_mean.binaryproto" \
      "caffe/data/ilsvrc12/synset_words.txt"
```

This would start the server with the googlenet alexnet model trained with ilsvrc12 dataset. Server listens on port 8000
```
nvidia-docker run --name=gpurestengine --net=host --rm \
  bitfusion/gpurestengine \
    inference \
      "caffe/models/bvlc_googlenet/deploy.prototxt" \
      "caffe/models/bvlc_googlenet/bvlc_googlenet.caffemodel" \
      "caffe/data/ilsvrc12/imagenet_mean.binaryproto" \
      "caffe/data/ilsvrc12/synset_words.txt"

```

## Working with docker containers

To view running containers or docker processes

```
$ docker ps -a

CONTAINER ID        IMAGE                     COMMAND                  CREATED             STATUS              PORTS               NAMES
4378dd2a49ae        bitfusion/gpurestengine   "inference caffe/mode"   7 minutes ago       Up 7 minutes                            gpurestengine

```

Connecting, making changes and saving your container

```
# 1. Connect
$ docker exec -i -t gpurestengine /bin/bash

# 2. Let's install vim and download a data set
root@ip-172-31-67-58:/#  apt-get update
root@ip-172-31-67-58:/#  apt-get install vim
root@ip-172-31-67-58:/#  wget http://vis-www.cs.umass.edu/lfw/lfw-deepfunneled.tgz
root@ip-172-31-67-58:/#  tar zxvf lfw-deepfunneled.tgz
root@ip-172-31-67-58:/#  exit

# 3 Commit your changes
$ docker commit 4378dd2a49ae bitfusion/gpurestengine
```

To verify your changes persisted, you can stop the container and start it back up with the following:

```
$ sudo service gpurestengine stop
$ sudo service gpurestengine start
$ docker exec -i -t gpurestengine /bin/bash

# You are now in the container.  Let's check if the data we downloaded is still there.
root@ip-172-31-67-58:/#  ls -lah | grep lfw

drwxr-xr-x 5751 root root 204K Jun 22 21:59 lfw-deepfunneled
-rw-r--r--    1 root root 104M Aug 23  2013 lfw-deepfunneled.tgz

root@ip-172-31-67-58:/# exit
```


### Training your own model with imagenet

To train your own model a good place to start is the following tutorial for training a model with imagenet: http://caffe.berkeleyvision.org/gathered/examples/imagenet.html


#### Training your own model with your own dataset:

To train on your own dataset you can follow the steps outlined here: https://github.com/BVLC/caffe/issues/550


## Current Issues

After restarting the AMI you may see this, note the *STATUS*

```
CONTAINER ID        IMAGE                     COMMAND                  CREATED             STATUS                       PORTS               NAMES
029217228cc3        bitfusion/gpurestengine   "inference caffe/mode"   16 minutes ago      Exited (137) 8 minutes ago                       gpurestengine
```

To fix it do the following:

```
$ docker rm { container ID }
$ sudo service gpurestengine stop
$ sudo service gpurestengine start
```

Example
```
$ docker rm 029217228cc3
$ sudo service gpurestengine stop
$ sudo service gpurestengine start
$ docker ps -a

CONTAINER ID        IMAGE                     COMMAND                  CREATED             STATUS              PORTS               NAMES
e538fd791c90        bitfusion/gpurestengine   "inference caffe/mode"   15 seconds ago      Up 14 seconds                           gpurestengine



```

## Support

Contact us a support@bitfusion.io

