# Scalable Web Application on Kubernetes

## Objective
1. Deploy an application on Kubernetes
2. Scale and Update Deployments
3. Setting up Autoscaler (HPA)

## Pre-requisite
1. IBM Cloud Account
2. kubernetes cluster Provisioned
3. Good to have IBM Cloud CLI installtion
4. Hey for generating load (Optional for testing load from outside )
Installation
Linux 64-bit: https://storage.googleapis.com/hey-release/hey_linux_amd64
Mac 64-bit: https://storage.googleapis.com/hey-release/hey_darwin_amd64
Windows 64-bit: https://storage.googleapis.com/hey-release/hey_windows_amd64


### 1. Deploy an application on Kubernetes

#### Step 1) Access Kubernetes Cluster using Web Terminal 
Go to Navigation Menu -> Kubernetes . From the 3 dots on the right side of mycluster , click on Web Terminal. 
<img src="./Images/webterminal1.png"
     alt="Markdown Monster icon"
     style="float: left; margin-right: 5px;" />
<img src="./Images/Webterminal2.png"
     alt="Markdown Monster icon"
     style="float: left; margin-right: 5px;" />
     
#### Step 2) Create the guestbook application deployment:

```kubectl create deployment guestbook --image=ibmcom/guestbook:v1```

Check the status of the running application, you can use:

```kubectl get pods```

You should see similar output O/P:
```console
   $ kubectl get pods
   NAME                          READY     STATUS              RESTARTS   AGE
   guestbook-87b756bd5-5dxsr    0/1       ContainerCreating   0          1m
   ```
It will take some time to get this pod in running state
``` console
$ kubectl get pods
NAME                              READY   STATUS    RESTARTS   AGE
guestbook-87b756bd5-5dxsr         1/1     Running   0          112m
```
#### Step 3) Expose the application 
Once the status reads Running, we need to expose that deployment as a Service so that it can be accessed from outside. By specifying a service type of NodePort, the service will also be mapped to a high-numbered port on each cluster node. The guestbook application listens on port 3000, so this is also specified in the command. 

Run:
```kubectl expose deployment guestbook --type="NodePort" --port=3000```
O/P:
``` console 
$ kubectl expose deployment guestbook --type="NodePort" --port=3000
service "guestbook" exposed
```
Check the service using command
``` kubectl get svc```

``` console
$ kubectl get svc
NAME         TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)          AGE
guestbook    NodePort    172.21.156.20    <none>        3000:30298/TCP   101m
```

Guestbook application is exposed at port 30298 on public ip of cluster 

#### Step 4) Access the guestbook app 

Find the public IP of the cluster 
use this command to get public if of your cluster

```ibmcloud ks workers -c <cluster name>```

O/P:
```console
$ ibmcloud ks workers -c mycluster
OK
ID                                                     Public IP       Private IP      Flavor   State    Status   Zone    Version   
kube-brgtjnkd0kv0nvd7np4g-mycluster-default-000000dd   184.173.1.140   10.76.195.111   free     normal   Ready    hou02   1.17.6_1527   
```

You will get public IP os the cluster by checking the worker node tab on cluster home page. 

<img src="./Images/publicip.png"
     alt="Markdown Monster icon"
     style="float: left; margin-right: 5px;" />


Access the page as 
```http://<publicip>:<Nodeport_port>```

eg: http://184.173.1.140:30298/
  
<img src="./Images/guestbook.png"
     alt="Markdown Monster icon"
     style="float: left; margin-right: 5px;" />

### 2. Scale the application
A replica is a copy of a pod that contains a running service. By having multiple replicas of a pod, you can ensure your deployment has the available resources to handle increasing load on your application.

Step 1) To Scale the guestbook application use command 

```kubectl scale --replicas=10 deployment guestbook```

O/P:
``` console
$ kubectl scale --replicas=10 deployment guestbook
deployment "guestbook" scaled
```
 Kubernetes will now try to match the desired state of 10 replicas by starting 9 new pods with the same configuration 
as the first.

   ```console
   $ kubectl rollout status deployment guestbook
	Waiting for rollout to finish: 1 of 10 updated replicas are available...
	Waiting for rollout to finish: 2 of 10 updated replicas are available...
	Waiting for rollout to finish: 3 of 10 updated replicas are available...
	Waiting for rollout to finish: 4 of 10 updated replicas are available...
	Waiting for rollout to finish: 5 of 10 updated replicas are available...
	Waiting for rollout to finish: 6 of 10 updated replicas are available...
	Waiting for rollout to finish: 7 of 10 updated replicas are available...
	Waiting for rollout to finish: 8 of 10 updated replicas are available...
	Waiting for rollout to finish: 9 of 10 updated replicas are available...
	deployment "guestbook" successfully rolled out
   ```
   
   ``` console
   $ kubectl rollout status deployment guestbook
deployment "guestbook" successfully rolled out
```
Step 2) Check number of pods running by usig command 
``` kubectl get pods```

