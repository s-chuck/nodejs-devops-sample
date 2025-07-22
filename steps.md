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

#kubernetes Architecture
**********Master_Node(ApiServer, Controller manager, Schedular , ETCD) -> Worker Node(kubectl, kuber-proxy => pods)***********
Visual: [ Your Host Laptop ]
     ↓
[ Minikube VM ]  ← Created via `minikube start`
    ├─ Control Plane (API server, scheduler, etc.)
    └─ Worker Node
         ├─ Pod 1 (runs container: your app)
         ├─ Pod 2 (runs container: your app)
         └─ Pod N ...
1. We have something called Master Node or Control Panel 
=> it consist of APISERVER(any request from client first comes here) -> it sends it to SCHEDULAR(saying we have this request schedule this to a node) -> At the same time APISERVER sends that request to CONTROLLER MANAGER(it have many controllers liek deployment controller , ReplicaSet controller) it overlooks all the objects using individual controllers and compares their actual state with the desired state if it's same then nothing it's working fine if not it takes action. -> Now the last thing in Control Panel is ETCD(it's a nosql database) store key value pairs all the info about nodes, pods, client requests etc.

2. Now we have Worker Nodes
=> Every worker node have kubectl and kube-proxy the kube-proxy allows inter communication b/w pods inside the same worker node and kubectl gets the request from control panel and execute them and sends the response to apiserver.Where the response is checked by Controller manager and the process goes on

EX. Now if a user says to create a pod , the user will use kubectl client(a client which is basically used to talk to our k8s application) after that ApiServer will receive the request it authenticate and validate the request if it's came from a valid client then sends an entry to ETCD to make a create pod entry , after that sends the instruction of creating a pod to schedular to find a node to create the pod , the schedular will find a node and sends it's details to apiserver then apiserver will contact the worker node and tell them to create the pod after receiving the response it again sends that detail to ETCD to store when this pod is created and where and then responds to user.

                    
# Now let's understand the file and file system in k8s
k8s uses yaml files to describe what you want to do with cluster.
-> Each .yaml file that we write is known as k8s objects. Objets are nothing but a blueprint that tells k8s what to do. Below is the list of most commonly used objects in K8s

->Deployment	               Tells K8s what app to run and how many copies (pods)
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deploy(name is given if we have to call this in future -> kubectl port-forward svc/nodejs-service) refer to service.yaml. in this command we are calling svc(service)/nodejs-service(with it's name specified in metadata)
spec:
  replicas: 3
-> These above 4 are the most imp. fields apiVersion, kind, metadata, spec.
-> In yaml sometimes we use (-) it is to tell that below these components are list.
->Service	                   Gives your app a stable IP or DNS name so others can access it
Here we configure the port in which the application works inside the container and if we want to acess the same thing we have to do port mapping. Just like we do in docker as container are whole new and insolated env. so whatever inside it doesn't affect the outside world. kubectl port-forward svc/nodejs-service 3000:80(3000 is the outside port we want to forward the container results.)
localhost:3000 → service (port 80) → pod (port 5000)

ports:
    - protocol: TCP
      port: 80
      targetPort: 5000
Here what is happening is port 80 is the port through which we can talk to k8s cluster the outside world and 5000 is the port which runs inside the container. 
->Namespace	               Organizes resources into separate folders or teams
->Ingress	                   Exposes your app to the internet, like setting up a gate
->ConfigMap/Secret	       Configuration or credentials for your app


Some of the most imp. commands
kubectl apply -f <file.yaml>          # Create or update resource
kubectl delete -f <file.yaml>         # Delete resource
kubectl create namespace <name>       # Create a namespace
kubectl get pods                      # List pods
kubectl get deployments               # List deployments
kubectl get services                  # List services
kubectl get namespaces                # List namespaces
kubectl get all                       # Show all resources in current namespace
kubectl describe pod <pod-name>       # Show detailed info (logs, events)
kubectl logs <pod-name>               # View pod logs
kubectl exec -it <pod-name> -- bash   # Enter running container
kubectl get ns                        # List all namespaces
kubectl config set-context --current --namespace=<ns>  # Switch current namespace
kubectl config get-contexts
kubectl config use-context <context-name>
kubectl config view

 -->


<!-- 
Step 5: Helm: Helm is like a package manager for k8s , we have to manually deploy each object in k8s using helm we just have to run one command and all done. Ex. apt in linux or choco in windows , helm is like for k8s.

-> Just like docker we upload our images in Helm we have something called charts(a chart is like set of k8s files bundled together.) in Artifacthub.io here all the charts are present and the running instance of that chart is known as Release.

we have to install helm and set it's path in env. variables now we are all set to go

-> Next step is to create a helm chart "helm create helm-nodejs" after that we would get a new directory in our root directory of helm-nodejs name it contains all the files and pre written code we don't have to make any changes into any of those files just to the values.yaml and Chart.yaml and all the other files extract the values from those so no need to do manually update values in all of those.After updateing our files in templates folder,(helm install nodejs-app(release name) ./helm-nodejs) we now have to deploy our helm chart into our k8 cluster. When a chart is up and running it's called release.
 -->

<!-- 
Step 6: Ingres(Ingress is an API object that manages external access to services in a cluster, typically over HTTP/HTTPS.)
In simple terms whatever service is running inside your k8s cluster we have to do kubectl port-forward 5000:80 so that this service can be acessed outside the browser.
-> Think of it as a smart router that exposes your internal services to the outside world.
-> It provides rules for routing external traffic to the right service and path.
-> Ingress works with an Ingress Controller (like NGINX) to manage actual traffic flow.

EX. Suppose your website has this url for your website now ingres doesn't change the url or domain name all of this would be same just the thing is we would be able to acess whatever is inside the cluster.
Home Page: https://fun.com/home

Contact Page: https://fun.com/contact_us

API: https://fun.com/api

 -->

 <!-- 
 Step 7: Argo CD(Argo CD is a declarative GitOps continuous delivery tool for Kubernetes.
It continuously monitors a Git repository and automatically applies the manifests (or Helm charts) to a Kubernetes cluster, ensuring the cluster state matches the Git-defined state.)
  1. Install ArgoCD
    kubectl create namespace argocd
    kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
  2. acess ArgoCD using
    kubectl port-forward svc/argocd-server -n argocd 8080:443
  3. Get login credentials
    kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d && echo
  4. Configure the following details in the ArgoCD app.
    Click “NEW APP”, then fill in:
    App Name: nodejs-app
    Project: default
    Sync Policy: Manual or Auto (your choice)
    Repo URL: your GitHub repo (must be public or connect private)
    Path: path to Helm chart directory (e.g. helm-nodejs)
    Cluster URL: https://kubernetes.default.svc
    Namespace: default
    Create.
  -->