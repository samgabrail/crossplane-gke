# Overview
Crossplane to create a GKE cluster

## Instructions

### Install the Up command-line
Download and install the Upbound up command-line.
```bash
curl -sL "https://cli.upbound.io" | sh
mv up /usr/local/bin/
```

Verify the version of up with up --version
```bash
$ up --version
v0.19.1
```

### Install Universal Crossplane
Install Upbound Universal Crossplane with the Up command-line.

```bash
$ up uxp install
UXP 1.13.2-up.2 installed
```
Verify the UXP pods are running
```bash
$ kubectl get pods -n upbound-system
NAME                                       READY   STATUS    RESTARTS   AGE
crossplane-77ff754998-k76zz                1/1     Running   0          40s
crossplane-rbac-manager-79b8bdd6d8-79577   1/1     Running   0          40s
```

### Install the official GCP Container Provider

```bash
kubectl apply -f gke_provider.yaml
```

After installing the provider, verify the install with kubectl get providers.
```bash
$ kubectl get providers
NAME                          INSTALLED   HEALTHY   PACKAGE                                                  AGE
provider-gcp-container        True        True      xpkg.upbound.io/upbound/provider-gcp-container:v0.41.0   47h
provider-gcp-storage          True        True      xpkg.upbound.io/upbound/provider-gcp-storage:v0.41.0     47h
upbound-provider-family-gcp   True        True      xpkg.upbound.io/upbound/provider-family-gcp:v0.41.0      47h
```

### Create a Kubernetes secret
The provider-gcp-container requires credentials to create and manage GCP resources.

Generate a GCP JSON key file
Create a JSON key file containing the GCP account credentials. GCP provides documentation on how to create a key file.

Here is an example key file:
```json
{
  "type": "service_account",
  "project_id": "caramel-goat-354919",
  "private_key_id": "e97e40a4a27661f12345678f4bd92139324dbf46",
  "private_key": "-----BEGIN PRIVATE KEY-----\nMIIEvwIBADANBgkqhkiG9w0BAQEFAASCBKkwggSlAgEAAoIBAQCwA+6MWRhmcPB3\nF/irb5MDPYAT6BWr7Vu/16U8FbCHk7xtsAWYjKXKHu5mGzum4F781sM0aMCeitlv\n+jr2y7Ny23S9uP5W2kfnD/lfj0EjCdfoaN3m7W0j4DrriJviV6ESeSdb0Ehg+iEW\ngNrkb/ljigYgsSLMuemby5lvJVINUazXJtGUEZew+iAOnI4/j/IrDXPCYVNo5z+b\neiMsDYWfccenWGOQf1hkbVWyKqzsInxu8NQef3tNhoUXNOn+/kgarOA5VTYvFUPr\n2l1P9TxzrcYuL8XK++HjVj5mcNaWXNN+jnFpxjMIJOiDJOZoAo0X7tuCJFXtAZbH\n9P61GjhbAgMBAAECggEARXo31kRw4jbgZFIdASa4hAXpoXHx4/x8Q9yOR4pUNR/2\nt+FMRCv4YTEWb01+nV9hfzISuYRDzBEIxS+jyLkda0/+48i69HOTAD0I9VRppLgE\ne97e40a4a27661f12345678f4bd92139324dbf46+2H7ulQDtbEgfcWpNMQcL2JiFq+WS\neh3H0gHSFFIWGnAM/xofrlhGsN64palZmbt2YiKXcHPT+WgLbD45mT5j9oMYxBJf\nPkUUX5QibSSBQyvNqCgRKHSnsY9yAkoNTbPnEV0clQ4FmSccogyS9uPEocQDefuY\nY7gpwSzjXpaw7tP5scK3NtWmmssi+dwDadfLrKF7oQKBgQDjIZ+jwAggCp7AYB/S\n6dznl5/G28Mw6CIM6kPgFnJ8P/C/Yi2y/OPKFKhMs2ecQI8lJfcvvpU/z+kZizcG\nr/7iRMR/SX8n1eqS8XfWKeBzIdwQmiKyRg2AKelGKljuVtI8sXKv9t6cm8RkWKuZ\n9uVroTCPWGpIrh2EMxLeOrlm0QKBgQDGYxoBvl5GfrOzjhYOa5GBgGYYPdE7kNny\nhpHE9CrPZFIcb5nGMlBCOfV+bqA9ALCXKFCr0eHhTjk9HjHfloxuxDmz34vC0xXG\ncegqfV9GNKZPDctysAlCWW/dMYw4+tzAgoG9Qm13Iyfi2Ikll7vfeMX7fH1cnJs0\nnYpN9LYPawKBgQCwMi09QoMLGDH+2pLVc0ZDAoSYJ3NMRUfk7Paqp784VAHW9bqt\n1zB+W3gTyDjgJdTl5IXVK+tsDUWu4yhUr8LylJY6iDF0HaZTR67HHMVZizLETk4M\nLfvbKKgmHkPO4NtG6gEmMESRCOVZUtAMKFPhIrIhAV2x9CBBpb1FWBjrgQKBgQCj\nkP3WRjDQipJ7DkEdLo9PaJ/EiOND60/m6BCzhGTvjVUt4M22XbFSiRrhXTB8W189\noZ2xrGBCNQ54V7bjE+tBQEQbC8rdnNAtR6kVrzyoU6xzLXp6Wq2nqLnUc4+bQypT\nBscVVfmO6stt+v5Iomvh+l+x05hAjVZh8Sog0AxzdQKBgQCMgMTXt0ZBs0ScrG9v\np5CGa18KC+S3oUOjK/qyACmCqhtd+hKHIxHx3/FQPBWb4rDJRsZHH7C6URR1pHzJ\nmhCWgKGsvYrXkNxtiyPXwnU7PNP9JNuCWa45dr/vE/uxcbccK4JnWJ8+Kk/9LEX0\nmjtDm7wtLVlTswYhP6AP69RoMQ==\n-----END PRIVATE KEY-----\n",
  "client_email": "my-sa-313@caramel-goat-354919.iam.gserviceaccount.com",
  "client_id": "103735491955093092925",
  "auth_uri": "https://accounts.google.com/o/oauth2/auth",
  "token_uri": "https://oauth2.googleapis.com/token",
  "auth_provider_x509_cert_url": "https://www.googleapis.com/oauth2/v1/certs",
  "client_x509_cert_url": "https://www.googleapis.com/robot/v1/metadata/x509/my-sa-313%40caramel-goat-354919.iam.gserviceaccount.com"
}
```
Save this JSON file as gcp-credentials.json.

