## WSL 安装 Ubuntu

WSL 2 设置为默认版本

```powershell
wsl --set-default-version 2

wsl --list
wsl --unregister CentOS8
wsl --list --online
wsl --install -d Ubuntu-20.04
```



## 安装Docker

```
#1、新软件列表和允许使用https
sudo apt-get update
sudo apt-get install \
    ca-certificates \
    curl \
    gnupg \
    lsb-release

#2、添加阿里源的GPG
curl -fsSL https://mirrors.aliyun.com/docker-ce/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg

#3、设置阿里源的docker仓库
 echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://mirrors.aliyun.com/docker-ce/linux/ubuntu \
  $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

#4.安装docker:

#4.1更新apt-get
sudo apt-get update

#4.2安装最新的docker版本
sudo apt-get install docker-ce docker-ce-cli containerd.io

#4.3启动docker
sudo service docker start

#4.4查看docker服务状态
sudo service docker status
```

## 安装MiniKube

参考：https://zhuanlan.zhihu.com/p/543458320

```
curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64
sudo install minikube-linux-amd64 /usr/local/bin/minikube

minikube start --registry-mirror=https://registry.docker-cn.com  --image-mirror-country cn --kubernetes-version=v1.26.3

验证
$kubectl get ns
NAME              STATUS   AGE
default           Active   102s
kube-node-lease   Active   104s
kube-public       Active   104s
kube-system       Active   104s
```

