# Demo 2 - CockroachDB Scalability under load

The purpose of this demonstration is to show how CockroachDB is able to scale to meet the needs of an increased load without any interruption to the workload that is running. We will use Roach Bank to increase the stress on  the CockroachDB cluster. Once it is under stress we will increase the number of CockroachDB node and observer the affect on the cluster.

Check the pods are running.
```
kubectl get po -n roach-bank --context $clus1
kubectl get po -n roach-bank --context $clus2
kubectl get po -n roach-bank --context $clus3
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

Increase the number of replicas for each of the StatefulSet in each of the regions.
```
kubectl scale statefulsets cockroachdb --replicas=6 -n $loc1 --context $clus1
kubectl scale statefulsets cockroachdb --replicas=6 -n $loc2 --context $clus2
kubectl scale statefulsets cockroachdb --replicas=6 -n $loc3 --context $clus3
```

It will now take a few mins for the new nodes to be added to the cluster and participate in the workload.
Check in the UI the load on the CockroachDB Cluster. What do you notice?

[CockroachDB UI Cluster Hardware Metrics](https://uksouth.mikebookham.co.uk:8080/#/metrics/hardware/cluster)

Scale Bank Client to increase the load on the Database.
```
kubectl scale deployment bank-client --replicas=4 -n roach-bank --context $clus1
kubectl scale deployment bank-client --replicas=4 -n roach-bank --context $clus2
kubectl scale deployment bank-client --replicas=4 -n roach-bank --context $clus3
```

Once you scale the bank-client replicas you will see an distinct increase in the number of QPS the cluster is now able to support.


Return to starting config. Remove the nodes one at a time, starting with the first cluster.
```
kubectl scale statefulsets cockroachdb --replicas=5 -n $loc1 --context $clus1
```
Wait until the cluster to have 0 under replicated ranges.


Remove the next node, and wait until the cluster to have 0 under replicated ranges.
```
kubectl scale statefulsets cockroachdb --replicas=4 -n $loc1 --context $clus1
```

Remove the next node, and wait until the cluster to have 0 under replicated ranges.
```
kubectl scale statefulsets cockroachdb --replicas=3 -n $loc1 --context $clus1
```

Delete nodes from the second region once the cluster has recovered. One by one.
```
kubectl scale statefulsets cockroachdb --replicas=5 -n $loc2 --context $clus2
```
```
kubectl scale statefulsets cockroachdb --replicas=4 -n $loc2 --context $clus2
```
```
kubectl scale statefulsets cockroachdb --replicas=3 -n $loc2 --context $clus2
```

Delete nodes from the third region once the cluster has recovered. Again, one by one.
```
kubectl scale statefulsets cockroachdb --replicas=5 -n $loc3 --context $clus3
```

```
kubectl scale statefulsets cockroachdb --replicas=4 -n $loc3 --context $clus3
```

```
kubectl scale statefulsets cockroachdb --replicas=3 -n $loc3 --context $clus3
```

Delete the PVC for the removed nodes.
```
kubectl delete pvc datadir-cockroachdb-3 datadir-cockroachdb-4 datadir-cockroachdb-5 -n $loc1 --context $clus1
kubectl delete pvc datadir-cockroachdb-3 datadir-cockroachdb-4 datadir-cockroachdb-5 -n $loc2 --context $clus2
kubectl delete pvc datadir-cockroachdb-3 datadir-cockroachdb-4 datadir-cockroachdb-5 -n $loc3 --context $clus3
```

Remove the the dead nodes. CHECK NODE NUMBERS IN THE UI!!!
```
kubectl exec -it cockroachdb-client-secure -n $loc1 --context $clus1 -- ./cockroach node decommission <node numbers> --certs-dir=/cockroach-certs --host=cockroachdb-public
```
