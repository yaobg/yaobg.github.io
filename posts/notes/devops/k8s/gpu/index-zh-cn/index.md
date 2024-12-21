# k8s调度gpu教程


<!--more-->

## 环境说明

| 工具         | 版本      | 备注                       |
|------------|---------|--------------------------|
| kubesphere | V4.1    | k8s管理平台                  |
| kubernetes | v1.23.7 | k8s                      |
| helm       |         | 包管理                      |
| harbor     |         | 镜像管理                     |
| docker     |         | 镜像                       |
| nvidia     |         | 显卡                       |
| CUDA       |         | NVIDIA 推出的一种通用并行计算平台和编程模 |

## 显卡

### 安装

**NVIDIA**

查看显卡

```shell
lspci | grep NVIDIA
```

[官网](https://www.nvidia.cn/geforce/drivers/)搜索对应的显卡下载

![1.png](images%2F1.png)
![2.png](images%2F2.png)
安装成功后，执行命令 nvidia-smi

```shell
+-----------------------------------------------------------------------------------------+
| NVIDIA-SMI 550.127.05             Driver Version: 550.127.05     CUDA Version: 12.4     |
|-----------------------------------------+------------------------+----------------------+
| GPU  Name                 Persistence-M | Bus-Id          Disp.A | Volatile Uncorr. ECC |
| Fan  Temp   Perf          Pwr:Usage/Cap |           Memory-Usage | GPU-Util  Compute M. |
|                                         |                        |               MIG M. |
|=========================================+========================+======================|
|   0  NVIDIA GeForce RTX 3060        Off |   00000000:01:00.0 Off |                  N/A |
|  0%   24C    P8             10W /  170W |     223MiB /  12288MiB |      0%      Default |
|                                         |                        |                  N/A |
+-----------------------------------------+------------------------+----------------------+
                                                                                         
+-----------------------------------------------------------------------------------------+
| Processes:                                                                              |
|  GPU   GI   CI        PID   Type   Process name                              GPU Memory |
|        ID   ID                                                               Usage      |
|=========================================================================================|
|    0   N/A  N/A      1039      G   /usr/lib/xorg/Xorg                             35MiB |
|    0   N/A  N/A     36553      G   /usr/lib/xorg/Xorg                             67MiB |
|    0   N/A  N/A     36731      G   /usr/bin/gnome-shell                          104MiB |
+-----------------------------------------------------------------------------------------+
```  

**CUDA**

CUDA（Compute Unified Device Architecture） 是 NVIDIA 推出的一种通用并行计算平台和编程模型，允许开发人员使用 C、C++
等编程语言编写高性能计算应用程序，它利用 GPU 的并行计算能力解决复杂的计算问题，特别是在深度学习、科学计算、图形处理等领域。所以一般情况下，安装完
NVIDIA 驱动后，CUDA 也可以一并安装上。
在下载 NVIDIA 驱动时，每个驱动版本都对应了一个 CUDA 版本，比如上面我们在下载驱动版本,它对应的 CUDA 版本为
12.4，所以我们就按照这个版本号来安装。首先进入 CUDA Toolkit Archive 页面，这里列出了所有的 CUDA 版本：
![3.png](images/3.png)

**在 Docker 容器中使用 GPU 资源**

测试下 GPU 是否可以在容器中使用：

```shell
# docker run --gpus all --rm centos:latest nvidia-smi
docker: Error response from daemon: could not select device driver "" with capabilities: [[gpu]].
```

可以看到命令执行报错了，想在 Docker 中使用 NVIDIA GPU 还必须安装 nvidia-container-runtime 运行时。

使用 NVIDIA Container Toolkit 来安装 nvidia-container-runtime，下面以ubuntu为例(其他参考官网Installation)

1、Configure the production repository

```shell
curl -fsSL https://nvidia.github.io/libnvidia-container/gpgkey | sudo gpg --dearmor -o /usr/share/keyrings/nvidia-container-toolkit-keyring.gpg \
  && curl -s -L https://nvidia.github.io/libnvidia-container/stable/deb/nvidia-container-toolkit.list | \
    sed 's#deb https://#deb [signed-by=/usr/share/keyrings/nvidia-container-toolkit-keyring.gpg] https://#g' | \
    sudo tee /etc/apt/sources.list.d/nvidia-container-toolkit.list
```

2、Update the packages list from the repository

```shell
sudo apt-get update
```

3、Install the NVIDIA Container Toolkit packages

```shell
sudo apt-get install -y nvidia-container-toolkit
```

4、安装 NVIDIA Container Toolkit 之后，再使用下面的命令将 Docker 的运行时配置成 nvidia-container-runtime

```shell
nvidia-ctk runtime configure --runtime=docker
```

这个命令的作用是修改 /etc/docker/daemon.json 配置文件

```shell
# cat /etc/docker/daemon.json
{
    "runtimes": {
        "nvidia": {
            "args": [],
            "path": "nvidia-container-runtime"
        }
    }
}
```

5、重启docker

```shell
systemctl restart docker
```

6、查看docker内是否可以使用gpu

```shell
docker run --gpus all --rm centos:latest nvidia-smi
```

## k8s调用GPU

### gpu-operator

GPU Operator 是 NVIDIA 提供的一个 Kubernetes 操作器，旨在简化在 Kubernetes 集群中安装和管理 NVIDIA GPU 驱动程序、CUDA 和其他与
GPU 相关的软件组件。它自动化了许多与 GPU 管理相关的任务，例如：

- 安装和管理 GPU 驱动：GPU Operator 能够自动在节点上安装和升级 NVIDIA GPU 驱动，并确保它们与集群中的其他组件兼容。

- 设备插件：GPU Operator 包括 NVIDIA Device Plugin，它使得 Kubernetes 集群能够识别和调度 GPU 资源，以便容器能够访问 GPU 加速。

- NVIDIA Toolkit：它包括了必要的工具（如 CUDA 和 NVIDIA Deep Learning SDK）来为运行 GPU 加速的容器提供支持。

- GPU Operator 的自动化：包括 GPU 驱动、工具和设备插件的自动安装和配置，简化了管理过程。

**1、安装**

```shell
helm repo list

helm repo add nvidia https://helm.ngc.nvidia.com/nvidia && helm repo update

helm install -n gpu-operator --create-namespace gpu-operator nvidia/gpu-operator --set driver.enabled=false

```

安装成功后，输出

```shell
NAME: gpu-operator
LAST DEPLOYED: Thu Dec 12 12:04:40 2024
NAMESPACE: gpu-operator
STATUS: deployed
REVISION: 1
TEST SUITE: None
```

查看deployment是否正常

```shell
k get deployment -n gpu-operator
```

```shell
NAME                                         READY   UP-TO-DATE   AVAILABLE   AGE
gpu-operator                                 1/1     1            1           3h45m
gpu-operator-node-feature-discovery-gc       1/1     1            1           3h45m
gpu-operator-node-feature-discovery-master   1/1     1            1           3h45m
```

**2、测试GPU容器**

GPU Operator 正确安装完成后，使用 CUDA 基础镜像，测试 K8s 是否能正确创建使用 GPU 资源的 Pod。

- 创建资源清单文件

```shell
apiVersion: v1
kind: Pod
metadata:
  name: cuda-ubuntu2204
spec:
  restartPolicy: OnFailure
  containers:
  - name: cuda-ubuntu2204
    image: "nvcr.io/nvidia/cuda:12.4.0-base-ubuntu22.04"
    resources:
      limits:
        nvidia.com/gpu: 1
    command: ["nvidia-smi"]

```

- 创建资源

```shell
kubectl apply -f cuda-ubuntu.yaml
```

- 查看日志

```shell
kubectl logs pod/cuda-ubuntu2204
```

- 清理测试资源

```shell
kubectl apply -f cuda-ubuntu.yaml
```

### 问题

**1、 registry.k8s.io/nfd/node-feature-discovery:v0.16.6 镜像拉取不下来**

找一个可以访问外网的机器，重现打tag，推送到harbor上，替换gpu-operator里面的镜像源为harbor地址

```shell
docker tag registry.k8s.io/nfd/node-feature-discovery:v0.16.6 192.168.50.7/library/node-feature-discovery:v0.16.6
```

**2、 gpu-operator-node-feature-discovery-master无法启动**

[问题原因](https://github.com/kubernetes-sigs/node-feature-discovery/issues/1730) 如果k8s版本低于1.24，需要关闭健康检测

```shell
k edit deployment gpu-operator-node-feature-discovery-master -n gpu-operator
```

删除如下代码：

```shell
  livenessProbe:
    grpc:
      port: 8082
      service: ''
    initialDelaySeconds: 10
    timeoutSeconds: 1
    periodSeconds: 10
    successThreshold: 1
    failureThreshold: 3
  readinessProbe:
    grpc:
      port: 8082
      service: ''
    initialDelaySeconds: 5
    timeoutSeconds: 1
    periodSeconds: 10
    successThreshold: 1
    failureThreshold: 10
```


---

> 作者: [姚保国](https://yaobg.github.io)  
> URL: http://localhost:1313/posts/notes/devops/k8s/gpu/index-zh-cn/  

