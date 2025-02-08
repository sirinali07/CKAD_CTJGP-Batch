## Security Context

### Task 1: Set the security context for a Pod

To specify security settings for a Pod, include the securityContext field in the Pod specification. 

The security settings that you specify for a Pod apply to all Containers in the Pod. Here is a configuration file for a Pod that has a securityContext and an emptyDir volume:

```
vi security-context1.yaml
```
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: security-context-pod1
spec:
  securityContext:
    runAsUser: 1000
    runAsGroup: 3000
    fsGroup: 2000
  volumes:
  - name: sec-ctx-vol
    emptyDir: {}
  containers:
  - name: sec-ctx-pod1-ctr
    image: busybox:1.28
    command: [ "sh", "-c", "sleep 1h" ]
    volumeMounts:
    - name: sec-ctx-vol
      mountPath: /data/demo
   
```
Create the Pod:
```
kubectl apply -f security-context1.yaml
```
Verify that the Pod's Container is running:
```
kubectl get pod security-context-pod1
```
Get a shell to the running Container:
```
kubectl exec -it security-context-pod1 -- sh
```
In your shell, list the running processes:
```
ps
```
The output shows that the processes are running as user 1000, which is the value of runAsUser:
```
id
```
The Output shows the UID, Group ID and all the groups

In your shell, navigate to /data, and list the one directory:
```
cd /data && ls -l
```
The output shows that the /data/demo directory has group ID 2000, which is the value of fsGroup.

In your shell, navigate to /data/demo, and create a file:
```
cd demo && echo hello > testfile
```
List the file in the /data/demo directory:
```
ls -l
```
The output shows that testfile has group ID 2000, which is the value of fsGroup.

Run the following command:
```
id
```

From the output, you can see that gid is 3000 which is same as the runAsGroup field. If the runAsGroup was omitted, the gid would remain as 0 (root) and the process will be able to interact with files that are owned by the root(0) group and groups that have the required group permissions for the root (0) group.
```
exit
```
### Task 2: Set the security context for a container

To specify security settings for a Container, include the securityContext field in the Container manifest. 

Security settings that you specify for a Container apply only to the individual Container, and they override settings made at the Pod level when there is overlap. Container settings do not affect the Pod's Volumes.

Below is the configuration file for a Pod that has one Container. Both the Pod and the Container have a securityContext field:
```
vi security-context-2.yaml
```
```yaml

apiVersion: v1
kind: Pod
metadata:
  name: security-context-pod2
spec:
  securityContext:
    runAsUser: 1000
  containers:
  - name: sec-ctx-pod2
    image: gcr.io/google-samples/node-hello:1.0
    securityContext:
      runAsUser: 2000
      allowPrivilegeEscalation: false
```
Create the Pod:
```
kubectl apply -f security-context-2.yaml
```
Verify that the Pod's Container is running:
```
kubectl get pod security-context-pod2
```
Get a shell into the running Container:
```
kubectl exec -it security-context-pod2 -- sh
```
In your shell, list the running processes:
```
ps aux
```

The output shows that the processes are running as user 2000. This is the value of runAsUser specified for the Container. It overrides the value 1000 that is specified for the Pod.

## TAsk 3 : Pod & Container Level Security Context
```
vi security-context-new.yaml
```
```
apiVersion: v1
kind: Pod
metadata:
  name: security-context-new
spec:
  securityContext:
    fsGroup: 2000     # common group of files
  volumes:
  - name: sec-ctx-vol
    emptyDir: {}
  containers:
  - name: ctr-1
    image: busybox:1.28
    command: [ "sh", "-c", "sleep 1h" ]
    volumeMounts:
    - name: sec-ctx-vol
      mountPath: /data/demo
    securityContext:
      runAsUser: 1010   #UID
      runAsGroup: 3010  #GID
  - name: ctr-2
    image: busybox:1.28
    command: [ "sh", "-c", "sleep 1h" ]
    volumeMounts:
    - name: sec-ctx-vol
      mountPath: /docs/demo
    securityContext:
      runAsUser: 1020   #UID
      runAsGroup: 3020  #GID
```
Create the Pod:
```
kubectl apply -f security-context-new.yaml
```
Verify that the Pod's Containers are running:
```
kubectl get po
```
Get a shell into the ctr-1 Container:
```
kubectl exec -it security-context-new -c ctr-1 -- sh
```
```
id
```
```
cd /data/demo/
```
```
echo hello > file1.txt
```
```
ls -l
```
```
exit
```
Get a shell into the ctr-2 Container:
```
kubectl exec -it security-context-new -c ctr-2 -- sh
```
```
id
```
```
cd /docs/demo/
```
```
echo hello > file2.txt
```
```
ls -l
```
```
exit
```


