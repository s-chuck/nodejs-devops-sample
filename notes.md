<!-- 
1. Replicaset: it is like saying that at any given time this many replicas should be running of our app.
2. apiVersrion: It's like a api version that we have to specify in the top of the file of any .yaml before making any request to k8s, as it's checked by APISERVER there.
3. Kind: it's also tells that what we want to do with this particular file, like the purpose of it.
4. metadata: from the name itself it contains the metadata of our file , metadata.name -> is like a primary key for our file in the ETCD. metadata.labels are identifiers(assume them as where clause in db) select * from pods where labels.name = "";
    metadata: {
        name: "nginx-123",  // Primary key (unique)
        labels: {           // Queryable tags
        app: "nginx",
        env: "prod"
        } 
    we can make our own custom labels too for ex. team: frontend-team4 like that.
5.Spec: It is like a blueprint that tells how things should behave.It's a place where we tell k8s what to create , of what type, with how much memory and everything related to.
6.Namspace: A namespace is a virtual cluster within a physical Kubernetes cluster, It is defined under metadata with key=namespace and value=(namespace name) before this we also have to create a namespace.yaml file and execute it first "kubectl create -f namespace.yaml" -> APISERVER -> ETCD (entry done) then we can use that namespace, it also provide logical isolation for:
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
-->