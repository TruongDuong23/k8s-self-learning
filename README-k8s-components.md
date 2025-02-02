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


Security of end users, who access the applications deployed on the cluster is managed by the applications themselves, internally.  --> out of discuss
Focus: on users' access to the k8s cluster (admins, usres, bots) for administrative purposes

So we are left with two types of users (users and service account): humans such as the administrators and developers, and robots such as other processes or services or applications that require access to the cluster.






