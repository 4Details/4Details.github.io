---
title: FATE 1.2 实践02-FATE集群部署
date: 2020-08-27 13:49:03
tags: [FATE,FL]
top: 11
---

# 2 配置FATE环境

第1步中虚拟机创建完成后的信息如下



| 主机名 | host155         | guest156        | guest157        |
| ------ | --------------- | --------------- | --------------- |
| IP     | 192.168.119.155 | 192.168.119.156 | 192.168.119.157 |
|        |                 |                 |                 |



下面的步骤均是依据官方文档操作 



## 2.1 基础环境配置

注意：不特殊说明，下面配置需要在所有机器上操作！！



由于在创建虚拟机过程中已经更改过每台主机名、创建用户app，所以文档中hostname配置可跳过。

root权限下创建文件夹  /data/projects 归属 app用户



```
//切换root用户，-p创建文件夹，给app用户赋予权限
su root
mkdir -p /data/projects
chown -R app /data/projects
```



### （1）关闭selinux

确认是否已安装selinux

centos系统执行：rpm -qa | grep selinux

ubuntu系统执行：apt list --installed | grep selinux

如果已安装了selinux就执行：setenforce 0



![关闭selinux](/FATE-1-2-实践02：FATE集群部署.assets/关闭selinux.png)



### （2）修改Linux最大打开文件数

切换root用户，在limits.conf文件最后添加5、6行代码，保存退出



```
su root

vim /etc/security/limits.conf

* soft nofile 65536
* hard nofile 65536
```



