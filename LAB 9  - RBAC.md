## RBAC

RBAC stands for Role-Based Access Control. 

It is a method of regulating access to resources based on the roles of individual users within an organization. With RBAC, you can specify what actions (such as create, read, update, delete) a user or group of users can perform on specific Kubernetes resources.

**Roles:** A role is a set of permissions that define what actions can be performed on Kubernetes resources within a namespace. For example, you can define a role that allows read-only access to pods or a role that allows full control over deployments.

**RoleBindings:** A role binding binds a role to a user, group, or service account within a namespace. It specifies who has access to the resources defined by the role. For example, you can create a role binding that associates a specific role with a particular user, granting them the permissions defined by that role.

**Cluster Role:** A ClusterRole is a set of permissions that can be applied across the entire Kubernetes cluster.ClusterRoles are not namespace-scoped; they apply cluster-wide. This means that permissions granted by a ClusterRole affect resources in all namespaces.

**Cluster Role Binding:** A ClusterRoleBinding binds a ClusterRole to a set of subjects (users, groups, or service accounts) across the entire cluster. It effectively grants the permissions defined in the ClusterRole to the subjects specified in the binding. Similar to ClusterRoles, ClusterRoleBindings are also cluster-wide and apply across all namespaces.


## Task 1: Launching the Kubernetes Dashboard using Cluster Role Binding

The Kubernetes Dashboard is a web-based user interface for Kubernetes clusters. It provides a graphical interface for users to manage and monitor their Kubernetes clusters and applications running within them.

The dashboard user interface is not deployed by default. To deploy it, run the following command:
```
kubectl apply -f https://raw.githubusercontent.com/kubernetes/dashboard/v2.7.0/aio/deploy/recommended.yaml
```
Enter the following commands to verify if the pods, services, and deployments have been created:
```
kubectl get pods -n kubernetes-dashboard -o wide
```
```
kubectl get deployment -n kubernetes-dashboard -o wide
```
```
kubectl get svc -n kubernetes-dashboard -o wide
```
To access the service outside the cluster, edit the service type from ClusterIP to NodePort using the following command:
```
kubectl edit svc -n kubernetes-dashboard kubernetes-dashboard
```
To confirm that the service type has been changed to NodePort, use the command:
```
kubectl get svc -n kubernetes-dashboard -o wide
```
To determine the location of the pod, run the following commands:
```
kubectl get pods -n kubernetes-dashboard -o wide
```
```
kubectl get svc -n kubernetes-dashboard -o wide
```
```
kubectl get nodes -o wide
```
Change the IP and NodePort accordingly:
IP : External IP of your master node

https://<"your worker-node-1">:<"NodePort">

Click on the Advanced button

Click Accept the Risk and Continue

Create a service account by running the following command, and then input the code in the master node:
```
vi ServiceAccount.yaml
```
```
apiVersion: v1
kind: ServiceAccount
metadata:
  name: admin-user
  namespace: kubernetes-dashboard
```

Apply the YAML file with the command:
```
kubectl apply -f ServiceAccount.yaml
```

Create a yaml file for cluster role binding using below command and code:
```
vi ClusterRoleBinding.yaml
```
```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: cr-binding1
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: cluster-admin
subjects:
- kind: ServiceAccount
  name: admin-user
  namespace: kubernetes-dashboard
```
Run the below command to create cluster role binding:
```
kubectl apply -f ClusterRoleBinding.yaml
```
Retrieve the token to log in by running the following command:
```
kubectl get secret $(kubectl get serviceaccount admin-user -n kubernetes-dashboard -o jsonpath='{.secrets[0].name}') -n kubernetes-dashboard -o jsonpath='{.data.token}' | base64 --decode
```
OR
```
kubectl -n kubernetes-dashboard create token admin-user
```
Copy the token and paste it into the Kubernetes dashboard login screen, then click Sign in

Cleanup: To delete the Kubernetes dashboard version 2.5, use the following command in the master node:
```
kubectl delete -f https://raw.githubusercontent.com/kubernetes/dashboard/v2.5.0/aio/deploy/recommended.yaml
```
By following these steps, you will be able to deploy the Kubernetes Dashboard, establish secure access, and navigate the interface to monitor and manage your Kubernetes cluster.

## Task 2: Implementing RBAC Using Namespaces (Allowing the User to access Namespace)

#### Prepare user
switch to the root user, create a new user named **test-user**, and switch to the newly created user's home directory.
```
sudo su
```
Create user and set password with below command:
```
adduser test-user
```
```
su test-user
```
```
cd /home/test-user
```
#### Generate TLS Certificates: 
Create a directory named cert to store TLS certificates. Then we generate a private key (test-user.key) and a Certificate Signing Request (CSR) (test-user.csr) for the user test-user.

