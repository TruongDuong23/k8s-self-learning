# Kubernetes Security
- kube-api: like center, controll all operations in k8s, interact with it through the kube-control utility or by accessing the API directly --> can perform almost operations on the cluster

## Authentication
### Who can access?
By the authentication mechanisms, there are many ways to authenticate to the API server:
- Files - Username and Passwords
- Files - Username and Tokens
- Certificates
- External Authentication providers - LDAP
- Service Accounts
### What can they do?
Authorization is implemented using role-based access controls where users are associated to groups with specific permissions. In addition, there are other authorization modules like the attribute-based access control
- RBAC Authorization
- ABAC Authorization
- Node Authorization
- Webhook Mode
Communication among components is secured using TLS encryption

As we know, there are many nodess, virtual,components... work with together. Beside, we have:
* users like administrators that access the cluster  to perform administrative tasks
* the developers that access the cluster to test or deploy applications
* end users who access the applications deployed on the cluster
* Third party applications (Bots) access the cluster for integration purpose
   
====> To secure our cluster, by securing the communication between internal components and securing management access to the cluster through authentication and authorization mechanisms

![image](https://github.com/user-attachments/assets/c679fab7-03e0-486b-afdf-191253346ef9)

### Account
Security of end users, who access the applications deployed on the cluster is managed by the applications themselves, internally.  --> out of discuss

Focus: on users' access to the k8s cluster (admins, usres, bots <-> Service Accounts) for administrative purposes

So we are left with two types of users (users and service account): humans such as the administrators and developers, and robots such as other processes or services or applications that require access to the cluster. 
Kubernetes does not manage user accounts natively. It relies on an external source like a file with user details or certificates or third party identify service like LDAP to manage these users. And so you can not create users in a Kubernetes cluster or view the list of users like this.

![image](https://github.com/user-attachments/assets/2bd12cf4-7c22-4d1d-b194-1fd12d1a1f79)

However, in case of service accounts, Kubernetes can manage them. You can create and manage service accounts using the k8s API. We have a section on service accounts exclusively where we discuss and practice more about service accounts

![image](https://github.com/user-attachments/assets/061d7b41-a386-4b05-9630-510faba9c8d3)

### Auth Mechanisms
All user access is managed by the API server whether you're accessing the cluster through kubecontrol tool or the API directly. All of these requets go through the kube-apiserver. The kube-apiserver authenticates the request before processing it. 

So how does the kube-apiserver authenticate?

There are different authentication mechanisms that can be configured. You can have a list of username and password in a static password file or usernames and tokens in a static token file or you can authenticate using certificates. And another option is to connect to third party authentication protocols like LDAP, Kerberos etc...

![image](https://github.com/user-attachments/assets/70afebf6-08a7-4acc-bd2f-4ba7c369ab84)

#### Static password and token files
Let's start with the simplest form of authentication. You can create a list of users and their passwords in a **CSV file** and use that as the code for user information. The fiel has three columns, password, username and user ID. We then pass the file name as an option to the kube-apiserver. Remember kube-apiserver.service and the various options we looked at earlier in this course. That is where you must specify this option **--basic-auth-file=user-details.csv**. 

![image](https://github.com/user-attachments/assets/ac8255fd-8e86-450e-9490-82e1b2cd3eb3)



You must then restart the kube-apiserver for these options to take effect. If you set up your cluster using the Kubeadm tool, then you must modify the kube-apiserver POD definition file. The kubeadm tool will automatically restart the kube-apiserver once you update this file

![image](https://github.com/user-attachments/assets/54d1fcfa-450a-4c7a-8ba0-277c3f8dbfcb)
![image](https://github.com/user-attachments/assets/113b71c5-7ff9-4b7e-bb62-b550ed8028f0)

#### Authenticate User
To authenticate using the basic credentials while accessing the API server, specify the user and password in a curl command like this

![image](https://github.com/user-attachments/assets/19e901db-9fdd-4339-afc3-7b53e3928b89)

#### Auth Mechanisms - Basic
In the CSV file with the user details that we saw, we can optionally have a fourth column with the group details to assign users to specific groups

![image](https://github.com/user-attachments/assets/0cc39692-ee55-4e39-bae8-f6e44286d9cc)

Similarly, instead of a static password file, you can have a static token file

![image](https://github.com/user-attachments/assets/fcd5499e-ab36-402e-94b4-e4f2f05bc0f6)

Here instead of password, you specify a token. Pass the token file as an option, tokenauth file to the kube-apiserver. While authenticating specify the token as an authorization barrier token to your request like this

![image](https://github.com/user-attachments/assets/394714a7-5b53-4da5-975a-3ee50ae0d823)

### Note
- This is not a recommended authentication mechanism
- Consider volume mount while providing the auth file in a kubeadm setup
- Set up Role Based Authorization for the new users


## TLS Certitficates (PRE_REQ)
A certificate is used to guarantee trust between two parties during a transaction. For example, when a user tries to access a web server, TLS certificates ensure that the communication between the user and the server is encrypted and the server is who it says it is.

![image](https://github.com/user-attachments/assets/0735dd0e-6188-4d17-8326-5ac14db717e8)

Let's take a look at a scenario.

![image](https://github.com/user-attachments/assets/fac267d8-d109-4a9a-af91-5a8c6bbc624b)

Without secure connectivity, if a user were to access his online banking application, the credentials he types in would be sent in a plain text format. The hacker sniffing network traffic could easily retrieve the credentials and use it to hack into the user's bank account.

![image](https://github.com/user-attachments/assets/1a6f11ce-1970-4780-8225-03a97ea6df9e)

So you must encrypt the data being transferred using encryption keys. The data is encrypted using a key, which is basically a set of random numbers and alphabets.

![image](https://github.com/user-attachments/assets/9bae5779-633a-450c-bf29-95edcde6cc54)

You add the random number to your data and you encrypt it into a format that can not be recognized.

![image](https://github.com/user-attachments/assets/df1bcc49-cbe5-47d6-ba22-6ecd43864e78)

The data is then sent to the server. The hacker sniffing the network gets the data, but can't do anything with it 

![image](https://github.com/user-attachments/assets/5240ea67-a484-4aef-b33d-421efe89c50d)

### Asymmetric encryption
However, the same is the case with the server receiving the data. It can not decrypt the data without the key, so a copy of the key must also be sent to the server so that the server can decrypt and read the message. Since the key is also sent over the same netwok, the attacker can sniff that as well and decrypt the data with it. This is known as **symmetric encryption**. It is a secure way of encryption, but since it uses the same key to encrypt and decrypt the data, and since the key has to be exchanged between the sender and the receiver, there is a risk of a hacker gaining access to the key and decrypting the data and that's where asymmetric encryption comes in. Instead of using a single key to encrypt and decrypt data, asymmetric encryption uses a pair of keys: a private key and a public key. We'll call it a private key and a public lock.

![image](https://github.com/user-attachments/assets/a6031707-4586-40ba-9a21-1f3fbbf90110)

A key, which is only with me so it's private. A lock that anyone can access, so it's public.

The trick here is, if you encrypt or lock the data with your lock, you can only open it with the associated key. So your key must always be secure with you and not be shared with anyone else, it's private. But the lock is public and maybe shared with others but they can only lock something with it. No matter what is locked using the public lock, it can only be unlocked by your private key.

![image](https://github.com/user-attachments/assets/72fb1213-f33e-4d70-bbde-c6e65be56be3)
- ID_RSA: the private key (Can be shared openly and is used to encrypt data or verify signatures)
- ID_RSA.pub: the public key (public lock) (Must be kept secret and is used to decrypt data or create signatures)
  
Then secure server by locking down all access to it, except through a door that is locked using your public lock. It's usually done by adding an entry with public key into the server's SSH authorized_keys file

![image](https://github.com/user-attachments/assets/45b891de-b0ae-4074-8bbb-8f746947ff56)

So the lock is public and anyone can attempt to break through. But as long as no one gets their hands on your private key, which is safe with you on your laptop, no one can gain access to the server. When you try to SSH, you specify the location of your private key in your SSH command.

What will you do when you have more than one server with your key pair? You can create copies of your public log and place them on as many servers as you want.

What if other users need access to your servers? They can generate their own public and private key pairs. You can create an additional door for them and lock it with their public locks, copy their public locks to all the servers

![image](https://github.com/user-attachments/assets/dcb137f9-d7e4-4c47-af86-e634b695bb7d)


Problem with symmetric encryption: the key used to encrypt data had to be sent to the server over the network along with the encrypted data. So there is a risk of hacker getting the key to decrypt the data.

What if we could somehow get the key to the server safely? To securely transfer the symmetric key from client to the server, we use asymmetric encryption. So we generate a public and private key pair on the server. We're going to refer to the public log as public key going forward.

When the user first accesses the web server using STTPS, he gets the public key from the server. Since the hacker is sniffing all traffic, let us assume he too gets a copy of the public key. 

![image](https://github.com/user-attachments/assets/d2b26084-9884-4456-b89b-cf5786172392)

The user, in fact, the user's browser then encrypts the symmetric key using the public key provided by the server. The symmetric key is now secure

![image](https://github.com/user-attachments/assets/8358ff4f-834b-4fea-927b-1122c4c946b9)

The user then sends this to the server. The hacker also gets a copy. The server uses the private key to decrypt the massage and retrieve the symmetry key from it. However, the hacker does not have the private key to decrypt and retrieve the symmetric key from the massage it received. The hacker only has the public key with which he can only lock or encrypt a message and not decrypt the message. The hacker is left with the encrypted messages and public keys with which he can't decrypt any data.

![image](https://github.com/user-attachments/assets/708bc3b3-4dbc-4947-8a5e-fbb89e56f46c)

The hacker now looks for new ways to hack into your account, and so he realizes that the only way he can get your credential is by getting you to type it into a form he presents. So he creates a website that looks exactly like your bank's website. The design is the same, the graphics are the same, the website is a replica of the actual bank's website. He hosts the website on his own server. So he generates his own set of public and private key pairs and configures them on his web server. And finally, he somehow manages to tweak your environment or your network to route your request going to your bank's website to his servers.

![image](https://github.com/user-attachments/assets/f6c48e90-6975-4718-853b-cf5fe4f9de27)

What if you could look at the key you received from the server and say if it's legitimate key from the real bank server? When the server sends the key, it does not send the key along, it sends a certificate that has the key in it, you will see that it is like an actual certificate but in a digital format. It has information about who the certificate is issued to the public key of that server, the location of that server, et cetera. On the right, you see the output of an actual certificate. Every certificate has a name on it, the person or subject to whom the certificate is issued to that is very important as that is the field that helps you validate their identify. This must match what the user types in the URL on his browser.

![image](https://github.com/user-attachments/assets/b35f290b-2299-452e-8591-78256d9e09fd)

So how do you look at a certificate and verify if it is legit? Who signed and issued the certificate?  If you generated a certificate then you'll have to sign it by yourself. That is known as a self-signed certificate.

How do you create a legitimate certificate for your web servers that the web browsers will trust? How do you get your certificates signed by someone with authority? That's where certificate authorities or CAs comes in. They're well known organizations that can sign and validate your certificates for you. Some of the popular ones are symantec, DigiCert, Comodo, GlobalSign, et cetara.

![image](https://github.com/user-attachments/assets/289e0b1f-89df-43e7-af6a-d753bfa9650f)

You generate a certificate signing a request or CSR (Certificate Signing Request) using the key you generated earlier and the domain name of your website.

![image](https://github.com/user-attachments/assets/e42c3b61-e1d4-42ab-aa5b-b5b3b094d56d)
- Validate Information
- Sign and send certificate

If hacker tried to get his certificate signed the same way, he will fail during the validation phase and his certificate would be rejected by the CA.

How do the browsers know that the CA itself was legimate?  
- For example, one if the certificate was signed by a fake CA, in this case, our certificate was signed by Symantec. How would the brownser know Symantec is a valid CA and that the certificate was in fact signed by Symantec and not by someone who says they are Symantec?
- The CAs themselves have a set of public and private key pairs. The CAs use their private keys to sign the certificates. The public keys of all the CAs are built in to the browsers. The browser uses the public key of the CA to validate tha the certificate was actually signed by the CA themselves.

You can actually see them in the settings of your web browser under certificates, they're under trusted CAs tab.

![image](https://github.com/user-attachments/assets/f19f23fe-4ad9-4456-9ec5-afe30eb99a63)

That help us ensure the public websites we visit like our banks, emails, et cetera, are legitimate. However, they don't help you validate sites, hosted privately, say within your organization. For example, for accessing your payroll or intenal email applications. For that, you can host your own private CAs. Most of these companies listed here have a private offering of their services, a CA server that you can deploy internally within the company. You can then have the public key of your internal CAs server installed on all your employees browsers and establish secure connectivity within your organization.

![image](https://github.com/user-attachments/assets/648f1c60-e581-41fc-8630-5b6c26c3fa9a)

### TLS-Summary
An admin uses a pair of keys to secure SSH connectivity to the servers. The server uses a pair of keys to secure a STPS traffic.

**CERTIFICATE AUTHORITY (CA)**
the server first sends a certificate signing request to a CA, the CA uses its private key to sign the CSR. (all users have a copy of the CAs public key)

![image](https://github.com/user-attachments/assets/923bc989-4173-4ff8-be00-18b28ff0b643)

The signed certificate is then sent back to the server. The server configures the web application with the signed certificate.

![image](https://github.com/user-attachments/assets/20509eb1-efe2-4aa4-b26e-a68e155780dc)

Whenever a user accesses the web application, the server first sends the certificate with its public key. The user, or rather the user's browser reads the certificate and uses the CAs public key to validate and retrieve the server's public key. It then generates a symmetric key that it wishes to use going forward for all communication.

The certificate authority generates its own set of keeper to sign certificates. The end user though only generates a single symmetric key. Once he establishes trust with the website, he uses his username and password to authenticate to the web server. But the server's keepers, the client was able to validate that the server is who they say they are but the server does not for sure know if the client is who they say they are. It could be a hacker impersonating a user by somehow gaining access to his credentials, not over the network for sure as we have secured it already with TLS, maybe by some other means.

![image](https://github.com/user-attachments/assets/00f92a39-cb0f-4b4b-a50e-e65dc0f3e73d)


## TLS in Kubernetes

![image](https://github.com/user-attachments/assets/e437e6f1-addf-428c-8d25-65d23b61d745)

The Kubernetes cluster consists of a set of master and worker nodes. Of course, all communication between these nodes need to be secure and must be encrypted. All interactions between all services and their clients need to be secure.

![image](https://github.com/user-attachments/assets/03ec3328-ef67-49fb-951d-509539e48954)

### Server Certificates for Servers
As we know already, the API server exposes an HTTPS service that other components, as well as external users, use to manage the Kubernetes cluster. So it is a server and it requires certificates to secure all communication with its clients. So we generate a certificate and key pair. We call it APIserver.cert and APIserver.key. We will try to stick to this naming convention going forward. Anything with a .CRT extension is the certificate and .key extension is the private key. These certificate names could be different in different Kubernetes setups depending on who and how the cluster was set up.

Another server in the cluster is the etcd server. The etcd server stores all information about the cluster. So it requires a pair of certificate and key for itself. We will call it etcdserver.crt and etcdserver.key.

The other server component in the cluster is on the worker nodes. They are the kubelet services. They also expose an HTTPS API endpoint that the kube-apiserver talks to interact with the worker nodes: kubelet.cert and kubelet.key.

![image](https://github.com/user-attachments/assets/213cdb8f-c397-4c3a-94bf-3bb1abe4f4a5)

### Client Certificates for Clients
Who are the clients who access these services? Are us, the administrators through kubectl Arrest API. The admin user requires a certificate and key pair to authenticate to the kube-apiserver: admin.crt and admin.key.

The scheduler talks to the kube-apiserver to look for pods that require scheduling and then get the API server to schedule the pods on the right worker nodes. The scheduler is a client that accesses the kube-apiserver. The scheduler is just another client, like the admin user. So the scheduler needs to validate its identity using a clien TLS certificate. So it needs its own pair of certificate and keys: scheduler.cert and scheduler.key.

The kube-controller-manager is another client that accesses the kube-apiserver, so it also requires a certificate for authentication to the kube-apiserver: controller-manager.crt and controller-manager.key

The kube-proxy requires a client certificate to authenticate to the kube-apiserver and so it requires its own pair of certificate and keys: kube-proxy.crt and kube-proxy.key.

![image](https://github.com/user-attachments/assets/30ffec67-60bd-4f51-9485-b81ef8408c29)

So there are many certificates. Let try and group them. 

![image](https://github.com/user-attachments/assets/e45be9d2-7b4c-4ca2-a684-6acecd3dc1fb)

There are a set of client certificates, mostly used by clients to connect to the kube-apiserver and there are s et of server site certificates used by the kube-apiserver, etcd server and kubelet to authenticate their clients.

![image](https://github.com/user-attachments/assets/18d4f8f0-0f53-4259-b2ec-59eb2026c915)
![image](https://github.com/user-attachments/assets/7ac41f25-9cba-4b92-93d2-16e8c98107d7)

### Certificate Creation
To generate certificates, there are different tools available such as EASYRSA, OpenSSL, CFSSL, etc. We will focus OpenSSL
#### Certificate Authority (CA)
First, we create a private key using the OpenSSL command (Generate Keys):
```
openssl genrsa -out ca.key 2048
```
!!! (image)
Then we use the OpenSSL Request command along with the key we just created to generate a certificate signing request. The certificate signing request is like a certificate with all of your details but with no signature (Certificate signing Request)
```
openssl req -new -key ca.key -subj "/CN=KUBERNETES-CA" -out ca.csr
```
!!!
In the certificate signing request we specify the name of the component the certificate is for in the Common Name or CN field. So certificate is named **KUBERNETES-CA**

Finally, we sign the certificate using the **openssl x509** command and by specifying the certificate signing request, we generated in the previous command. Since this is for the CA itself, it is self-signed by the CA using its owning private key that it is generated in the first step.
```
openssl x509 -req -in ca.csr -signkey ca.key -out ca.crt
```

![image](https://github.com/user-attachments/assets/a5e97341-0a54-4d3f-b9fd-b28ec5336dcb)

![image](https://github.com/user-attachments/assets/08dc009d-2a76-47a8-a094-8ae8e523a8ba)

example:
![image](https://github.com/user-attachments/assets/914d9872-b951-477c-ac8e-71b396a937b5)

Instead of using user and password then access like Rest API, we can config by yaml file for the certificates to use

![image](https://github.com/user-attachments/assets/02afafc8-3237-4f8b-8bb6-7a5a5e40bf1c)

Remember in the prerequisite lecture we mentioned that for clients to validate the certificates sent by the server and vice versa, they all need a copy of the certificate authorities public certificate. The one that we said is already installed within the user's browsers in case of a web application.

To secure communication between the different members in the cluster, we must generate additional peer certificates. Once the certificates are generated specify them while starting the ETCD server. 

![image](https://github.com/user-attachments/assets/0c85b3ee-1491-4244-9bc1-9de987b4704a)

The kubelets server is an ACTPS API server that runs on each node, responsible for managing the node. That is who the API server talks to monitor the node as well as send information regarding what pods to schedule on this node. Such as, you need a key certificate pair for each node in the cluster.

![image](https://github.com/user-attachments/assets/ea84427f-8a41-46a1-be05-a702b1662d00)


### View Certificate Details
There are many different methods to generate and manage certificates. If you were to deploy a k8s cluster from scratch, you generate all the certificates by yourself as above. Or else if you were to rely on an automated provisioning tool like KubeADM, it takes care of automatically generating and configuring the cluster for you. While you deploy all the components as native services on the notes in the hard way, the KubeADM tool deploys these as pods.

![image](https://github.com/user-attachments/assets/81cbc7a3-51c2-4d63-a315-4778f8c52308)

First, look into kube-apiserver anf find more details:
![image](https://github.com/user-attachments/assets/11a1fd21-34a0-4d8f-8170-7c93a63ab239)
![image](https://github.com/user-attachments/assets/8880a999-c0af-48e8-a10a-b656a720e47d)
![image](https://github.com/user-attachments/assets/78de5b3b-8710-4ac6-94b5-b2e005bb8ea6)
![image](https://github.com/user-attachments/assets/02414ba1-d0d9-4727-a038-28989aefbe4e)

The certificate requirements are listed in detail in the k8s documentation page.

![image](https://github.com/user-attachments/assets/0c4720e6-acd1-4a2b-9fa2-7d69411daf15)

#### Inspect Service Logs 
![image](https://github.com/user-attachments/assets/348fda94-1348-407b-aa6a-bc08ef20f792)


#### View Logs
In case you set up the cluster with Kube ADM then the various components are deployed as pods. So you can look at the logs, using the kube-control logs command followed by the pod name.

![image](https://github.com/user-attachments/assets/aff9c148-68eb-4729-9fc3-9439bd6bb18f)

Sometimes if the core components such as the k8s API server or the NCD server are down, the kube-control commands won't function. In that case, you have to go one level down to Docker to fetch the logs. List all the containers using the **docker ps -a** command

![image](https://github.com/user-attachments/assets/c1ca18c6-a0b0-4502-8c96-e97322ce8832)

And then view the logs using the Docker logs command followed by the container ID

![image](https://github.com/user-attachments/assets/43a09c9f-c52f-46d9-8ec9-23ba6864c13b)

### Certificate workflow & API
We'll look at how to manage certificates and what the certificate API is in k8s.

![image](https://github.com/user-attachments/assets/82785e96-44d5-47a1-b03a-2abaae17fb45)

A user first create a key, then generate a certificate signing request using the key with her name on it, then sends the request to the administrator.

![image](https://github.com/user-attachments/assets/85b64ee7-c8a5-4a40-89c7-0b934c741016)

Takes the key and creates a CertificateSigningRequest object. The CertificateSigningRequest object is created like any other k8s object, using a manifest file with the usual fields. The kind is CertificateSigningRequest. 

The request field is where you specify the certificate signing request sent by the user, but you don't specify it as plaintext. Instead, it must be encoded using the Base64 command. Then move the encoded text into the request field and then submit the request. 

![image](https://github.com/user-attachments/assets/e54c8cce-9bf6-4faf-a871-2ed32e4f3ab2)

Once the object is created, all certificate signing requests can be seen by administrators by running the **kubectl get csr** command. 

![image](https://github.com/user-attachments/assets/6561a3f9-4a9c-48ac-953a-a9f3fd1cad07)

Identify the new request and approve the request by running the **kubectl certificate approve** command.

![image](https://github.com/user-attachments/assets/1893a0e8-6d42-4284-b54c-d70b9dc486a0)

Kubernetes signs the certificate using the CA key pairs and generates a certificate for the user. This certificate can then be extracted and shared with the user. 

View the certificate by viewing it in a YAML format.

![image](https://github.com/user-attachments/assets/c5143300-9c75-45c6-8470-e7fc5279343c)

The generated certificate is part of the output, but as before, it is in a Base64 encoded format. To decoded and use the Base64 utility's decode option. This gives the certificate in a plaintext format. This can be shared with user

![image](https://github.com/user-attachments/assets/0b1226cd-b31f-4081-8271-80ed7d7fb8d3)

All the certificate-related operations are carried out by the controller-manager.

![image](https://github.com/user-attachments/assets/9e49d460-bcb6-49e9-a399-72edfce2c535)
![image](https://github.com/user-attachments/assets/8ac913e3-ff2a-46c7-b0fc-5f44a480c83a)

If anyone has to sign certificates, they need the CA server's root certificate and private key. The Controller-manager service configuration has 2 options where you can specify these.

![image](https://github.com/user-attachments/assets/d1d79e54-e945-43d5-a998-0827d10732ea)


## KubeConfig
![image](https://github.com/user-attachments/assets/8da86e3f-f8d6-4c9a-a63a-31ac91c4abe7)

You can move information to a configuration file called as kubeconfig and then specify this file as the kubeconfig option in your command. 

![image](https://github.com/user-attachments/assets/b68ce76a-c855-4a10-9336-fb5304629257)

By default, the kubectl tool looks for a file named config under a directory .kube under the user's home directory: **$HOME/.kube/config**

### KubeConfig File
![image](https://github.com/user-attachments/assets/45193ed7-7a35-4d48-a6b8-c69184b7d045)
![image](https://github.com/user-attachments/assets/c9cf5453-ff53-4005-87a7-f0ea8e1bb7f2)

The server specification in our command goes into cluster section (MyKubePlayground). The admin user's keys and certificates goes into user section (MyKubeAdmin). You then create a context that specifies to use the MyKubeAdmin user to access the MyKubePlayground cluster

![image](https://github.com/user-attachments/assets/880f3fc8-2e7f-4466-87f5-b14f477ef12a)

The kubeconfig file is in a YAML format. 

![image](https://github.com/user-attachments/assets/8b3cd649-a3c8-458c-98a2-c3f028b63476)
![image](https://github.com/user-attachments/assets/12b27293-c4ff-47ee-bf5a-3b2891e91fa6)

You can specify the default context used by adding a field current context to the kubeconfig file, specify the name of the context to use. In this case, kubectl will always use the context dev user at Google to access the Google clusters using the dev user's credentials.

To view the current file being used, run the kubectl config view command. 

![image](https://github.com/user-attachments/assets/93e0729d-b96f-4a41-ae22-d9207dd6fe09)

It lists the cluster's contexts and users, as well as the current context that is set. If you do not specify which kubeconfig file to use, it ends up using the default file located in the folder .kube in the user's home directory. Alternatively, you can specify a kubeconfig file by passing the kubeconfig option in the command line like this.

![image](https://github.com/user-attachments/assets/d429034b-4f81-40e0-a741-4f4f8e0f5f8e)

Run the kubectl config use context command to change the current context to the prod user at production context. And you also need to point to existed context like **kubectl config use-context research --kubeconfig /root/my-kube-config**

![image](https://github.com/user-attachments/assets/b62a83c9-6936-411a-ad2d-13c51869ed4a)

You can make other changes in the file, update or delete items in it using other variations of the kubectl config command. 

![image](https://github.com/user-attachments/assets/35bac44b-8063-4b67-98c5-a4a21fe780e6)

### Certificates in KubeConfig
![image](https://github.com/user-attachments/assets/ff511982-fb11-4ac0-b566-bd228999a58f)
![image](https://github.com/user-attachments/assets/1b0f5af3-b374-4f6d-bfb1-94aa32f92c27)

## API Groups
The Kubernetes API is grouped into multiple such groups based on their purpose, such as: 

![image](https://github.com/user-attachments/assets/16e8172f-7e69-4383-b8b2-c326d3d1f37a)

The metrics and health API are used to monitor the health of the cluster. The logs are used for integrating with third-party logging applications. We focus on apis, responsible for the cluster functionality.

![image](https://github.com/user-attachments/assets/ac638ef5-b5cb-489f-8a90-c24538c132b7)

The core group is where all core functionality exists, such as name, spaces, pods,..

![image](https://github.com/user-attachments/assets/c5c74742-f0a2-4a04-9c79-78fb4e1d0146)

The named group APIs are more organized and going forward, all the newer features are going to be made available through these named groups.

![image](https://github.com/user-attachments/assets/f4548119-91d7-446e-b715-d4f905b583a1)

Access your Kube API Server at port 6443 without any path and it will list you the available API groups. 

![image](https://github.com/user-attachments/assets/a45e2818-f692-41a9-8630-3f2725d7ec80)

And then, within the named API groups, it returns all the supported resource groups.

![image](https://github.com/user-attachments/assets/ca3894a3-4f71-49c8-bb87-c210caedf1b4)

If you were to access the API directly through curl, then you will not be allowed access except for certain APIs like version, as you have not specified any authentication mechanisms. So you have to authenticate to the API using your certificate files by passing them

![image](https://github.com/user-attachments/assets/70ae3a7f-21e6-4028-a653-0217c0182198)

### kubectl proxy
An alternate option is to start a Kube control proxy client. The kube control proxy command launches a proxy service locally on port 8001 and uses credentials and certificates from your kube config file to access the cluster. That way, you don't have to specify those in the curl command.

![image](https://github.com/user-attachments/assets/5cb10be7-02cd-4ade-9660-853634d4532a)

Here are two terms that kind of sound the same: The Kube proxy and Kube control proxy.

![image](https://github.com/user-attachments/assets/0319c8f0-283e-4529-8987-87356fbf4daf)

- The kube proxy is used to enable connectivity between pods and services across different nodes in the cluster. 
- The kube control proxy is an ACTP proxy service created by kube control utility to access the kube API server.

## Authorization
First of all, why do you need authorization in your cluster? 

As an administrator of the cluster, we were able to perform all sorts of operations in it, such as viewing various objects like pods, nodes,... As an admin, we are able to perform any operation but soon we will have others accessing the cluster as well such as the pther administrators, developers, testers or other applications like monitoring applications or continuous delivery applications like Jenkins, et cetera. So we will be creating accounts for them to access the cluster by creating usernames and passwords or tokens, or signed TL certificates or service accounts. But we don't want all of them to have the same level of access as us. 

For example, we don't want the developers to have access to modify our cluster configuration, like add or del nodes or the storage or networking configurations. We can allow them to view but not modify, but they could have access to deploying applications.

![image](https://github.com/user-attachments/assets/d8a4e20d-a5d2-4bd1-bb95-6ed2c80a53bb)

When we share our cluster between different organizations or teams, by logically partitioning it using namespaces, we want to restrict access to the users to their namespaces alone. 

### Authorization Mechanisms 
#### Node 
Kube API Server is accessed by users for management purposes, as well as the kubelets on node within the cluster for management process. The kubelet accesses the API server to read information about services and points, nodes and pods. The kubelet also reports to the Kube API Server with information about the node, such as it's status.

![image](https://github.com/user-attachments/assets/471f6c0d-b2bd-43c0-bfe5-e915c3b871f3)

#### ABAC
External access to the API. Attribute-based authorization is where you associate a user or a group of users with a set of permissions. In this case, dev user can view, create and delete pods. You do this by creating a policy file with a set of policies defined in adjacent format, this way you pass this file into the API server. 

![image](https://github.com/user-attachments/assets/13f94c65-b10a-48cf-8317-c7a7248bcf78)

Everytime you need to add or make a change in the security, you must edit this policy file manually and restart the Kube API Server.

![image](https://github.com/user-attachments/assets/6dad35f8-7a47-4eda-8ba8-116604ee7ff8)

#### Webhook
You want to manage authorization externally and not through the built-in mechanisms that we just discussed. 

For instance, Open Policy Agent is a third-party tool that helps with admission control and authorization. You have Kubernetes make an API call to the OPA with the information about the user and his access requirements, and have the OPA decide if the user should be permitted or not. Based on that response, the user is granted access.

![image](https://github.com/user-attachments/assets/fc3f6602-f9f8-4e9a-a221-869de87e2801)

#### Authorization Mode
2 more modes: Allways Allow and Always Deny.
- Always Allow: allows all requests without performing any authorization checks.
- Always Deny: deny all requests.

![image](https://github.com/user-attachments/assets/97fe9fae-94df-465d-a462-72ff5b6d737f)

The modes are set using the Authorization Mode Option on the Kube API Server. If you don't specify this option, it is set **Always Allow** by default. you may provide a comma separated list of multiple modes that you wish to use. 

![image](https://github.com/user-attachments/assets/13e64636-e4cd-495c-b63b-7989ddb09c0b)
![image](https://github.com/user-attachments/assets/9293bfb2-68a9-4f93-8ee9-364a097905db)

### Role Based Access Controls - RBAC
So how do we create a role? We do that by creating a role object. So we create a role definition file with the API version set to **rbac.authorization.k8s.io/v1**.

![image](https://github.com/user-attachments/assets/b922e5ca-b5fd-44e1-85a1-c576fec79982)

The next step is to link the user to that role. The roll binding object links a user object to a role.

![image](https://github.com/user-attachments/assets/c5dc718c-b788-40fa-9273-6dcc41f1e17f)

If you want to limit the dev user's access within a different namespace then specify the namespace within the metadata of the definition file while creating them. 

#### View RBAC
To view creating role

![image](https://github.com/user-attachments/assets/ab28d6d2-4e55-4972-accf-1af660dcf2ba)

To list role bindings

![image](https://github.com/user-attachments/assets/26c6ab52-d049-435e-bf63-ec1e8f8b2161)

To view more details about the role

![image](https://github.com/user-attachments/assets/4e811409-0b57-4c35-808f-b9b62eba4677)

Similarly, to view details about rule bindings 

![image](https://github.com/user-attachments/assets/710025e7-9e4e-41a3-9bde-137efc3b654a)

#### Check Access

![image](https://github.com/user-attachments/assets/4f5241df-e722-41f8-8a7c-ae0600b9ddf1)

If you're an administrator, then you can even impersonate another user to check their permission.

For instance, you were tasked to create necessary set of permissions for user to perform a set of operations for a user to perform a set of operations and you did that, but you would like to test if what you dis is working. You don't have to authenticate as the user to test it. Instead, you can use the same command with the AS user. Since we did not grant the deeloper permissions to create deployments, it returns no. The dev user has access to vreating pods through.

![image](https://github.com/user-attachments/assets/8d498b1e-bc69-44d1-b5d8-2045a676e6f3)

You can also specify the namespace in the command , the dev does not have permission to create a pod in the test namespace

![image](https://github.com/user-attachments/assets/ed54d917-e367-4024-ae94-4df3159652ea)

### Resource Names

![image](https://github.com/user-attachments/assets/0a5bbfca-8e10-4122-bdfd-3fa154a185b9)

We just saw how you can provide access to users for resource like pods within the namespace. For example, say you have 5 parts in namespace, you wanna give access to a user to pods, but not all pods. You can restrict access to the blue and orange pod alone by adding a **resourcenames** field to the rule.

![image](https://github.com/user-attachments/assets/2b221710-53d0-48ac-a1ae-730519c5251b)


## Cluster Roles and Role Bindings
### Roles

![image](https://github.com/user-attachments/assets/3cc8b5e4-0933-40b8-8c43-1a434758e72c)

### Namespace
Cluster-wide or cluster-scoped resources. They can't be associated to any particular namespace. So the resources are categorized as either namespaced or cluster-scoped.

![image](https://github.com/user-attachments/assets/24e1897e-0fe6-4d90-a30c-68103173a41f)

How to authorize a user to namespace resources, we used roles and role bindings for that. But how do we authorize users to cluster-wide resources like nodes or persistent volumes.That is where you use cluster roles and cluster rolebindings.

### Cluster roles
are just like roles, except they are for cluster-scoped resources. For example, a cluster admin role can be created to provide a cluster administrator permissions to view, create, or delete nodes in a cluster. Similarly, a storage administrator role can be created to authorize a storage admin to create persistent volumes and claims. Create a cluster role definition file.

![image](https://github.com/user-attachments/assets/8b102207-1e52-4ac3-8916-21206e6d67a5)


## Service Accounts
There are two types of accounts in Kubernetes, a user account and a service account.
- The user account is used by humans. A user account could be for an administrator accessing the cluster to perform administrative tasks, or a developer accessing the cluster to deploy applications, et cetera. 
- The service accounts are used by machines. A service account could be an account used by an application to interact with the Kubernetes cluster.

To create a service account, run command

![image](https://github.com/user-attachments/assets/3c4d5b1e-3ada-47dc-b429-d3722d8ea15e)

To view the serivce accounts, run command will list all the service accounts.

![image](https://github.com/user-attachments/assets/df33bc9c-0e66-464f-b8dc-2733c4c8081a)

When the service account is created, it also creates a token automatically. The service account token is waht must be used by the external application while authenticating to the Kubernetes API. The token, however, is stored as a secret object. 

![image](https://github.com/user-attachments/assets/96bb9919-3dbf-46a1-8ee0-d5484bd4fc04)

When a service account is created, it first creates the service account object, and then generates a token for the service account. It then creates a secret object ans stores that token inside the secret object. the secret object is then linked to the service account. To view the token, view the secret object by running command **kubectl describe secret**

![image](https://github.com/user-attachments/assets/f2d1436b-d02e-424d-b16e-058ad6bebef4)

This token can be used as an authentication bearer token while making a risk call to the Kubernetes API.

You can create a service account, assign the right permissions using role-based access control mechanisms, and export your service account tokens, and use it to configure your third-party application to authenticate to the Kubernetes API. 

![image](https://github.com/user-attachments/assets/da4661a5-9a0a-4ec7-b12a-03af29efcf97)

You don't have to provide it manually. If you go back and look at the list of service accounts, you will see that there is a default service account that exists already. For every namespace in Kubernetes, a service account named default is automatically created. Each namespace has its own default service account. Whenever a pod is created, the default service account and it's token are automatically mounted to that pod as a volume mount.

You will see that a volume is automatically created from the secret named default token, which is in fact the secret containing the token for this default service account. The secret token containing the token for this default service account. The secret token is mounted at location **/var/run/secrets/kubernetes.io/serviceaccount** inside the pod.

![image](https://github.com/user-attachments/assets/f91cef08-5433-4ce3-9247-d7302fbde921)

From inside the pod, if you run ls command to list the contents of the directory, you will see the secret mounted as three separate files. The one with the  actual token is the file's named token. 

![image](https://github.com/user-attachments/assets/3725e51b-aaab-4bca-b061-eb8c5ffa5e2b)

If you view contents of that file, you will see the token to be used for accessing the Kubernetes APi.

![image](https://github.com/user-attachments/assets/04108bf7-3db1-4ad4-bc5c-a2f46e4228c7)

Remember that the default service account is very much restricted. It only has permission to run basic Kubernetes API queries. If you'd like to use a different service account such as the one we just created, modify the pod definition file to include a service account field to include a service account field and specify the name of the new service account. Remember you can not edit the SA of an existing pod. You must delete and recreate the pod.

However, in case of deployment, you will able to edit the SA as any changes to the pod definition file, will automatically trigger a new rollout for the deployment.

![image](https://github.com/user-attachments/assets/e72b1c8f-9ec4-4f1f-a422-2922dbb270bc)

Kubernetes automatically mounts the default service account if you haven't explicitly specified any. You may choose not to mouunt a service account automatically by setting the **automountServiceAccountToken: false** field in the pod spec section.

Take the token and if you decode the token using command

![image](https://github.com/user-attachments/assets/1667ee04-61b5-48a3-b778-0f99ffe39c7d)

Or you could just copy and paste this token in the JWT website at jwt.io, you will see that it has no expiry date defined in the payload section here on the right.

![image](https://github.com/user-attachments/assets/a43788af-c3d0-4663-bbef-29483a637d0d)



















