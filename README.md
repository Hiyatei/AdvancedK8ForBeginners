# Advanced Kubernetes For Beginners

<img src="https://user-images.githubusercontent.com/1936716/122274216-ef9f8000-cea7-11eb-8efc-82c7e58cd239.png" width=“700” />

In this workshop we will explore kubernetes beyond simply spinning up a container in a pod.  

## Before starting
Workshop attendees will receave an email with the instance info prior to the workshop.

Notice that training cloud instances will be available only during the workshop and will be terminated **24 hours later**. If you are in our workshop we recommend using the provided cloud instance, you can relax as we have you covered: prerequisites are installed already.

**⚡ IMPORTANT NOTE:**
Everywhere in this repo you see `<YOURADDRESS>` replace with the URL for the instance you were given.  

## Table of content and resources

* [Workshop On YouTube](YOUTUBE LINK HERE)
* [Presentation](PDF OF SLIDES HERE)
* [Discord chat](DISCORD LINK HERE)

| Title  | Description
|---|---|
| **1 - Getting Connected** | [Instructions](#1-Getting-Connected)  |
| **2 - Control Loop and Operators** | [Instructions](#2-Control-Loop-and-Operators)  |
| **3 - Custom Resource Definitions (CRD)** | [Instructions](#3-Custom-Resource-Definitions-crd)  |
| **4 - Sidecars** | [Instructions](#4-Sidecars)  |
| **5 - ConfigMap and Secrets** | [Instructions](#5-ConfigMap-and-Secrets)  |
| **6 - Resources** | [Instructions](#Resources)  |

## 1. Getting Connected
**✅ Step 1a: The first step in the section.**

In your browser window, navigate to the url <YOURADDRESS>:3000 where your address is the one provided by Prasson.
  
When you arrive at the webpage you should be greeted by something similar to this.
<img src="https://user-images.githubusercontent.com/1936716/107884421-a23fe180-6eba-11eb-96d2-4c703ccb1dcf.png" width=“700” />

Click in the `Terminal` menu from the top of the page and select new terminal as shown below
<img src="https://user-images.githubusercontent.com/1936716/107884506-09f62c80-6ebb-11eb-9f7b-42bdb3444cc1.png" width=“700” />

Once you have opened the terminal run
```bash
kubectl get nodes
```

*📃output*

```bash
NAME                        STATUS   ROLES    AGE   VERSION
learning-cluster-master     Ready    master   49m   v1.19.4
learning-cluster-worker-0   Ready    <none>   49m   v1.19.4
learning-cluster-worker-1   Ready    <none>   49m   v1.19.4
ubuntu@learning-cluster-master:~/workshop$ 
```
If you see the above output you are ready for the lab.

## 2. Control Loop and Operators
  
To understand operators a bit more we will use kubectl to apply a pre built opperator for the Cassandra database.
  **✅ Step 1a: The first step in the section.** 
 
```bash
kubectl apply -f https://raw.githubusercontent.com/datastax/cass-operator/v1.6.0/docs/user/cass-operator-manifests-v1.19.yaml
```

*📃output*
```bash
Output from the above command     
```
  
Screenshot of the above working
<img src="https://user-images.githubusercontent.com/blah/blahblah.png" width=“700” />
  
  

## 3. Custom Resource Definitions (CRD)
  
Custom resource definitions are ways that you can create assets the function outside of the default kubernetes resource types.  This section pulls heavily from the kubernetes custom resource definition examples in the docs.
 
  
Before getting started check what CRDs are already on the cluster.
  
```bash
kubectl get crd
```
  
*📃output*
```bash
NAME                              CREATED AT
addons.k3s.cattle.io              2021-06-17T18:59:23Z
helmcharts.helm.cattle.io         2021-06-17T18:59:23Z
helmchartconfigs.helm.cattle.io   2021-06-17T18:59:23Z
```

Next apply the yaml for the new custom CRD.
  
```bash
kubectl apply -f  crd.yaml
``` 
  
Check the list of CRDs again.  Notice the new entry for our new asset.
```bash
kubectl get crd
``` 
  
Start a pod based of the new resource type we defined. 
```bash
kubectl apply -f example-pod.yaml
``` 

Check for running resources of the type crontab.
```bash
kubectl get crontab
``` 

If more information is required running the -o wide flag on the command will provide a full output in yaml format.
```bash
kubectl get ct -o yaml
```

As illistrated here a custom resource can easily be created allowing for much more expanded pod creation.
  
  
## 4. Sidecars
  
Much of the credit for this sidecar section goes to [this great tutorial on Medium](https://medium.com/bb-tutorials-and-thoughts/kubernetes-learn-sidecar-container-pattern-6d8c21f873d)
  
Before we apply anything lets pull the yaml we will use. 
  ```bash
  curl -0 https://gist.githubusercontent.com/bbachi/7d6c40fc8f660eed243f7e9cd31d99c8/raw/03801fa846760d7833ad10f0be69552eede7d828/manifest.yml > sidecars.yaml
```

*📃output*
```bash
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100  1478  100  1478    0     0  14211      0 --:--:-- --:--:-- --:--:-- 14211    
```
 
If we open up the yaml file in the file explorer we can see that we are setting up three containers.  One is a simple Nginx server and two are seperate sidecars.  Go ahead and apply the yaml.
 
Note we are creating a deployment instead of a single pod.
  ```bash
kubectl create -f sidecars.yaml
```
  
*📃output*
```bash
deployment.apps/nginx-webapp created
service/nginx-webapp created
```

Kubernetes will create a deployment object that has every component we just set up inside of it.  This deployment object can be created and destroyed by kubectl in a single step.
  
```bash
kubectl get deploy -o wide
```
  
*📃output*
```bash
NAME           READY   UP-TO-DATE   AVAILABLE   AGE   CONTAINERS                                             IMAGES                  SELECTOR
nginx-webapp   5/5     5            5           36s   sidecar-container1,sidecar-container2,main-container   busybox,busybox,nginx   app=nginx-webapp
```
  
To see the indavidual pods and ports that are now being used run the following.
  
```bash
kubectl get pods -o wide
```
   
*📃output*
```bash 
 NAME                           READY   STATUS    RESTARTS   AGE    IP           NODE                          NOMINATED NODE   READINESS GATES
nginx-webapp-f5cf77c9c-8789k   3/3     Running   0          101s   10.244.2.6   learning-cluster-0-worker-1   <none>           <none>
nginx-webapp-f5cf77c9c-rm64k   3/3     Running   0          101s   10.244.0.7   learning-cluster-0-master     <none>           <none>
nginx-webapp-f5cf77c9c-9k9vh   3/3     Running   0          101s   10.244.1.7   learning-cluster-0-worker-0   <none>           <none>
nginx-webapp-f5cf77c9c-d76q8   3/3     Running   0          101s   10.244.2.7   learning-cluster-0-worker-1   <none>           <none>
nginx-webapp-f5cf77c9c-5bfdc   3/3     Running   0          101s   10.244.1.6   learning-cluster-0-worker-0   <none>           <none>
```
  
```bash
kubectl get svc -o wide
```
  
*📃output*
```bash
NAME           TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)        AGE     SELECTOR
kubernetes     ClusterIP   10.43.0.1      <none>        443/TCP        20h     <none>
nginx-webapp   NodePort    10.43.218.89   <none>        80:32711/TCP   2m20s   app=nginx-webapp
```
  
We can even see more info if we run the following command.  

```bash
kubectl cluster-info
```
   
*📃output*
```bash
Kubernetes control plane is running at https://127.0.0.1:6443
CoreDNS is running at https://127.0.0.1:6443/api/v1/namespaces/kube-system/services/kube-dns:dns/proxy
Metrics-server is running at https://127.0.0.1:6443/api/v1/namespaces/kube-system/services/https:metrics-server:/proxy

To further debug and diagnose cluster problems, use 'kubectl cluster-info dump'.
```
  
Since everything is running on localhost we can simply go into the container and hit the web server. Replace YOURCONTAINERID with the container ID of your instance. 
  
```bash 
kubectl exec -it YOURCONTAINERID -c main-container -- /bin/sh
```
  
Curl isn't installed in this container so we will have to add it.  To do so run the following.
  
```bash  
apt-get update && apt-get install -y curl
```
  
   
*📃output*
```bash
Get:1 http://security.debian.org/debian-security buster/updates InRelease [65.4 kB]
Get:2 http://deb.debian.org/debian buster InRelease [121 kB]
Get:3 http://deb.debian.org/debian buster-updates InRelease [51.9 kB]
Get:4 http://security.debian.org/debian-security buster/updates/main amd64 Packages [292 kB]
Get:5 http://deb.debian.org/debian buster/main amd64 Packages [7907 kB]
Get:6 http://deb.debian.org/debian buster-updates/main amd64 Packages [10.9 kB]
Fetched 8449 kB in 2s (4806 kB/s)                     
Reading package lists... Done
Reading package lists... Done
Building dependency tree       
Reading state information... Done
curl is already the newest version (7.64.0-4+deb10u2).
0 upgraded, 0 newly installed, 0 to remove and 2 not upgraded.  
```
  
Now we can hit the localhost
```bash
curl localhost
```
   
*📃output*
```bash
...
echo Fri Jun 18 15:31:00 UTC 2021 Hi I am from Sidecar container 1
echo Fri Jun 18 15:31:00 UTC 2021 Hi I am from Sidecar container 2
echo Fri Jun 18 15:31:05 UTC 2021 Hi I am from Sidecar container 1
echo Fri Jun 18 15:31:05 UTC 2021 Hi I am from Sidecar container 2
echo Fri Jun 18 15:31:10 UTC 2021 Hi I am from Sidecar container 1
echo Fri Jun 18 15:31:10 UTC 2021 Hi I am from Sidecar container 2
echo Fri Jun 18 15:31:15 UTC 2021 Hi I am from Sidecar container 1
echo Fri Jun 18 15:31:15 UTC 2021 Hi I am from Sidecar container 2
echo Fri Jun 18 15:31:20 UTC 2021 Hi I am from Sidecar container 1
echo Fri Jun 18 15:31:20 UTC 2021 Hi I am from Sidecar container 2
echo Fri Jun 18 15:31:25 UTC 2021 Hi I am from Sidecar container 1
echo Fri Jun 18 15:31:25 UTC 2021 Hi I am from Sidecar container 2
echo Fri Jun 18 15:31:30 UTC 2021 Hi I am from Sidecar container 1
echo Fri Jun 18 15:31:30 UTC 2021 Hi I am from Sidecar container 2
```


## 5. ConfigMap and Secrets
  
  Description of the first section what we are going to try and do.

## 6. Resources
For further reading and labs go to 
[link name](URL) 
[link name](URL) 
[link name](URL) 
[link name](URL) 
[link name](URL) 