``` console
$ kubectl get pods
NAME                              READY   STATUS    RESTARTS   AGE
guestbook-87b756bd5-5dxsr         1/1     Running   0          138m
guestbook-87b756bd5-b9vfp         1/1     Running   0          61s
guestbook-87b756bd5-ftmxn         1/1     Running   0          61s
guestbook-87b756bd5-jwsqm         1/1     Running   0          61s
guestbook-87b756bd5-l46pv         1/1     Running   0          61s
guestbook-87b756bd5-q9gk9         1/1     Running   0          61s
guestbook-87b756bd5-vnw9t         1/1     Running   0          61s
guestbook-87b756bd5-vs56p         1/1     Running   0          61s
guestbook-87b756bd5-xtwck         1/1     Running   0          61s
guestbook-87b756bd5-zbqls         1/1     Running   0          61s

```
   
#### Kubernetes Update and rollback
Kubernetes allows you to do a rolling upgrade of your application to a new container image. Kubernetes allows you to easily update the running image but also allows you to easily undo a rollout if a problem is discovered during or after deployment.

In the previous lab, we used an image with a v1 tag. For our upgrade, we'll use the image with the v2 tag.

Step 1) Using kubectl, you can now update your deployment to use the v2 image. kubectl allows you to change details about existing resources with the set subcommand. We can use it to change the image being used.

```kubectl set image deployment/guestbook guestbook=ibmcom/guestbook:v2```

O/P:
``` console
 kubectl set image deployment/guestbook guestbook=ibmcom/guestbook:v2
deployment.apps/guestbook image updated
```
Check Rollout status using command
``` kubectl rollout status deployment/guestbook```

```console
$  kubectl rollout status deployment/guestbook
deployment "guestbook" successfully rolled out
```
Step 2) Test the application as before, by accessing ```http://<public-IP>:<nodeport>```(use the same as the previous lab) in the incognito mode of your browser to confirm your new code is active.
 
<img src="./Images/guestbookv2.png"
     alt="Markdown Monster icon"
     style="float: left; margin-right: 5px;" />
  
#### Step 3) Rollback your application
Use command "undo" to rollback the deplyment at previous version 

```kubectl rollout undo deployment guestbook```

O/P:
``` console
 kubectl rollout undo deployment guestbook
deployment.apps/guestbook rolled back
```

Check the pod status using 
``` kubectl get pods```

O/P:
``` console
$ kubectl get pods
NAME                              READY   STATUS        RESTARTS   AGE
guestbook-87b756bd5-4whpb         1/1     Running       0          13s
guestbook-87b756bd5-b6h6x         1/1     Running       0          13s
guestbook-87b756bd5-crh72         1/1     Running       0          9s
guestbook-87b756bd5-czjs2         1/1     Running       0          10s
guestbook-87b756bd5-dsv88         1/1     Running       0          13s
guestbook-87b756bd5-mh7xc         1/1     Running       0          13s
guestbook-87b756bd5-n2zt6         1/1     Running       0          9s
guestbook-87b756bd5-p9tbh         1/1     Running       0          13s
guestbook-87b756bd5-vmc6x         1/1     Running       0          9s
guestbook-87b756bd5-whhr8         1/1     Running       0          10s
guestbook-df976f65b-4lsgr         0/1     Terminating   0          10m
guestbook-df976f65b-82jd6         0/1     Terminating   0          10m
guestbook-df976f65b-gkf62         0/1     Terminating   0          10m
guestbook-df976f65b-jv9v7         0/1     Terminating   0          10m
guestbook-df976f65b-pf98c         0/1     Terminating   0          10m
guestbook-df976f65b-vsrfk         0/1     Terminating   0          10m
guestbook-df976f65b-x2mc7         0/1     Terminating   0          10m
guestbook-df976f65b-xxpsx         0/1     Terminating   0          10m
guestbook-df976f65b-z7tm9         0/1     Terminating   0          10m
load-generator-5fb4fb465b-rzsj2   1/1     Running       0          77m
logdna-agent-cjhjl                1/1     Running       0          47h
php-apache-79544c9bd9-cphmx       1/1     Running       0          78m
```

Check the broser again , you will see that you again get guestbook V1 app in your browser

#### Clean up
Before we continue, let's delete the application so we can learn about a different way to achieve the same results by using resource files instead of providing command line options.
To remove the deployment, use:

``` kubectl delete deployment guestbook```

To delete the service use

```  kubectl delete service guestbook```

We have finished checking scaling and rollout of application 


### 3. Setup auto-scaling for the app 

To demonstrate Horizontal Pod Autoscaler we will use a custom docker image based on the php-apache image

Execute following command to deply PHP-APACHE container 
```kubectl apply -f https://k8s.io/examples/application/php-apache.yaml```

setup Autoscaling using command 
```kubectl autoscale deployment php-apache --cpu-percent=50 --min=1 --max=10```

Check Horizontal Pod Autoscaling (HPA) using command 
```kubectl get hpa```

``` console
$ kubectl get hpa
NAME         REFERENCE               TARGETS   MINPODS   MAXPODS   REPLICAS   AGE
php-apache   Deployment/php-apache   0%/50%    1         10        1          4h6m
```

