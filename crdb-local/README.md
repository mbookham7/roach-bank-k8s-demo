## Deploy CockroachDB in a local Kubernetes cluster (if required)

If you need to run and test Roach Bank locally on you laptop you can also deploy CockroachDB into a local version of Kubernetes if required. If you follow these step by step instructions then you will be able to run CockroachDB along side Roach Bank in a local Kubernetes cluster like minikube, Docker Desktop or Rancher Desktop.

When deploying COckroachDB in Kubernetes we like to deploy the pods into a namespace named after the region that it is running in. This makes multi regional setups easy to understand and configure. As a result of this the first step is to set an environment variable to the name of your region.

Create environment variable
```
export loc1="uksouth"
```

Create the namespace inside Kubernetes for CockroachDB.
```
kubectl create namespace $loc1
```

All communication in CockroachDB is secured with certificates. The CockroachDB binary contains its own CA which can be used to generate all the required certificates. In the following steps the required certificates to secure CockroachDB are created.

First create the required folder structure to store the certificates.
```
mkdir certs my-safe-directory
```

With the Cockroach binary create the certificate authority.
```
cockroach cert create-ca \
--certs-dir=certs \
--ca-key=my-safe-directory/ca.key
```

Create the client certificate.
```
cockroach cert create-client \
root \
--certs-dir=certs \
--ca-key=my-safe-directory/ca.key
```

Add these certificates as kubernetes secrets into the required namespaces.
```
kubectl create secret \
generic cockroachdb.client.root \
--from-file=certs \
--namespace $loc1
```

Create the node certificates for each region. In this scenario there is just the one region.

```
cockroach cert create-node \
localhost 127.0.0.1 \
cockroachdb-public \
cockroachdb-public.$loc1 \
cockroachdb-public.$loc1.svc.cluster.local \
"*.cockroachdb" \
"*.cockroachdb.$loc1" \
"*.cockroachdb.$loc1.svc.cluster.local" \
--certs-dir=certs \
--ca-key=my-safe-directory/ca.key
```

Upload the secret.
```
kubectl create secret \
generic cockroachdb.node \
--from-file=certs \
--namespace $loc1
```

Deploy the CockroachDB kubernetes resources form the file provided
> There are some hard codes region names in these files. If you have changed the region names you will need to edit these files. (In this lab the region names used are uksouth, ukwest or northeurope). You may also want to adjust the replica count and resource requests and limits depending on your computer spec.
```
kubectl apply -f kubernetes/cockroachdb-statefulset-secure.yaml -n $loc1
```

Once the pods are deployed we need to initialize the cluster. This is done by getting a shell into the container and running the `cockroach init` command.
```
kubectl exec \
--namespace $loc1 \
-it cockroachdb-0 \
-- /cockroach/cockroach init \
--certs-dir=/cockroach/cockroach-certs
```

Check that all the pods have started successfully.
```
kubectl get pods --namespace $loc1
```

Next, create a secure client in the first region.
```
kubectl create -f kubernetes/client-secure.yaml --namespace $loc1
```

Now exec into the pod using the CockraochDB SQL client.
```
kubectl exec -it cockroachdb-client-secure -n $loc1 -- ./cockroach sql --certs-dir=/cockroach-certs --host=cockroachdb-public
```
Create a user, make that user an admin and create the `roach_bank` database.
```
CREATE USER craig WITH PASSWORD 'cockroach';
GRANT admin TO craig;
CREATE database roach_bank;
\q
```
You now have a CockroachDB cluster running to locally to test out new versions of Roach Bank as required. Port forward port `8080` to localhost so you are able to access the CockroachDB Admin UI from the browser.
```
kubectl port-forward cockroachdb-0 8080 -n $loc1
```

Clean up

```
rm -R certs my-safe-directory
```
