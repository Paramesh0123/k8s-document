# k8s-document

Labels & Selectors:
-------------------------

	1. Labels are key/value pairs that are attached to objects, such as pods.
	2. Selectors allows us to filter objects based on labels. For example, list all the objects which has label where env: dev
	3. To label the pod, use: kubectl label pod <pod-name> <label-key>:<label-value>
	4. To list all the labels assigned to pods, use: kubectl get pods --show-labels
	5. To list the pods with specific label attached, use: kubectl get pods -l <label-key>:<label-value>
	6. To delete the label for the pods, use: kubectl label pod <pod-name> <key>-
	
	
ReplicaSets:
---------------

	1. A Replicaset purpose is to maintain a stable set of replica pods running at any given time.
	2. Desired state - the state of pods which are desired/defined.
	3. Current state - the actual state of pods which are running.
	4. Replicaset will always try and maintain the desired state is equal to current state.

Replicaset.yml

apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name:webapp-rs
spec:
  replicas: 3
  selector:
    matchLabels:
      app: webapp
  template:
    metadata:
      labels:
        app: webapp
    spec:
      containers:
        - name: webapp-container
          image: rojaparamesh/webapp:v1
          ports:
            - containerPort: 3000

Deployments:
------------------

	1. Deployments provides replication functionality with the help of replicasets, along with various additional capability like rolling out of changes, rollback changes if required.

Deployment.yml

apiVersion: apps/v1
kind: Deployment
metadata:
  name: webapp-deployment
spec:
  replicas: 3
  selector:
    matchLabels:
      app: webapp
  template:
    metadata:
      labels:
        app: webapp
    spec:
      containers:
        - name: webapp-container
          image: rojaparamesh/webapp:v1
          ports:
            - containerPort: 3000

	2. Whenever we upgrade our application to newer version, it will create a revision. This revision helps to track the changes.

Commands:
----------------

	1. Kubectl rollout history deployment <deployment-name>
	2. Kubectl rollout history deployment <deployment-name> --revision <revision-number>  --> to see changes in that particular revision.
	3. Kubectl rollout undo deployment <deployment-name> --to-revision=<revision-number>
	4. Kubectl create deployment webapp-deployment --image=rojaparamesh/webapp:v1  --> It will create the deployment from the CLI.
	5. Kubectl create deployment webapp-deployment --image=rojaparamesh/webapp:v1 --replicas=3  --> It will create the deployment with 3 replicas.
	6. Kubectl set image deployment webapp-deployment <container-name>=<newimage-name> --record  --> it will set the new image for the existing deployment.
	7. Kubectl scale deployment webapp-deployment --replicas=10  --> It will scale the replicas.
	
	
	
Daemonsets:
----------------

	1. A Daemonset can ensures that all the nodes run a copy of a pod.
	2. As nodes are added to the cluster, pods are added to them.
	
	
Daemonset.yml

apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: monitoring-daemonset
spec:
  selector:
    matchLabels:
      app: node-monitoring
  template:
    metadata:
      app: node-monitoring
    spec:
      containers:
        - name: monitoring-container
          image: rojaparamesh/node-exporter:v1
          

NodeSelector:
------------------

	1. NodeSelector allows us to add a constraint about running a pod in a specific worker node.
	2. This can be achieved by adding the labels to the nodes.
	
Nodeselector.yml

apiVersion: v1
kind: Pod
metadata:
  name: webapp
spec:
  containers:
    - name: webapp
      image: rojaparamesh/webapp:v1
      ports:
        - containerPort: 3000
      nodeSelector:
        <label-key>:<label-value>


NodeAffinity:
------------------

	1. Node affinity is a set of rules used by the scheduler to determine where a pod can be placed.
	2. In k8s terms, it is referred as nodeselector and node affinity fields under podspec.
	3. Node affinity is conceptually similar to nodeselector - it allows you to constrain which nodes your pod is eligible to be scheduled on, based on labels on that node. There are 2 types of node affinity:
	
	 a. requiredDuringSchedulingIgnoredDuringExecution
	 b. preferredDuringSchedulingIgnoredDuringExecution
	
