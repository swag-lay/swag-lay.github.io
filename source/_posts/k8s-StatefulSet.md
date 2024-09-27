---
title: k8s-StatefulSet
date: 2023-07-22 09:17:03
categories:
- 云原生
- k8s
tags:
---

# 工作负载StatefulSet
主要用来管理有状态应用的工作负载API对象
管理某Pod集合的部署和扩缩，为这些Pod提供持久化存储和持久标识符。在StatefulSet中，为每个Pod维护一个由粘性的ID，每个Pod都是基于相同的规约来创建的，但是不能相互替换，无论怎么调度，每个Pod都有一个永久不变的ID

主要针对一下应用程序更好：
- 稳定的，唯一的网络标识符
- 稳定的，持久的存储
- 有序的，更好的部署和扩缩
- 有序的，自动的滚动更新

在生产环境中，加载卷建议采用ReadWriteOncePod，该卷可以被单个pod以读写模式挂载，不能被其他pod挂载
其他访问模式：
- ReadWriteOnce：可以被单个Pod以读写模式挂载
- ReadWriteMany：可以被多个Pod以读写模式挂载
- ReadOnlyMany：可以被多个Pod以只读模式挂载

样例：
```yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx
  labels:
    app: nginx
spec:
  ports:
  - port: 80
    name: web
  clusterIP: None
  selector:
    app: nginx
---
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: web
spec:
  selector:
    matchLabels:
      app: nginx # 必须匹配 .spec.template.metadata.labels
  serviceName: "nginx"
  replicas: 3 # 默认值是 1
  minReadySeconds: 10 # 默认值是 0
  template:
    metadata:
      labels:
        app: nginx # 必须匹配 .spec.selector.matchLabels
    spec:
      terminationGracePeriodSeconds: 10
      containers:
      - name: nginx
        image: registry.k8s.io/nginx-slim:0.24
        ports:
        - containerPort: 80
          name: web
        volumeMounts:
        - name: www
          mountPath: /usr/share/nginx/html
  volumeClaimTemplates:
  - metadata:
      name: www
    spec:
      accessModes: [ "ReadWriteOnce" ]
      storageClassName: "my-storage-class"
      resources:
        requests:
          storage: 1Gi
```
## 部署和扩缩
部署顺序是顺序依次的0-N-1
删除顺序是逆序的
当进行扩缩的时候，在该pod之前的pod必须是running和ready
一个pod终止之前，所有的继承者必须完全关闭

更新策略
.spec.updateStrategy
- OnDelete
- RollingUpdate 默认的滚动更新

PersistentVolumeClaim保留
spec.persistentVolumeClaimRetentionPolicy控制是否删除以及如何删除PVC
使用该字段必须在API服务器和控制器管理器启用StatefulSetAutoDeletePVC
- whenDeleted
- whenScaled
这两个策略可以配置下面两个值
- Delete
- Retain 默认

```sh
kubectl scale sts xxx --replicas=4 -n xxx
```


### 创建源码解析
将接口注册到controllerContext.InformerFactory中，包括pods，statefulsets，PersistentVolumeClaims，ControllerRevisions
informer会同步cache监听pod和statefulset对象的变更事件。最后会进行同步。

最后进行同步的时候会调用
```golang
func (ssc *StatefulSetController) sync(ctx context.Context, key string) error {
	startTime := time.Now()
	defer func() {
		klog.V(4).Infof("Finished syncing statefulset %q (%v)", key, time.Since(startTime))
	}()
	//拿到所在的namespace和name
	namespace, name, err := cache.SplitMetaNamespaceKey(key)
	if err != nil {
		return err
	}
	//获取到他的selector
	set, err := ssc.setLister.StatefulSets(namespace).Get(name)
	if errors.IsNotFound(err) {
		klog.Infof("StatefulSet has been deleted %v", key)
		return nil
	}
	if err != nil {
		utilruntime.HandleError(fmt.Errorf("unable to retrieve StatefulSet %v from store: %v", key, err))
		return err
	}

	selector, err := metav1.LabelSelectorAsSelector(set.Spec.Selector)
	if err != nil {
		utilruntime.HandleError(fmt.Errorf("error converting StatefulSet %v selector: %v", key, err))
		// This is a non-transient error, so don't retry.
		return nil
	}
	//这里的目的是为了检测孤儿资源，使他成为statefulset的一部分。因为有一些快照版本可能没有关联的控制器。 在中间会确认是否有权限关联这个资源，包括是否还存在，uid是否一致
	if err := ssc.adoptOrphanRevisions(ctx, set); err != nil {
		return err
	}
	//获取到管理的pod
	pods, err := ssc.getPodsForStatefulSet(ctx, set, selector)
	if err != nil {
		return err
	}
	//真正的同步
	return ssc.syncStatefulSet(ctx, set, pods)
}

func (ssc *StatefulSetController) syncStatefulSet(ctx context.Context, set *apps.StatefulSet, pods []*v1.Pod) error {
	klog.V(4).Infof("Syncing StatefulSet %v/%v with %d pods", set.Namespace, set.Name, len(pods))
	var status *apps.StatefulSetStatus
	var err error
	status, err = ssc.control.UpdateStatefulSet(ctx, set, pods)
	if err != nil {
		return err
	}
	klog.V(4).Infof("Successfully synced StatefulSet %s/%s successful", set.Namespace, set.Name)
	// One more sync to handle the clock skew. This is also helping in requeuing right after status update
	if utilfeature.DefaultFeatureGate.Enabled(features.StatefulSetMinReadySeconds) && set.Spec.MinReadySeconds > 0 && status != nil && status.AvailableReplicas != *set.Spec.Replicas {
		ssc.enqueueSSAfter(set, time.Duration(set.Spec.MinReadySeconds)*time.Second)
	}

	return nil
}

//在1.23版本中默认的并没有去更新了 需要用户去重写 包括对创建，更新，删除等操作 比如是是否处于删除状态。包括扩容对replicas的删除和重建
func (ssc *defaultStatefulSetControl) UpdateStatefulSet(ctx context.Context, set *apps.StatefulSet, pods []*v1.Pod) (*apps.StatefulSetStatus, error) {
	set = set.DeepCopy() // set is modified when a new revision is created in performUpdate. Make a copy now to avoid mutation errors.

	// list all revisions and sort them
	revisions, err := ssc.ListRevisions(set)
	if err != nil {
		return nil, err
	}
	history.SortControllerRevisions(revisions)

	currentRevision, updateRevision, status, err := ssc.performUpdate(ctx, set, pods, revisions)
	if err != nil {
		return nil, utilerrors.NewAggregate([]error{err, ssc.truncateHistory(set, pods, revisions, currentRevision, updateRevision)})
	}

	// maintain the set's revision history limit
	return status, ssc.truncateHistory(set, pods, revisions, currentRevision, updateRevision)
}
```