Use `kubectl create secret -n upbound-system` to generate the Kubernetes secret object inside the Universal Crossplane cluster.

```bash
kubectl create secret generic gcp-secret -n upbound-system --from-file=creds=./gcp-credentials.json
```

View the secret with `kubectl describe secret`
```bash
$ kubectl describe secret gcp-secret -n upbound-system
Name:         gcp-secret
Namespace:    upbound-system
Labels:       <none>
Annotations:  <none>

Type:  Opaque

Data
====
creds:  2334 bytes
```

### Create a ProviderConfig
Create a ProviderConfig Kubernetes configuration file to attach the GCP credentials to the installed official provider-gcp-storage.

```bash
kubectl apply -f gke_provider.yaml
```

Verify the ProviderConfig

```bash
$ kubectl describe providerconfigs
Name:         default
Namespace:
API Version:  gcp.upbound.io/v1beta1
Kind:         ProviderConfig
# Output truncated
Spec:
  Credentials:
    Secret Ref:
      Key:        creds
      Name:       gcp-secret
      Namespace:  upbound-system
    Source:       Secret
  Project ID:     crossplaneprojects
```

## Create 2 managed resources

```bash
kubectl apply -f gke_cluster_managed_resources.yaml
```

Check the status with:

```bash
kubectl get cluster
kubectl get nodepool
kubectl describe cluster
kubectl describe nodepool
```

### Access the New Cluster

Get the cluster name:

```bash
CLUSTER_NAME=$(kubectl get clusters.container.gcp.upbound.io gke-managed-resources -o=jsonpath='{.metadata.name}')
```

Get the cluster location:

```bash
CLUSTER_LOCATION=$(kubectl get clusters.container.gcp.upbound.io gke-managed-resources -o=jsonpath='{.spec.forProvider.location}')
```

Get the project ID:

```bash
PROJECT_ID=$(kubectl get providerconfig.gcp.upbound.io/default -o=jsonpath='{.spec.projectID}')
```

After running these commands, you will have environment variables CLUSTER_NAME, CLUSTER_LOCATION, and PROJECT_ID set with the respective values. Then, you can use them in your gcloud command like this:

```bash
gcloud container clusters get-credentials $CLUSTER_NAME --zone $CLUSTER_LOCATION --project $PROJECT_ID
```

> Note: You may run into this error: CRITICAL: ACTION REQUIRED: gke-gcloud-auth-plugin, which is needed for continued use of kubectl, was not found or is not executable. Install gke-gcloud-auth-plugin for use with kubectl by following https://cloud.google.com/blog/products/containers-kubernetes/kubectl-auth-changes-in-gke
If that happens, follow the link above to install the gke auth plugin.

## Create a Composite Resource Definition

```bash
kubectl apply -f cluster_XRD.yaml
```

## Create a Composition

```bash
kubectl apply -f cluster_composition.yaml
```

## Create a Claim

```bash
kubectl apply -f cluster_XR_claim.yaml
```

## Monitor the provisioning

```bash
kubectl get xcluster
kubectl describe xcluster
kubectl get cluster
kubectl describe cluster
kubectl get nodepool
kubectl describe nodepool
```