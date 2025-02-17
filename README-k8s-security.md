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



















