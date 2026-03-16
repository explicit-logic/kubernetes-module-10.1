# Module 10 - Container Orchestration with Kubernetes

This repository contains a demo project created as part of my **DevOps studies** in the **TechWorld with Nana – DevOps Bootcamp**.

https://www.techworld-with-nana.com/devops-bootcamp

***Demo Project:*** Deploy MongoDB and Mongo Express into local K8s cluster

***Technologies used:*** Kubernetes, Docker, MongoDB, Mongo Express

***Project Description:***

- Setup local K8s cluster with Minikube
- Deploy MongoDB and MongoExpress with configuration and credentials extracted into ConfigMap and Secret

---

The installation documentation for Minikube can be found here:

https://minikube.sigs.k8s.io/docs/start/

![](./images/status.png)

![](./images/cli.png)

---

### Deploy MongoDB and MongoExpress with configuration and credentials extracted into ConfigMap and Secret

- Overview

![](./images/overview.png)

![](./images/request-flow.png)

1. Create Mongo DB deployment config

```sh
kubectl create deploy mongodb-deployment --image mongo --dry-run=client -o yaml > mongo.yaml
```
See: [](./mongo.yaml)


2. Create secrets

Encode credentials

```sh
echo -n 'root-user' | base64
# cm9vdC11c2Vy

echo -n 'root-password' | base64
# cm9vdC1wYXNzd29yZA==
```
See: [](./mongo-secret.yaml)

Run:
```sh
kubectl apply -f mongo-secret.yaml
```

3. Add secret reference to deployment config

```yaml
        env:
        - name: MONGO_INITDB_ROOT_USERNAME
          valueFrom:
            secretKeyRef:
              name: mongodb-secret
              key: mongo-root-username

        - name: MONGO_INITDB_ROOT_PASSWORD
          valueFrom:
            secretKeyRef:
              name: mongodb-secret
              key: mongo-root-password
```

See: [](./mongo.yaml)


4. Launch mongo db deployment

Run:
```sh
kubectl apply -f mongo.yaml
```

![](./images/run-mongo.png)

5. Add mongodb service

Add these lines to `mongo.yaml` config:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: mongodb-service
spec:
  selector:
    app: mongodb-deployment
  ports:
  - protocol: TCP
    port: 27017
    targetPort: 27017
```
See: [](./mongo.yaml)

Run:
```sh
kubectl apply -f mongo.yaml
```

![](./images/mongodb-service.png)


6. Create a configmap with database url

```sh
kubectl create configmap mongodb-configmap --from-literal=database_url="mongodb-service:27017" --dry-run=client -o yaml > mongo-configmap.yaml
```

Run:
```sh
kubectl apply -f mongo-configmap.yaml
```


7. Create MongoExpress deployment

```sh
kubectl create deploy mongo-express --image mongo-express --dry-run=client -o yaml > mongo-express.yaml
```

Then add the following lines:

```yaml
   env:
        - name: ME_CONFIG_MONGODB_ADMINUSERNAME
          valueFrom:
            secretKeyRef:
              name: mongodb-secret
              key: mongo-root-username

        - name: ME_CONFIG_MONGODB_ADMINPASSWORD
          valueFrom:
            secretKeyRef:
              name: mongodb-secret
              key: mongo-root-password

        - name: DATABASE_URL
          valueFrom:
            configMapKeyRef:
              name: mongodb-configmap
              key: database_url

        - name: ME_CONFIG_MONGODB_URL
          value: "mongodb://$(ME_CONFIG_MONGODB_ADMINUSERNAME):$(ME_CONFIG_MONGODB_ADMINPASSWORD)@$(DATABASE_URL)"
```
See: [](./mongo-express.yaml)

Run

```sh
kubectl apply -f mongo-express.yaml
```

![](./images/mongo-express-deploy.png)


8. Create Mongo Express service

```yaml
---

apiVersion: v1
kind: Service
metadata:
  name: mongo-express-service
spec:
  selector:
    app: mongo-express
  type: LoadBalancer
  ports:
  - protocol: TCP
    port: 8081
    targetPort: 8081
    nodePort: 30000
```
See: [](./mongo-express.yaml)

run:
```sh
kubectl apply -f mongo-express.yaml
```

9. Assign public IP address

```sh
minikube service mongo-express-service
```

Sign In

user: `admin`
password: `pass`

Demo

![](./images/demo.gif)

