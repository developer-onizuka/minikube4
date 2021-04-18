# 1. Start minikube with 3nodes
```
PS D:\k8s\deployment> minikube.exe start --driver=virtualbox --nodes=3
* Microsoft Windows 10 Pro 10.0.19042 Build 19042 上の minikube v1.18.1
* minikube 1.19.0 が利用可能です! 以下のURLでダウンロードできます。 https://github.com/kubernetes/minikube/releases/tag/v1.19.0
* To disable this notice, run: 'minikube config set WantUpdateNotification false'

* 設定を元に、 virtualbox ドライバを使用します
* VM ブートイメージをダウンロードしています...
    > minikube-v1.18.0.iso.sha256: 65 B / 65 B [-------------] 100.00% ? p/s 0s
    > minikube-v1.18.0.iso: 212.99 MiB / 212.99 MiB  100.00% 532.35 KiB p/s 6m5
* コントロールプレーンのノード minikube を minikube 上で起動しています
* Kubernetes v1.20.2 のダウンロードの準備をしています
    > preloaded-images-k8s-v9-v1....: 491.22 MiB / 491.22 MiB  100.00% 653.91 K
* virtualbox VM (CPUs=2, Memory=2700MB, Disk=20000MB) を作成しています...
* Docker 20.10.3 で Kubernetes v1.20.2 を準備しています...
  - Generating certificates and keys ...
  - Booting up control plane ...
  - Configuring RBAC rules ...
* Configuring CNI (Container Networking Interface) ...
* Kubernetes コンポーネントを検証しています...
  - Using image gcr.io/k8s-minikube/storage-provisioner:v4
* 有効なアドオン: storage-provisioner, default-storageclass

* Starting node minikube-m02 in cluster minikube
* virtualbox VM (CPUs=2, Memory=2700MB, Disk=20000MB) を作成しています...
* ネットワーク オプションが見つかりました
  - NO_PROXY=192.168.99.103
  - no_proxy=192.168.99.103
* Docker 20.10.3 で Kubernetes v1.20.2 を準備しています...
  - env NO_PROXY=192.168.99.103
* Kubernetes コンポーネントを検証しています...

* Starting node minikube-m03 in cluster minikube
* virtualbox VM (CPUs=2, Memory=2700MB, Disk=20000MB) を作成しています...
* ネットワーク オプションが見つかりました
  - NO_PROXY=192.168.99.103,192.168.99.104
  - no_proxy=192.168.99.103,192.168.99.104
* Docker 20.10.3 で Kubernetes v1.20.2 を準備しています...
  - env NO_PROXY=192.168.99.103
  - env NO_PROXY=192.168.99.103,192.168.99.104
* Kubernetes コンポーネントを検証しています...
* Done! kubectl is now configured to use "minikube" cluster and "default" namespace by default
```

# 2.  Check the node list and status
```
PS D:\k8s\deployment> minikube.exe node list
minikube        192.168.99.103
minikube-m02    192.168.99.104
minikube-m03    192.168.99.105

PS C:\Users\developer> minikube.exe status
minikube
type: Control Plane
host: Running
kubelet: Running
apiserver: Running
kubeconfig: Configured
timeToStop: Nonexistent

minikube-m02
type: Worker
host: Running
kubelet: Running

minikube-m03
type: Worker
host: Running
kubelet: Running

```

# 3. Put labels on each node.
```
PS D:\k8s\deployment> kubectl.exe label nodes minikube location=tokyo
node/minikube labeled

PS D:\k8s\deployment> kubectl.exe label nodes minikube-m02 location=telaviv
node/minikube labeled

PS D:\k8s\deployment> kubectl.exe label nodes minikube-m03 location=houston
node/minikube labeled

PS C:\Users\developer> kubectl.exe describe node minikube | Select-String location

                    location=tokyo


PS C:\Users\developer> kubectl.exe describe node minikube-m02 | Select-String location

                    location=telaviv


PS C:\Users\developer> kubectl.exe describe node minikube-m03 | Select-String location

                    location=houston

```

