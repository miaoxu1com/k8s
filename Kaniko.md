# [Kaniko](https://segmentfault.com/a/1190000039713484)

## 前言

随着各企业上云进程的加速，用容器的方式来交付软件产品也变得越来越普遍，如何以更安全的方式来构建容器镜像，也就成了大家关注的话题。

## docker 构建镜像

docker是最近几年非常火热的容器技术，用docker来构建容器镜像也是常用的方法，在具备构建容器镜像所需的两个要素(**Dockerfile & 上下文**)的前提下，用下述命令就能构建一个容器镜像出来

```
$ docker build -t your_registry/your_repository:tag .
```

然后用 `docker push` 将镜像推送到镜像仓库

```
$ docker push your_registry/your_repository:tag
```

现在DevOps 的CI/CD环境大多数都会运行在容器内，镜像的构成也是在容器内完成的。这时候，通常用以下两种方式来完成容器内构建镜像的工作：

### 第一：挂载宿主机的socket文件到容器内部

```
$ docker run -it -v /var/run/docker.sock:/var/run/docker.sock -v /tmp/kaniko:/tmp/kaniko docker
```

然后在容器内部用 `docker build` 构建镜像

```sql
$ docker build -t dllhb/kaniko-test:v0.1 .
Sending build context to Docker daemon  5.632kB
Step 1/4 : FROM alpine:latest
latest: Pulling from library/alpine
89d9c30c1d48: Already exists
Digest: sha256:c19173c5ada610a5989151111163d28a67368362762534d8a8121ce95cf2bd5a
Status: Downloaded newer image for alpine:latest
 ---> 965ea09ff2eb
Step 2/4 : MAINTAINER <devops008@sina.com xiaomage> 
 ---> Running in 8a2b1dc13d6b
Removing intermediate container 8a2b1dc13d6b 
 ---> bd535532278d
Step 3/4 : RUN apk add busybox-extras curl
 ---> Running in fc254ad3d088
fetch http://dl-cdn.alpinelinux.org/alpine/v3.10/main/x86_64/APKINDEX.tar.gz
fetch http://dl-cdn.alpinelinux.org/alpine/v3.10/community/x86_64/APKINDEX.tar.gz
(1/5) Installing busybox-extras (1.30.1-r3)
Executing busybox-extras-1.30.1-r3.post-install
(2/5) Installing ca-certificates (20190108-r0)
(3/5) Installing nghttp2-libs (1.39.2-r0)
(4/5) Installing libcurl (7.66.0-r0)
(5/5) Installing curl (7.66.0-r0)
Executing busybox-1.30.1-r2.trigger
Executing ca-certificates-20190108-r0.trigger
OK: 7 MiB in 19 packages
Removing intermediate container fc254ad3d088
 ---> 1bbe81600a67
Step 4/4 : CMD ["echo","Hello DevOps"]
 ---> Running in 4b92a6a4b37e
Removing intermediate container 4b92a6a4b37e
 ---> de712b8cd7e5
Successfully built de712b8cd7e5
Successfully tagged dllhb/kaniko-test:v0.1
```

由于docker依赖于 `docker daemon` 进程， `docker daemon` 进程是一个 `Unixsocket` 连接，且 `/var/run/docker.sock` 文件是root权限，

```bash
$ ls -ltr /var/run/docker.sock
lrwxr-xr-x 1 root  daemon 69 Nov 26 15:13 /var/run/docker.sock
```

说明只有root权限才能访问 `docker daemon` 进程，在 `docker daemon` 无法暴露或者用户没有权限获取 `docker daemon` 进程的前提下，用 `docker build` 来构建镜像就变的非常困难了。

可能我们会想，docker build镜像无非就是需要docker命令能运行成功，只要在容器里面安装一个docker不就成功了吗？这也就是下面讲的第二种方法。

### 第二种: dind(docker-in-docker)

这种方式不需要挂载宿主机的socket文件，但是需要以 **--privileged** 权限来以dind镜像创建一个容器:

```shell
$ docker run --rm -it --privileged docker:18.06-dind
```

然后在容器里面构建容器镜像并推送至远端仓库。

dind能够满足构建容器镜像的需求，但是从上面的命令看，有一个参数：**--privileged** 。意味这这个容器具有一些特权，他可能会看到宿主机上的一些设备，而且能够执行mount命令。

