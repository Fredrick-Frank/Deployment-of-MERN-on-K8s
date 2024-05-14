# Deployment-of-MERN-on-K8s
----------------------------------------------------------------------------------
The deployment of a MERN (MongoDB, Expressjs, React and Nodejs) web using the base images on a Kubernetes Cluster. 
* The mongodb image was used to build the container image which was used to deploy the MERN app. 

In a kubernetes environment the use of a YAML file is used: 
-- Here we configured four yaml files
* The mongo-app-deployment.yaml: 
It contains the deployment file and the services:
apiVersion: apps/v1
kind: Deployment
metadata:
  name: mongodb-deployment
  labels:
    app: mongo
spec:
  replicas: 3
  selector:
    matchLabels:
      app: mongo
  template:
    metadata:
      labels:
        app: mongo
    spec:
      containers:
      - name: mongo
        image: mongo:6.0
        ports:
        - containerPort: 27017
        env:
        # Define the environment variable
        - name: MONGO_INITDB_ROOT_USERNAME
          valueFrom:
            secretKeyRef:
              # The ConfigMap containing the value you want to assign to SPECIAL_LEVEL_KEY
              name: mongo-secret
              key: mongo-user
        - name: MONGO_INITDB_ROOT_PASSWORD
          valueFrom:
            secretKeyRef:
              # The ConfigMap containing the value you want to assign to SPECIAL_LEVEL_KEY
              name: mongo-secret
              key: mongo-password
---

apiVersion: v1
kind: Service
metadata:
  name: mongo-service
spec:
  selector:
    app: mongo
  ports:
    - protocol: TCP
      port: 80
      targetPort: 27017

* mongo-config.yaml:
This mongo-config yaml file contains configmap. A configMap is a key-value array that is used to store non-confidential. Kubernetes stores API objects like ConfigMaps and Secrets within the etcd cluster. Etcd is essentially the brain of Kubernetes, since it stores all of the key-value objects that Kubernetes requires to orchestrate the containers.
apiVersion: v1
kind: ConfigMap
metadata:
  name: mongo-config
data:
  mongo-url: mongo-service

* The secret.yaml
The secret.yaml file contains the key-value store for the mongo-user & mongo-password. The base64 format was used to encode the user name and password.

apiVersion: v1
kind: Secret
metadata:
  name: mongo-secret
type: Opaque
data:
  mongo-user: bW9uZ291c2Vy
  mongo-password: bW9uZ29wYXNzd29yZA==


* The web-app.yaml file: This file contains both the deployment and services. Unlike the mongo-app-deployment that contains just the deployment definition file that consists of the image being mongodb, the containerport, referencing the secrets. The web-app.yaml file consist of a deployment file which has the configmap configured and the service with a type "LoadBalancer" that is used to communicate with the outside world with a targetport of 80.

apiVersion: apps/v1
kind: Deployment
metadata:
  name: webapp-deployment
  labels:
    app: webapp
spec:
  replicas: 1
  selector:
    matchLabels:
      app: webapp
  template:
    metadata:
      labels:
        app: webapp
    spec:
      containers:
      - name: mongo-express
        image: mongo-express:1.0.2-20
        ports:
        - containerPort: 8081
        env:
        # Define the environment variable
        - name: ME_CONFIG_MONGODB_ADMINUSERNAME
          valueFrom:
            secretKeyRef:
              # The ConfigMap containing the value you want to assign to SPECIAL_LEVEL_KEY
              name: mongo-secret
              key: mongo-user
        - name: ME_CONFIG_MONGODB_ADMINPASSWORD
          valueFrom:
            secretKeyRef:
              # The ConfigMap containing the value you want to assign to SPECIAL_LEVEL_KEY
              name: mongo-secret
              key: mongo-password
        - name: ME_CONFIG_MONGODB_SERVER
          valueFrom:
            configMapKeyRef:
              # The ConfigMap containing the value you want to assign to SPECIAL_LEVEL_KEY
              name: mongo-config
              key: mongo-url

---

apiVersion: v1
kind: Service
metadata:
  name: webapp-service
spec:
  type: LoadBalancer
  selector:
    app: webapp
  ports:
    - protocol: TCP
      port: 80
      targetPort: 80

