# Jenkins保姆级安装教程


## 📚 前言

Jenkins是当前最流行的持续集成工具之一,本文将为您提供一个详细的Jenkins安装配置指南。无论您是DevOps新手还是有经验的工程师,都能从本文中获得有价值的信息。

## 🚀 一、Docker安装Jenkins

### 1.1 创建配置文件

首先创建`docker-compose.yml`文件:

```yaml:docker-compose.yml
services:
  jenkins:
    image: jenkins/jenkins:2.452.3-lts
    container_name: jenkins
    restart: on-failure
    user: "0"
    environment:
      - JAVA_OPTS=-Duser.timezone=Asia/Shanghai
    ports:
      - "8082:8080"      # Web访问端口
      - "50000:50000"    # JNLP通信端口
    volumes:
      - /mnt/nfs/jenkins_home:/var/jenkins_home          # Jenkins数据目录
      - /etc/localtime:/etc/localtime                    # 时区同步
      - /usr/bin/docker:/usr/bin/docker                  # Docker命令
      - /var/run/docker.sock:/var/run/docker.sock        # Docker通信
      - /usr/local/bin/kubectl:/usr/bin/kubectl          # K8s命令
      - /usr/sbin/helm:/usr/bin/helm                     # Helm命令
      - $HOME/.kube:$HOME/.kube                          # K8s配置
      - /usr/local/maven-3.9:/usr/local/maven-3.9        # Maven目录
```

**📋 配置说明:**

| 配置项 | 说明 | 用途 |
|--------|------|------|
| ports | 8080:8080 | Jenkins Web界面访问 |
| | 50000:50000 | Jenkins代理节点通信 |
| volumes | /var/jenkins_home | 数据持久化存储 |
| | /usr/bin/docker | 支持Docker构建 |
| | /usr/bin/kubectl | 支持K8s部署 |
| | /usr/bin/helm | 支持Helm部署 |

### 1.2 启动容器

```bash
# 启动Jenkins容器
docker-compose up -d

# 验证容器状态
docker ps | grep jenkins
```

### 1.3 获取管理员密码

两种方式获取初始密码:

```bash
# 方式1: 查看容器日志
docker logs jenkins

# 方式2: 直接读取密码文件
cat /mnt/nfs/jenkins_home/secrets/initialAdminPassword
```

## 🔧 二、基础配置

### 2.1 配置插件源

为提升下载速度,建议更换为国内源:

```bash
# 1. 修改插件下载源
cd /mnt/nfs/jenkins_home/updates
sed -i 's/http:\/\/updates.jenkins-ci.org\/download/https:\/\/mirrors.tuna.tsinghua.edu.cn\/jenkins/g' default.json && \
sed -i 's/http:\/\/www.google.com/https:\/\/www.baidu.com/g' default.json

# 2. 修改更新中心
cat > /mnt/nfs/jenkins_home/hudson.model.UpdateCenter.xml << EOF
<?xml version='1.1' encoding='UTF-8'?>
<sites>
  <site>
    <id>default</id>
    <url>http://mirror.esuni.jp/jenkins/updates/update-center.json</url>
  </site>
</sites>
EOF
```

### 2.2 必备插件清单

**🔰 基础功能类**
| 插件名称 | 用途 | 必要性 |
|---------|------|--------|
| Locale | 中文支持 | ★★★★★ |
| Gitlab Plugin | Git集成 | ★★★★★ |
| Maven Integration | Maven构建 | ★★★★☆ |
| NodeJs | Node.js支持 | ★★★★☆ |

**🛠 部署工具类**
| 插件名称 | 用途 | 必要性 |
|---------|------|--------|
| Kubernetes | K8s集成 | ★★★★★ |
| Docker Pipeline | Docker支持 | ★★★★★ |
| Publish Over SSH | 远程部署 | ★★★★☆ |

**📊 可视化类**
| 插件名称 | 用途 | 必要性 |
|---------|------|--------|
| Blue Ocean | 流水线可视化 | ★★★★☆ |
| Build Pipeline | 构建流程图 | ★★★★☆ |
| AnsiColor | 控制台彩色输出 | ★★★☆☆ |

## 🔗 三、系统集成

### 3.1 GitLab集成

1. GitLab生成访问令牌
   ![img.png](images/4.png)

   ![img.png](images/5.png)
2. Jenkins添加GitLab凭据
   ![6.png](images/6.png)
3. 配置GitLab连接
   ![gitlab-3.png](images/gitlab-3.png)

