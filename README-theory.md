## Deploying your first Nginx Pod
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

Having both containers on the same pod allows them to communicate through the loopback interface (ifconfig lo) as if they were two processes running on the same host. They also share the same storage volume.

Let us see how a pod can host more than one container. Let’s take a look to the pods02.yaml file. It contains the following lines:
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
$ kubectl apply -f pods02.yaml
```
```Bash
$ kubectl apply -f pods02.yaml
```



















