### Task 1: Installing Helm 3 in Kubernetes
setting up Helm on Kubernetes
Run the following command to update the packages:
```
apt update
```
Download the Helm binary for Linux 386 architecture:
```
wget https://get.helm.sh/helm-v3.11.3-linux-386.tar.gz
```
Unpack the downloaded tarball: 
```
tar -xvzf helm-v3.11.3-linux-386.tar.gz
```
Check the contents of the extracted directory to ensure that the Helm binary is present:
```
ls
```
Move the Helm executable to the /bin directory to make it accessible system-wide:
 
```
mv linux-386/helm /bin/
```
Confirm that Helm is installed correctly by checking its version:
```
helm version
```
### Task 2: Install Wordpress chart using Helm

To check the version of Helm installed
```
helm version
```
To add a Helm chart repository to your local environment. Below we are adding two Helm Chart Repositories.
```
helm repo add bitnami https://charts.bitnami.com/bitnami 
```
To list the Helm chart repositories configured in your local environment.
```
helm repo list
```
To install the WordPress application using the Bitnami Helm chart with the release name my-wordpress
```
helm install my-wordpress bitnami/wordpress
```
To list all the releases currently installed on your Kubernetes cluster
```
helm list
```


#### Verify that WordPress has been set up 

```
kubectl get all
```
Verify that the pods are running
```
kubectl get pods
```

Also Notice that the front end of wordpress are part of deployment
```
kubectl get deploy
```
View the services using the below commands. Make note of the EXTERNAL IP of the LoadBalancer service(If LoadBalancer is integrated with Cluster)
```
kubectl get svc
```
Change the type of wordpress service to ***NodePort***
```
kubectl edit svc my-wordpress
```
verify the changes
```
kubectl get svc
```
Open the browser and paste the Public Ip of the Node along with service nodePort noted on the previous step. Observe that the WordPress site is up and running
> http://<Public-IP-of-the-Node>:<NodePort>
> i.e.. http://3.85.9.5:32123

#### Cleanup
List the current helm release and delete it
```
helm ls
```

To uninstall the WordPress application deployed using Helm with the release name my-wordpress
```
helm uninstall my-wordpress
```
```
helm ls
```
 To remove a Helm repository from your local Helm client configuration
```
helm repo remove bitnami
```
To remove Helm package manager
```
sudo apt-get remove helm
```
 
