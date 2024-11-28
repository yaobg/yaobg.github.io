# gitLab镜像部署


<!--more-->
## docker部署
1、镜像
```shell
docker run --detach \
  --hostname 192.168.50.7 \
  --publish 9043:443 --publish 9080:80 --publish 9022:22 \
  --name gitlab \
  --restart always \
  --volume /mnt/nfs/gitlab/config:/etc/gitlab \
  --volume /mnt/nfs/gitlab/logs:/var/log/gitlab \
  --volume /mnt/nfs/gitlab/data:/var/opt/gitlab \
  gitlab/gitlab-ce:latest
```
参数说明：

- hostname：是访问地址，如果是公网访问，需要写公网的ip
- publish：分别对应443 80 22端口
- volume：挂载地址，前面是服务器地址，后面对应的镜像内地址

注意事项：10080端口不能用，浏览器认为10080端口不安全

2、修改配置文件

进入容器
```shell
docker exec -it gitlab /bin/bash
```
修改配置文件
```shell
vi /etc/gitlab/gitlab.rb
```
新增配置如下：
```yaml
#gitlab访问地址，可以写域名。如果端口不写的话默认为80端口，注意此处不能写端口，不然无法访问
external_url 'http://192.168.50.7'
#ssh主机ip
gitlab_rails['gitlab_ssh_host'] = '192.168.50.7'
#ssh连接端口
gitlab_rails['gitlab_shell_ssh_port'] = 10022
#时区
gitlab_rails['time_zone'] = 'Asia/Shanghai'
#开启备份功能
gitlab_rails['manage_backup_path'] = true
#备份文件的权限
gitlab_rails['backup_archive_permissions'] = 0644
#保存备份 7 天
gitlab_rails['backup_keep_time'] = 604800
```
生效配置：
```shell
# 容器内执行
gitlab-ctl reconfigure
# 容器外执行
docker exec 容器名或容器ID gitlab-ctl reconfigure  
```

## 备份
### 手动备份
```yaml
# 第一种进行入容器执行命令的方法进行手工备份
docker exec -it 容器名或容器id bash  # 进入容器
gitlab-rake gitlab:backup:create   # 执行gitlab备份命令

# 第二种直接使用外部命令执行，一次完成
docker exec 容器名或容器id gitlab-rake gitlab:backup:create

```

### 自动备份
脚本
```yaml
#!/bin/bash
case "$1" in
  start)
    docker exec gitlab gitlab-rake gitlab:backup:create
    ;;
esac
```
定时器
```shell
crontab -e
# 新增下面一行，每天2点备份
0 2 * * * /mnt/nfs/gitlab.backup.sh start
```
## 常见问题
1、gitLab的root密码怎么查看
```yaml
# 进入容器
docker exec -it gitlab /bin/bash
# 查看密码
cat /etc/gitlab/initial_root_password
```

---

> 作者:   
> URL: https://yaobg.github.io/posts/notes/devops/docker/mirror/1-gitlab/  

