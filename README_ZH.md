# 时髦商店: 云原生微服务示例程序

本项目包含10层的微服务的程序。这个程序是一个基于web的电子商务网站，被称之为“时髦商店”。
用户可以在上面浏览商品，把这些商品加入购物车，最后买下他们。

**谷歌使用本程序来演示Kubernetes/GKE，Istio，Stackdriver, gRPC and OpenCensus技术的使用**。
本程序可以在任何Kubernetes集群上使用（比如本地的一个集群），以及谷歌的Kubernetes引擎。
它**只需一丁点到一点儿配置，甚至一丁点配置都不需要就可以部署**

假如你在使用这个示例，请**★Star**这个仓库来表达您的兴趣！

> 👓**致谷歌员工？的注意事项:** 如果你要使用这个程序，请先在
> [go/microservices-demo](http://go/microservices-demo) 填写表单

## 截图

| 主页                                                                                                       | 收银台截屏                                                                                                     |
| ----------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------ |
| [![Screenshot of store homepage](./docs/img/hipster-shop-frontend-1.png)](./docs/img/hipster-shop-frontend-1.png) | [![Screenshot of checkout screen](./docs/img/hipster-shop-frontend-2.png)](./docs/img/hipster-shop-frontend-2.png) |

## 服务架构

**时髦商店** 是一个包含许多不同语言的微服务组成，他们通过gRPC进行相互通信。
[![Architecture of
microservices](./docs/img/architecture-diagram.png)](./docs/img/architecture-diagram.png)

通过[`./pb` 文件夹](./pb)查找 **Protocol Buffers 描述文件**。 

| 服务                                                  | 语言          | 描述                                                                       |
| ---------------------------------------------------- | ------------- | -------------------------------------------------------------------------- |
| [frontend](./src/frontend)                           | Go            | 暴露一个HTTP服务器来服务网站。不需要注册/登录，自动的为所有用户生成session IDs。   |                                        |
| [cartservice](./src/cartservice)                     | C#            | 存储用户的购物车中的商品到Redis中并从Redis取出来。                              |
| [productcatalogservice](./src/productcatalogservice) | Go            | 提供JSON文件中的产品列表以及搜索产品和获取单个产品。                             |
| [currencyservice](./src/currencyservice)             | Node.js       | 将一种货币数量转换为另一种货币。它是最高QPS的服务。                              |
| [paymentservice](./src/paymentservice)               | Node.js       | 用给定的金额向给定的信用卡信息（模拟）收费，并返回交易ID。                        |
| [shippingservice](./src/shippingservice)             | Go            | 根据购物车给出运输成本估算。 将物品运送到给定的地址（模拟）                       |
| [emailservice](./src/emailservice)                   | Python        | 向用户发送订单确认电子邮件                                                    |
| [checkoutservice](./src/checkoutservice)             | Go            | 取出用户购物车，准备订单并安排付款，运输和邮件通知。                             |
| [recommendationservice](./src/recommendationservice) | Python        | 根据购物车中的商品，推荐其他商品。                                             |
| [adservice](./src/adservice)                         | Java          | 根据给定的上下文，提供文字类广告。                                             |                                                           |
| [loadgenerator](./src/loadgenerator)                 | Python/Locust | 将模拟现实用户购物流程的请求，连续发送到前端。                                   |

## 特性

- **[Kubernetes](https://kubernetes.io)/[GKE](https://cloud.google.com/kubernetes-engine/):**
  这个应用被设计在Kubernetes上运行 (本地的 "Docker for
  Desktop", 和云端 GKE).
- **[gRPC](https://grpc.io):** 微服务中大量使用gRPC调用来相互通信。
- **[Istio](https://istio.io):** 应用程序在Istio服务网格上运行。
- **[OpenCensus](https://opencensus.io/) Tracing:** Most services are
  instrumented using OpenCensus trace interceptors for gRPC/HTTP.
  跟踪:** 大部分服务使用OpenCensus来追踪gRPC/HTTP**.
- **[Stackdriver APM](https://cloud.google.com/stackdriver/):** 许多服务装配了**Profiling**, **Tracing** 和 **Debugging**。
  除此之外, 使用Istio来得到诸如请求/回应**Metrics**和**上下文图**开箱即用的特性。
  当程序不在谷歌云服务商运行时，这些代码的路径可能无效
- **[Skaffold](https://skaffold.dev):** 应用程序可以用个一个命令就可以部署到Kubernetes上。
- **Synthetic Load Generation:** 这个应用示例后台附带一个任务，通过 [Locust](https://locust.io/) 负载生成器在网站上
使用现实用户的使用模式。

## 安装

我们提供了三种安装方法：

1. **在本地的“Docker for Desktop中运行”** (~20分钟) 你将编译和部署微服务镜像到一个你的开发机上的单节点的Kubernetes集群。

2. **在谷歌Kubernetes引擎中(GKE)中运行”** (~30分钟) 你将构建，上传和部署容器镜像到谷歌云上的Kubernetes集群中。

3. **使用预构建容器镜像:** (~10分钟, 你仍需要依照上面的步骤，直到执行`skaffold run`命令）。
    有了这个选择后，你可以使用公开的预构建容器镜像来代替你自己去构建它们，自己构建太费时间了。

### 选择 1: 在本地的“桌面版Docker”运行

> 💡 如果你打算开发程序，或者只是尝试运行这个程序，这么推荐这种方式。

1. 安装工具来运行本地的Kubernetes集群:

   - kubectl (可以通过 `gcloud components install kubectl` 来安装)
   - 桌面版Docker (Mac/Windows): 他可以提供  Kubernetes 支持正如 [这里](https://docs.docker.com/docker-for-mac/kubernetes/)所说。
   - [skaffold]( https://skaffold.dev/docs/install/) (确保版本 ≥v0.20)

1. 启动 “桌面版Docker”. 找到 Preferences:

   - 选择 “Enable Kubernetes”,
   - 设置 CPU 数量最少三个, 内存至少6G
   - 在 "Disk" 选项, 设置至少 32 GB 的磁盘空间

1. 运行 `kubectl get nodes` 来验证你是否已经连接到 “Kubernetes on Docker”.

1. 运行 `skaffold run` (首次会很慢, 将会会费 ~20 分钟).
   这将会构建和部署应用。如果你需要在你修改代码时，自动的重新构建程序，那么用`skaffold dev`命令来运行。

1. 运行 `kubectl get pods` 来验证Pod已经就绪和运行。在你的机器上，应用前端应该会在 http://localhost:80 变得可用。

### 选择 2: 在谷歌Kubernetes引擎中(GKE)中运行

> 💡 假如你使用谷歌云服务平台并且在真实的集群中尝试，这么推荐这种方式。

1.  安装上述步骤中的指定的工具 (Docker, kubectl, skaffold)

1.  创建一个谷歌Kubernetes引擎，确保 `kubectl` 指向了这个集群。

    ```sh
    gcloud services enable container.googleapis.com
    ```

    ```sh
    gcloud container clusters create demo --enable-autoupgrade \
        --enable-autoscaling --min-nodes=3 --max-nodes=10 --num-nodes=5 --zone=us-central1-a
    ```

    ```
    kubectl get nodes
    ```

1.  在你的GCP项目中启用谷歌容器Registry (GCR)， 配置`docker` CLI 授权 GCR:

    ```sh
    gcloud services enable containerregistry.googleapis.com
    ```

    ```sh
    gcloud auth configure-docker -q
    ```

1.  在仓库的根目录, 运行 `skaffold run --default-repo=gcr.io/[PROJECT_ID]`,
    [PROJECT_ID] 是你的GCP项目ID。

    这个命令:

    - 构建容器镜像
    - 把他们推到GCR
    - 应用 `./kubernetes-manifests` 文件来部署程序到Kubernetes。

    **出错处理:** 如果你在Google Cloud Shell中看到 "No space left on device" 错误, 
    你可以在 Google Cloud Build 上构建
    [启用云构建API](https://console.cloud.google.com/flows/enableapi?apiid=cloudbuild.googleapis.com),
    然后运行`skaffold run -p gcb --default-repo=gcr.io/[PROJECT_ID]`替代上述命令。
   

1.  找到你应用的IP地址，然后访问它来确认安装

        kubectl get service frontend-external

    **出错处理:** 一个Kubernetes bug （将在1.12中修复）和Skaffold [bug](https://github.com/GoogleContainerTools/skaffold/issues/887),
    将导致即使在获得IP地址后，负载均衡器也不能工作。如果你看到这种情况，
    运行`kubectl get service frontend-external -o=yaml | kubectl apply -f-`来重新触发负载均衡器重配置。

### 选择 3: 使用预构建容器镜像

> 💡 如果你想用很少的步骤来快速部署应用到一个已经存在的集群，那么推荐你用这种方式。

**注意:** 假如你需要创建一个本地或者云上的Kubenetes集群，那么按照`选择 1` 或者 `选择 2`来直到你到`skaffold run`那步。

这个选择为你提供了一个预构建的镜像，它可以很简单的通过[release manifest](./release)来直接部署到一个已经存在的集群中。

**先决条件**: 一个已经在运行的Kubenetes集群 (本地或者云上都可以)。

1. 克隆这个仓库, 打开仓库所在目录
1. 运行 `kubectl apply -f ./release/kubernetes-manifests.yaml` 来部署应用。
1. 运行 `kubectl get pods` 查看pod是否就绪
1. 找到你应用的IP地址，然后访问它来确认安装。

   ```sh
   kubectl get service/frontend-external
   ```

### (可选) 部署在一个 装有Istio 的 GKE 集群中

> **注意:** 你需要按照上述GKE部署的步骤来，首先运行 `skaffold delete` 来删除已经部署的应用

1. 创建一个 GKE 集群 (在 "选择 2"中有描述)。

1. 使用 [Istio on GKE add-on](https://cloud.google.com/istio/docs/istio-on-gke/installing)
   来安装 Istio 到已经存在的 GKE 集群中。

   ```sh
   gcloud beta container clusters update demo \
       --zone=us-central1-a \
       --update-addons=Istio=ENABLED \
       --istio-config=auth=MTLS_PERMISSIVE
   ```

2. (Optional) Enable Stackdriver Tracing/Logging with Istio Stackdriver Adapter
   by [following this guide](https://cloud.google.com/istio/docs/istio-on-gke/installing#enabling_tracing_and_logging).

3. 安装自动sidecar注入(用label注解命名空间 `default` ):

   ```sh
   kubectl label namespace default istio-injection=enabled
   ```

4. 应用 [`./istio-manifests`](./istio-manifests) 中的清单文件.
   (只需要一次.)

   ```sh
   kubectl apply -f ./istio-manifests
   ```

5. 使用命令 `skaffold run --default-repo=gcr.io/[PROJECT_ID]`来部署

6. 运行 `kubectl get pods` 查看pod是否监控和就绪

7.  找到Istio 网关 Ingress或者服务的IP地址，然后访问应用

   ```sh
   INGRESS_HOST="$(kubectl -n istio-system get service istio-ingressgateway \
      -o jsonpath='{.status.loadBalancer.ingress[0].ip}')"
   echo "$INGRESS_HOST"
   ```

   ```sh
   curl -v "http://$INGRESS_HOST"
   ```

### 清理

如果你是通过`skaffold run`命令来部署的应用，你可以运行`skaffold delete`来清理已经部署的资源

如果你是通过`kubectl apply -f [...]`来部署的应用，你可以用`kubectl delete -f [...]`运行同样的参数来清理已部署资源

## Conferences featuring Hipster Shop

- [Google Cloud Next'18 London – Keynote](https://youtu.be/nIq2pkNcfEI?t=3071)
  showing Stackdriver Incident Response Management
- Google Cloud Next'18 SF
  - [Day 1 Keynote](https://youtu.be/vJ9OaAqfxo4?t=2416) showing GKE On-Prem
  - [Day 3 – Keynote](https://youtu.be/JQPOPV_VH5w?t=815) showing Stackdriver
    APM (Tracing, Code Search, Profiler, Google Cloud Build)
  - [Introduction to Service Management with Istio](https://www.youtube.com/watch?v=wCJrdKdD6UM&feature=youtu.be&t=586)
- [KubeCon EU 2019 - Reinventing Networking: A Deep Dive into Istio's Multicluster Gateways - Steve Dake, Independent](https://youtu.be/-t2BfT59zJA?t=982)

---

This is not an official Google project.
