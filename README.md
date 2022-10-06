# k3s-gitops-project-template
this is a gitops template project base on k3s with a smaple .net web app  
这是一个基于 k3s 的 gitops 流程演示，使用.net web 应用作为演示项目。


# 一、项目内容

## 目录结构
```
├─.github  
│  └─workflows    //github actions 文件
├─deploy  
│  ├─base         //用于部署yaml文件的基础目录
│  └─overlays     //对应不同环境的 overlay
│      ├─prod     //prod 环境
│      └─stage    //stage 测试环境，本例使用 stage环境作为演示
│          └─tls  //stage 环境ssl证书
├─setup           //用于初始化环境的一些 
└─src             //应用代码
    └─DemoProject //demo project 项目文件夹
```

## 使用 github actions 搭建 CI/CD
本例中创建了两个 workflow action
```
├─.github  
│  └─workflows
|       ├─docker-push.yaml  //主干发布时触发，生成镜像并push到docker hub
|       └─dotnet-build.yaml //dev分支提交时触发，测试生成是否成功
```


# 二、部署环境搭建
## 主要版本
OS：Ubuntu 22.04 LTS  
k3s: v1.24.4+k3s1  
argocd: v2.4.12

## step 1, 安装 k3s

官方安装脚本：  
`curl -sfL https://get.k3s.io | sh - `

若使用上面的脚本时出现超时，可使用镜像安装脚本:  
`sudo curl -sfL https://rancher-mirror.oss-cn-beijing.aliyuncs.com/k3s/k3s-install.sh | INSTALL_K3S_MIRROR=cn sh -`

安装完成后，使用下面的命令检查k3s是否成功运行  
`kubectl get all --all-namespaces `  

## step 2, 安装 argocd
k3s安装成功后，使用下面的命令安装 argo cd。  
```
kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
```

## step 3, 公开并登录 argo cd
上传 setup 目录中的 patch-argocd-service.yaml 文件，并通过下面的命令  
`kubectl -n argocd patch service argocd-server --patch-file  patch-argocd-service.yaml`  
将 argo cd 内置的 service 更新为 NodePort 来公开 argo cd 服务。  
执行完命令后，可以使用 https://{主机地址}:31080 来访问 argo cd 页面。  
默认用户名: `admin`  
默认密码可以通过下面的命令获取:  
`kubectl get secret -n argocd argocd-initial-admin-secret -o=jsonpath={.data.password} | base64 -d`


## step 4, 在 argo cd 中创建应用




# 三、实际流程





# 四、其他