# 4. Make mongo's yaml file and run it. Check where the pod is running. 
```
PS D:\k8s\deployment> cat .\mongo_latest_nodeSelector.yaml

apiVersion: apps/v1
kind: Deployment
metadata:
  name: mongo-test
spec:
  selector:
    matchLabels:
      run: mongo-test
  replicas: 1
  template:
    metadata:
      labels:
        run: mongo-test
    spec:
      containers:
      - name: mongodb
        image: docker.io/mongo
        ports:
        - containerPort: 27017
      nodeSelector:
        location: houston

PS D:\k8s\deployment> kubectl.exe create -f .\mongo_latest_nodeSelector.yaml
deployment.apps/mongo-test created

PS D:\k8s\deployment> kubectl.exe describe pod mongo-test
Name:         mongo-test-7cb544b5bd-2hdh9
Namespace:    default
Node:         minikube-m03/192.168.99.105
Annotations:  <none>
Status:       Running
IP:           10.244.2.3
IPs:
  IP:           10.244.2.3
Controlled By:  ReplicaSet/mongo-test-7cb544b5bd
Containers:
  mongodb:
    Container ID:   docker://cc5a60a64f51c83bc92abacd3e8f332456dfc12731514275f7f2b3f0d02d2816
    Image:          docker.io/mongo
    Image ID:       docker-pullable://mongo@sha256:b66f48968d757262e5c29979e6aa3af944d4ef166314146e1b3a788f0d191ac3
    Port:           27017/TCP
    Host Port:      0/TCP
    State:          Running
    Ready:          True
    Restart Count:  0
    Environment:    <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from default-token-6b7ps (ro)
Conditions:
  Type              Status
  Initialized       True
  Ready             True
  ContainersReady   True
  PodScheduled      True
Volumes:
  default-token-6b7ps:
    Type:        Secret (a volume populated by a Secret)
    Optional:    false
QoS Class:       BestEffort
Node-Selectors:  location=houston
Tolerations:     node.kubernetes.io/not-ready:NoExecute op=Exists for 300s
                 node.kubernetes.io/unreachable:NoExecute op=Exists for 300s
Events:
  Type    Reason     Age   From               Message
  ----    ------     ----  ----               -------
  Normal  Scheduled  20s   default-scheduler  Successfully assigned default/mongo-test-7cb544b5bd-2hdh9 to minikube-m03
  Normal  Pulling    19s   kubelet            Pulling image "docker.io/mongo"
  Normal  Pulled     15s   kubelet            Successfully pulled image "docker.io/mongo" in 4.408630186s
  Normal  Created    15s   kubelet            Created container mongodb
  Normal  Started    15s   kubelet            Started container mongodb

```

