# 创建EKS Worker节点组

默认情况下，集群Worker节点以NodeGroup的方式提供，NodeGroup依托于EC2的Autoscaling Group和Launch Templates，结合[Cluster Autoscaler](https://docs.aws.amazon.com/eks/latest/userguide/autoscaling.html)插件,可实现集群的自动横向扩展。

Worker节点主要运行kubelet、kube-proxy、VPC-CNI等，通过eksctl工具结合配置模版，可自动创建节点组，并加入到EKS集群，创建步骤如下：

a) 创建Worker NodeGroup模版

创建模版文件，名为`nodegroup.yaml`,内容如下，可以根据自己需求进行调整。

```yaml
apiVersion: eksctl.io/v1alpha5
kind: ClusterConfig

metadata:
  name: "demo"   # 集群名字
  region: "ap-southeast-1"    # 集群所在的region

managedNodeGroups:
  - name: Private-01
    instanceType: m6i.large    # worker节点使用的机器类型
    desiredCapacity: 3    # Autoscaling group的初始大小
    minSize: 3    # Autoscaling group的最小规模
    maxSize: 6    # Autoscaling group的最大规模 
    volumeSize: 30    # 节点EBS大小
    volumeType: gp3    # 节点存储类型
    ssh:
      allow: true # if not publicKeyPath will use ~/.ssh/id_rsa.pub as the default ssh key
      publicKeyName: demoUser    # 节点使用的ssh-key，需要提前创建好
    privateNetworking: true       # 节点放置在private network，public subnet的节点无需设置
#  - name: Public-01
#    instanceType: m6i.large
#    desiredCapacity: 3
#    minSize: 3
#    maxSize: 6
#    volumeSize: 30
#    volumeType: gp3    
#    ssh:
#      allow: true # if not publicKeyPath will use ~/.ssh/id_rsa.pub as the default ssh key
#      publicKeyName: demoUser
```

b) 创建Worker NodeGroup

```bash
ec2-user@~ > eksctl create nodegroup -f nodegroup.yaml
```

* 该过程需要较长的时间，耐心等待

c) 验证Worker节点

```bash
ec2-user@~ > kubectl get node
NAME                                                STATUS   ROLES    AGE   VERSION
ip-172-31-117-202.ap-southeast-1.compute.internal   Ready    <none>   16m   v1.22.6-eks-7d68063
ip-172-31-132-99.ap-southeast-1.compute.internal    Ready    <none>   16m   v1.22.6-eks-7d68063
ip-172-31-96-196.ap-southeast-1.compute.internal    Ready    <none>   16m   v1.22.6-eks-7d68063
```

至此，Worker节点组已创建完成，基础集群已经部署完成，一些用户可能需要额外的插件，参考下一步[额外的插件安装](./05-%E9%A2%9D%E5%A4%96%E6%8F%92%E4%BB%B6%E5%AE%89%E8%A3%85.md)