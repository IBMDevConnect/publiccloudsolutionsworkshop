# Public Cloud Solutions Workshop - 1


## 0. Register on IBM Cloud

Get the VCPI Link for IBM Cloud registration from the speaker for the session.

After successful registration Login into your account. 

## 1. Convert the IBM Cloud Lite account to Trial account

You will be provided with PROMO Code. 
Account -- Apply Promo Code

Complete the following steps to apply a feature code to your account:

1) In the IBM Cloud console, go to Manage > Account, and select Account settings.
2) Click Apply code.
3) Enter the feature code, which is typically a random alphanumeric value such as 'a1b2c3def456'.
4) Click Apply.

To know about extra resources and capabilities from this promo code refer to below link-
[IBM Cloud Sign up and PROMO Code](https://cloud.ibm.com/docs/account?topic=account-codes#codes)


## 2. Provision a Kubernetes Cluster on IBM Cloud

From Dashboard menu, select Kubernetes.

<img src="./img/k8s-1.png"
     alt="Markdown Monster icon"
     style="float: left; margin-right: 5px;" />
     

After that click on "Create Cluster", you will be asked following details to enter.

a - Select Free Cluster

b - Give a cluster name as "mycluster"

     
<img src="./img/k11.png"
     alt="Markdown Monster icon"
     style="float: left; margin-right: 10px;" />
     
     
Once the cluster is provisioned, Cluster will be shown in Normal State. 

     
<img src="./img/k8-3.png"
     alt="Markdown Monster icon"
     style="float: left; margin-right: 10px;" />     


## 3. Create a namespace in Container Registry

1. Create `Container Registry` service in IBM Cloud

2. Create a `namespace` within the container registry service

## 4. Access Kubernetes Cluster using Web Terminal

Once the cluster is provisioned, the kubernetes client CLI `kubectl` needs to be
configured to talk to the provisioned cluster.

This can also be done using your local desktops. You will have to install IBM Cloud Plugin Tools for doing that.

For this lab, you will be using Web Terminal, to work with your cluster. So installations on client machines is not required.
This feature is in beta currently. 

a. Access the Cluster, and go to Cluster details

<img src="./img/webterm1.png"
     alt="Markdown Monster icon"
     style="float: left; margin-right: 10px;" />   

b. Click on "Web terminal(beta) "
<img src="./img/webterm2.png"
     alt="Markdown Monster icon"
     style="float: left; margin-right: 10px;" /> 

c. Web based terminal opens at the bottom of the browser.     
<img src="./img/webterm3.png"
     alt="Markdown Monster icon"
     style="float: left; margin-right: 10px;" />       
      
     
Congratulations, you've now ready for coming week Lab on Kubernetes!