> dind还有很多问题，只是方便docker开发人员来测试docker，所以官方也说**running Docker inside Docker is generally not recommended**。具体的可以看看这篇博文: [https://jpetazzo.github.io/20...](https://link.segmentfault.com/?enc=QbilfqnTKjoEpbKFWtGMMQ%3D%3D.8oW6o7XA7m3j5ZZCes3fk0p7i0bRR8h5waT6NKfflR0pEumtuYvJsvbeFBtLlcT5pesMWEdTCCZP5bvr79Py8MkP%2FuEjNcJdxZ2%2FsSzcvR8%3D)

上述两种方法，都能满足在容器内构建容器镜像且推送镜像至远端仓库的需求，但是从security角度来讲，**需要root 权限(第一种方式)，提供特权(第二种方式)** 都使得风险增大，在Kubernetes 多租户的场景下，这种风险是不能接受的。那是否有一种不需要特殊权限，还能快速构建容器镜像的方法呢？答案就是下面将的Kaniko。

#### Kaniko

Kaniko是谷歌开源的一款用来构建容器镜像的工具。与docker不同，Kaniko 并不依赖于Docker daemon进程，完全是在用户空间根据Dockerfile的内容逐行执行命令来构建镜像，这就使得在一些无法获取 `docker daemon` 进程的环境下也能够构建镜像，比如在**标准的Kubernetes Cluster**上。

![image.png](https://segmentfault.com/img/bVcQNoZ)

Kaniko 以容器镜像的方式来运行的，同时需要三个参数: **Dockerfile，上下文，以及远端镜像仓库的地址**。

Kaniko会先提取基础镜像(Dockerfile FROM 之后的镜像)的文件系统，然后根据Dockerfile中所描述的，一条条执行命令，每一条命令执行完以后会在用户空间下面创建一个snapshot，并与存储与内存中的上一个状态进行比对，如果有变化，就将新的修改生成一个镜像层添加在基础镜像上，并且将相关的修改信息写入镜像元数据中。等所有命令执行完，kaniko会将最终镜像推送到指定的远端镜像仓库。

#### Kaniko 构建镜像

下面看一个demo。

```sql
$ cat Dockerfile
FROM alpine:latest
    
MAINTAINER <devops008@sina.com xiaomage>
    
RUN apk add busybox-extras curl
    
CMD ["echo","Hello DevOps"]
```

在kubernetes cluster上面创建一个pod，yaml文件如下：

```yaml
apiVersion: v1
kind: Pod
metadata:
name: kaniko
spec:
 containers:
 - name: kaniko
   image: gcr.io/kaniko-project/executor:latest
   args: ["--dockerfile=/workspace/Dockerfile",
          "--context=/workspace/",
          "--destination=dllhb/kaniko-test:v0.4"]
    volumeMounts:
      - name: kaniko-secret
        mountPath: /kaniko/.docker
      - name: dockerfile
        mountPath: /workspace/Dockerfile
        subPath: Dockerfile
restartPolicy: Never
volumes:
      - name: dockerfile
        configMap:
          name: dockerfile
      - name: kaniko-secret
         projected:
         sources:
         - secret:
              name: regcred
              items:
                - key: .dockerconfigjson
                  path: config.json
```

需要说明几点:

- args 部分

这部分就是上面所讲的，kaniko运行时需要三个参数: **Dockerfile(--dockerfile)，上下文(--context)，远端镜像仓库(--destination)**

- secret 部分

推送至指定远端镜像仓库需要credential的支持，所以需要将credential以secret的方式挂载到/kaniko/.docker/这个目录下，文件名称为config.json，内容如下:

```json
{   
    "auths": {
        "https://index.docker.io/v1/": {
            "auth": "AbcdEdfgEdggds="
       }
    }
    
}
```

> 其中auth的值为: `echo"docker_registry_username:docker_registry_password"|base64`

然后创建pod，查看状态，并看log

```csharp
$ kubectl -n kaniko apply -f kaniko.yaml
pod/kaniko created
$ kubectl -n kaniko get pods
NAME     READY   STATUS      RESTARTS   AGE
kaniko 1/1 Running 0 21s
$ kubectl -n kaniko logs -f kaniko
 1 INFO[0000] Resolved base name alpine:latest to alpine:latest
 2 INFO[0000] Resolved base name alpine:latest to alpine:latest
 3 INFO[0000] Downloading base image alpine:latest
 4 INFO[0001] Error while retrieving image from cache: getting fileinfo: stat /cachesha256:e4355b66995c96b4b468159fc5c7e3540fcef961189ca13fee877798649f51a: no such file or directory
 5 INFO[0001] Downloading base image alpine:latest
 6 INFO[0001] Built cross stage deps: map[]
 7 INFO[0001] Downloading base image alpine:latest
 8 INFO[0001] Error while retrieving image from cache: getting fileinfo: stat /cachesha256:e4355b66995c96b4b468159fc5c7e3540fcef961189ca13fee877798649f51a: no such file or directory
 9 INFO[0001] Downloading base image alpine:latest
 10 WARN[0002] maintainer is deprecated, skipping
 11 INFO[0002] Unpacking rootfs as cmd RUN apk add busybox-extrascurl requires it.
 12 INFO[0002] Taking snapshot of full filesystem...
 13 INFO[0002] RUN apk add busybox-extras curl
 14 INFO[0002] cmd: /bin/sh
 15 INFO[0002] args: [-c apk add busybox-extras curl]
 16 fetch http://dl-cdn.alpinelinux.org/alpine/v3.10/main/x86_64APKINDEX.tar.gz
 17 fetch http://dl-cdn.alpinelinux.org/alpine/v3.10/community/x86_64APKINDEX.tar.gz
 18 (1/5) Installing busybox-extras (1.30.1-r3)
 19 Executing busybox-extras-1.30.1-r3.post-install
 20 (2/5) Installing ca-certificates (20190108-r0)
 21 (3/5) Installing nghttp2-libs (1.39.2-r0)
 22 (4/5) Installing libcurl (7.66.0-r0)
 23 (5/5) Installing curl (7.66.0-r0)
 24 Executing busybox-1.30.1-r2.trigger
 25 Executing ca-certificates-20190108-r0.trigger
 26 OK: 7 MiB in 19 packages
 27 INFO[0003] Taking snapshot of full filesystem...
 28 INFO[0003] CMD ["echo","Hello DevOps"]
```

可以用新生成的镜像创建个容器来测试一下:

```bash
$ docker run --rm dllhb/kaniko-test:v0.5
Unable to find image 'dllhb/kaniko-test:v0.5' locally
v0.5: Pulling from dllhb/kaniko-test
89d9c30c1d48: Already exists
2beaee649353: Pull complete
Digest: sha256:461233efe63a8f16e8630ab92a31795fb0932d2910669131d9973f258070cca5
Status: Downloaded newer image for dllhb/kaniko-test:v0.5
Hello DevOps
```

可以看到输出了 `HelloDevOps` 这和Dockerfile所写是一致的。证明容器镜像已经构建完毕并已推送至远端仓库。

### Kaniko 与Jenkins pipeline的集成

Jenkins pipeline代码如下：

```python
podTemplate(
       label: label,
        serviceAccount: "your service account",
        namespace: "your jenkins namespace",
        yaml: """
kind: Pod
metadata:
  name: kaniko
spec:
  containers:
  - name: kaniko
    image: gcr.io/kaniko-project/executor:debug
    imagePullPolicy: Always
    command:
    - cat
    tty: true
    volumeMounts:
      - name: registry-secret
        mountPath: /kaniko/.docker
      - name: jenkins-home
        mountPath: /home/jenkins
  - name: maven
    image: maven:3-jdk-8
    imagePullPolicy: Always
    command:
    - cat
    tty: true
    volumeMounts:
      - name: jenkins-home
        mountPath: /home/jenkins
  volumes:
    - name: jenkins-home
      persistentVolumeClaim:
        claimName: jenkins-pvc
    - name: registry-secret
      secret:
        secretName: regcred
        items:
          - key: .dockerconfigjson
            path: config.json
"""
) {
    node(label)  {
        container('kaniko') {
           sh '/kaniko/executor -f Dockerfile -c . --destination=your_registry/your_repo:tag'
        }
    }
}
```

触发构建查看jenkins log：

![image.png](https://segmentfault.com/img/bVcQNr3)

可以看出，Kaniko可以在无法获取宿主机docker daemon进程或者没有特殊权限的情况下构建容器镜像，这种方式是非常适合在Kubernetes平台上构建容器镜像的，而且易于集成到 `DevOposJenkinspipeline` 里。