# Storage in Docker
## File system

![image](https://github.com/user-attachments/assets/748b443c-d41e-41c3-8ecc-ab270e18c788)

## Layered architecture
When Docker builds images, it builds these in a layered architecture. Each line of instruction in the Docker file creates a new layer in the Docker image with just the changes from the previous layer. 

![image](https://github.com/user-attachments/assets/af05c2d6-0c50-4e7a-8c23-b371e25943f9)

You can see when build images which have changes, Docker is not going to build the first three layers. Instead, it reuses the same three layers it built for the first application frm cache. And only creates the last two layers with the new sources and the new entrypoint. This way Docker builds images faster and efficiently saves disc space. Thus saving us a lot of time during rebuilds and updates.

![image](https://github.com/user-attachments/assets/131191ef-63fc-4477-a570-868c603b4743)

If i were to log into the newly created container and say create a new file called temp.text, it will create that file in the container layer which is read and write. 

![image](https://github.com/user-attachments/assets/176969e3-9246-450f-b64b-c7568d8dc0a2)

The image layer being read only just means that the files in these layers will not be modified in the image itself. So the image will remain the same all the time until you rebuild the image using the docker build command.

All of the data that was stored in the container layer also gets deleted. The change we made to the app.py and the new temp file we created will also get removed.

## Volumes

![image](https://github.com/user-attachments/assets/cc50b4aa-74bc-4c6b-bcf8-d5f77071c545)

## Storage drivers

![image](https://github.com/user-attachments/assets/4579ef8b-1d69-448f-b4f8-9791fe727630)

## Volume Driver Plugins in Docker

![image](https://github.com/user-attachments/assets/07591d70-1b19-486b-9d6d-c20071b563ca)

## Container Storage Interface (CSI)
In the past, K8s used Docker alone as the container runtime engine, and all the code to work with Docker was embedded within the Kubernetes source code. With other container run times coming in. such as RKT and CRI-O, it was important to open up and extend support to work with different container runtimes, and not be dependent on the Kubernetes source code.

And that's how Container Runtime Interface (CRI) came to be. The CRI  is a standard that defines how an orchestration solution like Kubernetes would communicate with container runtimes like Docker.

![image](https://github.com/user-attachments/assets/70ea77a8-74a0-4b0f-a0aa-ca45a428b9ae)

### Container Network Interface

![image](https://github.com/user-attachments/assets/f3d1d58f-3e2f-4668-9c06-bcddef29f12e)

### Container Storage Interface
The CSI was developed to support multiple storage solutions. With CSI, you can now write your own drivers for your own storage to work with Kubernets ( Portworx, Amazon EBS, Azure Disk,...)

![image](https://github.com/user-attachments/assets/01e67761-51c0-4272-bef8-f2531d65b7ff)

Everyone's got their own CSI drivers. Note that CSI is not a Kubernetes specific standard. It is meant to be a universal standard, and if implemented, allows any container orchestration tool to work with any storage vendor with supported plugin.

Here's what the CSI kind of looks like.

![image](https://github.com/user-attachments/assets/4e12dad5-d272-4f0a-a671-0d2889d8feb4)

For example, CSI says that when a pod is created and requires a volume, the container orchestrator, in this case Kubernetes, should call the create volume RPC and pass a set of details such as the volume name. The storage driver should implement this RPC and handle that request and provision a new volume on the storage array, and return the results of the operation. Similarly, container orchestrator should call the delete volume RPC when a volume is to be deleted, and the storage driver should implement the code to decommission the volume from the array when that call is made. And the specification details exactly what parameters should be sent by the caller, what should be received by the solution, and what error codes should be exchanged.

## Volumes
### Volumes in Docker
Docker containers are meant to be transient in nature, which means they are meant to last only for a short period of time. They're called upon when required to process data and destroyed once finished.

To persist data processed by the containers, we attach a volume to the containers when they are created. The data processed by the container is now placed in this volume, thereby retaining it permanently. Even if the container is deleted, the data generated or processed by it remains.

![image](https://github.com/user-attachments/assets/419a6202-91c3-458e-9900-c73ce6f8a439)

So how does that work in Kubernets world?

Just as in Docker, the pods created in Kubernetes are transient in nature. When a pod is created to process data, and then deleted, the data processed by it, gets deleted as well. 

### Volumes & Mounts

![image](https://github.com/user-attachments/assets/e44abe16-7fa3-4ead-a9ea-0d714cf149ab)

We create a simple pod like above. It then get deleted along with random number. To retain the number generated by the pod, we create a volume. And a volume needs a storage. When you create  a volume, you can choose to configure its storage in different ways.

![image](https://github.com/user-attachments/assets/60567e4b-9455-43fc-8ce2-cb003dc5fcee)

Any files created in the volume would be stored in the directory data on my node.

![image](https://github.com/user-attachments/assets/de034e57-b552-4f26-8d74-25baa203baca)

Once the volume is created, to access it from a container we mount the volume to a directory inside the container. We use the volume mounts field in each container to mount the data volume to the directory, slash O P T within the container. The random number will now be written to **/opt** mount inside the container, which happens to be on the data volume, which is in fact the data directory on the host.

When the pod gets deleted, the file with the random number still lives on the host.

![image](https://github.com/user-attachments/assets/e8a3e7ff-cc44-4a9d-9777-16b5da8c567c)

### Volume Types

![image](https://github.com/user-attachments/assets/c105a795-c983-49d8-a565-73ebb12aa2ed)

Let's take a step back and look at the volume storage options. We just used the host path option to configure it directly on the host as storage space for the volume. Now that works fine on a single node, however, it is not recommended for use in a multi node cluster. This is because the pods would use the slash data directory on all the nodes, and expect all of them to be the same and have the same data. Since they're on different servers, they're in fact, not the same. Unless you configure some kind of external replicated cluster storage solution. Kubernetes supports several types of different storage solutions, such as NFS, cluster affairs, Flocker, etc..., or public cloud solutions like AWS, EBS, Azure desk, or file, or Google's Persistent Desk.

For example, to configure an AWS elastic block store volume as the storage option for the volume, we replaced host path field of the volume with the AWS Elastic block store field, along with the volume ID and file system type. The volume storage will now be on AWS EBS.

![image](https://github.com/user-attachments/assets/6b0abcb0-5cfe-4c8d-85d5-59a3279397b7)

### Persistent Volumes

















