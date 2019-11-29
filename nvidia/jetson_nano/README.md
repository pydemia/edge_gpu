# Jetson Nano

## Set-up

### Boot-up

* Get an Image
* First Boot

### Network Config

### System Basic Setting

```sh
sudo apt update
sudo apt install vim htop -y
```

### Docker Setting

```sh
sudo apt-get remove docker docker-engine docker.io containerd runc -y
sudo apt-get update
sudo apt-get install \
    apt-transport-https \
    ca-certificates \
    curl \
    gnupg-agent \
    software-properties-common -y

curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -

sudo apt-key fingerprint 0EBFCD88

#sudo add-apt-repository \
#   "deb [arch=amd64] https://download.docker.com/linux/ubuntu \
#   $(lsb_release -cs) \
#   stable"

sudo add-apt-repository \
   "deb [arch=arm64] https://download.docker.com/linux/ubuntu \
   $(lsb_release -cs) \
   stable"

sudo apt update

sudo apt install docker-ce docker-ce-cli containerd.io -y

```

* Disable GUI

```sh
sudo systemctl set-default multi-user.target
#Removed /etc/systemd/system/default.target.
#Created symlink /etc/systemd/system/default.target → /lib/systemd/system/multi-user.target.

```

* High-Power mode(10W)
```sh
sudo nvpmodel -m 0
```

* Disable SWAP Memory (it can causes issues on k8s)
```sh
sudo swapoff -a
```

* Set the NVidia runtime as a default runtime in Docker. For this edit /etc/docker/daemon.json file, so it looks like this:

```sh
sudo vim /etc/docker/daemon.json
```

```json
{
  “default-runtime”: “nvidia”,
  “runtimes”: {
    “nvidia”: {
      “path”: “nvidia-container-runtime”,
      “runtimeArgs”: []
     }
   }
}
```

> Among the settings presented here, the one above is very, very, very crucial.
> You will experience a lot of issues if you fail to set this correctly.
> For details on why this is needed see for example here:
> https://github.com/NVIDIA/nvidia-docker/wiki/Advanced-topics#default-runtime.
> By changing the default runtime,
> you are sure that every Docker command and every Docker-based tool will be allowed to access the GPU.


* Update all packages you have
```sh
sudo apt-get update
sudo apt-get dist-upgrade
```

* Set a group `docker`   
Add current user to docker group to use docker command without sudo, following this guide: https://docs.docker.com/install/linux/linux-postinstall/. The required commands are following:
```sh
sudo groupadd docker
sudo usermod -aG docker $USER
newgrp docker
```

* Reboot
```sh
sudo shutdown -r now
```

* Set NVIDIA

```sh
sudo vim /etc/bash.bashrc
```

```sh
export PATH="${PATH}:/usr/local/cuda/bin"
export LD_LIBRARY_PATH="${LD_LIBRARY_PATH}:/usr/local/cuda/lib64"

```

```sh
sudo dpkg --get-selections | grep nvidia
sudo docker info | grep nvidia
```


* Building CUDA in Containers on Jetson

```sh
cd ~
mkdir /tmp/docker-build && cd /tmp/docker-build
cp -r /usr/local/cuda/samples/ ./
tee ./Dockerfile <<EOF
FROM nvcr.io/nvidia/l4t-base:r32.2

RUN apt-get update && apt-get install -y --no-install-recommends make g++
COPY ./samples /tmp/samples

WORKDIR /tmp/samples/1_Utilities/deviceQuery
RUN make clean && make

CMD ["./deviceQuery"]
EOF

sudo docker build -t devicequery .
sudo docker run -it --runtime nvidia devicequery
```

* Tag it
```sh
docker tag devicequery:latest pydemia/nvidia-jetson-nano:latest
docker push pydemia/nvidia-jetson-nano

```

* Run it

```sh
docker run -it --runtime nvidia pydemia/nvidia-jetson-nano

---------------------------------------------------------------
./deviceQuery Starting...

 CUDA Device Query (Runtime API) version (CUDART static linking)

Detected 1 CUDA Capable device(s)

Device 0: "NVIDIA Tegra X1"
  CUDA Driver Version / Runtime Version          10.0 / 10.0
  CUDA Capability Major/Minor version number:    5.3
  Total amount of global memory:                 3964 MBytes (4156932096 bytes)
  ( 1) Multiprocessors, (128) CUDA Cores/MP:     128 CUDA Cores
  GPU Max Clock rate:                            922 MHz (0.92 GHz)
  Memory Clock rate:                             13 Mhz
  Memory Bus Width:                              64-bit
  L2 Cache Size:                                 262144 bytes
  Maximum Texture Dimension Size (x,y,z)         1D=(65536), 2D=(65536, 65536), 3D=(4096, 4096, 4096)
  Maximum Layered 1D Texture Size, (num) layers  1D=(16384), 2048 layers
  Maximum Layered 2D Texture Size, (num) layers  2D=(16384, 16384), 2048 layers
  Total amount of constant memory:               65536 bytes
  Total amount of shared memory per block:       49152 bytes
  Total number of registers available per block: 32768
  Warp size:                                     32
  Maximum number of threads per multiprocessor:  2048
  Maximum number of threads per block:           1024
  Max dimension size of a thread block (x,y,z): (1024, 1024, 64)
  Max dimension size of a grid size    (x,y,z): (2147483647, 65535, 65535)
  Maximum memory pitch:                          2147483647 bytes
  Texture alignment:                             512 bytes
  Concurrent copy and kernel execution:          Yes with 1 copy engine(s)
  Run time limit on kernels:                     Yes
  Integrated GPU sharing Host Memory:            Yes
  Support host page-locked memory mapping:       Yes
  Alignment requirement for Surfaces:            Yes
  Device has ECC support:                        Disabled
  Device supports Unified Addressing (UVA):      Yes
  Device supports Compute Preemption:            No
  Supports Cooperative Kernel Launch:            No
  Supports MultiDevice Co-op Kernel Launch:      No
  Device PCI Domain ID / Bus ID / location ID:   0 / 0 / 0
  Compute Mode:
     < Default (multiple host threads can use ::cudaSetDevice() with device simultaneously) >

deviceQuery, CUDA Driver = CUDART, CUDA Driver Version = 10.0, CUDA Runtime Version = 10.0, NumDevs = 1
Result = PASS
---------------------------------------------------------------

```

### Test Docker GPU support


* Test a Docker image
we are ready to test if Docker runs correctly and supports GPU.
To make this easier, we created a dedicated Docker image with “deviceQuery” tool from the CUDA SDK which is used to query the GPU and present its capabilities. The command to run it is simple:
```diff
-docker run -it jitteam/devicequery ./deviceQuery
+docker run -it pydemia/nvidia-jetson-nano ./deviceQuery
```

Then, The output is the following:
```txt

```


### Kubernetes Setting

```sh


##

sudo apt-get update && sudo apt-get install -y apt-transport-https curl
curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add -
cat <<EOF | sudo tee /etc/apt/sources.list.d/kubernetes.list
deb https://apt.kubernetes.io/ kubernetes-xenial main
EOF
sudo apt-get update
sudo apt-get install -y kubelet kubeadm kubectl
sudo apt-mark hold kubelet kubeadm kubectl

```