So now we have a container running which can autoscale based on load . Max replica set should be 10 and replica set should increate as and when CPU utilization goes above 50%

Expose your pod outside if you want to do load testing using external tool 
php-apache runs on port 80, expose nodeport for container port 80 using command 
```kubectl expose deployment php-apache --type="NodePort" --port=80```
check the service created  using command
```kubectl get svc```

``` console 
 kubectl get svc
NAME         TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)        AGE
php-apache   NodePort    172.21.136.105   <none>        80:31053/TCP   136m
```
Now you can do load testing from external tools like jmeter, load test , hey . I used Hey. Installtion method is provided in pre-requisite link

command used to do load test is 
```hey -n 100 -c 2 http://<public ip>:<nodeport>```
``` console
$ hey -n 100 -c 2 http://184.173.1.140:31053/

Summary:
  Total:	26.5119 secs
  Slowest:	1.0935 secs
  Fastest:	0.4467 secs
  Average:	0.5224 secs
  Requests/sec:	3.7719
  
  Total data:	300 bytes
  Size/request:	3 bytes

Response time histogram:
  0.447 [1]	|
  0.511 [83]	|■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■
  0.576 [6]	|■■■
  0.641 [0]	|
  0.705 [0]	|
  0.770 [2]	|■
  0.835 [0]	|
  0.899 [0]	|
  0.964 [5]	|■■
  1.029 [2]	|■
  1.094 [1]	|


Latency distribution:
  10% in 0.4572 secs
  25% in 0.4649 secs
  50% in 0.4757 secs
  75% in 0.4928 secs
  90% in 0.7379 secs
  95% in 0.9458 secs
  99% in 1.0935 secs

Details (average, fastest, slowest):
  DNS+dialup:	0.0054 secs, 0.4467 secs, 1.0935 secs
  DNS-lookup:	0.0000 secs, 0.0000 secs, 0.0000 secs
  req write:	0.0000 secs, 0.0000 secs, 0.0005 secs
  resp wait:	0.5168 secs, 0.4466 secs, 1.0931 secs
  resp read:	0.0001 secs, 0.0000 secs, 0.0003 secs

Status code distribution:
  [200]	100 responses


```
On the kubernetes webterminal check hpa using command
```kubectl get hpa```
and check the running pods using 

```kubectl get pods```
``` console
$ kubectl get hpa
NAME         REFERENCE               TARGETS         MINPODS   MAXPODS   REPLICAS   AGE
php-apache   Deployment/php-apache   81%/50%         1         10        2          143m
shiprajain_81@k8s-terminal ~ (⎈ mycluster/brgtjnkd0kv0nvd7np4g:default)$ kubectl get pods
NAME                              READY   STATUS    RESTARTS   AGE
php-apache-79544c9bd9-7t4zr       0/1     Pending   0          1s
php-apache-79544c9bd9-bhqp6       1/1     Running   0          108s
php-apache-79544c9bd9-cphmx       1/1     Running   0          128m
php-apache-79544c9bd9-ng4cq       0/1     Pending   0          16s
php-apache-79544c9bd9-tptw2       0/1     Pending   0          16s
```

You should see the replica set increasing as and when the load is getting incresed
Alternatively 

use load tester image to check the load 
We will start a container, and send an infinite loop of queries to the php-apache service (please run it in a different terminal):
execute following command to create a load test container and get inside the container 
```kubectl run -it --rm load-generator --image=busybox /bin/sh```

``` console
$ kubectl run -it --rm load-generator --image=busybox /bin/sh
kubectl run --generator=deployment/apps.v1 is DEPRECATED and will be removed in a future version. Use kubectl run --generator=run-pod/v1 or kubectl create instead.
If you don't see a command prompt, try pressing enter.
Hit enter for command prompt
```
When asked to hit enter, hit enter to get the bash and enter following command
```while true; do wget -q -O- http://php-apache; done```

``` console
while true; do wget -q -O- http://php-apache; done
OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK
```

Let it run for some time and then using ctrl C and exit come out 
Immediately check kubectl get pods. You should see more than 1 pod running for php-apache

``` console
 kubectl get pods
NAME                              READY   STATUS    RESTARTS   AGE
load-generator-5fb4fb465b-m28d4   1/1     Running   0          2m53s
logdna-agent-cjhjl                1/1     Running   0          2d1h
php-apache-79544c9bd9-9l8sv       0/1     Pending   0          86s
php-apache-79544c9bd9-cphmx       1/1     Running   0          3h44m
php-apache-79544c9bd9-nwv5s       0/1     Pending   0          71s
php-apache-79544c9bd9-qktd9       0/1     Pending   0          86s
php-apache-79544c9bd9-rdgdh       1/1     Running   0          86s
```
Number of pods will come down to 1 after sometime 

We have finished Autoscaling using load testing

## Clean up

``` kubectl delete deployment php-apache```

``` kubectl delete pod <podname>```
```kubectl delete svc <service name>```
```kubectl delete hpa <hpa name>```






