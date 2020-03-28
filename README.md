# centos7一键部署deploy_Kubernetes:v1.13.4脚本

# 后续脚本更新统一由kkitDeploy项目持续更新相应脚本

致谢：本项目参考 https://github.com/luckman666/deploy_Kubernetes-v1.13.1  只是稍作调整

感谢作者luckman666 开源

centos7.4 版本已验证通过，可放心使用。有问题请提交issue，谢谢



请移步至kkitDeploy项目

https://github.com/luckman666/kkitdeploy_server

##########################################################################

k8s 1.15.2一键部署地址： https://github.com/luckman666/k8s1.15.2

k8s 1.15.1一键部署地址：https://github.com/luckman666/k8s1.15.1

k8s 1.15.0一键部署地址：https://github.com/luckman666/deploy_Kubernetes-v1.15.0

k8s 1.14.1一键部署地址：https://github.com/luckman666/deploy_Kubernetes-v1.14.1

k8s 1.13.1一键部署地址：https://github.com/luckman666/deploy_Kubernetes-v1.13.1

觉得不错给个star哦！我还会持续更新脚本，但是我想跨代更新，等啥时候出V1.15我在写V1.14的。讲真K8S实在太快了更新的！
注意事项：

1、使用git clone的同志们需要将文件夹里面的所有文件cp 到/root下面。确保所有文件都在/root下面。实在不好意思哈！我不想改了！我把路径写成了/root了
然后只需要在修改base.config里面的固定参数即可。

2、给.sh结尾的脚本赋权限。

3、然后只需执行./deploy_k8s_master.sh就可以啦！

4、tail -f setup.log 查看日志

5、物理机不用说了，要是虚拟机cpu必须最少是2个哦！切记


# 升级内核脚本（这个内核是否需要升级，我没测试过但是写在这里了。反正我是升级了。有需要的就升级吧！或者忽略去直接部署试试？）

执行upgradeKernel.sh就可以将内核升级到4+了，脚本内容如下：

#!/bin/bash

setupkernel(){

 rpm --import https://www.elrepo.org/RPM-GPG-KEY-elrepo.org

 rpm -Uvh http://www.elrepo.org/elrepo-release-7.0-3.el7.elrepo.noarch.rpm

 yum --enablerepo=elrepo-kernel install -y kernel-lt kernel-lt-devel

 grub2-set-default 0

 reboot

}

setupkernel

# 部署k8s集群具体实现步骤：

git clone https://github.com/zhaoyansheng163/deploy_Kubernetes-v1.13.4.git

cd deploy_Kubernetes-v1.13.4/

chmod -R 755 .

mv * /root

cd /root

编辑base.config里面的参数

./deploy_k8s_master.sh

备注1：如果因网速问题，多次尝试未成功。建议下载百度云盘的k8s镜像：k8s.tar  

链接：https://pan.baidu.com/s/1eDn6tbfb5URDlttUSQDR6w 
提取码：wjdc

然后在所有node上：docker load -i k8s.tar

然后注释掉 deploy_k8s_master.sh、node_install_k8s.sh  中的下列几行

	#for imagename in ${images[@]}; do
	#docker pull zhaoyansheng/$imagename
	#docker tag zhaoyansheng/$imagename k8s.gcr.io/$imagename
	#docker rmi zhaoyansheng/$imagename
	#done
	
	#docker pull quay.io/coreos/flannel:v0.10.0-amd64
然后再重新执行即可。

备注2(在云服务器验证通过)：生产环境部署有内网和公网IP的情况下，如果想让两个IP均可被管理，则需要修改 deploy_k8s_master.sh中下列的内容：

```
kubeadm init --kubernetes-version=$k8s_version --pod-network-cidr=10.244.0.0/16 --apiserver-advertise-address=$masterip
```

修改为：

```
kubeadm init --kubernetes-version=$k8s_version --pod-network-cidr=10.244.0.0/16  --apiserver-advertise-address=0.0.0.0 --apiserver-cert-extra-sans=$masterip,PUBLICIP1,PUBLICIP2
```




# base.config参数介绍：

masterIP：

masterip="192.168.1.107"

K8S版本：

k8s_version="v1.13.1"

服务器root密码

root_passwd=root123

多台主机的主机名前缀，主节点就叫k8s1，node叫k8s2依次后推

hostname=k8s

集群服务器IP地址

hostip='
192.168.1.107
192.168.1.108
192.168.1.109
'
再部署的时候严格按照我所给的示例参数写哦。换参数不要换格式，以免出错

# 部署完后进入到dashboard文件夹部署dashboard

cd dashboard

kubectl create -f .

然后查看部署情况以及登录的node节点端口

kubectl get service --all-namespaces | grep kubernetes-dashboard

例如结果：
kube-system   kubernetes-dashboard   NodePort    10.101.25.47   <none>        443:31660/TCP   22m
那么你就输入https://nodeIP:31660来登录
​	
查看登录时候的token

kubectl -n kube-system describe secret $(kubectl -n kube-system get secret | grep admin-user | awk '{print $1}')

关注公众号回复：k8s   获得k8s各个版本的一键部署脚本

![index4](https://github.com/luckman666/devops_kkit/blob/master/gzh.jpg)


