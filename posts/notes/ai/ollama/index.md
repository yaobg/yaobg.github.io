# Ollama - 本地部署开源大语言模型指南


<!--more-->

# Ollama私有化部署指南

## 什么是Ollama?

Ollama是一个强大的本地大语言模型(LLM)运行平台。它提供了比llama.cpp更简便的安装和使用方式,支持在本地运行多种开源模型:

- LLaMA系列模型
- Mistral系列模型  
- DeepSeek系列模型
- 其他兼容模型

通过Ollama,您可以轻松实现AI模型的本地化部署,无需依赖云服务API。

## 部署方案

### 方案一：本地直接部署

1. **下载安装**
   - 访问[Ollama官网](https://ollama.com)下载对应系统版本
   - 支持Windows/MacOS/Linux系统

2. **环境准备**
   - 确保系统已安装必要依赖
   - 推荐配置:
     - CPU: 4核以上
     - 内存: 8GB以上
     - 硬盘: 20GB以上可用空间

3. **环境变量配置**
   - 按照官方指南设置系统环境变量
   - 确保系统可正确识别Ollama命令

### 方案二：Docker容器部署

1. **获取Docker镜像**
   ```shell
   # 可选择以下镜像源:
   # 官方镜像
   docker pull ollama/ollama
   # 国内加速镜像
   docker pull 192.168.50.7/base/ollama
   ```

2. **创建容器网络**
   ```shell
   docker network create ollama-network
   ```

3. **启动Ollama容器**
   ```shell
   docker run -d \
     --network ollama-network \
     --gpus=all \
     -v /root/data/ollama:/root/.ollama \
     -p 11434:11434 \
     --name ollama \
     192.168.50.7/base/ollama
   ```

   **配置说明:**
   - `--network`: 指定容器网络,用于与Open WebUI互联
   - `--gpus`: GPU配置,需提前完成[Docker GPU配置](https://yaobg.github.io/posts/notes/devops/k8s/gpu/#%E5%AE%89%E8%A3%85)
   - `-v`: 数据卷挂载,用于持久化存储模型文件
   - `-p`: 端口映射,11434为Ollama默认服务端口

## Open WebUI集成

### 简介
Open WebUI是一个功能丰富的自托管AI平台,提供:
- 友好的用户界面
- 完全离线运行能力
- 多种LLM运行环境支持
- 内置RAG推理引擎

### Docker部署步骤

1. **拉取镜像**
   ```shell
   docker pull swr.cn-north-4.myhuaweicloud.com/ddn-k8s/ghcr.io/open-webui/open-webui:main
   ```

2. **启动容器**
   ```shell
   docker run -d \
     --network ollama-network \
     -p 3000:8080 \
     --add-host=host.docker.internal:host-gateway \
     -e HF_ENDPOINT=https://hf-mirror.com \
     -v /root/data/open-webui:/app/backend/data \
     --name open-webui \
     --restart always \
     swr.cn-north-4.myhuaweicloud.com/ddn-k8s/ghcr.io/open-webui/open-webui:main
   ```

## 常见问题解决

### 1. 模型下载速度变慢问题

可通过定时重启下载进程来优化:

```shell
nohup sh -c 'while true; do 
  ollama pull deepseek-r1:671b; 
  echo "等待600秒后重启..." >> output.log;
  sleep 600; 
  echo "停止旧进程..." >> output.log;
  pkill -f "ollama pull deepseek-r1:671b"; 
  echo "重启进程..." >> output.log;
done' > output.log 2>&1 &
echo $! > nohup_ollama.pid
```

### 2. 使用建议

- 建议使用SSD存储模型文件
- 首次运行大模型时预留足够内存
- 定期清理未使用的模型释放空间
- 配置GPU可显著提升推理性能

## 参考资源

- [Ollama官方文档](https://ollama.com/docs)
- [Open WebUI中文文档](https://openwebui-doc-zh.pages.dev/)
- [Docker GPU配置指南](https://yaobg.github.io/posts/notes/devops/k8s/gpu/)
- [国内镜像源](https://docker.aityp.com/)


---

> 作者:   
> URL: https://yaobg.github.io/posts/notes/ai/ollama/  

