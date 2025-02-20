# Kubernetes GPU 资源调度指南


## 环境要求

### 软件版本

| 组件         | 版本      | 说明                          |
|------------|---------|------------------------------|
| Kubernetes | v1.23.7 | 容器编排平台                      |
| KubeSphere | v4.1    | 企业级容器管理平台                   |
| Helm       | 最新版     | Kubernetes的包管理工具            |
| Docker     | 最新版     | 容器运行时                       |
| NVIDIA驱动  | 550+    | GPU驱动程序                     |
| CUDA       | 12.4    | NVIDIA并行计算平台                |

## NVIDIA环境配置

### 1. GPU驱动安装

#### 1.1 检查GPU设备

```shell
# 查看NVIDIA显卡型号
lspci | grep NVIDIA
```

#### 1.2 安装驱动

1. 访问[NVIDIA官方驱动下载页面](https://www.nvidia.cn/geforce/drivers/)
2. 选择对应显卡型号和操作系统
3. 下载并安装驱动

#### 1.3 验证安装

```shell
nvidia-smi
```

预期输出示例：
```
+-----------------------------------------------------------------------------------------+
| NVIDIA-SMI 550.127.05             Driver Version: 550.127.05     CUDA Version: 12.4     |
|-----------------------------------------+------------------------+----------------------+
| GPU  Name                 Persistence-M | Bus-Id          Disp.A | Volatile Uncorr. ECC |
| Fan  Temp   Perf          Pwr:Usage/Cap |           Memory-Usage | GPU-Util  Compute M. |
|=========================================+========================+======================|
|   0  NVIDIA GeForce RTX 3060        Off |   00000000:01:00.0 Off |                  N/A |
|  0%   24C    P8             10W /  170W |     223MiB /  12288MiB |      0%      Default |
+-----------------------------------------+------------------------+----------------------+
```

### 2. CUDA工具包安装

CUDA (Compute Unified Device Architecture) 是NVIDIA推出的并行计算平台，支持使用GPU进行通用计算。

1. 访问[CUDA Toolkit Archive](https://developer.nvidia.com/cuda-toolkit-archive)
2. 选择与GPU驱动匹配的CUDA版本（本例中为12.4）
3. 按照官方指南完成安装

## Docker GPU支持配置

### 1. 安装NVIDIA Container Toolkit

```shell
# 添加NVIDIA软件源
curl -fsSL https://nvidia.github.io/libnvidia-container/gpgkey | sudo gpg --dearmor -o /usr/share/keyrings/nvidia-container-toolkit-keyring.gpg \
  && curl -s -L https://nvidia.github.io/libnvidia-container/stable/deb/nvidia-container-toolkit.list | \
    sed 's#deb https://#deb [signed-by=/usr/share/keyrings/nvidia-container-toolkit-keyring.gpg] https://#g' | \
    sudo tee /etc/apt/sources.list.d/nvidia-container-toolkit.list

# 更新软件包列表
sudo apt-get update

# 安装NVIDIA Container Toolkit
sudo apt-get install -y nvidia-container-toolkit
```

### 2. 配置Docker运行时

```shell
# 配置nvidia-container-runtime
nvidia-ctk runtime configure --runtime=docker

# 重启Docker服务
systemctl restart docker
```

### 3. 验证Docker GPU支持

```shell
# 运行测试容器
docker run --gpus all --rm centos:latest nvidia-smi
```

## Kubernetes GPU资源调度

### 1. 部署GPU Operator

GPU Operator简化了Kubernetes集群中NVIDIA GPU的管理，提供以下功能：
- 自动化GPU驱动管理
- 设备插件配置
- NVIDIA工具包集成
- 运行时环境配置

#### 1.1 使用Helm安装

```shell
# 添加NVIDIA Helm仓库
helm repo add nvidia https://helm.ngc.nvidia.com/nvidia
helm repo update

# 安装GPU Operator
helm install -n gpu-operator --create-namespace gpu-operator nvidia/gpu-operator --set driver.enabled=false
```

#### 1.2 验证部署状态

```shell
kubectl get deployment -n gpu-operator
```

### 2. GPU工作负载测试

创建测试Pod验证GPU功能：

```yaml
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

## 常见问题解决

### 1. 镜像拉取问题

**问题**: registry.k8s.io镜像无法访问

**解决方案**:
```shell
# 使用私有镜像仓库
docker tag registry.k8s.io/nfd/node-feature-discovery:v0.16.6 192.168.50.7/library/node-feature-discovery:v0.16.6
```

### 2. GPU Operator启动问题

**问题**: K8s 1.24版本以下node-feature-discovery-master无法启动

```shell
k edit deployment gpu-operator-node-feature-discovery-master -n gpu-operator
# 删除健康检查探针配置
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

## 参考资源

- [NVIDIA驱动下载](https://www.nvidia.cn/geforce/drivers/)
- [CUDA Toolkit文档](https://docs.nvidia.com/cuda/)
- [GPU Operator指南](https://docs.nvidia.com/datacenter/cloud-native/gpu-operator/overview.html)
- [Kubernetes GPU调度文档](https://kubernetes.io/docs/tasks/manage-gpus/scheduling-gpus/)


---

> 作者: [姚保国](https://yaobg.github.io)  
> URL: https://yaobg.github.io/posts/notes/devops/k8s/gpu/  

