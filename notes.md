<!-- 
1. Replicaset: it is like saying that at any given time this many replicas should be running of our app.
2. apiVersrion: It's like a api version that we have to specify in the top of the file of any .yaml before making any request to k8s, as it's checked by APISERVER there.
3. Kind: it's also tells that what we want to do with this particular file, like the purpose of it.
4. metadata: 
        from the name itself it contains the metadata of our file , metadata.name -> is like a primary key for our file in the ETCD. metadata.labels are identifiers(assume them as where clause in db) select * from pods where labels.name = "";
        metadata: {
            name: "nginx-123",  // Primary key (unique)
            labels: {           // Queryable tags
                app: "nginx",
                env: "prod"
            } 
    we can make our own custom labels too for ex. team: frontend-team4 like that.
5.Spec:
     It is like a blueprint that tells how things should behave.It's a place where we tell k8s what to create , of what type, with how much memory and everything related to.
    -> Now let's understand how does spec behave and some of the tags inside it.
    (
    -> First understand this story 
    -> we create labels to individually identify the pods and for one more thing is to create an entry in ETCD for that k8s object saying this object has this label and just we talked above spec is like a blueprint for the k8s, so selector.matchLabels most of the time is similar to the above metadata.label to tell that the pods which have this lable is to be managed by this selector.
    ->Then we have another thing template.metadata.label now this one should be same as selector.matchLabels it is compulsory or either we would get an error. Now in template part we tell how pods should be created with what configurations and it would retreive info from ETCD about this specific label as The Deployment controller watches ETCD for updates and ensures the desired number of pods exist with the matching labels. ETCD doesn’t enforce, but stores, and controllers act on it.
    ->Inside the template till now we have seen the part with metadata and labels and how they should be same with matcHLabels now comes the real configuration part we use another spec: inside the template as it is k8s syntax to define the blueprint inside the spec: tag so we use another spec: containers: to tell from where to find the image and what should be it's name and which port it should run.
    )
    spec:
        replicas: 2(no. of replicas u want of application)
        selector:
            matchLabels:    
                app: nginx 
        Template:
            metadata:
                labels:
                    app: nginx
        spec:
        containers:
            - name: nodejs-app
            image: chucky12/nodejs-sample-app:latest
            ports:
                - containerPort: 5000
    (
    ->Now we have diff spec: in service.yaml let's understand that
    -> we have the above this as same as i won't be talking about the same thing.
    -> Let's talk about why do we even need Service every pod have diff. ip and if a pod crashes or malfunctions CONTROLLER creates a new pod with diff ip so we can't rely on CONTROLLER alone to find out each ip and then connect it, that's why we use one centralized method called Service it's like all tarffic goes to Service first then to pods.
        Client request,All traffic -> Service() -> {pod1, pod2, pod3}.
    ->That's why spec.selector.matchLabel have the label in which we want to apply that service.
    -> Now we need a doorway for client request to come in Service that is defined inside ports.
    -> Inside ports we have what protocol is being used in this Service and what would be the port which is used for listening the user request and teh targetPort. And as we know targetPort(Service.yaml) = Conatinerport(deployment.yaml) is same for all the replicas and the pods so all 3 pods have one container inside them which are running application on 5000 port and as service finds suitable as per load balancing it assigns the request to one of the pods.
    -> The type is like saying who can acess the service or where to expose this service , ClusterIP refers to give acess to other pods inside the cluster.There are others too read about them later.
    )
    spec:
    selector:
        app: nodejs-app
    ports:
        - protocol: TCP
        port: 80
        targetPort: 5000
    type: ClusterIP
6.Namspace: 
    A namespace is a virtual cluster within a physical Kubernetes cluster, It is defined under metadata with key=namespace and value=(namespace name) before this we also have to create a namespace.yaml file and execute it first "kubectl create -f namespace.yaml" -> APISERVER -> ETCD (entry done) then we can use that namespace, it also provide logical isolation for:
    Resources (Pods, Deployments, Services, etc.), Access Control (RBAC), Network Policies, Resource Allocation
    Key Uses of Namespaces
    Purpose	Description	Example
        1. Avoid Naming Conflicts	Resources with the same name can coexist in different namespaces.	dev/nginx and prod/nginx can both exist.
        2. Resource Isolation	Group resources by environment (dev, prod) or team (team-a, team-b).	kubectl get pods -n dev shows only Dev pods.
        3. Access Control (RBAC)	Restrict users/teams to specific namespaces.	Developers get edit role in dev but only view in prod.
        4. Resource Quotas	Limit CPU/memory/storage per namespace.	prod gets 16 CPUs, dev gets 4 CPUs.
        5. Network Segmentation	Control traffic between namespaces using NetworkPolicy.	Block test namespace from accessing prod.
        6. Environment Separation	Isolate configurations (e.g., dev uses test DB, prod uses HA DB).	configmap-db differs in dev vs. prod.
        7. Multi-Tenancy	Host multiple clients/teams securely in one cluster.	client-a and client-b have separate namespaces.
        8. CI/CD Pipelines	Dedicated namespaces for stages (build, stage, prod).	ci namespace runs build pods; prod runs live apps.
        9. Billing/Chargeback	Track resource usage per namespace for cost allocation.	kubectl top pods -n team-a shows Team A’s resource consumption.
7.
-->