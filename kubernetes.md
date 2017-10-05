---
layout: page-disqus
title: Kubernetes
permalink: /kubernetes/
published: true
---

Container - A system process

Docker - Here are my strong opinions on how you should build containment around your system process so it can best be isolated from other running proceses and from the host

Image - I will not depend on the host, I will bring all my dependencies with me

Kubernetes - I've got a lot of images, I think I need a bit of help managing them.  Here's my desired state, please enforce it

Let's take a stab at setting up a Kubernetes cluster
```
Item          | AWS                          | GCP
----          | ---                          | ---
Setup cluster | kops - Kubernetes Operations | gcloud
```

### Configuring tools to start a k8s cluster on AWS

For this exercise, let's use an Ubuntu Workstation - 16.04 LTS

  * Install kubectl - [https://kubernetes.io/docs/tasks/tools/install-kubectl/](https://kubernetes.io/docs/tasks/tools/install-kubectl/){:target="_blank"}

  * Install kops - [https://github.com/kubernetes/kops](https://github.com/kubernetes/kops){:target="_blank"}
  
  * Install AWS CLI - [https://aws.amazon.com/cli/](https://aws.amazon.com/cli){:target="_blank"}
  
  * Generate ssh keys to be used when loggin into the cluster - will utilize **ssh-keygen**
  
On AWS:

  * Configure a IAM user for kops.  Will attach the following policies (via a group):
  
    * AmazonEC2FullAccess
    * IAMFullAccess
    * AmazonS3FullAccess
    * AmazonVPCFullAccess
    * AmazonRoute53FullAccess
    
  * Create a S3 bucket to contain kops state
  
  * Configure a kubernetes subdomain via Route 53.  We will use **kubernetes.domainName.com**
  
### Using kops to create the cluster

To preview the changes to be made on AWS:

```kops create cluster --name=<clusterName> --state=s3://<bucket-name> --zones=us-east-1a --node-count=2 --node-size=t2.micro --master-size=t2.micro --dns-zone=kubernetes.<domain>.com --ssh-public-key=<publicKey>```

#### Interesting Options
  * Build a terraform manifest with ```--target=terraform``` then use **terraform apply** to apply the configuration
  
  * Build a private network topology with ```--topology private```

Finally configure your cluster with: ```kops update cluster <clusterName> --yes```

To validate cluster components: ```kops validate cluster```

And, to eventually delete the cluster: ```kops delete cluster --name=<clusterName>```


### Kebernetes

A typical Master + Worker Configuration

  * Master Server(s) - Manages the cluster
    * Etcd - key value store containing tbe cluster configuration
    * API server - RESTful interface
    * Scheduler - finds best fit (node) for pods
    * Controller Manager - cluster task manager
  * node - machine running kubelet
    * kubelet - node agent running on each node - I see job, I run job - Interacts with Docker
    * kube-proxy - node network proxy - manages iptables (FW) which forwards traffic to pods - NAT
  * pod - n containers (logical unit running on a node)
  * service - vip for pods (think load balancer) - forwards traffic to iptables
  * replication controller/replica set/deployment - templates for desired state
    
  
### Docker Hub

Let's push our [docker image](/blog/itNow#express) to Docker Hub to make it available to kubernetes


### Launching container (app) on Kubernetes

Create a pod definition
```
apiVersion: v1
kind: Pod
metadata:
  name: hvag-ninjas.hvag.com
  labels:
    app: hvag-ninjas
spec:
  containers:
  - name: hvag-ninjas
    image: markshaw/hvag-ninjas-express-mongo:1
    ports:
    - name: nodejs-port
      containerPort: 3000
```

Create a service definition
```
apiVersion: v1
kind: Service
metadata:
  name: hvag-ninjas-service
spec:
  ports:
  - port: 80
    targetPort: nodejs-port
    protocol: TCP
  selector:
    app: hvag-ninjas
  type: LoadBalancer
```

Use kubectl to create the pod and service on the cluster
```
kubectl create -f hvagNinjas-pod.yml
kubectl create -f hvagNinjas-service.yml
```

We should now be able to reach the node application via the DNS name for the load balancer created in AWS

We can also create an Alias record in Route 53 which targets the load balancer

**Traffic Flow**
  * DNS Address
    * Route 53 -> Load Balancer
      * Load balancer redirects port 80 traffic to one of the k8s cluster nodes
        * Node accepts traffic on instance port (NodePort) and redirects to pod
          * Pod is running node/express app which responds with message

**Teardown**
```
kubectl delete -f hvagNinjas-service.yml
kubectl delete -f hvagNinjas-pod.yml
```


### Deployment

Let's revisit the launch above utilizing a **Deployment**

With the Deployment, we will define the desired state of our application and depend on Kubernetes to maintain that desired state within the cluster

We are running the same image as above but have requested that the cluster maintain 3 running replicas

```
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  name: hvag-ninjas-deployment
spec:
  replicas: 3
  revisionHistoryLimit: 10
  template:
    metadata:
      labels:
        app: hvag-ninjas
    spec:
      containers:
      - name: hvag-ninjas-container
        image: markshaw/hvag-ninjas-express-mongo:1
        ports:
        - name: nodejs-port
          containerPort: 3000
```

Create/Run the Deployment

```
kubectl create -f hvagNinjas-deployment.yml --record
```

We use the --record option to keep additional historical info of the changes to the Deployment over time

Let's "expose" the deployment, making it visible on the network

```
kubectl expose deployment hvag-ninjas-deployment --port=80 --target-port=nodejs-port --name hvag-ninjas-service --type=LoadBalancer
```

This is the same as previously running 
```
kubectl create -f hvagNinjas-service.yml
```
Why?  Because in the end, it's all a call to the Kubernetes API

#### Update Deployment Image

Let's update the image being used by our deployment to a new version

```
kubectl set image deployment hvag-ninjas-deployment hvag-ninjas-container=markshaw/hvag-ninjas-express-mongo:2
```

That was easy!

You can view the history of your deployment 
```
kubectl rollout history deployment hvag-ninjas-deployment
```

It's also easy to 'rollback' to the previous deployment.  Let's undo what we just did

```
kubectl rollout undo deployment hvag-ninjas-deployment
```

You are able to rollback to a specific prior revision via --to-revision=#


#### Scaling Up/Down

Let's increase the number of replicas to 4

```
kubectl scale deployment hvag-ninjas-deployment --replicas=4
```

Perfroming 'kubectl get pods' will show the increased POD count

```
NAME                                      READY     STATUS    RESTARTS   AGE
hvag-ninjas-deployment-3218787200-bjwkj   1/1       Running   0          6m
hvag-ninjas-deployment-3218787200-brs2x   1/1       Running   0          6m
hvag-ninjas-deployment-3218787200-v6t4w   1/1       Running   0          6m
hvag-ninjas-deployment-3218787200-w8lgd   1/1       Running   0          43s
```

Note that throughout this exercise the service has been available; from a client perspective, there was no downtime.


#### Health Check

We can add a health check to our Deployment.  This will perform a HTTP GET to the application running in the containers in the pod.  Kubernetes will terminate the pod and relaunch if there is an issue with the application

livenessProbe (health check)

```
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  name: hvag-ninjas-deployment
spec:
  replicas: 3
  revisionHistoryLimit: 10
  template:
    metadata:
      labels:
        app: hvag-ninjas
    spec:
      containers:
      - name: hvag-ninjas-container
        image: markshaw/hvag-ninjas-express-mongo:1
        livenessProbe:
          httpGet:
            path: /api
            port: nodejs-port
          initialDelaySeconds: 15
          timeoutSeconds: 30
        ports:
        - name: nodejs-port
          containerPort: 3000
```

### Secrets

Our application utilizes a MongoDB back-end.  Let's attempt to use a [Kubernetes Secret](https://kubernetes.io/docs/concepts/configuration/secret){:target="_blank"} to store our DB login credentials

Time for a [running list of security concerns and mitigation steps](/blog/security#kubernetes)

Once the "Secret" object has been created, it can be referenced by the PODS.  We will utilize the option for using Secrets as environment variables within the containers

Create a secret definition
```
apiVersion: v1
kind: Secret
metadata:
  name: mongodb-secrets
type: Opaque
data:
  username: <uname here - base64>
  password: <pwd here - base64>
```

Modify the Deployment definition to reference the Secret (add env:)
```
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  name: hvag-ninjas-deployment
spec:
  replicas: 3
  revisionHistoryLimit: 10
  template:
    metadata:
      labels:
        app: hvag-ninjas
    spec:
      containers:
      - name: hvag-ninjas-container
        image: markshaw/hvag-ninjas-express-mongo:1
        livenessProbe:
          httpGet:
            path: /api
            port: nodejs-port
          initialDelaySeconds: 15
          timeoutSeconds: 30
        ports:
        - name: nodejs-port
          containerPort: 3000
        env:
        - name: MONGODB_USERNAME
          valueFrom:
            secretKeyRef:
              name: mongodb-secrets
              key: username
        - name: MONGODB_PASSWORD
          valueFrom:
            secretKeyRef:
              name: mongodb-secrets
              key: password
```

Create the Secret
```
kubectl create -f hvagNinjas-secret.yml
```


Create the Deployment
```
kubectl create -f hvagNinjas-deployment.yml
```

If you run 'kubectl get pod pod-name' you will see that the environment variables have been set
```
Environment:
      MONGODB_USERNAME:	<set to the key 'username' in secret 'mongodb-secrets'>	Optional: false
      MONGODB_PASSWORD:	<set to the key 'password' in secret 'mongodb-secrets'>	Optional: false
```

...




























