# k3s-gitops-project-template
这是一个基于 k3s 的 gitops 流程演示，使用.net web 应用作为演示项目。  
this is a gitops template project base on k3s with a smaple .net web app.  
为加速中国大陆地区的访问，该项目的 jihulab 镜像地址为：https://jihulab.com/wanglvhang/k3s-gitops-project-template

## 一、项目内容

### 目录结构
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

### 使用 github actions 搭建 CI/CD
本例中创建了两个 workflow action
```
├─.github  
│  └─workflows
|       ├─docker-push.yaml  //代码发布时触发，生成镜像并push到docker hub
|       └─dotnet-build.yaml //代码提交时触发，测试生成是否成功
```
本例没有添加自动化测试的流程。  
<br/>

## 二、部署环境搭建
### 主要版本
OS：Ubuntu 22.04 LTS  
k3s: v1.24.4+k3s1  
argocd: v2.4.12

### step 1, 安装 k3s

官方安装脚本：  
`curl -sfL https://get.k3s.io | sh - `

若使用上面的脚本时出现超时，可使用镜像安装脚本:  
`sudo curl -sfL https://rancher-mirror.oss-cn-beijing.aliyuncs.com/k3s/k3s-install.sh | INSTALL_K3S_MIRROR=cn sh -`

安装完成后，使用下面的命令检查k3s是否成功运行  
`kubectl get all --all-namespaces`  

### step 2, 安装 argocd
k3s安装成功后，使用下面的命令安装 argo cd。  
```
kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
```

### step 3, 公开并登录 argo cd
上传 setup 目录中的 patch-argocd-service.yaml 文件，并通过下面的命令  
`kubectl -n argocd patch service argocd-server --patch-file  patch-argocd-service.yaml`  
将 argo cd 内置的 service 更新为 NodePort 来公开 argo cd 服务。  
执行完命令后，可以使用 `https://{主机地址}:31080` 来访问 argo cd 页面。  

<img src="https://user-images.githubusercontent.com/936437/194257503-e5b926a8-00ce-4cfa-852d-8f442a5bbe26.png" alt="argo cd 主页" title="argo cd 主页" width=80%>


默认用户名: `admin`  
默认密码可以通过下面的命令获取:  
`kubectl get secret -n argocd argocd-initial-admin-secret -o=jsonpath={.data.password} | base64 -d`


### step 4, 在 argo cd 中创建应用  
进入 argo cd 主页后 点击左上角的[NEW APP]按钮添加新的部署，在弹出窗口中点击左上角的[EDIT AS YAML] 并拷贝下面的内容。  

```
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: demoproj
spec:
  destination:
    name: ''
    namespace: demo-app
    server: 'https://kubernetes.default.svc'
  source:
    path: deploy/overlays/stage/
    repoURL: 'https://github.com/wanglvhang/k3s-gitops-project-template.git'
    targetRevision: main
  project: default
  syncPolicy:
    automated: null
```
如下图所示
<br/>
<img src="https://user-images.githubusercontent.com/936437/194600297-1e76ef54-e621-4f05-8f9e-acb0789198ea.png" alt="创建app" title="创建app" width=80%>
<br/>
点击[CREATE]即可创建应用部署，若出现超时错误，可多尝试几次或将yaml中的 repoURL 替换为：
`https://jihulab.com/wanglvhang/k3s-gitops-project-template.git`  
<br/>
应用部署创建完成后你就可以查看部署状态并进行各种操作与设置，详细内容可查看 [argo cd 文档](https://argo-cd.readthedocs.io/en/stable/)。
<img src="https://user-images.githubusercontent.com/936437/194600806-a05ae51a-6ad1-4517-baec-0c0a268d5f0b.png" alt="argocd app synced" title="argocd app synced" width=80%>
<br/>
<br/>
手动触发同步或自动同步完成后，就可以通过主机ip地址访问部署好的应用页面了。
<img src="https://user-images.githubusercontent.com/936437/194612620-31915b1c-6030-4479-952d-24f5e3f0bf99.png" alt="demoproj app" title="demoproj app" width=80%>


本例的流程中除了最重要的 [**k3s**](https://k3s.io/)、[**github**](https://docs.github.com/cn) 和 [**argocd**](https://argo-cd.readthedocs.io/en/stable/) 之外还包含了其他工具或技术，如:  


- [github actions](https://docs.github.com/cn/actions) github 上的CI/CD工具。
- [kustomize](https://kustomize.io/)  kustomize 是 k8s 原生的，无模板的 yaml 配置工具。
- [traefik](https://doc.traefik.io/traefik/) k3s 内置了 traefik 作为反向代理服务器。
- [helmchart](https://helm.sh/zh/docs/topics/charts/) k3s 使用 hemlchart 安装 traefik，所以对 traefik 的配置默认使用 hemlchart config。
- [helmchart artifact](https://artifacthub.io/) helmchart 官方仓库。
- [dockerfile](https://docs.docker.com/engine/reference/builder/) 容器技术，主要是 Dockerfile 的编写。
- [yaml](https://yaml.org/) yaml文件的格式与基本语法。


这些工具或技术的细节不在本项目的讨论范围内，相关内容可查看对应文档。  
**在部署过程中有任何问题可在issues中提交，会尽量回答。**


## 三、场景

- 场景1，代码push  
当在 main 提交代码后，dotnet-build.yml workflow 会被触发并 build src目录中的 demoproj 项目。 

- 场景2，发布应用  
当在 main 发布后，docker-push.yml workflow 会被触发并构建 demoproj 镜像并上传到 dockerhub。修改 deploy/overlays/stage/kustomization.yaml 中的images:newTag 为最新的 image tag 签入后 argo cd 会自动更新部署。


## 四、其他

### secrets 管理  
本例中将 tls 和 配置信息都明文放在 /deploy/base/kustomization.yaml 中做为演示，生产环境中不应这么做!!!
通常的做法包括：
- 使用云端提供的 Vault 功能来保存机密信息
- 直接手动在 k8s 中创建机密信息
- 使用 Argo CD Vault Plugin
- 使用 Sealed Secrets

机密管理现在没有统一的标准，需要根据实际的情况选择具体的实践。

### kubernetes 管理工具
为了提高管理 kubernetes 的效率，推荐使用 k9s 和 lens 两款工具。

- [**k9s**](https://k9scli.io/) vim 风格的 kubernetes 管理工具。
- [**lens**](https://k8slens.dev/) GUI kubernetes 管理工具。

### k3s 镜像加速
k3s 默认使用 containerd 容器引擎，containerd 默认使用 docker hub, 为了加速镜像的获取，可以将 setup 目录中的 `registries.yaml` 文件拷贝到 k3s 主机上的 `/etc/rancher/k3s/` 目录中。



