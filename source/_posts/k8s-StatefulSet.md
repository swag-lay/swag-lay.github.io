---
title: k8s-StatefulSet
date: 2024-07-22 09:17:03
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