Nodeaffinity.yml

apiVersion: v1
kind: Pod
metadata:
  name: node-affinity
spec:
  affinity:
    nodeaffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
          - matchExpressions:
              - key: <label-key>
                operator: In
                values:
                  - <label-value>
  containers:
    - name: webapp
      image: rojaparamesh/webapp:v1
      ports:
        - containerPort: 3000


Resource Limits:
---------------------

	1. If you schedule a large application in a node which has limited resource, then it will soon lead to OOM(out of memory) or others and will lead to downtime.
	2. Requests and limits are two ways in which we can control the amount of resource that can be assigned to a pod(resource like cpu and memory).
	3. Requests - It means that a pod is guaranteed to get a specific amount of RAM or CPU which is defined.
	4. Limits - Make sure that container does not take node resources above a specific value.
	
	
Requests-limits.yml

apiVersion: v1
kind: Pod
metadata:
  name: webapp
spec:
  containers:
    - name: webapp-container
      image: rojaparamesh/webapp:v1
      ports:
        - containerPort: 3000
      resources:
        requests:
          memory: "64Mi"
          cpu: "0.5"
        limits:
          memory: "128Mi"
          cpu: "1"


Taints & Tolerations:
---------------------------
 
	1. Taints are used to repel the pods from a specific nodes.
	2. In order to enter the taint worker node, you need a special pass. And this pass is called toleration.

Commands:
---------------

	1. Kubectl taint nodes <node-name> <key>=<value>:NoSchedule  --> It will taint the node with some key and value associated with that key.

Tolerations.yml

apiVersion: apps/v1
kind: Deployment
metadata:
  name: webapp-deployment
spec:
  replicas: 5
  selector:
    matchLabels:
      app: webapp
  template:
    metadata:
      app: webapp
    spec:
      containers:
        - name: webapp-container
          image: rojaparamesh/webapp:v1
          ports: 
            - containerPort: 3000
          tolerations:
            - key: <key>
              operator: "Exists"
              effect: "NoSchedule"

	2. Kubectl apply -f tolerations.yml
	3. NoSchedule - New pods that do not match the taint are not scheduled on that node. Existing pods on the node remain.
	4. PreferNoSchedule - New pods that do not match the taint might be scheduled onto that node, but the scheduler tries not to. Existing pods on the node remain.
	5. NoExecute - New pods that do not match the taint cannot be scheduled  on to that node. Existing pods on the node that do not have a matching toleration are removed.
	
	
Services:
------------

	1. K8s service can act as an abstraction which can provide a single IP address and DNS through which pods can be accessed.
	2. Service is a gateway that distributes the incoming traffic between its endpoints.
	3. Endpoints are the underlying pods to which the traffic will be routed to.


ClusterIP:
------------

	1. Whenever service type is clusterIP, an internal clusterIP address is assigned to the service.
	2. Since an internal clusterip is assigned, it can only be reachable from within the cluster.
	3. This is the default service type.

NodePort:
-------------

	1. NodePort exposes the service on each node's IP at a static port(NodePort).
	2. You will be able to contact the nodeport service from outside the cluster by requesting <worker-ip>:<nodeport>.
	3. If the service type is NodePort, then k8s will allocate a port (default: 30000-32767) on every worker node.

Commands:
---------------

	1. Kubectl run nodeport-pod --labels="type=publicpod" --image=nginx
	2. Kubectl get pods --show-labels
	3. Now, create a nodeport service using yaml:

Nodeport-service.yml

apiVersion: v1
kind: Service
metadata:
  name: nodeport-service
spec:
  selector:
    type: publicpod
  type: NodePort
  ports:
    - port: 3000
      targetPort: 3000


LoadBalancer:
------------------

	1. LoadBalancer service type will automatically deploy an external load balancer.
	2. This load balancer takes care of routing requests to the underlying service.