### 3.2 Harbor集成
   ![img.png](images/7.png)

### 3.3 Maven配置

1. 配置Maven settings.xml
   ![maven-1.png](images/maven-1.png)
2. 设置Maven环境变量
   ![maven-2.png](images/maven-2.png)

## 📝 四、Pipeline示例

完整的CI/CD流水线示例:

```groovy
pipeline {
    agent any
    tools {
        maven 'maven-3.9'  // Maven 的名称需与全局工具配置中一致
    }
    options {
        timestamps()  // 为所有步骤启用时间戳
    }
    parameters {
        string(name: 'BRANCH_NAME', defaultValue: 'dev', description: 'Git branch')
        string(name: 'TAG', defaultValue: 'latest', description: 'Git Tag')
    }
    environment {
        GIT_URL = 'http://192.168.50.7:9080/wx-ai/data-platform-server.git'
        CREDENTIALSID = 'gitlab-user-ybg' // Git 凭据 ID
        VERSION = VersionNumber(
            projectStartDate: '1970-12-12',
            versionNumberString: '0.0.${BUILDS_ALL_TIME}',
            versionPrefix: '',
            worstResultForIncrement: 'SUCCESS'
        )
        HARBOR_REPO = "192.168.50.7/dev/data-platform-server:${env.VERSION}"  // Harbor 的 Docker 仓库地址
    }
    stages {
        stage('拉取代码') {
            steps {
                script {
                    // 检出代码
                    git credentialsId: env.CREDENTIALSID, url: env.GIT_URL, branch: params.BRANCH_NAME
                    if (params.TAG && params.TAG.startsWith("v")) {
                        echo "Checking out Git Tag: ${params.TAG}"
                        sh "git fetch --tags"
                        sh "git checkout tags/${params.TAG}"
                    }
                }
            }
        }
        stage('Maven Build') {
            steps {
                script {
                    // 使用 Maven 打包并重命名为 app.jar
                    sh """
                    echo "Starting Maven Build..."
                    mvn clean package -DskipTests -e -X
                    """
                }
            }
        }
        stage('编译镜像') {
            steps {
                script {
                    // 构建 Docker 镜像
                    sh """
                    ls -lh target
                    echo "Building Docker image: ${HARBOR_REPO}"
                    docker build -t ${HARBOR_REPO} .
                    """
                }
            }
        }
        stage('推送harbor') {
            steps {
                script {
                    // 登录并推送镜像到 Harbor
                    withCredentials([usernamePassword(credentialsId: 'harbor', passwordVariable: 'HARBOR_PASSWORD', usernameVariable: 'HARBOR_USERNAME')]) {
                        sh """
                        echo "Logging into Harbor..."
                        docker login 192.168.50.7 -u ${HARBOR_USERNAME} -p ${HARBOR_PASSWORD}
                        echo "Pushing Docker image to Harbor..."
                        docker push ${HARBOR_REPO}
                        """
                    }
                }
            }
        }
        stage('Deploy with Helm') {
            steps {
                script {
                    // 使用 Helm 部署
                    sh """
                    echo "Deploying with Helm..."
                    helm upgrade --install data-platform-server deploy/k8s-helm --namespace dev --set image.repository=${HARBOR_REPO}
                    """
                }
            }
        }
    }
    post {
        always {
            // 清理临时文件
            cleanWs()
        }
    }
}
```

## ❓ 五、常见问题

### 5.1 插件安装失败

**问题描述**: 插件安装过程中出现超时错误  
**解决方案**: 
1. 更换插件源
   Dashboard > 插件管理 > 高级 > 升级站点 > URL 更改为http://mirror.esuni.jp/jenkins/updates/update-center.json
### 5.2 如何生成pipeline语法
1、点击流水线语法

![8.png](images/8.png)

2、生成自己需要的语法

![9.png](images/9.png)

## 📚 参考资料

- [Jenkins官方文档](https://www.jenkins.io/doc/)
- [Jenkins Pipeline语法](https://www.jenkins.io/doc/book/pipeline/)
- [Docker官方文档](https://docs.docker.com/)
- [Kubernetes文档](https://kubernetes.io/docs/)

---
> 💡 **小贴士**: 定期备份Jenkins配置文件和数据目录,以防意外情况发生。

---

> 作者: [姚保国](https://yaobg.github.io)  
> URL: https://yaobg.github.io/posts/notes/devops/docker/mirror/2-jenkins/  

