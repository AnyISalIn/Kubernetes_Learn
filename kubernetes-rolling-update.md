# Kubernetes Learn (滚动更新)

`Kubernetes` 支持 `rolling update`, 每一次更新一个 `Pod`, 可以使用 `kubectl rolling-update` 命令来完成

`rolling-update` 仅支持 `Replication Controller(RC)`

```yaml
apiVersion: v1
kind: ReplicationController
metadata:
  name: my-nginx
spec:
  replicas: 5
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.7.9
        ports:
        - containerPort: 80
```

首先我们创建一个 `RC`

```shellscript
$ kubectl create -f kube-nginx-rc.yaml
replicationcontroller "my-nginx" created

$ kubectl get pods -o wide
NAME             READY     STATUS    RESTARTS   AGE       NODE
my-nginx-511vw   1/1       Running   0          1m        127.0.0.1
my-nginx-b6dat   1/1       Running   0          1m        127.0.0.1
my-nginx-sj6fz   1/1       Running   0          1m        127.0.0.1
my-nginx-vvcf6   1/1       Running   0          1m        127.0.0.1
my-nginx-z60jz   1/1       Running   0          1m        127.0.0.1
```

开始滚动更新

```shellscript
$ kubectl rolling-update my-nginx --image=nginx:latest  #过一段时间会使用新的image重建pod
Created my-nginx-ba3663b2e3a856975068e556f3d0556b
Scaling up my-nginx-ba3663b2e3a856975068e556f3d0556b from 0 to 5, scaling down my-nginx from 5 to 0 (keep 5 pods available, don't exceed 6 pods)
Scaling my-nginx-ba3663b2e3a856975068e556f3d0556b up to 1
Scaling my-nginx down to 4
Scaling my-nginx-ba3663b2e3a856975068e556f3d0556b up to 2
Scaling my-nginx down to 3

# 如果升级到一半, 发现出了 BUG, 我们也可以回滚

$ kubectl rolling-update my-nginx --rollback
Setting "my-nginx" replicas to 5
Continuing update with existing controller my-nginx.
Scaling up my-nginx from 3 to 5, scaling down my-nginx-ba3663b2e3a856975068e556f3d0556b from 2 to 0 (keep 5 pods available, don't exceed 6 pods)
Scaling my-nginx up to 4
Scaling my-nginx-ba3663b2e3a856975068e556f3d0556b down to 1
Scaling my-nginx up to 5
Scaling my-nginx-ba3663b2e3a856975068e556f3d0556b down to 0
Update succeeded. Deleting my-nginx-ba3663b2e3a856975068e556f3d0556b
replicationcontroller "my-nginx" rolling updated

$ kubectl get pods
NAME             READY     STATUS    RESTARTS   AGE
my-nginx-511vw   1/1       Running   0          14m
my-nginx-fsqjs   1/1       Running   0          2m
my-nginx-qte3t   1/1       Running   0          1m
my-nginx-sj6fz   1/1       Running   0          14m
my-nginx-z60jz   1/1       Running   0          14m
```
