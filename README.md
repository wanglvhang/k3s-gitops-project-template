# k3s-gitops-project-template
this is a gitops template project base on k3s with a smaple .net web app


working in progress....plan to upload in Oct.


部署初始环境
directory structure  
/src    包含应用代码 contains application source code  
/infra  包含定义k8s对象的.yaml文件 .yaml files of kubernetes objects
/cd     

# step-1：install k3s
this project using k3s as kubernetes cluster, only one command line is need to install k3s cluster and nessary components is already install.


# step-2: install argo-cd
argo-cd is a popular CD tool on Kubernetes.

follow below steps to install argo-cd



## set up argo-cd
after install argo-cd 


# step-3: set up CI with GitHub actions
利用 github 的 CI/CD 工具 actions 在发布时自动生成容器镜像并上传到 dockerhub。


# a standard process