![编辑最大打开文件数](https://cdn.nlark.com/yuque/0/2020/png/343708/1578365728781-dc1352aa-9d0c-4400-81a4-1e28ecca6bcd.png)





### （3） 添加主机映射



```
vim /etc/hosts
```



![添加主机映射](https://cdn.nlark.com/yuque/0/2020/png/343708/1578485849702-675aea11-871a-4201-8868-bc0a2653849d.png)



### （4）关闭虚拟机防火墙

在实际生产环境中需要配置防火墙端口规则！虚拟机环境下采用关闭防火墙的方式。

如果是Centos系统：



```
systemctl disable firewalld.service
systemctl stop firewalld.service
systemctl status firewalld.service
```

本例的环境是Centos系统

![关闭虚拟机防火墙](D:\workplace\GitSpace\Blogs\source\_posts\FATE-1-2-实践02：FATE集群部署.assets\1578380317690-ecb24d9b-0cf9-47df-b92e-03330fbd8e92.png)



如果是Ubuntu系统：



```
ufw disable
ufw status
```

### （5）给用户赋予sudo权限

root用户下对app用户赋予sudo权限



```
vim /etc/sudoers.d/app

app ALL=(ALL) ALL
app ALL=(ALL) NOPASSWD: ALL
Defaults !env_reset
```



### （6）配置ssh远程登录

**a. 切换app用户，生成rsa_id，具体根据下面代码**



![配置ssh登录01](D:\workplace\GitSpace\Blogs\source\_posts\FATE-1-2-实践02：FATE集群部署.assets\1578380760894-2368928b-1570-451c-aac0-649a1afbcacd.png)



**b.合并id_rsa_pub文件**



**步骤如下：三台机器分别将id_rsa.pub 写入authorized_keys文件中并且赋予权限chomd 600**

![image.png](D:\workplace\GitSpace\Blogs\source\_posts\FATE-1-2-实践02：FATE集群部署.assets\1578381412726-a1b69944-771d-4101-84ab-1f984304c52a-1598507951854.png)



155通过scp将authorized_keys文件发送到156上，并将156的id_rsa.pub文件写入，然后将生成的新文件发送至157上，并且写入157生成的id_rsa.pub文件，将最终生成的authorized_keys文件发送给155和156，此时通过ssh即可登录（在发送文件过程中建立连接需要输入密码）



155->156

![155->156](D:\workplace\GitSpace\Blogs\source\_posts\FATE-1-2-实践02：FATE集群部署.assets\1578381586236-3903e88e-0851-43ea-9d00-aeff64e00d90.png)



156写入， ->157



![156->157](D:\workplace\GitSpace\Blogs\source\_posts\FATE-1-2-实践02：FATE集群部署.assets\1578381654023-9f91941b-d097-4fee-abef-b1c2e5dae98d.png)



157写入，->155  ->156



![157->155 156](D:\workplace\GitSpace\Blogs\source\_posts\FATE-1-2-实践02：FATE集群部署.assets\1578381710128-b920208b-7d45-48a8-887f-c2dbdab925e6.png)



**上述完成之后即可通过  ssh usernane@ip访问某台虚拟机**

需要注意的是，访问自身需要重新建立一个连接。



建议配置以上步骤之后拍摄虚拟机快照，便于后期恢复重新部署。



## 2.2 FATE部署

完成基础环境配置之后，接下来就需要部署FATE的运行环境，本例使用的是[Installation](https://github.com/FederatedAI/FATE/blob/master/cluster-deploy/doc/Fate-cluster_deployment_guide_install_zh.md)文档。

下面的操作**只需在其中一台机器操作即可**，本例选用的是host 155。

### （1）下载压缩包

可以使用wget下载，也可以直接通过[链接](https://webank-ai-1251170195.cos.ap-guangzhou.myqcloud.com/FATE_install_v1.2.0.tar.gz)下载。

```
cd /data/projects
wget https://webank-ai-1251170195.cos.ap-guangzhou.myqcloud.com/FATE_install_v1.2.0.tar.gz
tar -xf FATE_install_v1.2.0.tar.gz
```



![下载压缩包](D:\workplace\GitSpace\Blogs\source\_posts\FATE-1-2-实践02：FATE集群部署.assets\1578399494803-2fb7d502-0815-4840-82f0-a68e4e8110dc.png)

### （2）修改configuration文件

可以使用vim，也可以直接使用编辑器打开并修改。

```
cd /data/projects/FATE/cluster-deploy/script
vi multinode_cluster_configuration.sh
```



![修改conf文件](D:\workplace\GitSpace\Blogs\source\_posts\FATE-1-2-实践02：FATE集群部署.assets\1578399969465-c030b14f-baac-4dc3-ba3b-96fdbb9150ba.png)

![image.png](https://cdn.nlark.com/yuque/0/2020/png/343708/1578399984843-bece1ed0-8147-4754-9ba9-14014be1d75b.png)



建议在部署之前拍摄快照，为防止在部署过程中因某些配置出错造成部署失败。

### （3）部署

本例中选择在各机器上部署所有组件。

```
cd FATE/cluster-deploy/scripts
# 部署所有组件
bash deploy_cluster_multinode.sh binary all 
# 只部署部分组件(可选：jdk python mysql redis fate_flow federatedml fateboard proxy federation roll meta-service egg)：
bash deploy_cluster_multinode.sh binary fate_flow
```



\-----------------------------------------------------------------------------------------------------------------------

-----------------------------------------------漫长地等待过程--------------------------------------------------------

\-----------------------------------------------------------------------------------------------------------------------



![](D:\workplace\GitSpace\Blogs\source\_posts\assets\1578399933245-b4e7ac81-a4a0-4520-accd-d8b6d3a7af41.png)



（部署时间暂未统计，时间需要2-4个小时，记录下了部署过程中终端产生的日志）



此处为语雀文档，点击链接查看：<https://www.yuque.com/u190689/qoiq3w/seeq38>



## 2.3 配置检查

请务必详细对照检查！！！

到各个目标服务器上进行检查对应模块的配置是否准确，每个模块的对应配置文件所在路径可在此配置文件下查看，参考：<https://github.com/FederatedAI/FATE/blob/master/cluster-deploy/doc/configuration.md>





至此，FATE v1.2的安装部署全部结束，接下来测试环境。

建议部署成功之后拍摄快照，便于将来找到还原点。