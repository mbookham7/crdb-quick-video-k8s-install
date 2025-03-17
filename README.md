# 🚀 CockroachDB - Quick Single Region Installation on Kubernetes

This guide provides a quick and streamlined way to install **CockroachDB** on **Kubernetes** in a single region.  

## 📌 Prerequisites

Ensure you have the following before proceeding:

- **Kubernetes Cluster** (Minikube, Kind, GKE, AKS, etc.)
- **kubectl** installed and configured
- **git** installed
- **curl** installed

---

## 🛠 Step 1 - Download the CockroachDB Binary

First, download the CockroachDB binary and move it to the appropriate directory.

```sh
git clone https://github.com/mbookham7/crdb-quick-video-k8s-install.git
cd crdb-quick-video-k8s-install

curl -O https://binaries.cockroachdb.com/cockroach-v25.1.0.darwin-11.0-arm64.tgz
tar -xzvf cockroach-v25.1.0.darwin-11.0-arm64.tgz

sudo mv cockroach-v25.1.0.darwin-11.0-arm64/cockroach /usr/local/bin/
cockroach version
```

---

## 🔐 Step 2 - Generate Certificates and Upload as Kubernetes Secrets

CockroachDB requires secure certificates for authentication.  

### 📂 Create Folder Structure

```sh
mkdir certs
mkdir my-safe-directory
```

### 🏷 Generate CA and Client Certificates

Use the `cockroach` binary to generate **CA (Certificate Authority)** and **client certificates**.

```sh
cockroach cert create-ca --certs-dir=certs --ca-key=my-safe-directory/ca.key
cockroach cert create-client root --certs-dir=certs --ca-key=my-safe-directory/ca.key
```

Upload the **client certificate** as a Kubernetes secret.

```sh
kubectl create secret generic cockroachdb.client.root --from-file=certs
```

### 🔑 Generate Node Certificates  

Each node requires a **node certificate** with all **trusted Subject Alternative Names (SANs)**.

```sh
cockroach cert create-node --certs-dir=certs --ca-key=my-safe-directory/ca.key     localhost 127.0.0.1     'cockroachdb-public' 'cockroachdb-public.default'     'cockroachdb-public.default.svc.cluster.local'     '*.cockroachdb' '*.cockroachdb.default' '*.cockroachdb.default.svc.cluster.local'
```

Upload the **node certificate** as a Kubernetes secret.

```sh
kubectl create secret generic cockroachdb.node --from-file=certs
```

---

## 🚀 Step 3 - Deploy CockroachDB on Kubernetes

Deploy the **StatefulSet** YAML file and initialize the cluster.

```sh
kubectl create -f cockroachdb-statefulset.yaml
```

### 🔍 Check Pod Status  

Ensure that all `cockroachdb` pods are running and ready.

```sh
kubectl get pods
```

### 🏗 Initialize the Cluster

Once the pods are ready, **exec** into one of the CockroachDB pods and initialize the cluster.

```sh
kubectl exec -it cockroachdb-0 -- /cockroach/cockroach init --certs-dir=/cockroach/cockroach-certs
```

---

## 👤 Step 4 - Create a Database User  

Create a **secure client pod** to connect to the database.

```sh
kubectl create -f client-secure.yaml
```

### 💻 Connect to CockroachDB SQL Client

```sh
kubectl exec -it cockroachdb-client-secure -- ./cockroach sql     --certs-dir=/cockroach-certs --host=cockroachdb-public
```

### 🏗 Create a New Database User

Inside the **SQL client**, create a new user and grant them **admin privileges**.

```sql
CREATE USER craig WITH PASSWORD 'cockroach';
GRANT admin TO craig;
```

Exit the SQL shell.

```sql
\q
```

---

## 🌍 Step 5 - Access the CockroachDB Console  

To access the **DBConsole**, use `kubectl port-forward`:

```sh
kubectl port-forward service/cockroachdb-public 8080
```

Then open [http://localhost:8080](http://localhost:8080) in your browser.

---

## 🎯 Conclusion  

You have successfully installed **CockroachDB** on Kubernetes in a single region! 🎉  
From here, you can:  

✅ **Start creating databases and tables**  
✅ **Monitor cluster health using the DBConsole**  
✅ **Scale and configure CockroachDB as needed**  

Happy coding! 🚀
