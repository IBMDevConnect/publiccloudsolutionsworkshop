# IBM Cloud - IBM Kubernetes Service - Deploying WordPress and MySQL with Persistent Volumes

This tutorial shows you how to deploy a WordPress site and a MySQL database on IBM Kubernetes. Both applications use PersistentVolumes and PersistentVolumeClaims to store data.
WordPress is an open-source website creation platform that is written in PHP and uses a MySQL database

Please note : This workshop is intended to show how to use persistence volume and persistence volume claim . Hence created Mysql DB and a user for wordpress is not part of this tutorial 

# Objectives
1. Create storage class
2. Create PersistentVolumeClaims and PersistentVolumes
3. Create Deployment File - WordPress and Mysql
4. Create a kustomization.yaml with a Secret generator
5. MySQL resource configs
6. WordPress resource configs
7. Deploy  Application
8. Access Application
9. Clean up


##  Create Storage Class
On kubernetes terminal , create a new file name `storageclass.yaml` and paste below content

``` Console
kind: StorageClass
apiVersion: storage.k8s.io/v1beta1
metadata:
 name: ibmc-block-bronze
 labels:
  product: ibm-storage-enable-for-containers
provisioner: ubiquity/flex
parameters:
 profile: :ibmc-block-bronze
 fstype: ext4
 backend: scbe
```
use command below command to create storage class
```kubectl create -f storageclass.yaml```
We created a storage class name ibmc-block-bronze
Check if storage class created using command ``` kubectl get sc```

``` console
$ kubectl get sc
NAME                PROVISIONER     RECLAIMPOLICY   VOLUMEBINDINGMODE   ALLOWVOLUMEEXPANSION   AGE
ibmc-block-bronze   ubiquity/flex   Delete          Immediate           false                  29m
```

##  Create PersistentVolumeClaims and PersistentVolumes

Now create Persistance Volume and Persistence volume claim 
Create new file using vi editor names mysqlpv.yaml. Copy the contents provided below

``` console
apiVersion: v1
kind: PersistentVolume
metadata:
  name: mysql-persistent-storage
  labels:
    type: local
spec:
  storageClassName: ibmc-block-bronze
  capacity:
    storage: 1Gi
  accessModes:
    - ReadWriteOnce
  hostPath:
    path: "/var/lib/mysql"
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: mysql-pv-claim
spec:
  storageClassName: ibmc-block-bronze
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 500M
```

Copy another file named wordpresspv.yaml , and copy the content provided below

``` console
apiVersion: v1
kind: PersistentVolume
metadata:
  name: wordpress-persistent-storage
  labels:
    type: local
spec:
  storageClassName: ibmc-block-bronze
  capacity:
    storage: 1Gi
  accessModes:
    - ReadWriteOnce
  hostPath:
    path: "/var/www/html"
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: wp-pv-claim
spec:
  storageClassName: ibmc-block-bronze
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 500M
```

Once file are created use command below to create PV and PVc

```kubectl apply -f mysqlpv.yaml```
```kubectl apply -f wordpresspv.yaml```

Check if PV are created using command 
``` kubectl get pv```
``` console
$ kubectl get pv
NAME                           CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS   CLAIM                    STORAGECLASS        REASON   AGE
mysql-persistent-storage       1Gi        RWO            Retain           Bound    default/mysql-pv-claim   ibmc-block-bronze            32m
wordpress-persistent-storage   1Gi        RWO            Retain           Bound    default/wp-pv-claim      ibmc-block-bronze            32m
````
Check if pvc are created using command 
```kubectl get pvc```

```console
$ kubectl get pvc
NAME             STATUS   VOLUME                         CAPACITY   ACCESS MODES   STORAGECLASS        AGE
mysql-pv-claim   Bound    mysql-persistent-storage       1Gi        RWO            ibmc-block-bronze   33m
wp-pv-claim      Bound    wordpress-persistent-storage   1Gi        RWO            ibmc-block-bronze   33m

```
## Create Deployment File  - WordPress and MySQL

Now Create deployment yaml file for wordpress and mysql

Create mysql-deployment.yaml and copy the content below

``` console
apiVersion: v1
kind: Service
metadata:
  name: wordpress-mysql
  labels:
    app: wordpress
