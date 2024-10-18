+++
title = 'resource/kind, client-go 和 apimachinery'
date = 2024-08-27T15:00:00+08:00
tags = ["container"]
+++

> 这篇文章是我在[工作室 wiki 上的文章](https://west2-online.feishu.cn/wiki/QP2NweWRViTbSZkjThGcwS7Pnwh)转载，原文章是富文本，比目前阅读来说会更加突出重点一些

## 问题背景

这个暑假在做一个推荐器的重构工作，先前的推荐器使用了 k8s 自身的 controller-runtime 库来实现，但是这个项目内联了一套自己的控制器框架与实现流程，很明显需要精简掉这个原生的控制器框架。我原先也觉得挺简单的，实际上手后难度还是蛮大的....但不是代码难写，流程、概念太多了

经过一段时间的研究，注意到这个项目自带的这个框架更像是做了一层剥离，直接提供出最直接的对象，例如 Informer、DynamicClient 等，自然问题就转到了 client-go 这个 k8s 官方提供的客户端框架和 apimachinery 这个 api 库了，实话实说比以往的一些项目来的折磨，因为要比较熟悉 k8s 的一些概念以及它的设计理念

我现在(2024.08)写的重构推荐器仅支持 Deployment，不支持诸如 StatefulSet 等其他 Kind，因为在代码中写死了通过 一个 DeploymentGetter 来获取 Deployment，而没有事先判定资源的类型。导师 review 代码后提出了使用 Unstructed 结构体来实现通用的 Kind。这句话突然让我意识到了为什么说 k8s 中一切皆资源，这也是这个文章的来由。

> 写这个的原因主要也是对自己这段时间的学习做一个总结，自己真的是太菜了(复读)。在这里涉及到具体的情况，会用我这段时间使用的 katalyst 来作为例子

后续的内容会以这个项目的推荐器为基础，为大家逐步了解资源的概念、client-go 库的作用以及实际用法、apimachinery 的作用

## "k8s中一切皆资源"

### 从一个例子说起

对于没有接触过的同学（包括前不久的我）来说，“没有事先判定资源的类型”这句话每个字都看得懂，但凑起来就看不懂了。我们先从一个标准的 Deployment 部署 yaml 来说起

> 前段时间和同学相约回高中和高中的竞赛教练叙叙旧，饭后闲余时间和朋友聊起了这个项目，直接被大量的专有名词搞晕了。所以对于一名有计算机背景的同学来说，k8s 也不是那么快速上手的，虽然到处都是 yaml，但是概念绕绕弯弯，如果你同样是零基础，还是不要跳过下面这个配置的注释

```yaml
apiVersion: apps/v1  # 指定了使用的 Kubernetes 的 API 版本
kind: Deployment     # 标注这个配置的资源类型是 Deployment
metadata:
  name: shared-normal-deplo0yment
  namespace: default
spec:
  replicas: 1 # 副本数量，即我们需要为这个Deployment 创建多少个Pod
  selector:
    matchLabels:
      app: shared-normal # 进行 Pod 筛选，Deployment 会管理带有这些标签的 Pod
  template:
    metadata:
      labels:
        app: shared-normal # Pod的标签
      annotations: # annotations 即为注解，我们可以理解为为这个Pod带上了额外的信息
        "katalyst.kubewharf.io/qos_level": shared_cores
    spec:
      containers:
      - name: stress # 容器名称
        image: joedval/stress:latest
        command: # 启动时执行的命令
          - stress
          - -c
          - "1"
        imagePullPolicy: IfNotPresent # 拉取的策略，这里说的是如果已经存在则不再拉取
        resources: # 资源请求和限制
          requests:
            cpu: "2"
            memory: 1Gi
          limits:
            cpu: "2"
            memory: 1Gi
      # schedulerName: katalyst-scheduler # 指定调度器，这里被注释掉就使用默认调度器
```
如果是一个纯新手，看着这些注释可能也不太理解，我们可以用白话来翻译：

> 我们创建一个资源，这个资源类型是 Deployment，这个资源定义了一个管理策略，管理所有 app 标签为`shared-normal`的 Pod，并且我们期望这个 Pod 的规格如下，它有一个容器，这个容器的名字是....

这样子你就可以理解到我们实际上是声明了一个管理策略，同时描述了一下我们所管理的 Pod 的期望状态。如果你进行了实践，比如通过`kubectl apply`了这个配置，你肯定会有另一个疑惑——这不是描述了一个管理的期望状态？那为什么我 apply 后会自动帮我创建 Pod？

k8s 在解析完这个配置后，会触发 Deployment 控制器的响应，这个控制器需要确保我们声明中的期望状态和集群的实际状态所匹配。当我们在一个空集群应用了这个配置，Deployment 控制器会触发响应，然后会捕获到“集群中没有这个 Pod”的情况，之后就会自动创建 Pod。

现在让我们先把重点放在这个 Kind，其他的先忽略，我们以问题开始：这个 Kind 是只有固定几种的吗？如果我们要自定义一个新的 Kind，是可行的吗？如果可行，应该怎么做？

在我刚开始的时候，我一直以为这个 Kind 是固定的，它就只有 Deployment、StatefulSet、DaemonSet 等几类类型，甚至我使用的`appsv1.AppsV1Interface`中间列举的也就这几种，这更加让我陷入了自我误导的陷阱中

> 有兴趣的可以在自己的 k8s 集群中敲一个`kubectl api-resources`，它会列举这个集群中目前支持的所有 API 资源，然后你可以敲一下`kubectl api-resources | grep deployment`，你会发现 Deployment 只是这其中的一个Kind


事实上像 Deployment 这种就是个内建类型，deployments 就是个内建资源，这些都是可以拓展的，具体可以看下面这一部分

### CR/CRD 和 Kind

这需要和 CR/CRD 进行结合，如果你先前有了解，你肯定知道这个是 CustomResource(Definition)的简写，从字面就能知道它的意思，但我们往往忽略了它的实际用意——当我们声明一个 CRD 和 CR 时，实际上在做什么？为什么有 CRD 还需要有一个 CR？

当我们通过kubectl apply一个 CRD 时，k8s 会在 api server中动态注册一个新的路径，这样可以通过 k8s api 来和你的自定义资源进行交互。这也意味着我们可以像使用内置资源（比如 Pod 和 Deployment）一样通过 kubectl 来增删改查我们自己的自定义资源

当我们创建 CRD 时，我们只是说：我们有这样一个资源 x，它的名称是 a，API 组是 b，版本和元数据分别是 c 和 d。这就好像我们写了一个 Interface，里面定义了具体函数的参数和返回值，但是没有进行实现

而 CR 做的就是“实现”这个步骤，CR 包含了资源的规格（spec），而规格可以理解为这个资源的期望状态，但我们声明期望状态时肯定希望有一个“范围”，总不能给全部资源都套上这个期望吧。这便是matchLabels的作用，之后这个资源的控制器会接收到你 CR 的创建、更新和删除的事件，并进行响应，这就是后面会涉及到的informer机制

CRD 通常是经过我们编写指定的 go 文件，如katalyst-api 中写了 recommendation 资源的 CRD代码（具体在这），再通过controller-gen，k8s 官方编写的一个代码/配置生成工具，为我们来生成 CRD yaml。这个生成出来的是一个挺大的配置文件。

我们使用 CR 时，则是基于 CRD 来进行编写，以这个 recommendation 为例，我们的 CR 长这样

```yaml
apiVersion: recommendation.katalyst.kubewharf.io/v1alpha1 # 我们使用的api版本
kind: ResourceRecommend # 就是我们定义的资源类型，这个要和 CRD 中的一致
metadata:
  name: shared-normal-deployment-recommendation
  namespace: default
spec:
  resourcePolicy:
    algorithmPolicy:
      algorithm: percentile # 推荐器所使用的算法
      extensions: {}
      recommender: default  # 推荐器
    containerPolicies:      # 我们对容器的策略
      - containerName: "*"  # 匹配所有的容器
        controlledResourcesPolicies:
          - resourceName: "cpu" # 对 CPU 资源进行策略配置
            bufferPercent: 10
            controlledValues: "RequestsOnly"
            minAllowed: "1"
            maxAllowed: "3"
          - resourceName: "memory"
            bufferPercent: 10
            controlledValues: "RequestsOnly"
            minAllowed: "500Mi"
            maxAllowed: "2Gi"
  targetRef: # 这个CR 所匹配的资源
    apiVersion: "apps/v1"
    kind: "Deployment"
    name: "shared-normal-deployment"
```

除去 CR 中具体的一些配置，我们可以把目标放在targetRef中，这个指的是这个 CR 期望匹配的资源的类型。

> 我们把所有东西都抽象成了资源，而各个东西之间的绑定实际上就是资源的绑定，在这里我们把 CR 绑定在了符合 targetRef 信息的资源中（在这里是一个 deployment，它的名字是 shared-normal-deployment）

当我们 apply 这个配置（当然，在apply这个 CR 前需要先 apply 它的 CRD，你肯定是先写 Interface 再写具体的 Implement 的）后，k8s 会寻找是否有指定的控制器（controller）负责管理这个 CR，一旦找到了，就会给这个控制器的 Informer 发送响应，进而被控制器进行接管，我们可以以代码为例。

```go
// 注册相应的回调事件，这个 recInformer 就是我们前面这个 CR 的 Informer
// 当遇见 CR 被添加、更新时进行回调函数的执行，而 Informer 会对回调函数传入这个资源的 obj 对象
recInformer.Informer().AddEventHandlerWithResyncPeriod(cache.ResourceEventHandlerFuncs{
    AddFunc:    recController.addRec,
    UpdateFunc: recController.updateRec,
}, recConf.RecSyncPeriod)
```

这样就会相对清楚了，k8s 中不论是 Deployment 还是我们自定义的 Recommendation 这样的东西，它都是“资源-控制器”组合，除了用 Interface 和 Implement 做类比，我觉得更好的还是将其理解为数据结构-基于这个数据结构的变量-对这个变量进行操作的函数。现在我们定义完了资源，就要开始思考我们该如何编写这个控制器，这就要引出 client-go 和 apimachinery 两个库了

### 怎么创建一个 CR/CRD？

在开始思考编写控制器前，插播一下 CR/CRD 的创建，实际上是通过k8s 的一个项目 kubernetes/code-generator 来实现的，这个代码生成工具为我们生成CRD，我们便可以通过 CRD 进一步生成 CR。

> 实际上这个项目不止有这样的功能，它还可以同步生成 lister、informer 的代码以及其他代码。ister 用于枚举特定的资源（通过 namspace 和 name），informer 则负责监视(watch)资源变化及对特定情况进行事件触发

也就是说，创建 CR/CRD 实际上是通过先编写go 代码，后通过生成工具生成的，我们可以参考这个 Recommendation CRD 的代码，定义了很多结构体，并且在中间插入代码生成工具的指令（如一些 comment 是// +k8s xxx这样的格式），定义完后我们通过 codegen cli 就可以生成指定的CRD yaml，再通过 kubectl apply 上传至集群。

至此我们的 CRD 已经做完了，接下来还需要做是基于 CRD 去生成 CR，而此时我们就可以按照 CRD 这个 yaml 中的内容来编写，但是这个文章主要还是捋一下大纲，具体的细节，比如如何去根据这个 CRD 来编写我们的 CR，这里就不展开了

## Resource? Kind?

恐怕很多人对这两个概念很奇怪，我是从 GVR、GVK 开始困惑的——这似乎是一个东西。

从一个例子开始，Deployment，这个查一查就知道是什么东西，实际上也是一个资源——吗？不是的，Deployment 是 Kind，而 deployments 是资源。

> - Kind 指的是一个 Kubernetes 对象的类型。在 Kubernetes API 中，每个对象都有一个 Kind 字段，它表示该对象的类型，比如 Pod, Service, Deployment 等。Kind 是一个抽象的概念，它定义了一个对象应有的属性和行为。
> - 在 Kubernetes 的 API 定义中，Kind 是在 Go 语言的结构体中定义的，这个结构体描述了一个对象的规范（Spec）和状态（Status）。


Resource 和 Kind 之间的关系更像是 CR/CRD 之间的关系，定义一个标准（kind）而具体使用时则是 Resource

这看起来更像是一种解耦，但实际上也有它的大智慧，比如说不同的 Kind 实际上可能会管理到同一个资源——比如 HPA（HorizontalPodAutoscaler）和 Deployment 两个 Kind 都可能会管理到 ReplicaSet

> Deployment 控制器负责管理 ReplicaSet，它会根据 Deployment 的定义来创建、更新和删除 ReplicaSet。当你创建或更新一个 Deployment 时，Deployment 控制器会相应地创建或更新一个 ReplicaSet。
>
> 而HorizontalPodAutoscaler (HPA) 负责自动调整目标资源（如 Deployment、ReplicaSet 或 StatefulSet）的副本数量。HPA 通过 API 监控目标资源的性能指标，然后根据这些指标来调整目标资源的 spec.replicas 字段，这实际上会影响到由目标资源控制的 ReplicaSet。

## 通过 Client-go来管理我们的资源

我们可以通过 client-go 库开对 k8s 进行一些管理。这个库提供了 k8s 集群的一种抽象，我们可以通过这个库与 k8s 集群进行交互，就像它们 README 中说的

> Go clients for talking to a kubernetes cluster.

这个库是比较庞大的，而我们只需要用到其中的一些内容，可以先从 client-go 的结构开始说起，但会更侧重于如何实现这样一个控制器。

> 这样的一个控制器：指的是管理某个 CR 的 Controller，在这里我们沿用前面提到的 ResourceRecommend 这个 CR

### 基本结构

![](https://img.w2fzu.com/etc/202410181522309.png)

通用标准上 Client-go 库支持 4 种客户端，其中 RESTClient 客户端是最基础的，在它上面的 ClientSet、DynamicClient、DiscoveryClient 都是基于它来实现的，在这里我们可以进行一些简单的区别

| Client | 解释 |
| --------------- | ------------------------------------------------------------ |
| ClientSet       | 在 RESTClient 的基础上封装了对 Resource、Version 的管理方法，但只能管理 k8s 自带的资源，这和下文提到 katalyst 内建的 KubernetesClient 是一回事 |
| DynamicClient   | 在 ClientSet 的基础上进行拓展，它可以管理 k8s 中的所有资源，不论是内置资源还是 CRD，它不需要预先定义的 Go 类型来表示资源，而是使用 `unstructured.Unstructured` 类型来处理所有的 Kubernetes 资源 |
| DiscoveryClient | 用于发现 kube-apiserver 所支持的GVR（Group、Version、Resources），即资源组、资源版本和资源信息，我们可以使用这个客户端来了解集群中可用的 API 资源，也可以利用它来构建 mapper，形成对 Kind 和 Resource 的映射 |

现在我们就清晰多了，如果我们要管理我们上文提到的这个 Recommendation CR，我们肯定是需要基于 DynamicClient 来编写代码，因为它可以管理 CR 资源。

> 实际上在挖katalyst 代码的时候注意到 client-go 还有一个 metadataClient，它用于访问集群中所有资源的元数据。只能拿到例如资源版本、标签这一类的，不包含 spec 和 status 内容

### 为什么需要，以及怎么进行资源校验

> 这个问题实际上是我接过项目后问导师的第一个相较于其他问题来说有点质量的问题....这看起来非常不可理喻，我们发布了这个 CR，为什么还需要我们在修改前进行校验？导师说明是为了避免这个配置被其他的东西给修改掉，我们才需要校验。但这个“其他的东西”是什么？

在 k8s 中，不只是控制器可以编辑 CR，外部其他条件（如管理集群的用户本身）也可以修改 CR，但修改后的 CR 不一定满足我们的要求。以前面的 CR 为例，它要成为合格的 CR，就要符合 CRD 中的内容，而不是省略了一些必要字段。（这里没有说多一些内容，是因为 Controller 不会去管理多出来的东西，它只对它“所需要的”字段进行校验）

校验的过程没有那么难懂，但这里的“没那么难”也是后话了，初接触的可能会懵逼好一阵。从整体上来说，校验实际上就是从集群中取出这个 CR -> 将 CR 的 yaml 映射到结构体中 -> 对每个结构体进行检查.

#### 取出CR

还是这个项目 [kubernetes/code-generator](https://github.com/kubernetes/code-generator)，它会生成 lister 的代码，而这个 lister 负责枚举我们的 CR 资源。生成 lister 和生成 CRD 的文件是同一个（如前面提到的[Recommendation CRD](https://github.com/kubewharf/katalyst-api/blob/main/pkg/apis/recommendation/v1alpha1/types.go)），它只需要保证在符合一定规范的目录下，便可以一份代码通过多个生成工具生成不同类型的产物。

生成好 lister 代码后我们就可以引用这个 lister 来使用了，但是大家想过这个 lister 该如何用上吗？这边以 katalyst 为例子，来稍微入门一些些（如果觉得晦涩难懂可以跳过，尤其是大一的同学）

我读了一下他们自己的一个实现，Katalyst 有一个自己内部的一个 GenericContext（直译过来就是通用上下文），这个 Context 负责了各个client、Informer 的初始化，后续要用到的 informer、lister、client 都可以从这里取出来

首先将目标关注到[这个文件](https://github.com/kubewharf/katalyst-core/blob/main/cmd/katalyst-controller/app/controller.go)上（建议左右横屏对比着看），在这里有一个很明显的 Run 方法，其中有一句

```go
// line 49
clientSet, err := client.BuildGenericClient(conf.GenericConfiguration.ClientConnection, opt.MasterURL,
    opt.KubeConfig, fmt.Sprintf("%v", consts.KatalystComponentController))
```

这个函数负责构建通用客户端，在先头的一些前置处理（比如 BuildConfigFromFlags），又经过几个函数的跳转，会在[这个文件](https://github.com/kubewharf/katalyst-core/blob/main/pkg/client/genericclient.go)中的`newForConfig()`进行具体的客户端初始化，其中像 metaClient、kubeClient 这些都是直接引用client-go这个包中的初始化函数，但是还有一个他们自己的`internalClient`，你会发现这个初始化函数是在katalyst-api中的，我们定位到[这个文件](https://github.com/kubewharf/katalyst-api/blob/main/pkg/client/clientset/versioned/clientset.go)中，在第 137 行开始了它的初始化步骤，我们截取部分关键代码

```go
func NewForConfigAndClient(c *rest.Config, httpClient *http.Client) (*Clientset, error) {
    /* 省略代码 */
    cs.overcommitV1alpha1, err = overcommitv1alpha1.NewForConfigAndClient(&configShallowCopy, httpClient)
    if err != nil {
       return nil, err
    }
    cs.recommendationV1alpha1, err = recommendationv1alpha1.NewForConfigAndClient(&configShallowCopy, httpClient)
    if err != nil {
       return nil, err
    }
    /* 省略代码 */

    cs.DiscoveryClient, err = discovery.NewDiscoveryClientForConfigAndClient(&configShallowCopy, httpClient)
    if err != nil {
       return nil, err
    }
    return &cs, nil
}
```

既然是推荐器，我们自然会将目标放在`recommendationv1alpha1.NewForConfigAndClient`这个函数上，结果一跳进去你就会打通任督二脉——这个函数实际上是 client-gen 代码生成工具生成的！

那么一切都明晰了，我们在初始化控制器前先初始化了GenericContext，而这个 GenericContext 实际上除了对各个 k8s 客户端做初始化，也封装了自己的一个 InternalClient，在这里我们对我们自定义 CRD 的几个client、informer 进行初始化

如果你看的仔细，你会发现这个实际上初始化的是一个 client，那我们前面说的 informer，lister 又在哪？把目光放回 NewGenericContext() 这个方法上，当我们调用完 BuildGenericClient()函数，并设置一些信号量（处理优雅退出）后就是这个方法，具体在[这个文件](https://github.com/kubewharf/katalyst-core/blob/main/cmd/base/context.go)中，这个方法中会对 InternalInformerFactory 做初始化，具体代码在

```go
// line 122
internalInformerFactory = externalversions.NewSharedInformerFactoryWithOptions(clientSet.InternalClient, time.Hour*24,
    externalversions.WithTweakListOptions(func(options *metav1.ListOptions) {
       options.LabelSelector = labelSelector
    }))
// 其他几个 Informer 工厂初始化调用的也都是 client-go 库的，而这个内建 Informer 工厂则是 katalyst-api 提供的，和前面客户端初始化类似
```

而这个`NewSharedInformerFactoryWithOptions`实际上就是做了一个对象的封装，往里面塞了我们前面初始化得到的 client，之后就完成了初始化过程（其他不是很有必要的就省略了），而我们使用的时候只需要通过以下代码就可以拿到 informer 和 lister，这时候再回顾一下这两个的作用，informer 是负责事件通知，lister 是负责资源获取

```go
recInformer := controlCtx.InternalInformerFactory.Recommendation().V1alpha1().ResourceRecommends()
recLister:  recInformer.Lister()
// 其中 controlCtx 就是我们前面初始化的通用上下文，然后我们就可以通过下面这个来获取到了

recLister.ResourceRecommends(namespace).Get(name) // 获取精确资源
recLister.ResourceRecoomends(namespace).List(labels.everything()) // 获取资源列表（按照 label 筛选，此处 everything 指的就是全部的资源）
```

#### 将 CR 映射到结构体中

在 katalyst 这个框架设计下，实际控制器运行时，informer 发起响应事件会自动附带这个 CR 的 obj（如果是 update 事件会附带 old CR 和 new CR 的 obj），我们做一个转换就可以拿到了，这个 Lister 的作用实际上是没有的。因此我们可以直接拿到 CR 的结构体

例如，我们注册回调事件后，代码是这样的

```go
// 在 Controller 初始化时注册回调函数
recInformer.Informer().AddEventHandler(cache.ResourceEventHandlerFuncs{
    AddFunc:    recController.addRec,
    UpdateFunc: recController.updateRec,
    DeleteFunc: recController.deleteRec,
})

// 某个回调函数
func (rrc *ResourceRecommendController) addRec(obj interface{}) {
    v, ok := obj.(*v1alpha1.ResourceRecommend) // 直接转换到 CR 结构体
    if !ok {
       klog.Errorf("[resource-recommend] cannot convert obj to *apis.ResourceRecommend: %v", obj)
       return
    }
    klog.V(4).Infof("[resource-recommend] notice addition of ResourceRecommend %s", v.Name)
    rrc.enqueueRec(v)
}
```

#### 对 CR 进行校验

既然直接映射到结构体，我们的校验就方便很多了，把目标放在[这个文件](https://github.com/kubewharf/katalyst-core/blob/main/pkg/util/resource-recommend/types/recommendation/recommendation.go)上，其中有一个函数`SetConfig()`，这个函数将会校验CR 并提取出我们所需要的东西，而这几个校验方法都被写在了[这个文件](https://github.com/kubewharf/katalyst-core/blob/main/pkg/util/resource-recommend/types/recommendation/validate.go)中，仔细阅读可以发现实际上的校验就是检查必须项是否为空、是否内容不符合我们所需要的内容

比如，在`ValidateAndExtractTargetRef`函数中，我们会检查 targetRef 的 Name 是否为空（我们需要根据这个 Name 提取出我们想要的Pod，因此肯定不能为空）。而在`ValidateAndExtractAlgorithmPolicy`函数中，我们也会检查AlgorithmPolicy 字段内容是否是我们内部支持的算法之一

## 那 apimachinery 有什么用呢？

是不是觉得到这里就结束了？我们还没提 apimachinery 这个库，但它穿插在整个本身

### 从总结开始

我们前面介绍了 k8s 的资源、CRD/CR、client-go 这个库以及基本的使用方法、informer、lister、codegen等，那它们之间的关系是什么？这里还是用之前那个问题作为解答：我们应该如何编写一个完备的自定义控制器？

需要编写自定义控制器，那么我们肯定需要先编写 CRD，先对我们的资源内容进行一个定义。之后我们需要为特定的应用部署自己的 CR，这个 CR 是基于 CRD 来进行编写的。而 CRD 怎么编写？我们是倒推的——先写 CRD 的结构体，之后利用 codegen 利用这个结构体来生成CR，Lister、Informer、Client 等信息。

Apply CR 后，我们还需要对这个 CR 进行管理，也就是我们的控制器，而控制器肯定要取出CR，进行自己的控制流程，之后再应用新的 CR。比如这个推荐器，需要取出 CR，然后对应用进行重新计算推荐资源，再部署新的 CR 回去，这个步骤会涉及到 client-go，我们需要和集群做交互。

这里面就会涉及到一个细节，控制器肯定不是做轮询的，一个一个应用遍历检查，这样太耗费时间，而且产生了不必要的算力浪费，因此我们有 Informer，它专门负责监控我们的资源（不论是 CR 自定义资源还是 Pod、Deployment 这样的内建资源）是否产生了修改/新增/删除事件，当产生事件时我们再执行回调函数（callback），此时才由我们控制器来接管、处理。

> 以我们的推荐器Controller 为例，我们在初始化完 Controller 后会有多个 worker 以协程方式运行在后台，当 informer 调用回调函数时，会提取出 CR，然后将这个 work 加入 workqueue，一旦加入 workqueue，我们的 worker 就会从队列中提取出来这个任务，然后完成它。一切都是异步的，很好的体现了 go 的协程特性。
现在我们捋清了这些关系，那apimachinery 库在这之间有什么用呢？

### 基础设施的角色

> apimachinery 库在 Kubernetes 的自定义控制器开发中扮演着基础设施的角色

实际上，在我们编写最初的 CRD 的结构体文件时，就已经用到了 apimachinery，不信你回到[这个文件](https://github.com/kubewharf/katalyst-api/blob/main/pkg/apis/recommendation/v1alpha1/types.go)去看一下 import。在这个库的 README 中，k8s 官方是这么定义的

> Scheme, typing, encoding, decoding, and conversion packages for Kubernetes and Kubernetes-like API objects.

这个库包含了各种类型的对象定义，比如 ObjectMeta、TypeMeta，亦或是客户端连接设置（比如 rest.Config），或者资源处理接口（runtime.Object）。请注意，这些都是一些设置、对象定义，亦或是一些通用的函数。而通常没有具体的逻辑实现

更广泛的说这个库就是抽象出一些非常通用的内容，然后独立出来，这样诸如 client-go，以及我们自定义控制器，都可以直接复用这个库来达到统一的标准，这也是它的设计哲学——通用的、可扩展的、可复用的组件，这也是被称之为基石的原因，我们基于它来构建复杂的控制器逻辑，同时保持代码的简洁和高效

比如我们前面提到的 client-go，也引用了 apimachinery，它更像是提供了一个标准来让围绕 k8s 进行的开发更加统一


### Katalyst-API 和 apimachinery 是否是同一个意思

如果大家有去了解 [katalyst-api](https://github.com/kubewharf/katalyst-core) 这个库，可以感觉到这个库似乎就是 apimachinery的翻版，甚至设计哲学都能对得上，但是在我的理解中，它更像是 client-go 和 apimachinery 的结合，我们的katalyst-core就是基于这个 katalyst-api 库来构建的

## 参考

| 标题                         | 地址                                                         |
| ---------------------------- | ------------------------------------------------------------ |
| Client-go 四种客户端交互对象 | https://yuanshisen.com/Client-go%20%E5%9B%9B%E7%A7%8D%E5%AE%A2%E6%88%B7%E7%AB%AF%E4%BA%A4%E4%BA%92%E5%AF%B9%E8%B1%A1/index.html |
| 深入 Kubernetes API 源码实现 | https://morven.life/posts/k8s-api-2/                         |
| katalyst-core                | https://github.com/kubewharf/katalyst-core                   |
| katalyst-api                 | https://github.com/kubewharf/katalyst-api                    |