```
mkdir cert && cd cert
```
```
openssl genrsa -out test-user.key 2048
```
```
openssl req -new -key test-user.key -out test-user.csr 
```
Verify the above created certificate files:
```
ls -l
```

Switch back to the  **root** user and jump to cert directory
Note:-Either run "exit" or "su root" (if root passswor is set) command to switch root user
```
exit
```
```
cd /home/test-user/cert/
```
Sign the CSR to generate a certificate (test-user.crt) using the Kubernetes cluster's CA certificate and key.
```
openssl x509 -req -in test-user.csr -CA /etc/kubernetes/pki/ca.crt -CAkey /etc/kubernetes/pki/ca.key -CAcreateserial -out test-user.crt -days 500
```
Change the ownership of the certificate file to the test-user
```
chown test-user:test-user test-user.crt
```
Now switch to test-user again
```
su test-user
```
Set the credentials for the  test-user by specifying the user's certificate and key
```
kubectl config set-credentials test-user --client-certificate=/home/test-user/cert/test-user.crt --client-key=/home/test-user/cert/test-user.key
```
Set the context for the user to ensure they are associated with the correct cluster. 
```
kubectl config set-context test-user-context --cluster kubernetes --namespace devops --user test-user
```
```
kubectl config use-context test-user-context
```
Copy Paste the "Cluster info"  from root users config  (***/root/.kube/config***)file to the ***/home/test-user/.kube/config*** file as mention below  

> - cluster:                 
>       certificate-authority-data: LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tCk1JSURCVENDQWUyZ0F3SUJBZ0lJVmh6dlZVYll2eDh3RFFZSktvWklodmNOQVFFTEJRQXdGVEVUTUJFR0ExVUUKQXhNS2EzVmlaWEp1WlhSbGN6QWVGdzB5TkRBek1UZ3dNalUzTWpaYUZ3MHpOREF6TVRZd016QXlNalphTUJVeApFekFSQmdOVkJBTVRDbXQxWW1WeWJtVjBaWE13Z2dFaU1BMEdDU3FHU0liM0RRRUJBUVVBQTRJQkR3QXdnZ0VLCkFvSUJBUUR1ZkYrRmVJWDdkR0MrZEhSeXY0czk0eld3JYeDF0VWpSWE1uZ1o2N24wVkEvQnNYNHExb29qZFZnUVgvQVFHdGFXb2ZUTkdkRgpDd2JTT2FyTm5vejMwZDBYVGNsT2IybVpDMEdUNW5POE1WczNDbW5pVGl6eTJ5MkxpdjhsYk03bFJwWjQ5ajR3CnRTU2NVSWZKS3Z6ZzBIUEwyMlhpYWtuK2FXbk9idHo0SlFHZWlEQ3dRR2JXMEYrbHBsbVEycmM0b0VHYWlxWHkKeUVNejNiejNFcyt5Ci0tLS0tRU5EIENFUlRJRklDQVRFLS0tLS0K  
>         server: https://172.31.2.53:6443 
>    name: kubernetes
>   
```
vi /home/test-user/.kube/config
```

Finally, view the kubeconfig file to ensure the changes are reflected and display current contexts
```
cat /home/test-user/.kube/config
```
```
kubectl config get-contexts
```

#### Prepare Namespace and grant access to test-user by configuring RBAC
Switch back to the root user  
Note:-Either run "exit" or "su root" (if root passswor is set) command to switch root user
```
exit
```
create a namespace named "devops", and deploy a pod within that namespace
```
kubectl create ns devops
```
```
kubectl -n devops run ng-pod --image=nginx --port=80
```
Create a role  named "test-user-role" that grants permissions to list, create, and delete pods and services within the devops namespace
```
kubectl -n devops create role dev-role --verb=get,list,create,delete --resource=pods,services
```
Now create a role binding that associates the role with the test-user
```
kubectl -n devops create rolebinding dev-rb --role=dev-role --user=test-user
```
Verify the Role and RoleBinding created in the previous step
```
kubectl -n devops get role,rolebinding
```
```
kubectl -n devops describe role
```
```
kubectl -n devops describe rolebinding
```

Switch back to the test-user, and verify that the user has access to the devops namespace by listing the pods within that namespace. Additionally, we run a command to create a new pod (pod-1) within the devops namespace to further confirm the user's permissions.

```
su test-user
```
```
kubectl get pod -n devops
```
```
kubectl -n devops run dev-pod --image=nginx --port=80 --expose
```
By following these steps, Being test-user you will be able to interact with resources within the "devops" namespace, creating and listing pods and services.

