# nagp-kubernetes-repo
This project repo is for maintaining yaml files and read me file required for managing the deployment of nagp-rest-service project.

There are seven yaml files currently for handling the deployment of nagp-rest-service project and required my sql database.

Steps and commands to follow in below mentioned order for deployment.

1) Set up a standard GCP kubernetes cluster with required configurations.

2) nagp-storage-class.yaml --> This yaml file is for defining the storage class and required provisioner. Run below command using this yaml file.

   kubectl apply -f nagp-storage-class.yaml

   This will set up storage class which we can verify using command.

   kubectl get storageclass

3) nagp-mysql-configurations-secret.yaml --> This yaml file is for managing the secret. This contains required datebase credentials base 64 encoded.Run below 
   command using this yaml file.

   kubectl apply -f nagp-mysql-configurations-secret.yaml

   This will set up secret which we can verify using command.

   kubectl get secret

4) nagp-mysql-db-statefulset.yaml --> This yaml file is for managing the stateful set required for setting my sql pod. This has replica set value equal to 
   one which means that only one pod will be there always for my sql database. This has OnDelete stategy which means pod can be automatically deleted after 
   appying updated yaml but manual intervention would be needed. To set up this , please use below command.

   kubectl apply -f nagp-mysql-db-statefulset.yaml

   To verify use below command

   kubectl get statefulset

5) nagp-mysql-database-headless-service.yaml --> This yaml file is responsible for setting up headless service which would be required along with stateful 
   set for my sql database deployment.
    
   To set up this , please use below command.

   kubectl apply -f nagp-mysql-database-headless-service.yaml 

   To verify use below command

   kubectl get svc

6) nagp-env-config-map.yaml --> This yaml file is responsible for setting up required environment variables needed for nagp rest service application.
   
   To set up this , run below command

   kubectl apply -f nagp-env-config-map.yaml

   Use below command to verify.

   kubectl get configmap

7) nagp-project-deployment.yaml --> This yaml file is for responsible for setting up pods and deploying the nagp rest service application. This has replica 
   set value equal to four which means that four pods will be always there. This has RoolingUpdate stategy which means pods will be incrementally updated    
   with new ones.

   To set up deployment , please use below command.

   kubectl apply -f nagp-project-deployment.yaml

   To verify use this --> kubectl get deploy

9) nagp-load-balance-service.yaml --> This yaml is responsble for setting up required load balancer for project. We have exposed port 31063 for load balancer  
   in yaml. Make sure , same port is exposed in firewall GCP.

   To set up load balancer, use below command.

   kubectl apply -f nagp-project-deployment.yaml

   To verify use this --> kubectl get services

10) At end of all steps , if you run below command , you would see five pods running. One is for my sql database and other fours for nagp rest service 
   application.

   kubectl get po


Now as my sql database is there. We can use below command to set up database and load it with initial required data.

Run command to have bash shell of my sql pod. mysql-set-0 is pod name.

kubectl exec -it mysql-set-0 bash

Now run mysql -p command. It will ask password to connect with db. Then run below sql queries to set up data.

    DROP DATABASE IF EXISTS ems;
    CREATE DATABASE ems;
    USE ems;
    CREATE TABLE orders (id INT, user_key INT, side VARCHAR(5), amount Double, quantity Double);
    INSERT INTO orders VALUES (1, 3020, 'BUY', 5000, null);
    INSERT INTO orders VALUES (2, 3020, 'SELL', null, 15);
    INSERT INTO orders VALUES (3, 3021, 'BUY', 2700, null);
    INSERT INTO orders VALUES (4, 3021, 'BUY', 2900, null);
    INSERT INTO orders VALUES (5, 3023, 'SELL', null, 13);
    INSERT INTO orders VALUES (6, 3024, 'SELL', null, 10);
    INSERT INTO orders VALUES (7, 3024, 'SELL', null, 9);

11) Now hit required API from postman to verify results.

    http://<external ip of load balancer>:31063/getOrdersByUser/3020
