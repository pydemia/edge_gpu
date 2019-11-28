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


### Test Docker GPU support

* Test a Docker image
we are ready to test if Docker runs correctly and supports GPU.
To make this easier, we created a dedicated Docker image with “deviceQuery” tool from the CUDA SDK which is used to query the GPU and present its capabilities. The command to run it is simple:
```sh
docker run -it jitteam/devicequery ./deviceQuery
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
