#Section 3
## 1 - Manual Scheduling

How do we schedule pod manually?

Every pod has a `nodeName` field. Generally added later by kubernetes  
The scheduler goes through the pods and checks for the ones which don't have that property  

So, without a scheduler, the easiest way is to specify the `nodeName` property  
* `nodeName` can only be set at creation
* Can create a `Binding` object otherwise, to bind a pod to a node

## 2 - Labels and Selectors
### labels
* `labels` are just strings added to the kubernetes items
* `selectors` can be used to filter objects based on labels

To filter the pods :  
* `kubectl get pods -l=function=backend`
* `kubectl get pods --selector function=backend`  

Internally, kubernetes uses label to link objects together  
For example, when creating a ReplicaSet : 
* `labels` under metadata are labels for the ReplicaSet object
* `labels` under template section are the labels of the pods
* `selector -> matchLabels` are then used to by the ReplicaSet to discover the pods

###Annotations
Cannot be used for selecting
Used for a lot of stuff, for example prometheus auto-discovery of scraping endpoints

See [pod-label](pod-label-definition.yaml)

## 3 - Taints and Tolerations

`Taint` are labels-like added to the node  
By default, none of the pods can go on a tainted node. You need to add `tolerations` to make sure the node is populated

Command:
* `kubectl taint nodes node-name key=value:taint-effect`
* `taint-effect`
    * `NoSchedule` -> Pods are not scheduled on the node
    * `PreferNoSchedule`-> Try to avoid scheduling pod, but not guaranteed
    * `NoExecute` -> New pods are not scheduled on the node, and existing pods will be evicted
    
Example
* `kubectl taint nodes node1 app=blue:NoSchedule`

Can also be added in yaml files  

!!!Taint only restrict which pods can be run on which node. It doesn't force the scheduling of certain pods on certain nodes!!!

Special case of the master node  
No pods are scheduled on master node by default  
The master node has a special taint which prevents pods to be scheduled here  

See [pod-tolerations](pod-tolerations-definition.yaml)

## 4 - Nodes Selectors

Assuming multiples nodes with different sizes  
See [pod-nodeselector](pod-nodeselector-definition.yaml)

Can be added as a section in a yaml file  
The node selectors sections specify a label which has to match the labels on the nodes

To label nodes: 
* `kubectl label nodes node-name key=value`
* `kubectl label nodes node1 size=large`  

Nice, but only match single label  
How do we do
* Pod on medium or large node?
* Pod not on small node?

## 5 - Node Affinity

The goal is to ensure some pods are scheduled on some nodes

See [pod-nodeaffinity.yaml](pod-nodeaffinity-definition.yaml). Lots of options  

See `NodeAffinityTypes` for the possible behaviors  
* `DuringScheduling` -> Considered when placing the pods on the node
    * `required` -> Has to be placed on node matching the selectors. If no matching nodes, pod is not scheduled
    * `preferred` -> If no matching nodes, pod will still be scheduled on any available nodes
* `DuringExecution` -> Considered after pods are running, when we make changes to the nodes
    * `Ignored` -> Changing the nodes labels doesn't evict currently running pods