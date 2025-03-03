# Autoscaling
- Horizontal Pod Autoscaling (HPA): adding more resources to existing application.
- Vertical Pod Autoscaling (VPA): adding more instances or more servers to your system.

![image](https://github.com/user-attachments/assets/9ce53d96-6047-450e-baa4-eb262e149100)

We mentioned that one of the major purposes of a container orchestrator like Kubernetes is to host applications in the form of containers and scale up and down based on demand. So, there are two types of scaling in Kubernetes. What we just spoke about is scaling workloads.

So, you can scale the workloads by adding or removing containers or pods onto the cluster. Another approach to scaling is scaling the underlying cluster itself. So, that's adding or removing more servers or infrastructure to your cluster.

![image](https://github.com/user-attachments/assets/12579099-cb00-46c3-b332-a9583aeaaa24)

So, there's scaling workloads and scaling the cluster infra. So when it comes to the cluster, horizontal scaling refers to adding more nodes to the cluster. Vertical scaling would mean increasing the resources on existing nodes in the cluster. When it comes to scaling workloads, horizontal scaling would mean creating more pods, and vertical scaling would mean increasing the resources allocated to existing pods.

![image](https://github.com/user-attachments/assets/a90dd339-f67e-4cb9-85dd-a952a370004f)

So, there are two ways of scaling. There's the manual way and the automated way.

![image](https://github.com/user-attachments/assets/ca506033-2c81-4696-9663-5aba90466b14)

## Horizontal Pod Autoscaler (HPA)

![image](https://github.com/user-attachments/assets/b93d69ee-2bdd-4f87-8dc6-9dafe937e334)

So, the Horizontal Pod Autoscaler continuously monitors the metrics as we did manually using the top command. It then automatically increases or decreases the number of pods in a deployment stateful set or replica set based on the CPU memory, or custom metrics. And if the CPU memory or memory usage goes too high, HPA creates more pods to handle that. And if it drops, it removes the extra pods to save resources. And this balances the thresholds. And note that it can also track multiple different types of metrics, which we'll refer to in a few minutes.

![image](https://github.com/user-attachments/assets/fa99f7b3-9181-4c4f-9467-e2b7e42736a5)

For the given NGINX deployment, we can configure a Horizontal Pod Autoscaler by running the kubectl autoscale command targeting the deployment my-app, and specifying a CPU threshold of 50% with a minimum of 1 and maximum of 10 pods. So when this command is run, Kubernetes creates a Horizontal Pod Autoscaler for this deployment that first reads the limits configured on the pod, and then learns that it's set to 500 millicore. It then continuously pulls the metric server to monitor the usage, and when the usage goes beyond 50%, it modifies the number of replicas to scale up or down depending on the usage. So, to view the status of the created HPA, run the **kubectl get hpa** command, and it lists the current HPA. The targets column shows the current CPU usage versus the threshold we have set, and the minimum and maximum, and the current count of replicas. So, it would never go beyond the maximum that we have specified when scaling up and it would not go beyond the minimum that we have specified when scaling down.

![image](https://github.com/user-attachments/assets/0255b36c-0479-48a0-a426-58caa88c18db)

## Metrics

![image](https://github.com/user-attachments/assets/5a94881c-a93d-49ed-9d0d-9e165b182e17)

So, talking about metrics server, we spoke about the metrics server that HPA depends on to get current resource utilization numbers. Now what we have been referring to is the internal metrics server, but there are also other resources that we can refer to, such as a Custom Metrics Adapter that can retrieve information from other internal sources like a workload deployed in a cluster. However, these are still internal sources. We can also refer to external sources such as tools or other instances that are outside of the Kubernetes cluster, such as a Datadog or Dynatrace instance using an external adapter.

## In-place Resize of Pod Resources

![image](https://github.com/user-attachments/assets/3872181e-ca62-4b5a-b3f6-55c27468f7fb)

If you change resource requirements of a pod in a deployment, the default behavior is to delete the existing pod and then spin up a new pod with the new changes. So any changes to the resource definitions on a pod does not happen in place, which means the pod needs to be killed and the new pod with the new resource definitions need to be created.

![image](https://github.com/user-attachments/assets/78532166-dd5f-435d-a083-a3fda282f144)

![image](https://github.com/user-attachments/assets/62f13a50-f6f2-4c1a-a5f4-728ed27ee27b)

Now, we know that this can be disruptive, especially for stateful workloads. So there is an improvement that is being worked upon called as in-place update of pod resources. This is a feature that is currently in alpha as of Kubernetes release 1.27 and is not enabled by default. Now, when it goes to the beta or stable stage in the future, it will become enabled by default. But as with this recording, it is not. So, it is still available with the features that are part of the Kubernetes cluster. You just have to enable it. So to enable this feature, you must set the feature flag called InPlacePodVerticalScaling to True.

So the new resize policy options allow you to specify a restart policy for each resource.

![image](https://github.com/user-attachments/assets/d6c73b8d-f17e-4e6b-b020-fc09d654036f)

In this example, we have defined that a change in CPU resource will not require the pod to be restarted, and a change in memory will require a restart of the pod. And next, we make a change to a resource such as updating the CPU to one. And you can see that the pod does not need to be killed. Instead, it can simply be updated with the new resources and so it can just increase in size. So that explains how we can resize a CPU or a memory resource assigned to a container without really restarting it.

### Limitations
- Only CPU and memory resources can be changed.
- Pod QoS Class can not change.
- Init containers and Ephemeral Containers can not be resized.
- Resource requests and limits can not be removed once set.
- A containers's memory limit may not be reduced below its usage. If a request puts a container in this state, the resize status will remain in InProgress untill the desired memory limit becomes feasible.
- Windows pods can not be resized.


## Vertical Pod Autoscaling (VPA)

![image](https://github.com/user-attachments/assets/49d85421-971f-4ce4-a6df-de3f0dcf2a59)

So, similar to the Horizontal Pod Autoscaler, the Vertical Pod Autoscaler will continuously monitors the metrics and then it automatically increases or decreases the resources assigned to the pods in a deployment and thus, balances the workload.

![image](https://github.com/user-attachments/assets/a903fedd-bafc-4e6e-92f2-4178ae6e7eb3)

So unlike the Horizontal Pod Autoscaler, the Vertical Pod Autoscalers do not come built in. As such, we must deploy it.

![image](https://github.com/user-attachments/assets/c91ecd9f-552c-479c-a0b9-b49f8b6bbd80)

So, we first apply the Vertical Pod Autoscaler definition file available in the GitHub repo.

![image](https://github.com/user-attachments/assets/ffe42be0-d9b8-49ec-8a99-17eda660e4db)

And then, when we run the kubectl get pods command in the kube-system namespace and search for vpa, we will be able to see that there are multiple components deployed. So there is the admission-controller, recommender, and the updater service which should be running.

The VPA deployment consists of multiple components:
- VPA Admission Controller: intervenes the pod creation process and uses the recommendations from the Recommender again to then mutate the pod spec to apply the recommended CPU and memory values at startup. And this ensures that the newly created pods start with the correct resource requests.
- VPA Updater: detects pods that are running with suboptimal resources and evicts them when an update is needed. So, it gets the information from the Recommender and monitors the pod. And if a pod needs to be updated, it evicts them. So, that means it just terminates the pod.
- VPA Recommender: responsible for continuously monitoring resource usage from the Kubernetes metrics API and collects historical and live usage data for pods, and then provides recommendations on optimal CPU and memory values. And the Recommender itself does not modify the pod directly, or it only suggests changes.

![image](https://github.com/user-attachments/assets/3d3e944e-056c-4c9d-a6ba-d47b52969819)

So basically, the VPA Recommender collects information, the Updater monitors or gets the information from the Recommender, compares that to the actual pod, and if the pod is beyond the threshold, it kills the pod. And whether it kills or not, that depends on the policy that we will talk about in a bit. But ideally, it would kill the pod, and then the Admission Controller intervenes because when a pod is killed, it's automatically... The deployment will automatically recreate the pod. And when it does that, the Admission Controller intervenes and updates the resources so that the pod now comes up with the new size.

![image](https://github.com/user-attachments/assets/5a393e87-e19d-41a9-a42a-35c59a52fdc7)

There are four modes that VPA operates in.

![image](https://github.com/user-attachments/assets/355f9e13-ba35-4ace-9680-315d41558c8f)

Now, VPA will monitor resource usage and suggest adjustments, so we can check the recommendations with the command kubectl describe vpa followed by the name of the VPA.

![image](https://github.com/user-attachments/assets/e73045a1-b450-4913-93ec-37f2452dfe63)

### Key Differences

![image](https://github.com/user-attachments/assets/949fe77e-c59f-43ae-87b4-11f68e098369)















