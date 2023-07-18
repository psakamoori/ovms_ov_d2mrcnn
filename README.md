# 1. OVMS Buidling from source

## Prerequisites

1. [Docker Engine](https://docs.docker.com/engine/)
1. Ubuntu 20.04, Ubuntu 22.04 or RedHat 8.7 host
1. make
1. bash

## Makefile and building

Makefile located in root directory of this repository contains all targets needed to build docker images and binary packages.

It contains `docker_build` target which by default builds multiple docker images:
- `openvino/model_server:latest` - smallest release image containing only neccessary files to run model server on CPU
- `openvino/model_server:latest-gpu` - release image containing support for Intel GPU and CPU
- `openvino/model_server:latest-nginx-mtls` - release image containing examplary NGINX MTLS configuration
- `openvino/model_server-build:latest` - image with builder environment containing all the tools to build OVMS

The `docker_build` target also prepares binary package to run OVMS as standalone application and shared library to link against user written C/C++ applications.

```bash
git clone https://github.com/openvinotoolkit/model_server
cd model_server
```
## Building Options with OpenVINO engineering commit/branch:

```bash
make docker_build OV_USE_BINARY=0 OV_SOURCE_BRANCH="63b18adf68e44c2d1759d25e5d1c0ea6ebe78844" OV_SOURCE_ORG=openvinotoolkit RUN_TESTS=0 CHECK_COVERAGE=0 MEDIAPIPE_DISABLE=0 JOBS=2
````
NOTE: Number of compilation jobs. By default it is set to the number of CPU cores. On hosts with low RAM, this value can be reduced to avoid out of memory errors during the compilation.

## Build Client Docker Image:

To build the docker image and tag it as benchmark_client run:

```
cd model_server/demos/benchmark/python
docker build . -t benchmark_client
```

# 2. Model Prepearation for OVMS serving:

## Download Mask R-CNN and unzip below folder to <user choosen path> (for ex: "~\home\psakamoori\")
[Detectron2 Mask R-CNN OpenVINO IR model files](https://drive.google.com/file/d/1c-g_aY9pCUjaR5cS6uLXav4O4ysSb8z3/view?usp=sharing)

Below is models folder structure

```
models
└── d2mrcnn
  └── 1
     ├── mask_rcnn_R_50_FPN_3x.bin
     └── mask_rcnn_R_50_FPN_3x.xml
```

# 3. Performance benchmarking

## Running OVMS model_server

Docker image with Support for Intel CPU and GPU.

cd {to models directory root path} (ex: "~\home\psakamoori\")

```
docker run -u $(id -u) -v $(pwd)/models:/models -p 9000:9000 -p 9001:9001 --device /dev/dri openvino/model_server-gpu:latest --model_name d2mrcnn --model_path /models/d2mrcnn --port 9001 --rest_port 9000 --target_device GPU.1 --plugin_config "{\"NUM_STREAMS\":\"4\"}" --nireq 8
```
Note: Above command will add Intel GPU device support with "--device /dev/dri" device launch and using dGPU (--target_device GPU.1 ) as inference acceleartor.
Without these two options, Intel CPU will be default inference device.

## Running benchmark_client:

```
docker run --network host benchmark_client -a localhost -r 9000 -m d2mrcnn -p 9001 -t 20 -b 3 --print_all
```
Throughput performance can be read from outputlogs : Ex: window_brutto_frame_rate: 33.119598421801705