spec:
  ports:
    - port: 3306
  selector:
    app: wordpress
    tier: mysql
  clusterIP: None
---
apiVersion: apps/v1 # for versions before 1.9.0 use apps/v1beta2
kind: Deployment
metadata:
  name: wordpress-mysql
  labels:
    app: wordpress
spec:
  selector:
    matchLabels:
      app: wordpress
      tier: mysql
  strategy:
    type: Recreate
  template:
    metadata:
      labels:
        app: wordpress
        tier: mysql
    spec:
      containers:
      - image: mysql:5.6
        name: mysql
        env:
        - name: MYSQL_ROOT_PASSWORD
          valueFrom:
            secretKeyRef:
              name: mysql-pass
              key: password
        ports:
        - containerPort: 3306
          name: mysql
        volumeMounts:
        - name: mysql-persistent-storage
          mountPath: /var/lib/mysql
      volumes:
      - name: mysql-persistent-storage
        persistentVolumeClaim:
          claimName: mysql-pv-claim
```

create another file named wordpress-deployment.yaml and copy content below

``` console
apiVersion: v1
kind: Service
metadata:
  name: wordpress
  labels:
    app: wordpress
spec:
  ports:
    - port: 80
  selector:
    app: wordpress
    tier: frontend
  type: NodePort
---
apiVersion: apps/v1 # for versions before 1.9.0 use apps/v1beta2
kind: Deployment
metadata:
  name: wordpress
  labels:
    app: wordpress
spec:
  selector:
    matchLabels:
      app: wordpress
      tier: frontend
  strategy:
    type: Recreate
  template:
    metadata:
      labels:
        app: wordpress
        tier: frontend
    spec:
      containers:
      - image: wordpress:4.8-apache
        name: wordpress
        env:
        - name: WORDPRESS_DB_HOST
          value: wordpress-mysql
        - name: WORDPRESS_DB_PASSWORD
          valueFrom:
            secretKeyRef:
              name: mysql-pass
              key: password
        ports:
        - containerPort: 80
          name: wordpress
        volumeMounts:
        - name: wordpress-persistent-storage
          mountPath: /var/www/html
      volumes:
      - name: wordpress-persistent-storage
        persistentVolumeClaim:
          claimName: wp-pv-claim
```

## Create a kustomization.yaml with a Secret generator

We will need a secret to store the password for mysql . Create a kustomization file which will create secret file and other deployments

create Kustomization.yaml using command 
``` cat <<EOF >>./kustomization.yaml``` and paste content below

Provide the password of you choice. and make sure name of the deplyment file under resource tag are correct. In the last line write EOF and come out of editing 

``` console
> secretGenerator:
> - name: mysql-pass
>  literals:
>  - password=YOUR_PASSWORD
>  resources:
>  - mysql-deployment.yaml
>  - wordpress-deployment.yaml
> EOF

```  

Please make sure the indentation is correct. JSON file needs proper indentation.
You can use vi editor as well to create ./kustomization.yaml

## Deploy  Application

We have configured al the resources , now we need to deploy them all 
use command ```kubectl apply -k ./```

``` console
 kubectl apply -k ./
secret/mysql-pass-7tt4f27774 created
service/wordpress-mysql created
service/wordpress created
deployment.apps/wordpress-mysql created
deployment.apps/wordpress created
````

Check pods using command ``` kubectl get pods```
check service using ```kubectl get svc```

``` console
 kubectl get svc
NAME              TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)        AGE
kubernetes        ClusterIP   172.21.0.1       <none>        443/TCP        3d8h
php-apache        NodePort    172.21.136.105   <none>        80:31053/TCP   27h
wordpress         NodePort    172.21.240.64    <none>        80:31257/TCP   39m
wordpress-mysql   ClusterIP   None             <none>        3306/TCP       39m
```
## Access Application Deployed

you can access the application on <publicip>:<port>
  
  eg http://184.173.1.140:31257/
  
  
## Clean Up
Its alwasy good practice to clean up the data 
we will delete all the resources that we have created 

```kubectl delete -k ./```
  
  
