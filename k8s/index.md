## 容器是什么
容器的本质就是进程，根据 linux 的 namespace、cgrops、rootfs隔离出来

namespace： 隔离

cgroups： 限制资源

rootfs： 根文件系统

### 容器的分层 - 7层
- 最上层：读写层，可以提交到 docker hub 上被他们修改使用，删除的文件会被写入 whiteout 中，来屏蔽这个被删除的文件
- init 层：主要是修改 /etc/hosts 等数据，不会被 commit 
- 最低下五层：只读层，rootfs 根文件系统，一般也是操作系统的文件目录

### 卷【volume】的挂载
在 rootfs 准备好之后，chroot 之前，将宿主机的目录挂载到容器中，在执行这个挂载操作，容器已经创建了，所以在宿主机上不可见，在容器内可见这个挂载事件

### 容器主要组成
- rootfs，就是容器的镜像，是容器的静态视图
- namespace+cgroups，容器运行时，是容器的动态视图

## k8s 设计与架构
- master：包含 api-server，controller-manager（控制器管理器），scheduler（调度器）
- etcd：持久化 k8s 整个集群数据
- node：kubelet（主要负责与容器运行时交互，调用网络插件和存储插件为容器配置网络和数据持久化）

k8s 的本质是 平台的平台，帮助用户构建上层平台的基础平台

## k8s 编排原理
### pod
#### 什么是 pod
是一组共享了某些资源的容器
- 共享同一个网络
- 共享同一个数据卷
- 可以同时运行一个或者多个容器，这些容器共享网络、存储等资源

#### pod 的资源共享需要启用一个中间容器
- infra 容器，永远第一个被启动，永远处于暂停状态
- 占用资源少
- 一个特殊的镜像