# Overview
Intro to Crossplane

## Instructions

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
