---
title: k8s-pod
date: 2023-09-27 17:17:20
categories:
- 云原生
- k8s
tags:
---

# 创建pod的流程
新建pod的流程，当一个pod完成调用，需要与一个node绑定，然后pod触发kubelet在循环控制里注册的handler，通过检查pod在kubelet内存中的状态，判断是否为新调度的pod，从而触发add。然后kubelet为这个pod生成对应的podStatus，包括检查volume，然后调用容器。

```golang
func (kl *Kubelet) HandlePodAdditions(pods []*v1.Pod) {
	start := kl.clock.Now()
	//日期排序
	sort.Sort(sliceutils.PodsByCreationTime(pods))
	for _, pod := range pods {
		existingPods := kl.podManager.GetPods()
		// Always add the pod to the pod manager. Kubelet relies on the pod
		// manager as the source of truth for the desired state. If a pod does
		// not exist in the pod manager, it means that it has been deleted in
		// the apiserver and no action (other than cleanup) is required.
		kl.podManager.AddPod(pod)

		if kubetypes.IsMirrorPod(pod) {
			kl.handleMirrorPod(pod, start)
			continue
		}

		// Only go through the admission process if the pod is not requested
		// for termination by another part of the kubelet. If the pod is already
		// using resources (previously admitted), the pod worker is going to be
		// shutting it down. If the pod hasn't started yet, we know that when
		// the pod worker is invoked it will also avoid setting up the pod, so
		// we simply avoid doing any work.
		if !kl.podWorkers.IsPodTerminationRequested(pod.UID) {
			// We failed pods that we rejected, so activePods include all admitted
			// pods that are alive.
			activePods := kl.filterOutInactivePods(existingPods)

			// Check if we can admit the pod; if not, reject it.
			//看是否能在计算机上创建 资源是否足够
			if ok, reason, message := kl.canAdmitPod(activePods, pod); !ok {
				kl.rejectPod(pod, reason, message)
				continue
			}
		}
		mirrorPod, _ := kl.podManager.GetMirrorPodByPod(pod)
		//调用dispatchWork 转而调用update
		kl.dispatchWork(pod, kubetypes.SyncPodCreate, mirrorPod, start)
	}
}

在1.23版本，将定期检查变到
func (kl *Kubelet) syncPod(ctx context.Context, updateType kubetypes.SyncPodType, pod, mirrorPod *v1.Pod, podStatus *kubecontainer.PodStatus) (isTerminal bool, err error)
同时在这个函数里面有对容器的创建，判断网络模式等

最终的启动容器在startContainer
1.拉取镜像 涉及到怎么拉区 从本地 还是远程
2.创建container 容器的配置信息等 
3.启动
-- 启动完涉及到日志管理，如果采用了cni则需要修改日志路径 若该版本本身具有cni 就需要移除日志的符号链接
4.做一些预处理 比如生命周期，容器name，容器id设置
```