# 5. Make employee's yaml file and run it. Check where the pod is running. 
```
PS D:\k8s\deployment> cat .\employee_latest_nodeSelector.yaml

apiVersion: apps/v1
kind: Deployment
metadata:
  name: employee-test
spec:
  selector:
    matchLabels:
      run: employee-test
  replicas: 1
  template:
    metadata:
      labels:
        run: employee-test
    spec:
      containers:
      - name: employee
        image: developeronizuka/employee
        ports:
        - containerPort: 5001
        - containerPort: 5000
        env:
        - name: MONGO
          value: "10.244.2.3"
        command: ["/usr/local/dotnet/publish/Employee"]
      nodeSelector:
        location: telaviv

PS D:\k8s\deployment> kubectl.exe create -f .\employee_latest_nodeSelector.yaml
deployment.apps/employee-test created

PS D:\k8s\deployment> kubectl.exe describe pod employee-test
Name:         employee-test-9d7bb4d74-w6nrw
Namespace:    default
Priority:     0
Node:         minikube-m02/192.168.99.104
Start Time:   Sun, 18 Apr 2021 00:05:35 +0900
Labels:       pod-template-hash=9d7bb4d74
              run=employee-test
Annotations:  <none>
Status:       Running
IP:           10.244.1.4
IPs:
  IP:           10.244.1.4
Controlled By:  ReplicaSet/employee-test-9d7bb4d74
Containers:
  employee:
    Container ID:  docker://d36453d8dd1b4139c24b22d9edc752d33f646a6f0c19622e8fe925d49e4b1dc5
    Image:         developeronizuka/employee
    Image ID:      docker-pullable://developeronizuka/employee@sha256:ad36f06fcb5aa8d4da7dc36ac9bf42223617c3330e8dfcaef1b5a30ba9f71084
    Ports:         5001/TCP, 5000/TCP
    Host Ports:    0/TCP, 0/TCP
    Command:
      /usr/local/dotnet/publish/Employee
    State:          Running
      Started:      Sun, 18 Apr 2021 00:05:42 +0900
    Ready:          True
    Restart Count:  0
    Environment:
      MONGO:  10.244.2.3
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from default-token-6b7ps (ro)
Conditions:
  Type              Status
  Initialized       True
  Ready             True
  ContainersReady   True
  PodScheduled      True
Volumes:
  default-token-6b7ps:
    Type:        Secret (a volume populated by a Secret)
    SecretName:  default-token-6b7ps
    Optional:    false
QoS Class:       BestEffort
Node-Selectors:  location=telaviv
Tolerations:     node.kubernetes.io/not-ready:NoExecute op=Exists for 300s
                 node.kubernetes.io/unreachable:NoExecute op=Exists for 300s
Events:          <none>

```

# 6. Create the persistent volume and make nginx's config on it.
```
PS D:\k8s\volume> kubectl.exe create -f .\nginx-pv.yaml
persistentvolume/hostpath-pv created

PS D:\k8s\volume> kubectl.exe create -f .\nginx-pvc.yaml
persistentvolumeclaim/hostpath-pvc created


PS C:\Users\developer> ssh docker@192.168.99.103
docker@192.168.99.103's password: tcuser

$ cd /tmp/data
$ sudo vi default.conf
upstream proxy.com {
        server 10.244.1.4:5001;
}

server {
        listen 80;
        server_name localhost;
        location / {
                root /usr/share/nginx/html;
                index index.html index.htm;
                proxy_pass https://proxy.com;
        }
}

```


# 7. Make nginx's yaml file and run it.  
```
PS D:\k8s\deployment> cat .\ngingx_1.14.2_nodeSelector.yaml

apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-test
spec:
  selector:
    matchLabels:
      run: nginx-test
  replicas: 1
  template:
    metadata:
      labels:
        run: nginx-test
    spec:
      containers:
      - name: nginx
        image: nginx:1.14.2
        volumeMounts:
        - name: nginx-conf
          mountPath: /etc/nginx/conf.d
        ports:
        - containerPort: 80
      volumes:
      - name: nginx-conf
        persistentVolumeClaim:
         claimName: hostpath-pvc
      nodeSelector:
        location: tokyo

PS D:\k8s\deployment> kubectl.exe create -f .\ngingx_1.14.2_nodeSelector.yaml
deployment.apps/nginx-test created
```

# 8. Expose the nginx pod and Access the following URL.
```
PS D:\k8s\deployment> kubectl.exe expose deployment nginx-test --type=LoadBalancer
service/nginx-test exposed

PS D:\k8s\deployment> minikube.exe service list
|-------------|------------|--------------|-----------------------------|
|  NAMESPACE  |    NAME    | TARGET PORT  |             URL             |
|-------------|------------|--------------|-----------------------------|
| default     | kubernetes | No node port |
| default     | nginx-test |           80 | http://192.168.99.103:30466 |
| kube-system | kube-dns   | No node port |
|-------------|------------|--------------|-----------------------------|

```