Commands:
---------------

	1. Kubectl run lb-pod --labels="type=loadbalancer" --image=nginx
	2. Kubectl get pods --show-labels
	3. Now, create a LoadBalancer service yaml file like:

Loadbalancer-service.yml

apiVersion: v1
kind: Service
metadata:
  name: lb-service
spec:
  type: LoadBalancer
  selector:
    type: loadbalancer
  ports:
    - port: 3000
      targetPort: 3000

	4. Once you apply the service yaml file changes, it will automatically create a loadbalancer In the cloud. If you are using bare metal, you need to create your own load balancer.


Ingress:
----------

	1. Whenever making use of LoadBalancer service type, out of box, you can make use of a single website.
	2. K8s ingress is a collection of routing rules which governs how external users access the services running within the k8s cluster.
	3. Ingress can provide various features which includes:
	 a. Load Balancing
	 b. SSL termination
	 c. Name based virtual hosting
	4. There are two sub-components when we discuss about ingress:
	 a. Ingress Resource
	 b. Ingress controllers
	5. Ingress resource contains set of routing rules based on which traffic is routed to a service.
	6. Ingress controller takes care of the layer 7 proxy to implement ingress rules.
	7. You must have an ingress controller to satisfy an ingress. Only creating an ingress resource has no effect.


Name-based virtual hosting:
-------------------------------------

	1. Name based virtual hosts supports routing HTTP traffic to multiple host names at the same IP address.

Ingress.yml

apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: name-based-virtual hosting
spec:
  rules:
    - host: jenkins.website
      http:
        paths:
          - pathType: Prefix
            path: "/"
            backend:
              service:
                name: jenkins-service
                port:
                  number: 8080

	2. Now create the pods and services to see whether the ingress is working or not.


Deploying Ingress Controller:
--------------------------------------

Please look into the documentation and deploy nginx ingress controller.


Namespaces:
------------------

	1. Grouping of all the same resources to one place is known as namespaces.

Commands:
----------------

	1. Kubectl get namespace --> It will list all the existing namespaces.
	2. Kubectl create namespace <namespace-name>  --> It will create a new namespace.
	3. Kubectl run nginx --image=nginx --namespace <namespace-name> --> It will create pods under namespace.
	4. Kubectl get pods -n <namespace-name>  --> It will list all the pods under the namespace.


Service Accounts:
-----------------------

	1. K8s cluster have 2 types of accounts.
	 a. User Accounts (for Humans)
	 b. Service Account (For Applications)
	2. To connect to a k8s cluster, an entity needs to have some kind of authentication credentials.
	3. Various different components of k8s uses service accounts to communicate with the cluster. To see them, run: kubectl get serviceaccounts --all-namespaces
	4. Let's assume that a service account  is associated with pod A.
	5. Pod A can use the token associated with the service account to perform some actions on k8s cluster.
	6. The token is seen in the path - /var/run/secrets/kubernetes.io/serviceaccount/token
	7. When you create a cluster, k8s automatically creates a serviceaccount object named "default" for every namespace in your cluster.
	8. Each service account in k8s can be associated with certain permissions. You can assign permissions on what the token associated with the service account is allowed to do.
	9. The default service accounts in each namespace get no permissions by default other than the default API discovery permissions that k8s grants to all authenticated principals if RBAC is enabled.
	10. If you deploy a pod in a namespace and you don't manually assign a service account to the pod, k8s assigns the default service account for that namespace to the pod.
	11. You can use the "serviceAccountName" to specify custom service account token as part of the pod.
	12. To create a service account , use: kubectl create sa <sa-name>
	13. To list the service accounts, use: kubectl get sa


apiVersion: v1
kind: Pod
metadata:
  name: pod-sa
spec:
  serviceAccountName: <sa-name>
  containers:
    - name: Jenkins
      image: rojaparamesh/Jenkins:v1
      ports:
        - containerPort: 8080


Understanding Authentication:
-----------------------------------------
