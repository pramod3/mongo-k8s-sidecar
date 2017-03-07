# Mongo Kubernetes Replica Set Sidecar

This project is as a PoC to setup a mongo replica set using Kubernetes. It should handle resizing of any type and be 
resilient to the various conditions both mongo and kubernetes can find themselves in.

## How to use it

The docker image is hosted on docker hub and can be found here:  
https://hub.docker.com/r/cvallance/mongo-k8s-sidecar/

An example kubernetes replication controller can be found in the examples directory on github here:  
https://github.com/cvallance/mongo-k8s-sidecar

There you will also find some helper scripts to test out creating the replica set and resizing it.

### Settings

- KUBE_NAMESPACE  
  Required: NO  
  The namespace to look up pods in. Not setting it will search for pods in all namespaces.
- MONGO_SIDECAR_POD_LABELS  
  Required: YES  
  This should be a be a comma separated list of key values the same as the podTemplate labels. See above for example.
- MONGO_SIDECAR_SLEEP_SECONDS  
  Required: NO  
  Default: 5  
  This is how long to sleep between work cycles.
- MONGO_SIDECAR_UNHEALTHY_SECONDS  
  Required: NO  
  Default: 15  
  This is how many seconds a replica set member has to get healthy before automatically being removed from the replica set.
- KUBERNETES_MONGO_SERVICE_NAME  
  Required: NO  
  This should point to the MongoDB Kubernetes service that identifies all the pods. It is used for setting up the DNS
  configuration when applying this to stateful sets.  

### Note about a preconfigured cluster.

In its default configuration the sidecar uses the pods' IPs for MongodDB replica names. An example follows:
```
[ { _id: 1,
   name: '10.48.0.70:27017',
   stateStr: 'PRIMARY',
   ...},
 { _id: 2,
   name: '10.48.0.72:27017',
   stateStr: 'SECONDARY',
   ...},
 { _id: 3,
   name: '10.48.0.73:27017',
   stateStr: 'SECONDARY',
   ...} ]
```
`Note` that if you have already configured replica set, having different names (than the pod IPs) could lead to duplicates
for the same pod. This is not good, since Mongo cannot deal well with duplicates and may break the whole replica set and
leave it in a non-working state! Before moving to the sidecar, make sure you use the IPs for names of the pods. Always make
sure to test it before applying it on production.

If you want to use the StatefulSets' stable network IDs, you have to make sure that you use the `KUBERNETES_MONGO_SERVICE_NAME`
environmental variable. Then the MongoDB replica set node names could look like this:
```
[ { _id: 1,
   name: 'mongo-prod-0.mongodb.mongon.svc.cluster.local:27017',
   stateStr: 'PRIMARY',
   ...},
 { _id: 2,
   name: 'mongo-prod-1.mongodb.mongon.svc.cluster.local:27017',
   stateStr: 'SECONDARY',
   ...},
 { _id: 2,
   name: 'mongo-prod-2.mongodb.mongon.svc.cluster.local:27017',
   stateStr: 'SECONDARY',
   ...} ]
```
StatefulSet name: `mongo-prod`.  
Headless service name: `mongodb`.  
Namespace: `mongons`.

Read more about the stable network ids 
<a href="https://kubernetes.io/docs/concepts/abstractions/controllers/statefulsets/#stable-network-id">here</a>.

An example of the DNS name of a pod, using those stable network IDs looks like this:
`$(statefulset name)-$(ordinal).$(service name).$(namespace).svc.cluster.local`.
The stateful set name + the ordinal form the pod name, the service name is passed via `KUBERNETES_MONGO_SERVICE_NAME`,
the namespace is extracted from the pod metadata and the rest is static.

Another thing to consider when running a cluster with the mongo-k8s-sidecar, it will prefer the stateful set stable
network ID over the pod IP. Also if you have pods already having the IP as identifier, it should not add an additional
entry for it, using the stable network ID, it should only add it for new entries in the cluster.

## Debugging

TODO: Instructions for cloning, mounting and watching

## Still to do

- Add tests!
- Add to circleCi
- Alter k8s call so that we don't have to filter in memory
