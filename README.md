# CockroachDB - Quick Single Region install on Kubernetes

## Step 1 - Download the Cockroach Binary

Download the binary and move it to the correct directory.
```sh
git clone https://github.com/mbookham7/crdb-quick-video-k8s-install.git
cd crdb-quick-video-k8s-install
curl https://binaries.cockroachdb.com/cockroach-v25.1.0.darwin-11.0-arm64.tgz | tar -xz
sudo mv cockroach-v25.1.0.darwin-11.0-arm64/cockroach /usr/local/bin/
cockroach version
```
## Step 2 - Create certificates and upload them as Kubernetes secrets

Create the folder structure
```sh
mkdir certs
mkdir my-safe-directory
```

Use the `cockroach` binary to generate certificates and upload them as Kubernetes secrets.
```sh
cockroach cert create-ca --certs-dir=certs --ca-key=my-safe-directory/ca.key
cockroach cert create-client root --certs-dir=certs --ca-key=my-safe-directory/ca.key
kubectl create secret generic cockroachdb.client.root --from-file=certs
cockroach cert create-node --certs-dir=certs --ca-key=my-safe-directory/ca.key localhost 127.0.0.1 'cockroachdb-public' 'cockroachdb-public.default' 'cockroachdb-public.default.svc.cluster.local' '*.cockroachdb' '*.cockroachdb.default' '*.cockroachdb.default.svc.cluster.local'
kubectl create secret generic cockroachdb.node --from-file=certs
```

## Step 3 - Deploy CockroachDB

Deploy the stateful YAML file and initialize the cluster.
```sh
kubectl create -f cockroachdb-statefulset.yaml
```

Check the status of the `cockroachdb` pods, make sure that all the container images are pulled and the cluster is ready to be initialised.
```sh
kubectl get pods
```

Now exec into one of the `cockroachdb` pods and initialise the cluster.
```
kubectl exec -it cockroachdb-0 -- /cockroach/cockroach init --certs-dir=/cockroach/cockroach-certs
```

## Step 4 - Create a Database User

Create a secure client pod to connect to the database and create a user.
```sh
kubectl create -f client-secure.yaml
```

Exec into the pod using `sql`.
```sh
kubectl exec -it cockroachdb-client-secure -- ./cockroach sql --certs-dir=/cockroach-certs --host=cockroachdb-public
```

Use the SQL client to create a use and grant them the admin role.
```sh
CREATE USER craig WITH PASSWORD 'cockroach';
GRANT admin TO craig;
```

```sh
\q
```

```sh
kubectl port-forward service/cockroachdb-public 8080
```
