# 1. Put Labels on each node
```
PS D:\k8s\deployment> kubectl.exe label nodes minikube location=tokyo
node/minikube labeled
PS D:\k8s\deployment> kubectl.exe label nodes minikube-m02 location=telaviv
node/minikube-m02 labeled
PS D:\k8s\deployment> kubectl.exe label nodes minikube-m03 location=houston
node/minikube-m03 labeled

PS D:\k8s\deployment> kubectl.exe describe nodes minikube | Select-String "location"

                    location=tokyo

PS D:\k8s\deployment> kubectl.exe describe nodes minikube-m02 | Select-String "location"

                    location=telaviv

PS D:\k8s\deployment> kubectl.exe describe nodes minikube-m03 | Select-String "location"

                    location=houston
```

# 2. Edit yaml file for Mongo and create Deployment and run it
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
Name:         mongo-test-7cb544b5bd-bh8th
Namespace:    default
Priority:     0
Node:         minikube-m03/192.168.99.102
Start Time:   Sat, 17 Apr 2021 21:34:43 +0900
Labels:       pod-template-hash=7cb544b5bd
              run=mongo-test
Annotations:  <none>
Status:       Running
IP:           172.17.0.2
IPs:
  IP:           172.17.0.2
Controlled By:  ReplicaSet/mongo-test-7cb544b5bd
Containers:
  mongodb:
    Container ID:   docker://1e740672dc827e28a993ebe1d415e817b18997ecae0ba7aff0e292df37141d06
    Image:          docker.io/mongo
    Image ID:       docker-pullable://mongo@sha256:b66f48968d757262e5c29979e6aa3af944d4ef166314146e1b3a788f0d191ac3
    Port:           27017/TCP
    Host Port:      0/TCP
    State:          Running
      Started:      Sat, 17 Apr 2021 21:39:11 +0900
    Ready:          True
    Restart Count:  0
    Environment:    <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from default-token-cjxcb (ro)
Conditions:
  Type              Status
  Initialized       True
  Ready             True
  ContainersReady   True
  PodScheduled      True
Volumes:
  default-token-cjxcb:
    Type:        Secret (a volume populated by a Secret)
    SecretName:  default-token-cjxcb
    Optional:    false
QoS Class:       BestEffort
Node-Selectors:  location=houston
Tolerations:     node.kubernetes.io/not-ready:NoExecute op=Exists for 300s
                 node.kubernetes.io/unreachable:NoExecute op=Exists for 300s
Events:
  Type    Reason     Age   From               Message
  ----    ------     ----  ----               -------
  Normal  Scheduled  16m   default-scheduler  Successfully assigned default/mongo-test-7cb544b5bd-bh8th to minikube-m03
  Normal  Pulling    16m   kubelet            Pulling image "docker.io/mongo"
  Normal  Pulled     12m   kubelet            Successfully pulled image "docker.io/mongo" in 4m27.704188904s
  Normal  Created    12m   kubelet            Created container mongodb
  Normal  Started    12m   kubelet            Started container mongodb
```

