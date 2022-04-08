# k8s_postgres

## Simple PostgreSQL deployment on k8s

### Introduction

This definistion file provides a simply PostgreSQL deployment on Kubernetes cluster. In this example we understand some basic concepts of k8s and how these are applied to deploy a single postgres instance. On kubernetes the containers are run in the PODs. Pods are the smallest distributable compute units you can create and manage in Kubernetes. Deployments give us agility to scale, upgrade and update our application, so not in this case but in solution where is implemented a cluster database provide us HA and business continuity. 

For better understand relationship between pod and deployment i suggest to read also documentation about ReplicaSet. In kubernetes we can define infrastructure as code using YAML file. We can analyze deployment definition file.

 ```
 apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: pg-deploy
  name: pg-deploy
spec:
  replicas: 1
  selector:
    matchLabels:
      app: pg-deploy
  strategy:
    type: RollingUpdate
  template:
    metadata:
      creationTimestamp: null
      labels:
        app: pg-deploy
    spec:
      containers:
       - name: pg-container
         image: postgres:14
         resources:
          limits:
            cpu: 1
            memory: 2Gi
          requests:
            cpu: 500m
            memory: 1Gi
         #args: ["-c", "max_connections=10","-c","shared_buffers=128MB"]
         ports:
          - containerPort: 5432
         envFrom:
          - configMapRef:
             name: pg-config
          - secretRef:
             name: pg-secret
         volumeMounts:
          - mountPath: /var/lib/postgresql/data
            name: pg-storage
      volumes:
        - name: pg-storage
          persistentVolumeClaim:
            claimName: pg-pv-claim
 ```

We can see four section :

  1. we find kubernetes API where object is defined, name and some metadata that we can assign to object. This section is the same for all kubernetes object.
  
  ```
  apiVersion: apps/v1
  kind: Deployment
  metadata:
   labels:
    app: pg-deploy
   name: pg-deploy
  ```
  
  2. number of instance in ReplicaSet, matchLabel to identify on which POD deployment is referred and strategy used to update application.
  
  ```
  spec:
  replicas: 1
  selector:
    matchLabels:
      app: pg-deploy
  strategy:
    type: RollingUpdate
  ```
   
  3. POD template: container’s image, container’s name, container’s resources in term of CPU and memory and his limit, port on which container listen (5432        for postgres), environment variable to configure postgres instance and finally mounting path for data directory.
  ```
  template:
   metadata:
     creationTimestamp: null
     labels:
       app: pg-deploy
   spec:
     containers:
      - name: pg-container
        image: postgres:14
        resources:
         limits:
           cpu: 1
           memory: 2Gi
         requests:
           cpu: 500m
           memory: 1Gi
        #args: ["-c", "max_connections=10","-c","shared_buffers=128MB"]
        ports:
         - containerPort: 5432
        envFrom:
         - configMapRef:
            name: pg-config
         - secretRef:
            name: pg-secret
        volumeMounts:
         - mountPath: /var/lib/postgresql/data
           name: pg-storage  
   ```
  4. to provide persistence we need an object in kubernetes named PersistentVolume. To claim this volume in a POD we use PersistentVolumeClaim. In this            example pg-pv-claim is used to provide a volume mounted on mountPath on container executed in a pod that is part of pg-deploy kubernetes deployment. 
  ```
  volumes:
       - name: pg-storage
         persistentVolumeClaim:
           claimName: pg-pv-claim
  ```

Now we can see command and step that perform postgres deploy on cluster :

    git clone https://github.com/GioBVVF/k8s_postgres.git
    cd k8s_postgres
    kubectl create -f pg-configmap.yaml
    kubectl create -f pg-secret.yaml
    kubectl create -f pg-pv.yaml
    kubectl create -f pg-pvc.yaml
    kubectl create -f pg-deploy.yaml
    waiting that pod is in running state (kubectl get pod -l app=pg-deploy)
    kubectl create -f pg-service-deploy.yaml 
    
Ok now have a postgres installation on k8s and we need to connect to. In this infrastructure we have exposed postgres out of k8s cluster using a LoadBalancer Service on port 30011. With psql:

```
psql -h <IP_NODE_WHERE_POD_IS_RUNNING> -p 30011 -U admin admin
```

you can get IP_NODE_WHERE_POD_IS_RUNNING with command :

```
kubectl get pod -l app=pg-deploy -o jsonpath='{.items[*].status.hostIP}'
```

Configuration and sensitive data are injected into pod by configmap and secret . To show user and password for database connection:

```
kubectl get secrets pg-secret -o jsonpath='{.data.POSTGRES_PASSWORD}' | base64 -d

kubectl get secrets pg-secret -o jsonpath='{.data.POSTGRES_USER}' | base64 -d
```

License
-------

BSD

Author Information
------------------

[Giovanni Barbato](https://github.com/GioBVVF)
