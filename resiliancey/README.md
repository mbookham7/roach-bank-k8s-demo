# Demo 1 Cockroach Resilience under load

CockroachDB is a highly scalable and resilient relational database. In this demo you will witness how the database can survive a node failure and also a regional failure. 

Due to the wat in which CockraochDB replicates data around the nodes of the cluster we are able to delete nodes without impact.
```
kubectl get pods --context $clus1 --namespace $loc1
kubectl get pods --context $clus2 --namespace $loc2
kubectl get pods --context $clus3 --namespace $loc3
```

If the deployments have been scale to zero you will see no pods. Scale them back up for the demo.
Bank Server first.
```
kubectl scale deployment bank-server --replicas=2 -n roach-bank --context $clus1
kubectl scale deployment bank-server --replicas=2 -n roach-bank --context $clus2
kubectl scale deployment bank-server --replicas=2 -n roach-bank --context $clus3
```

Then the client. Starting with 2 replicas per region.
```
kubectl scale deployment bank-client --replicas=1 -n roach-bank --context $clus1
kubectl scale deployment bank-client --replicas=1 -n roach-bank --context $clus2
kubectl scale deployment bank-client --replicas=1 -n roach-bank --context $clus3
```

Tail the logs from one of the bank-client pods.
```
kubectl get po -n roach-bank --context $clus2
```

Grab the pod name and tail the logs.
```
kubectl logs -f --tail 10 $bank-client-pod -n roach-bank --context $clus2
```

Delete a single node or pod in k8s terms form any region.
```
kubectl delete po cockroachdb-0 -n $loc1 --context $clus1
```

Delete a single node or pod in k8s terms form any region.
```
kubectl delete po cockroachdb-0 -n $loc3 --context $clus3
```

Scale StatefulSet to zero
```
kubectl scale statefulsets cockroachdb --replicas=0 -n $loc1 --context $clus1
```

Scale back to three nodes.
```
kubectl scale statefulsets cockroachdb --replicas=3 -n $loc1 --context $clus1
```
