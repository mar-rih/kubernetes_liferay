# kubernetes_liferay
This project is buid and tested using kuberentes release of AWS, through EKS services. and using postgres.

## Version
liferay 6.2 ce-ga6
postgres:9.5

## Getting Started
These instructions will get you a copy of the project up and running on your local machine for development and testing purposes.

### Prerequisites
* Install EKS cluser in AWS.
* kubectl - kubernetes command tool.
* aws - command tool.
* ECR - docker registery of AWS
* build your flavor of liferay docker images and uploading it to your ECR registery
* after build you image and upload it to your docker registery, just change the image value in /lifferay_frontend_k8s/liferay_deployement.yml

### Liferay flavor - docker file example
```
FROM marrih/naseej-medad:latest

MAINTAINER Marwan Alrihawi <marwan.alrihawi@naseej.com>
RUN ln -s /opt/liferay-portal-6.2-ce-ga6 /opt/liferay-portal
ENV LIFERAY_PATH_DIR=/opt/liferay-portal

# add configuration liferay file
ADD lep/portal-bundle.properties /opt/liferay-portal-6.2-ce-ga6/portal-bundle.properties
ADD lep/portal-bd-MYSQL.properties /opt/liferay-portal-6.2-ce-ga6/portal-bd-MYSQL.properties
ADD lep/portal-bd-POSTGRESQL.properties /opt/liferay-portal-6.2-ce-ga6/portal-bd-POSTGRESQL.properties

ADD lep/portal-ext.properties /opt/liferay-portal-6.2-ce-ga6/portal-ext.properties
ADD lep/portal-setup-wizard.properties /opt/portal-setup-wizard.properties

# Optional requirements
```
You can use helper_svc_k8s, as a docker daemon, to build your image. just create the pods as per following command:
```
kubectl create /helper_svc_k8s/helper_deploy.yml
```
you can call shell command of this pods using following command

```
kubectl exec -it deb-service-**** -- /sh
```
After creating the your liferay flavor you can now run the image in EKS cluster as per following

#### database POSTGRES deploy
1- create the presistance volume
```
kubectl create -f /pgs_db_k8s/postgres-pv.yml
```
2- create deployement and the service
```
kubectl create -f /pgs_db_k8s/postgres.yml
```
notes: you can check if the previouse command is succeed to create the database by following commmand:
```
kubectl exec -it  postgres-****** -- psql -h postgres -d postgresdb -U admin -c "\c"
```
 you can test the existed databases using this PSQL command 
```
 kubectl exec -it postgres-****** -- psql -h postgres -d postgresdb -U admin -c "\l"
```
   password which is "1q2q2w3w"

### Liferay deploy
 ```
 kubectl create -f /lifferay_frontend_k8s/liferay-pvs.yml
 kubectl create -f /lifferay_frontend_k8s/liferay_claims.yml
 kubectl create -f /lifferay_frontend_k8s/liferay_services.yml
 kubectl create -f /lifferay_frontend_k8s/liferay-ingress.yml
 kubectl create -f /lifferay_frontend_k8s/liferay_deployement.yml
```

## Authors

* **Marwan Alrihawi** - *DevOps Eng* - [mar-rih](https://github.com/mar-rih)






