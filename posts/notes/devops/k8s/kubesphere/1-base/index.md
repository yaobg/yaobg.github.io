# kubeSphere


<!--more-->

## 简介
### kubeKey
kubeKey相比kubeadm安装更加简单方便。
### kubeSphere
KubeSphere，是基于 Kubernetes 内核的分布式多租户商用云原生操作系统。在开源能力的基础上，在多云集群管理、微服务治理、应用管理等多个核心业务场景进行功能延伸。

[官方地址](https://kubesphere.io/zh)
## 安装
### 准备工作
1、安装需要的包
```shell
sudo apt install socat conntrack ebtables ipset -y
```
2、修改机器hostname和hosts
```shell
# 查看hostname
hostnamectl
# 修改hostname
sudo hostnamectl set-hostname new-hostname
```
3、docker安装
```shell
# 确保系统是最新的
sudo apt update
sudo apt upgrade -y
# 安装必要的软件包
sudo apt install -y \
    apt-transport-https \
    ca-certificates \
    curl \
    software-properties-common

#添加 Docker 的官方 GPG 密钥
sudo install -m 0755 -d /etc/apt/keyrings
# 注意官方镜像源有问题，替换成阿里云的
curl -fsSL https://mirrors.aliyun.com/docker-ce/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc

# 更新镜像仓库
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://mirrors.aliyun.com/docker-ce/linux/ubuntu/ \
  $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
sudo apt-get update

# 安装docker
sudo apt-get install docker-ce docker-ce-cli containerd.io

# 测试
sudo docker run hello-world
```

4、NFS(非必须)

客户端

1、所有集群节点安装NFS 客户端
```shell
sudo apt install nfs-common
```
服务端（192.168.0.1）

1、安装 nfs-kernel-server
```shell
sudo apt install nfs-kernel-server
```

2、配置共享目录：

例如，我们要共享 /k8s/data 目录。首先，创建目录并设置权限：
```shell
sudo mkdir -p /k8s/data
sudo chown nobody:nogroup /k8s/data
sudo chmod 777 /k8s/data
```

3、编辑 /etc/exports 文件，添加共享目录的配置：
打开 /etc/exports 文件进行编辑：
```shell
# 挂载192.168.0网段
/k8s/data 192.168.0.0/24(rw,sync,no_subtree_check) 
```

4、更新 NFS 共享配置
```shell
sudo exportfs -a
# 重启
sudo systemctl start nfs-kernel-server
sudo systemctl enable nfs-kernel-server
```
### kubeKey

1、**机器配置**

==🔴准备三台机器==

| 节点     | ip          |
|--------|-------------|
| master | 192.168.0.1 |
| node1  | 192.168.0.2 |
| node2  | 192.168.0.3 |

系统版本信息：

- 操作系统:ubuntu20.04.6
- KubeSphere：4.1.2
- Kubernetes：v1.28.13

2、下载kubeKey
```shell
curl -sfL https://get-kk.kubesphere.io | sh -
sudo chmod +x kk
```

3、创建配置文件
```shell
# 查看k8s版本
./kk version --show-supported-k8s
# 创建配置文件，建议默认版本号，有的系统版本高版本的k8s无法安装上
./kk create config 
或
./kk create config --with-kubernetes <Kubernetes version>
```

4、执行以下命令编辑安装配置文件 **config-sample.yaml**
```shell
vi config-sample.yaml
```

- 配置文件如下：
```yaml
apiVersion: kubekey.kubesphere.io/v1alpha2
kind: Cluster
metadata:
  name: sample
spec:
  hosts:
  - {name: master, address: 192.168.0.2, internalAddress: 192.168.0.2, user: ubuntu, password: Testing123}
  - {name: node1, address: 192.168.0.3, internalAddress: 192.168.0.3, user: ubuntu, password: Testing123}
  - {name: node2, address: 192.168.0.4, internalAddress: 192.168.0.4, user: ubuntu, password: Testing123}
  roleGroups:
    etcd:
    - master
    control-plane:
    - master
    worker:
    - node1
    - node2
  controlPlaneEndpoint:
    internalLoadbalancer: haproxy # 如需部署⾼可⽤集群，且⽆负载均衡器可⽤，可开启该参数，做集群内部负载均衡
    domain: lb.kubesphere.local
    address: ""
    port: 6443
  kubernetes:
    version: v1.28.13
    clusterName: cluster.local
    containerManager: containerd # 部署 kubernetes v1.24+ 版本，建议将 containerManager 设置为 containerd
  network:
    plugin: calico
    kubePodsCIDR: 10.233.64.0/18
    kubeServiceCIDR: 10.233.0.0/18
    ## multus support. https://github.com/k8snetworkplumbingwg/multus-cni
    enableMultusCNI: false
  registry:
    privateRegistry: ""
    registryMirrors: []
    insecureRegistries: []
  addons: []
```
**注意**如果addons没有配置可持久存储，后续需要配置[NFS](#nfs)
### KubeSphere
1、安装helm
```shell
curl https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3 | bash
```
2、在集群节点，执行以下命令安装 KubeSphere Core。
```shell
helm upgrade --install -n kubesphere-system --create-namespace ks-core https://charts.kubesphere.io/main/ks-core-1.1.3.tgz --debug --wait --set global.imageRegistry=swr.cn-southwest-2.myhuaweicloud.com/ks --set extension.imageRegistry=swr.cn-southwest-2.myhuaweicloud.com/ks
```
3、如果显示如下信息，则表明 ks-core 安装成功：
```yaml
NOTES:
Thank you for choosing KubeSphere Helm Chart.

Please be patient and wait for several seconds for the KubeSphere deployment to complete.

1. Wait for Deployment Completion

    Confirm that all KubeSphere components are running by executing the following command:

    kubectl get pods -n kubesphere-system

2. Access the KubeSphere Console

    Once the deployment is complete, you can access the KubeSphere console using the following URL:

    http://192.168.0.1:30880

3. Login to KubeSphere Console

    Use the following credentials to log in:

    Account: admin
    Password: P@88w0rd

NOTE: It is highly recommended to change the default password immediately after the first login.

For additional information and details, please visit https://kubesphere.io.
```
## 集群管理

### 存储

#### NFS

1、下载NFS Subdir External Provisioner
```shell
wget https://github.com/kubernetes-sigs/nfs-subdir-external-provisioner/archive/refs/tags/nfs-subdir-external-provisioner-4.0.18.zip
unzip nfs-subdir-external-provisioner-4.0.18.zip
cd nfs-subdir-external-provisioner-nfs-subdir-external-provisioner-4.0.18/
```

2、创建NameSpace
可选配置，默认的 NameSpace 为 default，为了便于资源区分管理，可以创建一个新的命名空间。
```shell
kubectl create ns nfs-system
# 替换资源清单
 sed -i'' "s/namespace:.*/namespace: nfs-system/g" ./deploy/rbac.yaml ./deploy/deployment.yaml
```

3、创建RBAC
```shell
kubectl create -f deploy/rbac.yaml
```

4、配置 NFS subdir external provisioner

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nfs-client-provisioner
  labels:
    app: nfs-client-provisioner
  # replace with namespace where provisioner is deployed
  namespace: nfs-system
spec:
  replicas: 1
  strategy:
    type: Recreate
  selector:
    matchLabels:
      app: nfs-client-provisioner
  template:
    metadata:
      labels:
        app: nfs-client-provisioner
    spec:
      serviceAccountName: nfs-client-provisioner
      containers:
        - name: nfs-client-provisioner
          image: swr.cn-north-4.myhuaweicloud.com/ddn-k8s/k8s.gcr.io/sig-storage/nfs-subdir-external-provisioner:v4.0.2
          volumeMounts:
            - name: nfs-client-root
              mountPath: /persistentvolumes
          env:
            - name: PROVISIONER_NAME
              value: k8s-sigs.io/nfs-subdir-external-provisioner
            - name: NFS_SERVER
              value: 192.168.0.1
            - name: NFS_PATH
              value: /k8s/data
      volumes:
        - name: nfs-client-root
          nfs:
            server: 192.168.0.1
            path: /k8s/data
```

- 执行
```shell
 kubectl apply -f deploy/deployment.yaml
```

- 查看
```shell
kubectl get deployment,pods -n nfs-system
```

5、 部署 Storage Class

```shell
kubectl apply -f deploy/class.yaml
```

## 问题

1、下载超时

如果您访问 GitHub/Googleapis 受限，请登录任意集群节点，执行以下命令设置下载区域：
```shell
export KKZONE=cn
```
2、kubeKey安装一直失败

需要注意是否是k8s版本的问题


---

> 作者:   
> URL: https://yaobg.github.io/posts/notes/devops/k8s/kubesphere/1-base/  

