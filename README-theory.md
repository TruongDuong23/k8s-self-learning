## Deploying your first Nginx Pod  - https://collabnix.github.io/kubelabs/pods101/deploy-your-first-nginx-pod.html
### K8s Pods?
- k8s Pods are the foundational unit for all higher k8s objects
- A pod hosts one or more containers
- Can be created using either a command or a YAML/JSON file
- k8s will attempt to restart a failing pod by default
### Why k8s use pod as the smallest deployable unit and not a single container?
To manage a container, Kubernetes needs additional information, such as a restart policy, which defines what to do with a container when it terminates, or a liveness probe, which defines an action to detect if a process in a container is still alive from the application’s perspective, such as a web server responding to HTTP requests.

Instead of overloading the existing “thing” with additional properties, Kubernetes architects have decided to use a new entity, the Pod, that logically contains (wraps) one or more containers that should be managed as a single entity.
### Why k8s allow more than one contianer in a Pod?
Containers in a Pod run on a “logical host”; they use the same network namespace (in other words, the same IP address and port space), and the same IPC namespace. They can also use shared volumes. These properties make it possible for these containers to efficiently communicate, ensuring data locality. Also, Pods enable you to manage several tightly coupled application containers as a single unit.

So if an application needs several containers running on the same host, why not just make a single container with everything you need? Well first, you’re likely to violate the “one process per container” principle. 
This is important because with multiple processes in the same container it is harder to troubleshoot the container. That is because logs from different processes will be mixed together and it is harder manage the processes lifecycle. Second, using several containers for an application is simpler, more transparent, and enables decoupling software dependencies. Also, more granular containers can be reused between teams.

