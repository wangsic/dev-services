

# 部署微服务

在每一套环境中，部署微服务都分为两个步骤，即部署基础设施和微服务本身。基础设施指的是微服务运行期间所需的数据库、缓存等服务。

当针对某一个特定的环境部署时，在后面的命令行参数中需要输入以下两个参数：
1. 环境名称（`--env`）
2. 部署后缀（`--suffix`）

**环境名称**指的是，当前要执行的部署任务的目标环境是哪一个，可以选择的值有：
* `dev` 表示开发环境，即开发人员自用的环境
* `stage`  表示预生产环境

**部署后缀**请参考 [文档首页](https://github.com/netconf-cn2019-workshop/dev-services/blob/master/README.md) 的说明。

### 部署基础设施

针对每一个环境，比如 `dev` 或者 `stage`，只需要部署一次基础设施。之后，多次部署微服务时，可持续使用这些基础设施。

```sh
./provision-infra.sh --env <env> --suffix <suffix>
```

如需删除已部署的基础设施，请手动执行：

```
# 请先切换到正确的 Kubernetes namespace 下
kubectl delete deployments,services,configmaps -l tier=infrastructure
```


### 部署微服务


针对每一个环境，比如 `dev` 或者 `stage`，微服务可以多次部署。

部署步骤是：

1. 编辑变量文件
2. 执行部署命令行


#### 编辑变量文件

找到项目目录下的 `services/vars` 文件，使用文本编辑器编辑其中的变量。各个变量的含义如下：

| 变量 |  描述  |  
|----|----|
| DNS_SUFFIX | 环境中 Ingress 使用的顶级域名 |
| INGRESS_APIVERSION | Ingress 的 apiVersion 值。如果 Kubernetes 集群版本小于 `1.14.0`，请使用 `extensions/v1beta1`，否则请使用 `networking.k8s.io/v1beta1` |
| REGISTRY_SERVER | 存储了微服务的容器镜像注册表（registry）位置 |
| ENVIRONMENT_NAME | （无需手动修改，由命令行参数指定，此值会自动更新）本次部署的目标环境名称。支持 `dev`，`stage`，`prod` |
| DEPLOY_SUFFIX | （无需手动修改，由命令行参数指定，此值会自动更新）本次部署后缀。关于此参数的更多说明，请参考 [文档首页](https://github.com/netconf-cn2019-workshop/dev-services/blob/master/README.md) |
| IMAGE_VERSION | （无需手动修改，由 `service-list` 文件指定后，此值会自动更新）本次部署的各微服务的镜像版本 |

找到项目目录下的 `services/service-list` 文件，将其中每行的最后一项（按冒号 `:` 分隔），改为你希望部署的镜像的版本号。

#### 执行部署命令行

```sh
./provision-services.sh --env <env> --suffix <suffix>
```

如需删除已部署的微服务，请手动执行：

```
# 请先切换到正确的 Kubernetes namespace 下
# kubectl config set-context $(kubectl config current-context) --namespace "<env>-<suffix>"
kubectl delete deployments,services,configmaps,ingress -l 'tier in (backend, frontend)'
```

### 访问微服务

使用以下命令查看可用的网站入口：

```sh
kubectl get ingress
```

你应该能够获取类似下面的输出，访问 `HOSTS` 那一列的值即可访问相应的服务：

```
NAME                       HOSTS                             ADDRESS      PORTS   AGE
ecommerce-webapp-ingress   ecommerce-user1.aks.cloudapp.cn   10.28.6.51   80      27h
```