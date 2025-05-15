[TOC]



## 了解Argo-CD

```sh
Argo CD 是以 Kubernetes 作为基础设施，遵循声明式 GitOps 理念的持续交付（continuous delivery, CD）工具，支持多种配置管理工具，包括 ksonnet/jsonnet、kustomize 和 Helm 等。它的配置和使用非常简单，并且自带一个简单易用的可视化界面。
按照官方定义，Argo CD 被实现为一个 Kubernetes 控制器，它会持续监控正在运行的应用，并将当前的实际状态与 Git 仓库中声明的期望状态进行比较，如果实际状态不符合期望状态，就会更新应用的实际状态以匹配期望状态。
```

### GitOps 是什么？

- **参考文献：[What is GitOps and what should you know about it?](https://devs-group.medium.com/what-is-gitops-and-what-should-you-know-about-it-fcaa3e3c1dd5)**

#### DevOps你肯定很熟悉，所以GitOps vs DevOps比较一下大家就能快速了解GitOps到底是什么

![img](https://img2024.cnblogs.com/blog/3394728/202404/3394728-20240420102027432-1475057372.png)

```sh
从广义上来看，GitOps 与 DevOps 并不冲突，GitOps 是一种技术手段，而 DevOps 是一种文化。GitOps 是一种实现持续交付（Continuous Delivery）、持续部署（Continuous Deployment）和基础设施即代码（IaC）的工具和框架，它是支持 DevOps 文化的。

从狭义上来看，GitOps 与 DevOps 有以下几个区别：

首先，GitOps 是以目标为导向的。它使用 Git 来维护期望状态，并不断调整实际状态，最终与期望状态相匹配。而 DevOps 更多关注的是最佳实践，这些实践可以普遍应用于企业的每一个流程。

其次，GitOps 采取声明式的操作方法，而 DevOps 同时接受声明式和命令式的方法，所以 DevOps 除了适用于容器环境之外，还适用于虚拟机和裸机环境。

最后，GitOps 重新定义了云原生场景下的 CI/CD，它以 Git 作为中心的不可变状态声明，以加快持续部署速度。
```

#### GitOps 的设计哲学

```sh
1. 声明式
必须通过声明式来描述系统的期望状态。例如 Kubernetes，众多现代云原生工具都是声明式的，Kubernetes 只是其中的一种。

2. 版本控制/不可变
因为所有的状态声明都存储在 Git 仓库中，并且把 Git 仓库作为单一事实来源，那么所有的操作都是从 Git 仓库里驱动的，而且保留了完整的版本历史，方便回滚。有了 Git 优秀的安全保障，也可以使用 SSH 密钥来签署 commits，对代码的作者和出处实施强有力的安全保障。

3. 自动应用变更
Git 仓库中声明的期望状态发生了任何变更，都可以立即应用到系统中，而且不需要安装配置额外工具（比如 kubectl），也不需要配置 Kubernetes 的认证授权。

4. 持续的 Reconciliation
Reconciliation 其实最早是 Kubernetes 里的一个概念，表示的是确保系统的实际状态与期望状态一致的过程。具体的实现方式是在目标环境中安装一个 agent，一旦实际状态与期望状态不匹配，agent 就会进行自动修复。这里的修复比 Kubernetes 的故障自愈更高级，即使是手动修改了集群的编排清单，集群也会被恢复到 Git 仓库中的清单所描述的状态。
```

#### Push vs Pull

- **CD 流水线有两种模式：Push 和 Pull**

##### Push 模式

```sh
目前大多数 CI/CD 工具都使用基于 Push 的部署模式，例如 Jenkins。这种模式一般都会在 CI 流水线运行完成后执行一个命令（比如 kubectl）将应用部署到目标环境中。

这种 CD 模式的缺陷很明显：

需要在环境安装配置额外管理工具（比如 kubectl）；
需要 Kubernetes 对其进行授权；
需要云平台授权；
无法感知部署状态。也就无法感知期望状态与实际状态的偏差，需要借助额外的方案来保障一致性。
Kubernetes 集群或者云平台对 CI 系统的授权凭证在集群或云平台的信任域之外，不受集群或云平台的安全策略保护，因此 CI 系统很容易被当成非法攻击的载体。
```

##### Pull 模式

```sh
Pull 模式会在目标环境中安装一个 Agent，例如在 Kubernetes 集群中就靠 Operator 来充当这个 Agent。Operator 会周期性地监控目标环境的实际状态，并与 Git 仓库中的期望状态进行比较，如果实际状态不符合期望状态，Operator 就会更新基础设施的实际状态以匹配期望状态。

只有 Git 的变更可以作为期望状态的唯一来源，除此之外，任何人都不可以对集群进行任何更改，即使你修改了，也会被 Operator 还原为期望状态，这也就是传说中的不可变基础设施。

目前基于 Pull 模式的 CD 工具有 Argo CD，Flux CD 以及 ks-devops。
```

##### GitOps 的优势

- 一般 GitOps 首选的都是基于 Pull 的部署模式，因为这种模式有很多不可替代的优势。

```sh
3.1更强大的安全保障
上面已经提到了，使用 GitOps 不需要任何 Kubernetes 或者云平台的凭证来执行部署，Kubernetes 集群内的 Argo CD 或者 Flux CD 只需要访问 Git 仓库，并通过 Pull 模式来更新即可。

另一方面，Git 由用于跟踪和管理代码变更的强大密码学支持，拥有对变更进行签名以证明作者身份和来源的能力，这是保障集群安全的关键。

3.2Git 作为事实的唯一真实来源
因为所有的应用包括基础设施的声明式配置都保存在 Git 中，并把 Git 作为应用系统的唯一事实来源，因此可以利用 Git 的强大功能操作所有东西，例如版本控制、历史记录、审计和回滚等等，无需使用 kubectl 这样的工具来操作。

3.3提高生产力
Git 也是开发人员非常熟悉的工具，通过 Git 不断迭代，可以提高生产率，加快开发和部署速度，更快地推出新产品，同时提高系统的稳定性和可靠性。

3.4更容易合规的审计
使用 GitOps 的基础设施可以像任何软件项目一样使用 Git 来管理，所以同样可以对其进行质量审计。当有人需要对基础设施进行更改时，会创建一个 Pull Request，等相关人员对其进行 Code Review 之后，更改才可以应用到系统中。
```

##### 总结

```sh
GitOps 是对现有 DevOps 文化的补充，它使用 Git 这样的版本控制系统来自动部署基础设施，部署过程清晰可见，可以查看和跟踪对系统进行的任何变更，提高了生产力、安全性和合规性。而且 GitOps 提供了更优雅的可观测性，可以实时观测部署状态，并采取行动使实际状态与期望状态保持一致。

而且在 GitOps 中，整个系统都是通过声明式来描述的，天然适合云原生环境，因为 Kubernetes 也是这么设计的
```

### 为什么要使用Argo CD？

#### **先来看看传统 CD 工作流**

```sh
这种 CD 模式存在一些明显的缺陷：

需要安装和配置额外的工具（例如 kubectl）；
需要 Kubernetes 进行授权；
需要获得云平台的授权；
无法感知部署状态，因此无法准确评估期望状态与实际状态之间的偏差，需要依赖其他方案来确保一致性。
```

#### Argo CD 相对于传统 CD 工具的“优越之处”

##### Git 作为应用的主源

```sh
在 Argo CD 的世界里，Git 就是他的皇帝，所有 Kubernetes 的配置都藏在 Git 仓库里。不需要再费力地手动更新应用，也不必担心命令行操作搞砸一切。只需通过 Git 这个“万能钥匙”就可以轻松搞定。

更棒的是，Argo CD 还能负责监控 Git 仓库里的期望状态，同时也会查看集群中应用的实际状态。两者对比一下，只要不对味，Argo CD 就会出手帮你调整，保证一切按照预期运行。你把集群状态想象成舞台上的角色，Argo CD 就是你的导演，时刻保证他们在台上表演得有模有样。

有时候，我们可能想快速更新应用或者测试一下效果。此时，通过 Git 更新可能有点慢，别着急，我们可以调整一下 Argo CD 的设置，让它对手动修改的部分“睁一只眼闭一只眼”，只发出一声警告，提醒管理员要记得把更新提交到 Git 仓库中。
```

##### 快速回滚

```sh
Argo CD 定期从 Git 拉取最新配置并将其应用到集群中。一旦发现最新配置导致应用出现故障（例如应用启动失败），我们可以通过 Git 历史记录迅速将应用状态回滚到之前的可用状态。

如果你在多个 Kubernetes 集群中使用同一个 Git 仓库，那这个功能的价值将更加凸显。你不需要在不同集群中分别执行 kubectl delete 或 helm uninstall 等命令手动回滚，只需将 Git 仓库回滚到之前的版本，Argo CD 就会自动同步并将应用状态还原。
```

##### 灾备利器

```sh
如果你的 Kubernetes 集群遭遇故障且无法迅速恢复，别担心。你只需创建一个新的集群，并将Argo CD连接到Git仓库。这个仓库包含了整个集群的所有配置声明。最终，新集群的状态将与之前的旧集群状态完全一致，无需人工干预。
```

##### 使用 Git 实现访问控制

```sh
通常情况下，生产环境不允许所有人访问 Kubernetes 集群。直接在 Kubernetes 集群中管理访问权限需要使用复杂的 RBAC 规则。而通过 Git 仓库管理权限要简单得多。例如，所有人（DevOps 团队、运维团队、研发团队等）都可以向仓库提交 Pull Request，但只有高级工程师才有权限合并 Pull Request。

这样做的好处是，除了集群管理员和少数特定人员外，其他人无需直接访问 Kubernetes 集群，只需访问 Git 仓库即可。对于程序而言也是如此，类似于 Jenkins 这样的 CI 工具也无需访问 Kubernetes，因为只有 Argo CD 才能应用配置清单。并且，Argo CD 已部署在 Kubernetes 集群中，其必要的访问权限已经配置完备，因此不必向任意人或工具提供集群访问证书，从而提供更高级的安全保障。
```

##### 扩展 Kubernetes

```sh
虽然 Argo CD 可以部署在 Kubernetes 集群中并享受 Kubernetes 的诸多好处，但它并不是 Kubernetes 的专属。毕竟，Jenkins 也可以部署在 Kubernetes 中。那么，Argo CD 有何特殊之处呢？

事实上，Argo CD 灵活巧妙地利用了 Kubernetes 集群中的许多功能来实现自身的目标。例如，所有资源都存储在 Etcd 集群中，并利用 Kubernetes 的控制器监控应用的实际状态，随时与期望状态进行对比。

这样做的最直接好处就是实时感知应用的部署状态。例如，当你在 Git 仓库中更新配置清单中的镜像版本时，Argo CD 会立即将集
```

## 二.部署Argo-CD

### 去[github Arogo-CD]([2.3.5 · Releases · argoproj/argo-cd (github.com)](https://github.com/argoproj/argo-cd/releases?expanded=true&page=6&q=2.3.5))官网选择合适的版本

- 这里我用的2.2.0版本，因为最新版本有个Bug，小伙伴们可以自行研究（可能是K8S版本问题）--->（已结局，权限问题，修改yaml文件即可）
- Bug: argo-repo-server Pod启动报错 ：issue: gpg ... --gen-key failed exit status 2
- [解决方案]([argo-repo-server issue: gpg ... --gen-key failed exit status 2 · Issue #9809 · argoproj/argo-cd (github.com)](https://github.com/argoproj/argo-cd/issues/9809))

### 部署

```none
kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/v2.2.0/manifests/install.yaml
```

### 开放端口并访问

- 这里我用kubectl转发端口，小伙伴们可以用service或ingress实现外部访问！

```sh
kubectl port-forward svc/argocd-server -n argocd --address 0.0.0.0 8082:443
```

![img](https://img2024.cnblogs.com/blog/3394728/202404/3394728-20240420133124875-44810580.png)

### 查看密码

```sh
kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d; echo
#得到密码：xxxxxxxxxx
#用户名: admin
```

### 下载客户端argocd命令行工具并修改密码

```sh
wget https://github.com/argoproj/argo-cd/releases/download/v2.3.5/argocd-linux-amd64
mv argocd-linux-amd64 /usr/bin/argocd
chmod +x /usr/bin/argocd
argocd -v

[root@lc-master-1 ~]# argocd login 192.168.0.71:8082
WARNING: server certificate had error: x509: cannot validate certificate for 192.168.0.71 because it doesn't contain any IP SANs. Proceed insecurely (y/n)? y
Username: admin
Password: 
'admin:login' logged in successfully
Context '192.168.0.71:8082' updated

[root@lc-master-1 ~]# argocd account update-password
*** Enter password of currently logged in user (admin): 
*** Enter new password for user admin: 
*** Confirm new password for user admin: 
Password updated
Context '192.168.0.71:8082' updated

[root@lc-master-1 ~]# argocd logout  192.168.0.71:8082
Logged out from '192.168.0.71:8082'
[root@lc-master-1 ~]# argocd login  192.168.0.71:8082
WARNING: server certificate had error: x509: cannot validate certificate for 192.168.0.71 because it doesn't contain any IP SANs. Proceed insecurely (y/n)? y
Username: admin
Password: 
'admin:login' logged in successfully
Context '192.168.0.71:8082' updated
```

### Web页面介绍

#### 设置介绍

![img](https://img2024.cnblogs.com/blog/3394728/202404/3394728-20240420102143231-2047530009.png)

#### 添加仓库地址

![img](https://img2024.cnblogs.com/blog/3394728/202404/3394728-20240420130837822-1109676108.png)
![img](https://img2024.cnblogs.com/blog/3394728/202404/3394728-20240420130904494-685592270.png)
![img](https://img2024.cnblogs.com/blog/3394728/202404/3394728-20240420130923598-494819299.png)

#### 创建项目

![img](https://img2024.cnblogs.com/blog/3394728/202404/3394728-20240420131052995-2089292510.png)

#### 部署应用

![img](https://img2024.cnblogs.com/blog/3394728/202404/3394728-20240420131411215-323363767.png)

![img](https://img2024.cnblogs.com/blog/3394728/202404/3394728-20240420131429306-2054356714.png)

![img](https://img2024.cnblogs.com/blog/3394728/202404/3394728-20240420131457349-769674449.png)

![img](https://img2024.cnblogs.com/blog/3394728/202404/3394728-20240420131525338-1496683401.png)

![img](https://img2024.cnblogs.com/blog/3394728/202404/3394728-20240420131548214-1804418154.png)

![img](https://img2024.cnblogs.com/blog/3394728/202404/3394728-20240420131604978-1941786923.png)

![img](https://img2024.cnblogs.com/blog/3394728/202404/3394728-20240420131628292-953353078.png)

#### 查看部署结果并访问！

![img](https://img2024.cnblogs.com/blog/3394728/202404/3394728-20240420131731910-123855303.png)

- **这里看到全部成功，我们来访问一下**

![img](https://img2024.cnblogs.com/blog/3394728/202404/3394728-20240420131752258-1568097803.png)

### 配置文件实现

- **实际上与Web界面一样，只不过变成了可见的配置文件格式**

#### 配置解析

```sh
[root@lc-master-1 ceshi]# cat application.yaml 
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: myapp-argo-application
  namespace: argocd
spec:
  project: default

#git来源地址
  source:
    repoURL: https://gitee.com/Licheng996/argo-cd.git
    targetRevision: HEAD
    path: argo-demo
#部署目标
  destination: 
    server: https://kubernetes.default.svc
    namespace: myapp

#指定自动同步策略和频率，不配置时需要手动触发同步。
  syncPolicy:
#定义同步方式
    syncOptions:
#如果不存在这个 namespace，就会自动创建它
    - CreateNamespace=true
#检测到实际状态与期望状态不一致时，采取的同步措施
    automated:
#当集群世纪状态不符合期望状态时，自动同步
      selfHeal: true
#自动同步时，删除 Git 中不存在的资源
      prune: true
```

#### 部署

```sh
[root@lc-master-1 ceshi]# kubectl apply -f application.yaml
application.argoproj.io/myapp-argo-application created
```

![img](https://img2024.cnblogs.com/blog/3394728/202404/3394728-20240420132125829-1729083971.png)

## 三.在 ArgoCD 中添加多个集群:

### 开放端口

```sh
kubectl port-forward svc/argocd-server -n argocd --address 0.0.0.0 8082:443
```

#### 查询密码：

```sh
kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="
{.data.password}" | base64 -d; echo
```

#### 登录:

```sh
[root@lc-master-1 ~]# argocd login 192.168.0.71:8082
WARNING: server certificate had error: x509: cannot validate certificate for 192.168.0.71 because it doesn't contain any IP SANs. Proceed insecurely (y/n)? y
Username: admin
Password: 
'admin:login' logged in successfully
Context '192.168.0.71:8082' updated
```

#### 查看集群:

```sh
[root@lc-master-1 ~]# argocd cluster list
```

#### 准备添加集群：

```sh
[root@k8s-master-1 ~]# kubectl config view --flatten > /tmp/kubeconfig-cluster-a
[root@k8s-master-1 tmp]# scp /tmp/kubeconfig-cluster-a  root@192.168.0.71:/tmp/

[root@lc-master-1 ~]# kubectl config get-contexts --kubeconfig /tmp/kubeconfig-cluster-a
CURRENT   NAME                          CLUSTER      AUTHINFO           NAMESPACE

*         kubernetes-admin@kubernetes   kubernetes   kubernetes-admin   
          [root@lc-master-1 ~]# argocd cluster add CONTEXT_NAME --name cluster-a --kubeconfig /tmp/kubeconfig-cluster-a
          FATA[0000] Context CONTEXT_NAME does not exist in kubeconfig 
          [root@lc-master-1 ~]# argocd cluster add kubernetes-admin@kubernetes --name cluster-a --kubeconfig /tmp/kubeconfig-cluster-a
          WARNING: This will create a service account `argocd-manager` on the cluster referenced by context `kubernetes-admin@kubernetes` with full cluster level admin privileges. Do you want to continue [y/N]? y
          INFO[0004] ServiceAccount "argocd-manager" created in namespace "kube-system" 
          INFO[0004] ClusterRole "argocd-manager-role" created    
          INFO[0004] ClusterRoleBinding "argocd-manager-role-binding" created 
          Cluster 'https://192.168.0.20:6443' added
          [root@lc-master-1 ~]# argocd cluster list
          SERVER                          NAME        VERSION  STATUS      MESSAGE                                                  PROJECT
          https://192.168.0.20:6443       cluster-a            Unknown     Cluster has no applications and is not being monitored.  
          https://kubernetes.default.svc  in-cluster  1.19     Successful                            
```

出现Unknown Cluster has no applications and is not being monitored 不用担心，不影响界面部署，部署一次后就是Successful