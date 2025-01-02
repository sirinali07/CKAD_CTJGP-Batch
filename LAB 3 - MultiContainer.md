## Multi-container

A **multi-container pod** in Kubernetes contains more than one container that runs within the same pod. These containers share the same network namespace and can communicate using localhost. They also share storage volumes defined at the pod level.

A **sidecar container** is used to provide supporting functionality to the main application container. It's common in scenarios where you want to offload certain responsibilities from the main container. Its an Auxiliary containers that enhance or support the main container (e.g., logging, monitoring, proxy services).

An **init container** is a special type of container that runs before the main application containers. It ensures that certain prerequisites are met before the main containers start.
  ***Examples:***
  * Ensures the main application has the required configuration file (script) before starting.
  * Ensures the database is ready before the main application starts.
  * Prepares data that the main application will use.
  * Validates that all required environment variables are set before the main container starts.

---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

### Task 1: Sidecar container
```
vi sidecar.yaml
```
Add the given content, by pressing `INSERT`
```
apiVersion: v1
kind: Pod
metadata:
  name: sidecar-pod
spec:
  containers:
  - name: main-container
    image: nginx:latest
    ports:
    - containerPort: 80
    # Main application container

  - name: sidecar-container
    image: debian:latest
    command: ['sh', '-c', 'apt update && apt install curl -y && while true; do echo "Sidecar Running"; sleep 10; done']
    # Sidecar container
```
save the file using `ESCAPE + :wq!`
```	
kubectl apply -f sidecar.yaml
```
```
kubectl get pod
```
```
kubectl exec -it sidecar-pod -c sidecar-container -- sh
```
```
kubectl exec -it sidecar-pod -c main-container -- sh
```
### Task 2: Init container
```
vi init.yaml
```
Add the given content, by pressing `INSERT`
```
apiVersion: v1
kind: Pod
metadata:
  name: init-pod
spec:
  containers:
  - name: main-container
    image: registry.access.redhat.com/ubi8/ubi:latest
    command: ['sh', '-c', 'echo The app is running! && sleep 3600']
    # Main application container

  initContainers:
  - name: init-container
    image: registry.access.redhat.com/ubi8/ubi:latest
    command: ['sh', '-c', 'until getent hosts myservice; do echo waiting for myservice; sleep 2; done;']
    # Init container


```
save the file using `ESCAPE + :wq!`
```	
kubectl apply -f init.yaml
```
```
kubectl get pod
```
```
kubectl describe pod init-pod
```
```
vi init-myservice.yaml
```
Add the given content, by pressing `INSERT`
```
kind: Service
apiVersion: v1
metadata:
 name: myservice
spec:
 ports:
 - protocol: TCP
   port: 80
   targetPort: 9376
```
save the file using `ESCAPE + :wq!`
```
kubectl create -f init-myservice.yaml
```
```
kubectl get pod
```
```
kubectl get svc
```

### Task 3: Cleanup the resources using below command
```
kubectl delete -f sidecar.yaml
```
```
kubectl delete -f init.yaml
```
```
kubectl delete -f init-myservice.yaml
```
