<!-- This is Nothing but my way to document whatever i am doing. -->


<!-- Step 1 -->
<!-- 
# Created a Dockerfile.
# Gonna use multi stage in this as building an app in one container and then run in another cleaner smaller container.
# Build Stage -> Compile, install dependencies, bundle code.
# Production Stage -> Running the app in clean env. 
# How to write a dockerFile 
FROM node:20-alpine AS builder (image that we have to use builder is the name of the stage.)
WORKDIR /app(it's like saying cd into this and from now on whatever is written in dockerfile happens inside this. This directory keeps things in one place inside a container.)

COPY package*.json ./(copying the dependency files to install it later.)
RUN npm ci(clean installation)

COPY . .(copy everything from your local working directory to the container working dire)

#Now the docker file is created , we are shifting our project to docker -> (docker build -t nodejs-sample-app .) -> (docker run -p 5000:5000 nodejs-sample-app)
-->



<!-- STEP 2 -->
<!-- 
# Now the next thing is setting up CI pipeline 
# first we have to create this inside .github/workflows/ as we are going to use github actions.
# After that let's come to the syntax why are we writing what we are writing.
name: CI Pipeline(name of our workflow it appears in the github actions workflow too.)

on:(The name is itself valid it means that execute this when we are pushing to main branch or pulling from there.)
  push:
    branches: [ main ]
  pull_request:
    branches: [ main ]

jobs:(jobs are something like set of steps in a workflow that is executed on a same runner. Jobs run in parallel by default.)
  build-and-test:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout Code
        uses: actions/checkout@v4(we can find these from github marketplace. Checkout is used to extract code from repo and pasting it on runner.)

      - name: Set up Node.js
        uses: actions/setup-node@v4(once the code is extracted using checkout then setup node js)
        with:
          node-version: 20

      - name: Install dependencies
        run: npm ci

      - name: Run basic lint or test (optional)
        run: echo "No tests configured, skipping for now"

      - name: Build Docker image
        run: docker build -t nodejs-sample-app .

 -->


 <!-- Step 3 -->
 <!-- 
 #Now we have to find a way to build and upload our image in docker hub this step is important as later we need to pull this image from docker to use it in k8s and helm.
 #Before that we have to save our docker hub username and pwd in our github actions secrets : settings -> secret and variables -> Actions -> New repo secret , it is essential so that we don't enter secrets explicitly.
 #Let's come to code now
        - name: Login to docker hub
          uses: docker/login-action@v3(can easily find in github marketplace)
          with:
            username: ${{ secrets.DOCKER_USERNAME }}(retreiving username and pwd from secrets.)
            password: ${{ secrets.DOCKER_PASSWORD }}
        
        - name: Build and Push Image to docker(Building the image and pushing it using github action libraries.)
          uses: docker/build-push-action@v6
          with:
            push: true
            tags: |  (This is a trick when we have to use multiple lines put a pipe | it symbolizes we gonna use many lines.just a trick to make code clean.)

                (Now why are we using two tags for a same image it's because the latest one is for production and one with github.sha(first of all sha return commit id of the latest commit so) the commit id , if something goes wrong and we have to rollback so we can sort according to the commit id.)            
                chucky12/${{ env.IMAGE_NAME }}:latest 
                chucky12/${{ env.IMAGE_NAME }}:${{ github.sha }}
  -->


<!-- Step 4 -->
<!--
# What is K8s?
-> It's like a manager for your containers it has also many other functionalities, but this one is primary.
Ex. Kubernetes is like:
🧑‍🍳 A restaurant kitchen manager
Makes sure dishes (apps) are cooked (running)
Sends the right dish to the right customer (networking)
If a chef burns out (crashes), replaces them (load balancing)
Can cook more if more customers come (scaling)
Follows a recipe book (YAMLs) to make things consistent

# Some of the features are:
-> zero downtime while deploying new files , that is acheived by rolling updates means it first update one pod then another and keep shifting the traffic from old to new that's how it is acheived.
-> it acts like a traffic manager many pod/replica of the same app is running at the same time if one crashes it transfers that traffic to another healthy pod and replace the unhealthy one.
-> Auto restart your whole app if it crashes.
-> Scale up and Scale down(auto scale)

# What is a Node and a pod -> a pod is nothing but a group of containers working together to run an application, as one app have many things to do that's why we using multi containers(pod) in k8s. Node on the other hand is like a building and pod is like a single appartment, one node consist of many pods.

# What is a k8s cluster -> it's like a group of machine(real or virtual) that works together to run the application.
Now we can setup k8 cluster in our local machine using minikube or kind. 

When you run minikube start, it creates a single VM that behaves like a full Kubernetes cluster, acting as both the control plane (master) and the worker node.
Inside this VM:
You define how many pods should run (via deployment YAML)
Kubernetes runs those pods inside the VM
Each pod is like a wrapper for your containerized app
The control plane monitors and manages these pods automatically
                    
Visual: [ Your Host Laptop ]
     ↓
[ Minikube VM ]  ← Created via `minikube start`
    ├─ Control Plane (API server, scheduler, etc.)
    └─ Worker Node
         ├─ Pod 1 (runs container: your app)
         ├─ Pod 2 (runs container: your app)
         └─ Pod N ...

# Now let's understand the file and file system in k8s
k8s uses yaml files to describe what you want to do with cluster.
-> Each .yaml file that we write is known as k8s objects. Objets are nothing but a blueprint that tells k8s what to do. Below is the list of most commonly used objects in K8s

->Deployment	               Tells K8s what app to run and how many copies (pods)
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deploy
spec:
  replicas: 3

->Service	                   Gives your app a stable IP or DNS name so others can access it
apiVersion: v1
kind: Service
metadata:
  name: nginx-service
spec:
  selector:
  ports:


->Namespace	               Organizes resources into separate folders or teams

->Ingress	                   Exposes your app to the internet, like setting up a gate
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: my-ingress
spec:
  rules:
  
->ConfigMap/Secret	       Configuration or credentials for your app
apiVersion: v1                                        
kind: ConfigMap
metadata:
  name: app-config
data:
  DB_URL: "postgres://db:5432"

apiVersion: v1
kind: Secret
metadata:
  name: db-secret
type: Opaque
data:
  DB_PASSWORD: cGFzc3dvcmQxMjM=  # "password123" in base64

 -->