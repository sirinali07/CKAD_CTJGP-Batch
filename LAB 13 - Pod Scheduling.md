## Pod Scheduling

### Task 1: Pod Scheduling using Node labels

Node Labels: Like many other Kubernetes objects, nodes have labels. You can attach labels manually. Kubernetes also populates a standard set of labels on all nodes in a cluster.

Node Selector: Node Selector is the simplest recommended form of node selection constraint. You can add the `nodeSelector` field to your Pod specification and specify the node labels you want the target node to have. Kubernetes only schedules the Pod onto nodes that have each of the labels you specify.

List node labels
```
kubectl get nodes --show-labels
```
Label the node
```
kubectl label nodes node1 disktype=ssd1
```
List your nodes to check labels 
```
kubectl get nodes --show-labels | grep "disktype=ssd1"
```
```
vi nlns-pod.yaml
```
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nlns-nginx-pod
  labels:
    env: test
spec:
  containers:
  - name: nlns-nginx-ctr
    image: nginx
  nodeSelector:
    disktype: ssd

```
```
kubectl apply -f nlns-pod.yaml
```
```
kubectl get pods -o wide
```
The Pod goes into the pending state as the node selector does not match.

Either change the Label on the Node or the Node Selector specifications in the pod.

Changing the Label on the node. For this Unlabel the Node and mark it with the correct label
```
kubectl label nodes node1 disktype-
```
```
kubectl label nodes node1 disktype=ssd
```
List your nodes to check labels 
```
kubectl get nodes --show-labels | grep "disktype=ssd"
```
List the pods. You will see the the Pod has gone into the running state.
```
kubectl get po -o wide
```
Before signing off, remove the label
```
kubectl label nodes node1 disk type-
```


### Task 2: Pod Scheduling using Node Name / Host Name

Node Name is a more direct form of node selection than affinity or nodeSelector. nodeName is a field in the Pod spec. If the nodeName field is not empty, the scheduler ignores the Pod and the kubelet on the named node tries to place the Pod on that node. Using nodeName overrules using nodeSelector or affinity and anti-affinity rules.

Some of the limitations of using nodeName to select nodes are:

* If the named node does not exist, the Pod will not run, and in some cases may be automatically deleted.
* If the named node does not have the resources to accommodate the Pod, the Pod will fail and its reason will indicate why, for example OutOfmemory or OutOfcpu.
* Node names in cloud environments are not always predictable or stable.

```
vi node-name.yaml
```
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-nodename
  labels:
    env: test
spec:
  containers:
  - name: nginx
    image: nginx
  nodeName: {node-name / hostname }
```
```
kubectl apply -f node-name.yaml
```
Check your pod is running on the targeted node
```
kubectl get pods -o wide 
```

### Task 3: Pod Scheduling using Node Affinity and Node Anti Affinity
#### Node Affinity
Node affinity is conceptually similar to nodeSelector, allowing you to constrain which nodes your Pod can be scheduled on based on node label

List the nodes in your cluster, along with their labels:
```
kubectl get nodes --show-labels
```
Schedule a Pod using required node affinity.

This manifest describes a Pod that has a requiredDuringSchedulingIgnoredDuringExecution node affinity,disktype: ssd. This means that the pod will get scheduled only on a node that has a disktype=ssd label.

```
vi pod-nginx-required-affinity.yaml
```
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx
spec:
  affinity:
    nodeAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
        - matchExpressions:
          - key: disktype
            operator: In
            values:
            - ssd            
  containers:
  - name: nginx
    image: nginx
    imagePullPolicy: IfNotPresent
```
Apply the manifest to create a Pod that is scheduled onto your chosen node
```
kubectl apply -f pod-nginx-required-affinity.yaml
```
Check if the pod is running. If not describe the pod
```
kubectl get po -o wide
```
Choose one of your nodes, and add a label to it. Where <your-node-name> is the name of your chosen node.
```
kubectl label nodes node1 disktype=ssd
```
Verify that your chosen node has a disktype=ssd label
```
kubectl get nodes --show-labels
```
Verify that the pod is running on your chosen node:
```
kubectl get pods --output=wide
```
Remove the label on the node
```
kubectl label nodes node1 disktype-
```
Check if this effects the Pod status.
```
kubectl get po -o wide
```

This manifest describes a Pod that has a preferredDuringSchedulingIgnoredDuringExecution node affinity,disktype: ssd. This means that the pod will prefer a node that has a disktype=ssd label.
```
vi pod-nginx-preferred-affinity.yaml
```
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx1
spec:
  affinity:
    nodeAffinity:
      preferredDuringSchedulingIgnoredDuringExecution:
      - weight: 1
        preference:
          matchExpressions:
          - key: disktype
            operator: In
            values:
            - ssd          
  containers:
  - name: nginx
    image: nginx
    imagePullPolicy: IfNotPresent
```

Apply the manifest to create a Pod that is scheduled onto your chosen node:
```
kubectl apply -f pod-nginx-preferred-affinity.yaml
```
Verify that the pod is running on your chosen node:
```
kubectl get pods --output=wide
```
#### Node Anti Affinity
Create a Pod with Node Anti Affinity
```
vi na-nginx.yaml
```
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: na-nginx
spec:
  affinity:
    nodeAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
        - matchExpressions:
          - key: disktype
            operator: NotIn   #In = NodeAffinity  #NotIn = NodeAntiAffinity
            values:
            - hdd    
  containers:
  - name: nginx
    image: nginx
```
Apply the manifest to create a Pod that shouldn't scheduled onto your chosen node

```
kubectl apply -f  na-nginx.yaml
 ```
Check if this effects the Pod status
 ```
kubectl get pod -o wide
 ```

### Task 4: Pod Scheduling using Pod Affinity

```
vi depend-pod.yaml
```
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pa-nginx-pod1
  labels:
    app: pa-nginx-app
spec:
  containers:
  - name: pa-nginx-ctr1
    image: nginx
```
```
kubectl apply -f depend-pod.yaml
```
```
kubectl get pods -o wide

```

Now create pod with Pod Affinity
```
vi pod-affinity.yaml
```
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pa-nginx-pod2
spec:
  containers:
  - name: pa-nginx-ctr2
    image: nginx
  affinity:
    podAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
      - labelSelector:
          matchExpressions:
          - key: app
            operator: In
            values:
            - pa-nginx-app
        topologyKey: kubernetes.io/hostname
```
```
kubectl apply -f pod-affinity.yaml
```
```
kubectl get pods -o wide
```
Notice both the Pods are in the same Node

### Task 5: Pod Anti Affinity

Now create a pod anti affinity
```
vi pod-antiaff.yaml
```
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx
spec:
  selector:
    matchLabels:
      app: nginx
  replicas: 3
  template:
    metadata:
      labels:
        app: nginx
    spec:
      affinity:
        podAntiAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
          - labelSelector:
              matchExpressions:
              - key: app
                operator: In
                values:
                - pa-nginx-app
            topologyKey: "kubernetes.io/hostname"
      containers:
      - name: nginx
        image: nginx
```
```
kubectl apply -f  pod-antiaff.yaml
```
```
kubectl get pod -o wide 
```
### Task 6: CleanUp Resources

Delete all the resources created on the above tasks.
