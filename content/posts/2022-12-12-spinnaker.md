---
title: "Deploying Spinnaker in a Kubernetes cluster"
date: 2022-12-12
tags: ["DevOps"]
authors: ["kevinan9"]
---

# Deploying Spinnaker in a Kubernetes cluster

### Pre-requisites
Kubernetes cluster with atleast **4 cores** and **12GB memory**

### Start Halyard in a new Docker container on your current machine.
```
mkdir ~/.hal

mkdir ~/.kube

docker run -p 8084:8084 -p 9000:9000 \
    --name halyard --rm \
    -v ~/.hal:/home/spinnaker/.hal \
    -v ~/.kube:/home/spinnaker/.kube \
    -d \
    us-docker.pkg.dev/spinnaker-community/docker/halyard:stable
```
> Note: If you install Halyard in a Docker container, you will need to manually change permissions on the mounted ~/.hal directory to ensure Halyard can read and write to it.
> ~/.kube is used to store k8s cluster credentials


Then in a separate shell, connect to Halyard:
```
docker exec -it halyard bash
```
You can interact with Halyard from here.


### make sure that the provider is enabled:
In container:
```
hal config provider kubernetes enable
```

Then add the account:
```
CONTEXT=$(kubectl config current-context)
hal config provider kubernetes account add my-k8s-account     --context $CONTEXT
```
> my-k8s-account is used to map to your k8s cluster account configed in ~/.kube

```
hal config deploy edit --type distributed --account-name my-k8s-account
```

### Distributed installation on Kubernetes
Run the following command, using the $ACCOUNT name you created when you configured the provider:
```
hal config deploy edit --type distributed --account-name $ACCOUNT
```
> here $ACCOUNT is my-k8s-account in this guide

### Editing your external storage settings(AWS s3 for this guide)
```
hal config storage s3 edit \
    --access-key-id $YOUR_ACCESS_KEY_ID \
    --secret-access-key $YOUR_SECRET_ACCESS_KEY \
    --region [your s3 region] \
    --bucket [your s3 bucket name]
```
Finally, set the storage source to S3:

### Pick a version
```
hal config storage edit --type s3
```

```
hal version list

hal config version edit --version 1.29.2
```
> There is a bug Spinnaker 1.27.3 in which spin-gate will be checking health status forever


### Deploy Spinnaker
```
hal deploy apply
```


```
kubectl get svc -n spinnaker
```
![image](https://user-images.githubusercontent.com/1268001/208226091-c0d670a2-313a-4b30-b121-14d8bc81c608.png)

> At this point, all the services are deployed, well, no external IP addresses for desc and gate.

### Further configuration needed to see Spinnaker's UI
- Exposing Spinnaker to End Users
![image](https://user-images.githubusercontent.com/1268001/208170592-3411716e-64cc-4eae-9514-51ed49237117.png)

```
kubectl -n spinnaker edit svc spin-deck
kubectl -n spinnaker edit svc spin-gate
```
![image](https://user-images.githubusercontent.com/1268001/208226131-96d931dd-7221-414f-8b9c-36b4f8de5d3f.png)

```
hal deploy apply
```

Take a look at all the services again
```
kubectl get svc -n spinnaker
```
![image](https://user-images.githubusercontent.com/1268001/208226176-2a90d094-f68e-41f7-b541-e1149284a3f3.png)
You will see deck and gate are now LoadBalancer, copy their external IP addresses for further use.

```
hal config security ui edit --override-base-url "http://[external ip:port of deck service]"
hal config security api edit --override-base-url "http://[external ip:port of gate service]"
hal deploy apply
```

In your brower, open url: http://[deck service ip:port]
![image](https://user-images.githubusercontent.com/1268001/208226276-08e807fd-ae11-4cbb-bdf2-22f67bd3a106.png)
- If 'Applications' or 'Projects' pages are failed to load, please enable CORS on your web browser:
Chrome：https://chrome.google.com/webstore/detail/allow-cors-access-control/lhobafahddgcelffkeicbaginigeejlf
Firefox: https://addons.mozilla.org/en-US/firefox/addon/access-control-allow-origin/

---

### Back Up Your Config
First of all, let's see the directory under .hal
![image](https://user-images.githubusercontent.com/1268001/208317646-1e69d6c5-2c59-491f-a1a9-2d797f0d6dff.png)
Among these files, there are some auto generated ones, which will be overwritten by Halyard.
![image](https://user-images.githubusercontent.com/1268001/208317676-655c53a1-614d-4590-aef9-5c7931fe1da6.png)

> `settings.js` is for service `deck` only, that's why you won't be able find a config file named `desc.yml`.
> 
> You can validate that your Custom Profiles have been picked up and applied to their services by checking `~/.hal/$DEPLOYMENT/history/service-profiles.yml`.

Custom Profile for Deck 
At any point in time, you can run
```
hal backup create
```
This will produce a tar file that contains all linked local files, and a modified halconfig file that points to the local files within the tarball.
> warning: This includes all secrets you’ve supplied to hal. Keep this safe!

Given that tar file, you can at any time for any machine/user running Halyard run
```
hal backup restore --backup-path <backup-name>.tar
```
and Halyard will expand & replace the existing ~/.hal directory with the backup.
   
### Uninstall & Cleanup
uninstall Spinnaker from cluster
```
hal deploy clean
```
uninstall halyard
```
docker rm halyard
```

### Refs
[Spinnaker Custom Settings](https://spinnaker.io/docs/reference/halyard/custom/)

[Deploying Spinnaker in Kubernetes cluster](https://www.youtube.com/watch?v=9EUyMjR6jSc)
