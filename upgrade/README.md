# Perform a Rolling Upgrade

This is done by updating the image tag in the statefulSet manifest.

```
      containers:
      - name: cockroachdb
        image: cockroachdb/cockroach:v23.1.3
        imagePullPolicy: IfNotPresent
```

Each manifest needs to be applied in turn, a region at a time to ensure there is no disruption to service.

Apply region one and observe the changes in the CockroachDB UI
``````
kubectl -n $loc1 apply -f ./upgrade/manifest/uksouth-cockroachdb-statefulset-secure.yaml --context $clus1
```

Wait for the rollout to complete before moving on to next region.
```
kubectl -n $loc2 apply -f ./upgrade/manifest/ukwest-cockroachdb-statefulset-secure.yaml --context $clus2
```

Again wait for the rollout to finish and then finish with the final region
```
kubectl -n $loc3 apply -f ./upgrade/manifest/northeurope-cockroachdb-statefulset-secure.yaml --context $clus3
```