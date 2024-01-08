# Overview
Crossplane to createa a GKE cluster

## Instructions

Install the Up command-line
Download and install the Upbound up command-line.

curl -sL "https://cli.upbound.io" | sh
mv up /usr/local/bin/
Verify the version of up with up --version

$ up --version
v0.19.1
Note: official providers only support up command-line versions v0.13.0 or later.

Install Universal Crossplane
Install Upbound Universal Crossplane with the Up command-line.

$ up uxp install
UXP 1.13.2-up.2 installed
Note: Official provider-families only support crossplane version 1.12.1 or UXP version 1.12.1-up.1 or later.

Verify the UXP pods are running with kubectl get pods -n upbound-system

$ kubectl get pods -n upbound-system
NAME                                       READY   STATUS    RESTARTS   AGE
crossplane-77ff754998-k76zz                1/1     Running   0          40s
crossplane-rbac-manager-79b8bdd6d8-79577   1/1     Running   0          40s
Install the official GCP provider-family
Install the official provider-family into the Kubernetes cluster with a Kubernetes configuration file. For instance, let's install the provider-gcp-storage

Note: The first provider installed of a family also installs an extra provider-family Provider. The provider-family provider manages the ProviderConfig for all other providers in the same family.

apiVersion: pkg.crossplane.io/v1
kind: Provider
metadata:
  name: provider-gcp-storage
spec:
  package: xpkg.upbound.io/upbound/provider-gcp-storage:<version>
Apply this configuration with kubectl apply -f.

After installing the provider, verify the install with kubectl get providers.

$ kubectl get providers
NAME                          INSTALLED   HEALTHY   PACKAGE                                                AGE
provider-gcp-storage          True        True      xpkg.upbound.io/upbound/provider-gcp-storage:v0.36.0   78s
upbound-provider-family-gcp   True        True      xpkg.upbound.io/upbound/provider-family-gcp:v0.36.0    70s
It may take up to 5 minutes to report HEALTHY.

If you are going to use your own registry please check Install Providers in an offline environment.

Create a Kubernetes secret
The provider-gcp-storage requires credentials to create and manage GCP resources.

Generate a GCP JSON key file
Create a JSON key file containing the GCP account credentials. GCP provides documentation on how to create a key file.

Here is an example key file:

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
Save this JSON file as gcp-credentials.json.

Create a Kubernetes secret with GCP credentials
Use kubectl create secret -n upbound-system to generate the Kubernetes secret object inside the Universal Crossplane cluster.

kubectl create secret generic gcp-secret -n upbound-system --from-file=creds=./gcp-credentials.json

View the secret with kubectl describe secret

$ kubectl describe secret gcp-secret -n upbound-system
Name:         gcp-secret
Namespace:    upbound-system
Labels:       <none>
Annotations:  <none>

Type:  Opaque

Data
====
creds:  2334 bytes
Create a ProviderConfig
Create a ProviderConfig Kubernetes configuration file to attach the GCP credentials to the installed official provider-gcp-storage.

Note: the ProviderConfig must contain the correct GCP project ID. The project ID must match the project_id from the JSON key file.

apiVersion: gcp.upbound.io/v1beta1
kind: ProviderConfig
metadata:
  name: default
spec:
  projectID: <PROJECT_ID>
  credentials:
    source: Secret
    secretRef:
      namespace: upbound-system
      name: gcp-secret
      key: creds
Apply this configuration with kubectl apply -f.

Note: the ProviderConfig value spec.secretRef.name must match the name of the secret in kubectl get secrets -n upbound-system and spec.secretRef.key must match the value in the Data section of the secret.

Verify the ProviderConfig with kubectl describe providerconfigs.

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
  Project ID:     caramel-goat-354919
Create a managed resource
Create a managed resource to verify the provider-gcp-storage is functioning.

This example creates a GCP storage bucket, which requires a globally unique name.

Generate a unique bucket name from the command line.

echo "upbound-bucket-"$(head -n 4096 /dev/urandom | openssl sha1 | tail -c 10)

For example

$ echo "upbound-bucket-"$(head -n 4096 /dev/urandom | openssl sha1 | tail -c 10)
upbound-bucket-21e85e732
Use this bucket name for metadata.annotations.crossplane.io/external-name value.

Create a Bucket configuration file. Replace <BUCKET NAME> with the upbound-bucket- generated name.

apiVersion: storage.gcp.upbound.io/v1beta1
kind: Bucket
metadata:
  name: example
  labels:
  annotations:
    crossplane.io/external-name: <BUCKET NAME>
spec:
  forProvider:
    location: US
    storageClass: MULTI_REGIONAL
  providerConfigRef:
    name: default
  deletionPolicy: Delete
Note: the spec.providerConfigRef.name must match the ProviderConfig metadata.name value.

Apply this configuration with kubectl apply -f.

Use kubectl get managed to verify bucket creation.

$ kubectl get bucket
NAME      READY   SYNCED   EXTERNAL-NAME              AGE
example   True    True     upbound-bucket-4a917c947   90s








<!-- #################### -->
### Step 1: Install Crossplane

Let's use helm to install crossplane:

```bash
helm repo add \
crossplane-stable https://charts.crossplane.io/stable && helm repo update

helm install crossplane \
crossplane-stable/crossplane \
--namespace crossplane-system \
--create-namespace
```

Check that the pods came up

```bash
kubectl get pods -n crossplane-system
```

Look at the new API end-points with kubectl api-resources

```bash
kubectl api-resources  | grep crossplane
```

### Step 2: Create a Kubernetes secret for AWS 

Rename the `aws-credentials-example.txt` file to `aws-credentials.txt` and add your AWS credentials to the file. Make sure you don't check this file into git.

Now create the secret below

```bash
kubectl create secret \
generic aws-secret \
-n crossplane-system \
--from-file=creds=./aws-credentials.txt
```

Check that the secret was created:
```bash
kubectl describe secret aws-secret
```

> Note that if your AWS creds change, you can delete this secret and recreate it after updating your `aws-credentials.txt` file. Make sure you don't check this file into git.

### Step 3: Get the AWS provider and provider config ready

Now let's configure the AWS provider and use the credentials we created.

```bash
kubectl apply -f provider.yaml
```

You now have your Kubernetes cluster ready with crossplane installed.

### Step 4: Create the S3 Bucket


Apply the configuration:

```bash
kubectl apply -f s3bucket.yaml
```

### Step 5: Verify Resource Status
Check the status of the resource claim to ensure the S3 bucket has been successfully provisioned:

```bash
kubectl get bucket
```

Upon successful provisioning, you should see the following output:

```
NAME                  READY   SYNCED   EXTERNAL-NAME         AGE
tekanaid-crossplane   True    True     tekanaid-crossplane   11m
```

### Step 6: Verify Drift Correction

Now delete the bucket from the AWS console and wait some time for crossplane to recreate the bucket.

Run the command below to see the status of the bucket:

```bash
kubectl describe bucket
```

### Step 7: Cleanup

Now when ready to delete the bucket run the command below:

```bash
kubectl delete -f s3bucket.yaml
```

### Conclusion

In this demo, we walked through the process of creating an AWS S3 bucket using Crossplane. This showcases how Crossplane facilitates cloud resource provisioning through a declarative API, abstracting the complexities of individual cloud providers and providing a unified, Kubernetes-native interface for cloud infrastructure management.
