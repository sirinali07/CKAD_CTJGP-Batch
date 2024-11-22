## Multi-container

### Task 1: Sidecar container
```
vi sidecar.yaml
```
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
```
kubectl create -f init-myservice.yaml
```
```
kubectl get pod
```
```
kubectl get svc
```
