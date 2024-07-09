# Kubernetes

## Core Concepts:
- Architecture:
  - Worker nodes: Host application as containers
  - Master nodes: Responsible for managing, planning, scheduling and monitoring worker nodes.
    - It does this with the help of components together known as *control plane components*

  - __Control Plane Components:__
    1. ETCD cluster:
      - ETCD is a database that store information in a key:value format
    2. Kube scheduler:
      - Identifies the right node to place a container on, based on the container's resource requirements, worker node's capacity, or any other policies or constraints such as taints and tolerations or node affinity that are on them
    3. Controller Manager
      - It is the process that manages all the different controllers in a cluster.
      - In K8S, a controller is a process that continuously monitors the state of various components within the system and works towards bringing the whole system to the desired functioning state.
        - Node controller:
          - Responsible for onboarding new nodes to the cluster and handling the situations where a node becomes unavailable/ destroyed
          - It checks the status of the nodes and takes necessary actions to keep the application running	
        - Replication controller- monitors the state of replicasets and ensures that the desired number of containers are running at all times within the set/replication group. If a pod dies, it creates another one.
        
    4. Kube-apiserver: 
      - Primary management component of the cluster
      - It is responsible for orchestrating all operations within the cluster
      - It exposes the kubernetes API which is used by external users to perform management operations on the cluster, as well as the various controllers to monitor the state of the cluster and make the necessary changes as required, and by the worker nodes to communicate with the server
  
  - __Components on all nodes:__
    - Container Runtime Engine
      - The software that runs the containers
    - Kubelet
      - An agent that runs on each node in a cluster.
      - It listens for instructions from the API-server and deploys or destroys containers on the nodes as required
      - The API-Server periodically fetches status reports from the kubelet to monitor the status of nodes and containers on them
    - Kube-proxy
      - Communication between worker nodes is enabled by the kube-proxy service
      - It ensures that the necessary rules are in place on the worker nodes to allow containers running on them to reach each other

- __ETCD__
  - The ETCD data store stores regarding the cluster configuration (stores all the information about the cluster)- nodes, pods, configs, secrets, accounts, roles, bindings, etc.
  - Port 2379
  - The `etcdctl` client is a command line client for ETCD
  - Every change you make to your cluster are updated in the etcd server. Only once it is updated in the etcd server is the change considered to be complete

- __Kube API server__
  - When using the kubectl command it reaches out to the api-server. the API server then authenticates the request and validates it.
  - It then retrieves the data from the ETCD cluster and responds back with the requested information
  - eg. if trying to create a pod:
    - The Api server creates a pod object without assigning it to a node. It updates the information in the ETCD server, then updates the user that the pod has been creates
    - The scheduler continuously monitors the API server and realizes that there is a new pod with no node assigned.
    - The scheduler identifies the right node to place the new pod on and communicated that back to the api-server
    - The API server then updates the information in the etcd cluster
    - The API server then passes that information to the kubelet in the appropriate worker node
    - The kubelet then creates the pod on the node and instructs the container runtime engine to deploy the application image.
    - The Kubelet then updates the status back to the API server, and the API server then upates the data back in the ETCD server
  - A similar pattern in done every time a change is requested, because the API server is in the middle of every task that needs to be performed to make a change in the cluster
  - Kube API server is the only component that interacts directly with the ETCD data store
  - The other components use the API server to perform updates in the cluster

- __Kube Controller Manager__
  - Node controller:
    - The node controller is responsible monitoring the status of the nodes and taking necessary actions to keep the applications running, through the API server
    - It checks the status of the nodes every five seconds
      - *Node monitor period* every 5s, 
        - *Node Monitor Grace period* (waits 40s before marking unreachable) 
        - *Pod Eviction Timeout* (waits 5 minutes for it to come back up after it has been deemed unreacheable)
          - If it does not come back up, it removes the pods scheduled on that node and provisions them on the healthy nodes if they are part of a replica set.
  - Replication Controller:
    - Responsible for monitoring the status of replica sets and ensuring that the desired number of pods are available at all times within the sets. if a pod dies, it creates another one.
  - Other controllers: Service account controller, deployment controller, namespace controller, endpoint controller, job controller, PV- protection controller, PV-Binder controller, cronJob, statefulSet, replicaSet, etc
  - Display commands: `kubectl get pods -n kube-system` 
    - `ps -aux | grep kube-controller-manager`
  - By default all controllers are enabled but you can choose to enable a select few

- __Kube Scheduler__
  - The scheduler is only responsible for deciding which pod goes on which node and does not actually place the pod on the node; that is the job of the kubelet.
  - Scheduler decides where the pods are based on certain criteria.
  - Phases:
    - Filter nodes. e.g. on CPU and memory resources requested by the pod
    - Rank nodes:
      - Uses a priority function to assign a score to the nodes on a scale of 1-10
  - ** More covered in *Scheduling*
  
- __Kubelet__
  - It registers the node with a kubenetes cluster
  - When it receives instruction to load a container or a pod on the node, it requests the container runtime engine to pull the required image and run an instance
  - The kubelet then continues to monitor the status of the pod and containers in it and reports to the API server on a timely basis

- __Kube-proxy__
  - Within a cluster, every pod can reach every other pod. This is accomplished by deploying a pod networking solution to the cluster
  - A pod network is an internal virtual network that spans across all the nodes in the cluster to which all the pods connect to
  - Through this network, they are able to communicate with each other
  - Ensures necessary rules are in place on the worker nodes to allow the containers running on them to reach each other. Eg. An application in node1 being able to reach the database in node2.
  - Looks for new services and each time a new service is created, it creates the appropriate rules on each node to forward traffic from those services to the back-end pod eg. ip-tables rule… in this case, it creates an IP table rule on each node in the cluster to forward traffic heading to the IP of the service to the IP of the actual pod

-