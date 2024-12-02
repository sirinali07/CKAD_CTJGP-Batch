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


### Task 5: Pod Scheduling using Taints and Tolerations

Node affinity is a property of Pods that attracts them to a set of nodes (either as a preference or a hard requirement). Taints are the opposite -- they allow a node to repel a set of pods.

Tolerations are applied to pods. Tolerations allow the scheduler to schedule pods with matching taints. Tolerations allow scheduling but don't guarantee scheduling: the scheduler also evaluates other parameters as part of its function.

Taints and tolerations work together to ensure that pods are not scheduled onto inappropriate nodes. One or more taints are applied to a node; this marks that the node should not accept any pods that do not tolerate the taints.

Deploy some pods and check on which nodes they are getting scheduled 
```
kubectl run pod-1 --image nginx
```
```
kubectl get po -o wide
```
Label the node
```
kubectl label nodes node1 disktype=ssd
```
Deploy a pod with the same Node Selector
```
vi pod.yaml
```
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod1
  labels:
    env: test
spec:
  containers:
  - name: pod1-crt
    image: nginx
  nodeSelector:
    disktype: ssd

```
Apply the yaml file
```
kubectl apply -f pod.yaml
```
Check on which Node it gets scheduled.
```
kubectl get pods -o wide
```
Taint Node1 to NoSchedule
```
kubectl taint nodes node1 key1=value1:NoSchedule
```
Replace pod1
```
kubectl replace -f pod.yaml --force
```
The Pod goes in a Pending State, since it has NodeSelector mapping it to  Node1 but the Node1 is tainted to NoSchedule

Add toleration to the same pod and replace
```
vi pod.yaml
```
```
tolerations:
- key: "key1"
  operator: "Equal"
  value: "value1"
  effect: "NoSchedule"
```
```
kubectl replace -f pod.yaml --force
```
Check on which node it gets scheduled
```
kubectl get po -o wide
```
Remove the NodeSelector and replace the Pod
```
kubectl replace -f pod.yaml --force
```
Check where the Pod has scheduled now....
```
kubectl get po -o wide
```
Taint the node on which this pod gets deployed with `NoExecute`.
```
kubectl taint nodes <node-name> key1=value1:NoExecute
```
```
kubectl get po -o wide
```
The Pod gets evicted.

Remove the taint NoExecute
```
kubectl taint nodes node1 key1=value1:NoExecute-
```
Apply the taint PreferNoSchedule to both node1 and node2
```
kubectl taint nodes node1 key1=value1:PreferNoSchedule
```
```
kubectl taint nodes node2 key1=value1:PreferNoSchedule
```
Trying deploying another pod without any toleration.

Notice it gets scheduled on Node2. reason being Node1 has NoSchedule Taint still enabled.
```
kubectl run pod-test --image nignx
```

CLEANUP
Check the taints on all the ndoes and untaint the ones that you have applied.
