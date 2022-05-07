# 安装集群Control Plane

集群Control Plane主要托管了kube-apiserver、kube-scheduler、kube-controller-manager、etcd等组件，通过eksctl工具结合配置模版，快速建立EKS Control Plane，并在过程中自动创建Control Plane需要的IAM Role/Policy，SecurityGroup等配置。Control Plane创建步骤如下：

a) 创建Control plane的eksctl模版

创建模版文件，名为`cluster.yaml`,内容如下，根据自己需求调整：

```yaml
apiVersion: eksctl.io/v1alpha5
kind: ClusterConfig

metadata:
  name: "demo"   # 集群名字
  region: "ap-southeast-1"    # 集群所在的region
  version: "1.22"    # eks版本

iam:
  withOIDC: true    # OIDC配置，很重要，AWS Loadbalancer Controller等addon都需要

vpc:
  id: "vpc-0822ab256262822eb"    # 集群所在的vpc
  cidr: "172.31.0.0/16"    # vpc的CIDR
  subnets:
    public:
      ap-southeast-1a:    # 替换为AWS Management Console显示的信息，需要确保与AZ信息与Subnet ID对应
        id: "subnet-09243387ed13d6b56"    #替换为cloudformation的输出
      ap-southeast-1b:
        id: "subnet-04efc342407b23d8a"
      ap-southeast-1c:
        id: "subnet-0409576d8ba60fa53"
    private:
      ap-southeast-1a:
        id: "subnet-0218d913fd8663f15"
      ap-southeast-1b:
        id: "subnet-0ad79cd4e328794f8"
      ap-southeast-1c:
        id: "subnet-03601d164fc485234"

cloudWatch:
  clusterLogging:
    enableTypes: ["api", "audit", "authenticator", "controllerManager","scheduler"]   # 开启控制平面的日志功能，日志输出到cloudwatch的日志组里
    # all supported types: "api", "audit", "authenticator", "controllerManager", "scheduler"
    # supported special values: "*" and "all"
```

b) 创建EKS Control plane

```bash
ec2-user@~ > eksctl create cluster -f cluster.yaml
```

* 该过程持续时间较久，一般需要十几分钟，请确保网络稳定，耐心等待
* eksctl除了创建Control plane，会自动帮忙配置好kubeconfig文件，如需要也可以手动执行`aws eks update-kubeconfig --name <cluster-name>`创建kubeconfig

c) 验证集群状态

```bash
ec2-user@~ > kubectl cluster-info
Kubernetes control plane is running at https://<xxxxxx>.gr7.ap-southeast-1.eks.amazonaws.com
CoreDNS is running at https://<xxxxxx>.gr7.ap-southeast-1.eks.amazonaws.com/api/v1/namespaces/kube-system/services/kube-dns:dns/proxy
```

至此，EKS的Control Plane创建完成，可进行下一步[集群Worker部署](./04-%E9%9B%86%E7%BE%A4Worker%E9%83%A8%E7%BD%B2.md)