### Atypical Pod creation Workflow
![image](https://github.com/collabnix/kubelabs/assets/34368930/458ca251-8d23-4a9b-a48d-e3172d7236c6)
The workflow for creating a Pod in k8s typically involves the following steps:
- Create a Pod Manifest: A Pod is defined using a YAML or JSON manifest file that describes its desired state. The manifest includes information such as the Pod name, container specifications, networking details, and any additional configurations.
- Apply the Manifest: Use the kubectl apply command to apply the Pod manifest and create the Pod. For example:
```Bash
kubectl apply -f pod.yaml
``` 
- API Server Validation: The kubectl apply command sends the Pod manifest to the Kubernetes API server. The API server validates the manifest’s syntax and checks for any conflicts or errors.
- Pod Scheduler: Once the Pod manifest is validated, the Kubernetes scheduler assigns the Pod to a suitable worker node. The scheduler takes into account factors such as resource availability, node affinity rules, and other scheduling constraints.
- Container Creation: The assigned worker node receives the Pod specification and initiates the creation of containers within the Pod. The container runtime, such as Docker or containerd, pulls the container images specified in the Pod manifest and starts the containers.
- Pod Status: The Pod goes through different status phases, including “Pending” while it’s being scheduled, “Running” when the containers are successfully started, and “Completed” or “Failed” when the Pod’s primary container finishes its execution.
- Monitoring and Logging: Kubernetes provides various monitoring and logging mechanisms to track the status, resource usage, and events related to the Pod. You can use tools like Prometheus, Grafana, or Kubernetes Dashboard to monitor and visualize Pod metrics.
### Pre-requisite:
- Play with Kubernetes
- Clone the repo
### Steps
```Bash
git clone https://github.com/collabnix/kubelabs
cd kubelabs/pods101
kubectl apply -f pods01.yaml
```
### Viewing your Pods
```Bash
kubectl get pods
```
### Which Node is this pod running on?
```Bash
kubectl get pods -o wide
```
```Bash
$ kubectl describe po webserver
Name:               webserver
Namespace:          default
Priority:           0
PriorityClassName:  <none>
Node:               gke-standard-cluster-1-default-pool-78257330-5hs8/10.128.0.3
Start Time:         Thu, 28 Nov 2019 13:02:19 +0530
Labels:             <none>
Annotations:        kubectl.kubernetes.io/last-applied-configuration:
                      {"apiVersion":"v1","kind":"Pod","metadata":{"annotations":{},"name":"webserver","namespace":"default"},"spec":{"containers":[{"image":"ngi...
                    kubernetes.io/limit-ranger: LimitRanger plugin set: cpu request for container webserver
Status:             Running
IP:                 10.8.0.3
Containers:
  webserver:
    Container ID:   docker://ff06c3e6877724ec706485374936ac6163aff10822246a40093eb82b9113189c
    Image:          nginx:latest
    Image ID:       docker-pullable://nginx@sha256:189cce606b29fb2a33ebc2fcecfa8e33b0b99740da4737133cdbcee92f3aba0a
    Port:           80/TCP
    Host Port:      0/TCP
    State:          Running
      Started:      Thu, 28 Nov 2019 13:02:25 +0530
    Ready:          True
    Restart Count:  0
    Requests:
      cpu:        100m
    Environment:  <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from default-token-mpxxg (ro)
Conditions:
  Type              Status
  Initialized       True
  Ready             True
  ContainersReady   True
  PodScheduled      True
Volumes:
  default-token-mpxxg:
    Type:        Secret (a volume populated by a Secret)
    SecretName:  default-token-mpxxg
    Optional:    false
QoS Class:       Burstable
Node-Selectors:  <none>
Tolerations:     node.kubernetes.io/not-ready:NoExecute for 300s
                 node.kubernetes.io/unreachable:NoExecute for 300s
Events:
  Type    Reason     Age    From                                                        Message
  ----    ------     ----   ----                                                        -------
  Normal  Scheduled  2m54s  default-scheduler                                           Successfully assigned default/webserver to gke-standard-cluster-1-default-pool-78257330-5hs8
  Normal  Pulling    2m53s  kubelet, gke-standard-cluster-1-default-pool-78257330-5hs8  pulling image "nginx:latest"
  Normal  Pulled     2m50s  kubelet, gke-standard-cluster-1-default-pool-78257330-5hs8  Successfully pulled image "nginx:latest"
  Normal  Created    2m48s  kubelet, gke-standard-cluster-1-default-pool-78257330-5hs8  Created container
  Normal  Started    2m48s  kubelet, gke-standard-cluster-1-default-pool-78257330-5hs8  Started container
```
### Output in JSON
```Bash
$ kubectl get pods -o json
{
    "apiVersion": "v1",
    "items": [
        {
            "apiVersion": "v1",
            "kind": "Pod",
            "metadata": {
                "annotations": {
                    "kubectl.kubernetes.io/last-applied-configuration": "{\"apiVersion\":\"v1\",\"kind\":\"Pod\",\"metadata\":{\"annotations\":{},\"name\":\"webserver\",\"namespace\":\"default\"},\"spec\":{\"con
tainers\":[{\"image\":\"nginx:latest\",\"name\":\"webserver\",\"ports\":[{\"containerPort\":80}]}]}}\n",
                    "kubernetes.io/limit-ranger": "LimitRanger plugin set: cpu request for container webserver"
                },
                "creationTimestamp": "2019-11-28T08:48:28Z",
                "name": "webserver",
                "namespace": "default",
                "resourceVersion": "20080",
                "selfLink": "/api/v1/namespaces/default/pods/webserver",
                "uid": "d8e0b56b-11bb-11ea-a1bf-42010a800006"
            },
            "spec": {
                "containers": [
                    {
                        "image": "nginx:latest",
                        "imagePullPolicy": "Always",
                        "name": "webserver",
                        "ports": [
                            {
                                "containerPort": 80,
                                "protocol": "TCP"
                            }
                        ],
                        "resources": {
                            "requests": {
                                "cpu": "100m"
                            }
                        },
                        "terminationMessagePath": "/dev/termination-log",
                        "terminationMessagePolicy": "File",
```
### Executing Commands Against Pods
```Bash
$ kubectl exec -it webserver -- /bin/bash
root@webserver:/#
```
```Bash
root@webserver:/# cat /etc/os-release
PRETTY_NAME="Debian GNU/Linux 10 (buster)"
NAME="Debian GNU/Linux"
VERSION_ID="10"
VERSION="10 (buster)"
VERSION_CODENAME=buster
ID=debian
HOME_URL="https://www.debian.org/"
SUPPORT_URL="https://www.debian.org/support"
BUG_REPORT_URL="https://bugs.debian.org/"
```
Please exit from the shell (/bin/bash) session
```Bash
root@webserver:/# exit
```
### Deleting the Pod
```Bash
$ kubectl delete -f pods01.yaml
pod "webserver" deleted

$ kubectl get po -o wide
No resources found.
```
### Get logs of Pod
```Bash
$ kubectl logs webserver

/docker-entrypoint.sh: /docker-entrypoint.d/ is not empty, will attempt to perform configuration
/docker-entrypoint.sh: Looking for shell scripts in /docker-entrypoint.d/
/docker-entrypoint.sh: Launching /docker-entrypoint.d/10-listen-on-ipv6-by-default.sh
10-listen-on-ipv6-by-default.sh: Getting the checksum of /etc/nginx/conf.d/default.conf
10-listen-on-ipv6-by-default.sh: Enabled listen on IPv6 in /etc/nginx/conf.d/default.conf
/docker-entrypoint.sh: Launching /docker-entrypoint.d/20-envsubst-on-templates.sh
/docker-entrypoint.sh: Configuration complete; ready for start up
```

## Adding a 2nd container to a Pod
In the microservices architecture, each module should live in its own space and communicate with other modules following a set of rules. 
But, sometimes we need to deviate a little from this principle. Suppose you have an Nginx web server running and we need to analyze its web logs in real-time. 
The logs we need to parse are obtained from GET requests to the web server. The developers created a log watcher application that will do this job and they built a container for it. 
In typical conditions, you’d have a pod for Nginx and another for the log watcher. However, we need to eliminate any network latency so that the watcher can analyze logs the moment they are available. 
A solution for this is to place both containers on the same pod.

Having both containers on the same pod allows them to communicate through the loopback interface (```ifconfig lo```) as if they were two processes running on the same host. They also share the same storage volume.

Let us see how a pod can host more than one container. Let’s take a look to the ```pods02.yaml``` file. It contains the following lines:
```Bash
apiVersion: v1
kind: Pod
metadata:
  name: webserver
spec:
  containers:
  - name: webserver
    image: nginx:latest
    ports:
    - containerPort: 80
  - name: webwatcher
    image: afakharany/watcher:latest
```
Run the following command:
```Bash
$ kubectl apply -f pods02.yaml
```
```Bash
$ kubectl get po -o wide
NAME        READY   STATUS              RESTARTS   AGE   IP       NODE                                                NOMINATED NODE   READINESS GATES
webserver   0/2     ContainerCreating   0          13s   <none>   gke-standard-cluster-1-default-pool-78257330-5hs8   <none>           <none>
```
```Bash
$ kubectl get po,svc,deploy
NAME            READY   STATUS    RESTARTS   AGE
pod/webserver   2/2     Running   0          3m6s
NAME                 TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
service/kubernetes   ClusterIP   10.12.0.1    <none>        443/TCP   107m
```
```Bash
$ kubectl get po -o wide
NAME        READY   STATUS    RESTARTS   AGE     IP         NODE                                                NOMINATED NODE   READINESS GATES
webserver   2/2     Running   0          3m37s   10.8.0.5   gke-standard-cluster-1-default-pool-78257330-5hs8   <none>           <none>
```
### How to verify 2 containers are running inside a Pod?
```Bash
$ kubectl describe po
```
```Bash
Containers:
  webserver:
    Container ID:   docker://0564fcb88f7c329610e7da24cba9de6555c0183814cf517e55d2816c6539b829
    Image:          nginx:latest
    Image ID:       docker-pullable://nginx@sha256:36b77d8bb27ffca25c7f6f53cadd059aca2747d46fb6ef34064e31727325784e
    Port:           80/TCP
    State:          Running
      Started:      Wed, 08 Jan 2020 13:21:57 +0530
    Ready:          True
    Restart Count:  0
    Requests:
      cpu:        100m
    Environment:  <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from default-token-xhgmm (ro)
  webwatcher:
    Container ID:   docker://4cebbb220f7f9695f4d6492509e58152ba661f3ab8f4b5d0a7adec6c61bdde26
    Image:          afakharany/watcher:latest
    Image ID:       docker-pullable://afakharany/watcher@sha256:43d1b12bb4ce6e549e85447678a28a8e7b9d4fc398938a6f3e57d2908a9b7d80
    Port:           <none>
    State:          Running
      Started:      Wed, 08 Jan 2020 13:22:26 +0530
    Ready:          True
    Restart Count:  0
    Requests:
```
Since we have two containers in a pod, we will need to use the ```-c``` option with ```kubectl``` when we need to address a specific container. For example:
```Bash
$ kubectl exec -it webserver -c webwatcher -- /bin/bash

root@webserver:/# cat /etc/hosts
# Kubernetes-managed hosts file.
127.0.0.1       localhost
::1     localhost ip6-localhost ip6-loopback
fe00::0 ip6-localnet
fe00::0 ip6-mcastprefix
fe00::1 ip6-allnodes
fe00::2 ip6-allrouters
10.8.0.5        webserver
```
Please exit from the shell (```/bin/bash```) session.
```Bash
root@webserver:/# exit
```
### Cleaning up
```Bash
kubectl delete -f pods02.yaml
```
### Example of Multi-Container Pod
Let’s talk about communication between containers in a Pod. Having multiple containers in a single Pod makes it relatively straightforward for them to communicate with each other. They can do this using several different methods.

#### Use Cases
The primary purpose of a multi-container Pod is to support co-located, co-managed helper processes for a primary application. There are some general patterns for using helper processes in Pods:

```Sidecar containers``` help the main container. Some examples include log or data change watchers, monitoring adapters, and so on. A log watcher, for example, can be built once by a different team and reused across different applications. Another example of a sidecar container is a file or data loader that generates data for the main container.

```Proxies, bridges, and adapters``` connect the main container with the external world. For example, Apache HTTP server or nginx can serve static files. It can also act as a reverse proxy to a web application in the main container to log and limit HTTP requests. Another example is a helper container that re-routes requests from the main container to the external world. This makes it possible for the main container to connect to the localhost to access, for example, an external database, but without any service discovery.
#### Shared volumes in a Kubernetes Pod
In Kubernetes, you can use a shared Kubernetes Volume as a simple and efficient way to share data between containers in a Pod. For most cases, it is sufficient to use a directory on the host that is shared with all containers within a Pod.

Kubernetes Volumes enables data to survive container restarts, but these volumes have the same lifetime as the Pod. That means that the volume (and the data it holds) exists exactly as long as that Pod exists. If that Pod is deleted for any reason, even if an identical replacement is created, the shared Volume is also destroyed and created anew.

A standard use case for a multi-container Pod with a shared Volume is when one container writes logs or other files to the shared directory, and the other container reads from the shared directory. For example, we can create a Pod like so (```pods03.yaml```):
```Bash
apiVersion: v1
kind: Pod
metadata:
  name: mc1
spec:
  volumes:
  - name: html
    emptyDir: {}
  containers:
  - name: 1st
    image: nginx
    volumeMounts:
    - name: html
      mountPath: /usr/share/nginx/html
  - name: 2nd
    image: debian
    volumeMounts:
    - name: html
      mountPath: /html
    command: ["/bin/sh", "-c"]
    args:
      - while true; do
          date >> /html/index.html;
          sleep 1;
        done
```
In this file (```pods03.yaml```) a volume named ```html``` has been defined. Its type is ```emptyDir```, which means that the volume is first created when a Pod is assigned to a node, and exists as long as that Pod is running on that node. As the name says, it is initially empty. The ```1st``` container runs nginx server and has the shared volume mounted to the directory ```/usr/share/nginx/html```. The ```2nd``` container uses the Debian image and has the shared volume mounted to the directory ```/html```. Every second, the ```2nd``` container adds the current date and time into the ```index.html``` file, which is located in the shared volume. When the user makes an HTTP request to the Pod, the Nginx server reads this file and transfers it back to the user in response to the request.
![image](https://raw.githubusercontent.com/collabnix/kubelabs/master/pods101/multicontainerpod.png)
```Bash
kubectl apply -f pods03.yaml
```
```Bash
[Captains-Bay]🚩 >  kubectl get po,svc
NAME      READY     STATUS    RESTARTS   AGE
po/mc1    2/2       Running   0          11s

NAME             TYPE        CLUSTER-IP    EXTERNAL-IP   PORT(S)   AGE
svc/kubernetes   ClusterIP   10.15.240.1   <none>        443/TCP   1h
[Captains-Bay]🚩 >  kubectl describe po mc1
Name:         mc1
Namespace:    default
Node:         gke-k8s-lab1-default-pool-fd9ef5ad-pc18/10.140.0.16
Start Time:   Wed, 08 Jan 2020 14:29:08 +0530
Labels:       <none>
Annotations:  kubectl.kubernetes.io/last-applied-configuration={"apiVersion":"v1","kind":"Pod","metadata":{"annotations":{},"name":"mc1","namespace":"default"},"spec":{"containers":[{"image":"nginx","name":"1st","v...
              kubernetes.io/limit-ranger=LimitRanger plugin set: cpu request for container 1st; cpu request for container 2nd
Status:       Running
IP:           10.12.2.6
Containers:
  1st:
    Container ID:   docker://b08eb646f90f981cd36c605bf8fead3ca62178c7863598fd4558cb026ed067dd
    Image:          nginx
    Image ID:       docker-pullable://nginx@sha256:36b77d8bb27ffca25c7f6f53cadd059aca2747d46fb6ef34064e31727325784e
    Port:           <none>
    State:          Running
      Started:      Wed, 08 Jan 2020 14:29:09 +0530
    Ready:          True
    Restart Count:  0
    Requests:
      cpu:        100m
    Environment:  <none>
    Mounts:
      /usr/share/nginx/html from html (rw)
      /var/run/secrets/kubernetes.io/serviceaccount from default-token-xhgmm (ro)
  2nd:
    Container ID:  docker://63180b4128d477810d6062342f4b8e499de684ffd69ad245c29118e1661eafcb
    Image:         debian
    Image ID:      docker-pullable://debian@sha256:c99ed5d068d4f7ff36c7a6f31810defebecca3a92267fefbe0e0cf2d9639115a
    Port:          <none>
    Command:
      /bin/sh
      -c
    Args:
      while true; do date >> /html/index.html; sleep 1; done
    State:          Running
      Started:      Wed, 08 Jan 2020 14:29:14 +0530
    Ready:          True
    Restart Count:  0
    Requests:
      cpu:        100m
    Environment:  <none>
    Mounts:
      /html from html (rw)
      /var/run/secrets/kubernetes.io/serviceaccount from default-token-xhgmm (ro)
Conditions:
  Type              Status
  Initialized       True 
  Ready             True 
  ContainersReady   True 
  PodScheduled      True 
Volumes:
  html:
    Type:    EmptyDir (a temporary directory that shares a pod's lifetime)
    Medium:  
  default-token-xhgmm:
    Type:        Secret (a volume populated by a Secret)
    SecretName:  default-token-xhgmm
    Optional:    false
QoS Class:       Burstable
Node-Selectors:  <none>
Tolerations:     node.kubernetes.io/not-ready:NoExecute for 300s
                 node.kubernetes.io/unreachable:NoExecute for 300s
Events:
  Type    Reason     Age   From                                              Message
  ----    ------     ----  ----                                              -------
  Normal  Scheduled  18s   default-scheduler                                 Successfully assigned default/mc1 to gke-k8s-lab1-default-pool-fd9ef5ad-pc18
  Normal  Pulling    17s   kubelet, gke-k8s-lab1-default-pool-fd9ef5ad-pc18  pulling image "nginx"
  Normal  Pulled     17s   kubelet, gke-k8s-lab1-default-pool-fd9ef5ad-pc18  Successfully pulled image "nginx"
  Normal  Created    17s   kubelet, gke-k8s-lab1-default-pool-fd9ef5ad-pc18  Created container
  Normal  Started    17s   kubelet, gke-k8s-lab1-default-pool-fd9ef5ad-pc18  Started container
  Normal  Pulling    17s   kubelet, gke-k8s-lab1-default-pool-fd9ef5ad-pc18  pulling image "debian"
  Normal  Pulled     13s   kubelet, gke-k8s-lab1-default-pool-fd9ef5ad-pc18  Successfully pulled image "debian"
  Normal  Created    12s   kubelet, gke-k8s-lab1-default-pool-fd9ef5ad-pc18  Created container
  Normal  Started    12s   kubelet, gke-k8s-lab1-default-pool-fd9ef5ad-pc18  Started container
```
```Bash
$ kubectl exec mc1 -c 1st -- /bin/cat /usr/share/nginx/html/index.html
...
Wed Jan  8 08:59:14 UTC 2020
Wed Jan  8 08:59:15 UTC 2020
Wed Jan  8 08:59:16 UTC 2020
 
$ kubectl exec mc1 -c 2nd -- /bin/cat /html/index.html
...
Wed Jan  8 08:59:14 UTC 2020
Wed Jan  8 08:59:15 UTC 2020
Wed Jan  8 08:59:16 UTC 2020
```
#### Cleaning Up
```Bash
kubectl delete -f pods03.yaml
```
#### Running Promethus along with Nginx Container + Config in a single Pod
```Bash
kubectl apply -f pods105.yaml
```

### Conclusion




## Exception knowledge
![image](https://github.com/user-attachments/assets/045884e7-c1a3-4c05-899b-0c7e07a933d6)


# Note:
##JSON PATH in Kubectl:

![image](https://github.com/user-attachments/assets/2e3a949d-d669-4d0d-9ad6-45d3219e1bf7)

##JSON PATH for Sort

![image](https://github.com/user-attachments/assets/cad66b70-d725-41f5-8b24-5362823aa1ef)
















