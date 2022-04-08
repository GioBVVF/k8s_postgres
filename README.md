# k8s_postgres

## Simple PostgreSQL deployment on k8s

### Introduction

This definistion file provides a simply PostgreSQL deployment on Kubernetes cluster. In this example we understand some basic concepts of k8s and how these are applied to deploy a single postgres instance. On kubernetes the containers are run in the PODs. Pods are the smallest distributable compute units you can create and manage in Kubernetes. Deployments give us agility to scale, upgrade and update our application, so not in this case but in solution where is implemented a cluster database provide us HA and business continuity. 

                          ![immagine](https://user-images.githubusercontent.com/45388549/162480520-2a168d0c-4e9f-4f6f-8a94-cb020f7be912.png)

For better understand relationship between pod and deployment i suggest to read also documentation about ReplicaSet. In kubernetes we can define infrastructure as code using YAML file. We can analyze deployment definition file.

                          ![immagine](https://user-images.githubusercontent.com/45388549/162480717-4d5856e3-fada-425e-ac57-c1a044319c50.png)

We can see four section :

  -1 we find kubernetes API where object is defined, name and some metadata that we can assign to object. This section is the same for all kubernetes object.
  
  -2 number of instance in ReplicaSet, matchLabel to identify on which POD deployment is referred and strategy used to update application.
  
  -3 POD template: container’s image, container’s name, container’s resources in term of CPU and memory and his limit, port on which container listen (5432        for postgres), environment variable to configure postgres instance and finally mounting path for data directory.
    
  -4 to provide persistence we need an object in kubernetes named PersistentVolume. To claim this volume in a POD we use PersistentVolumeClaim. In this            example pg-pv-claim is used to provide a volume mounted on mountPath on container executed in a pod that is part of pg-deploy kubernetes deployment. 
    
License
-------

BSD

Author Information
------------------

[Giovanni Barbato](https://github.com/GioBVVF)
