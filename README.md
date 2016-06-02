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

The GPU Rest API start on boot and can be configured using the provided "/etc/default/gpurestengine" script:


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
  863665633681.dkr.ecr.us-east-1.amazonaws.com/bitfusion/infratech:gpurestengine \
    inference \
      "caffe/models/bvlc_reference_caffenet/deploy.prototxt" \
      "caffe/models/bvlc_reference_caffenet/bvlc_reference_caffenet.caffemodel" \
      "caffe/data/ilsvrc12/imagenet_mean.binaryproto" \
      "caffe/data/ilsvrc12/synset_words.txt"
```

This would start the server with the bvlc alexnet model trained with ilsvrc12 dataset. Server listens on port 8000

```
nvidia-docker run --name=gpurestengine --net=host --rm \
  863665633681.dkr.ecr.us-east-1.amazonaws.com/bitfusion/infratech:gpurestengine \
    inference \
      "caffe/models/bvlc_alexnet/deploy.prototxt" \
      "caffe/models/bvlc_alexnet/bvlc_alexnet.caffemodel" \
      "caffe/data/ilsvrc12/imagenet_mean.binaryproto" \
      "caffe/data/ilsvrc12/synset_words.txt"
```

This would start the server with the googlenet alexnet model trained with ilsvrc12 dataset. Server listens on port 8000
```
nvidia-docker run --name=gpurestengine --net=host --rm \
  863665633681.dkr.ecr.us-east-1.amazonaws.com/bitfusion/infratech:gpurestengine \
    inference \
      "caffe/models/bvlc_googlenet/deploy.prototxt" \
      "caffe/models/bvlc_googlenet/bvlc_googlenet.caffemodel" \
      "caffe/data/ilsvrc12/imagenet_mean.binaryproto" \
      "caffe/data/ilsvrc12/synset_words.txt"

```

## Accessing the container & creating a model

To build your own model you need to access the container after you SSH into the EC2 instance. You can do this by running:

```
docker run --interactive --rm -t bitfusion/gpurestengine /bin/bash
```

### Training your own model with imagenet

To train your own model a good place to start is the following tutorial for training a model with imagenet: http://caffe.berkeleyvision.org/gathered/examples/imagenet.html


#### Training your own model with your own dataset:

To train on your own dataset you can follow the steps outlined here: https://github.com/BVLC/caffe/issues/550

## Support

Contact us a support@bitfusion.io

