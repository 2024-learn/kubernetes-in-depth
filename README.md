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
        - Other controllers: Service account controller, deployment controller, namespace controller, endpoint controller, job controller, PV- protection controller, PV-Binder controller, cronJob, statefulSet, replicaSet, etc
        - View options: `kubectl get pods -n kube-system; ps -aux | grep kube-controller-manager`
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
  
  - __Worker node components__
    - Kube-proxy
      - Communication between worker nodes is enabled by the kube-proxy service
      - It ensures that the necessary rules are in place on the worker nodes to allow containers running on them to reach each other

- __ETCD__
  - Port 2379
  - The `etcdctl` client is a command line client for ETCD
    - You can use it to restore and retrieve key-value pairs
    - `./etcdctl --version`: displays the version of `etcdctl`
    - With the release of version 3.4, the default API version is set to 3
    - To chabge the `etcdctl` to work with the API v3.0, either set the env. variable called etcd_api to 3
      - `ETCDCTL_API=3 ./etcdctl version`
      - Or export as an env.variable
      - ` export ETCDCTL_API=3 ./etcdctl version`
    - With version 3, --version is now a command



  - *Node monitor period* every 5s, 
  - *Node Monitor Grace period* (waits 40s before marking unreachable) 
  - *Pod Eviction Timeout* (waits 5 minutes for it to come back up after it has been deemed unreacheable)
    - If it does not come back up, it removes the pods scheduled on that node and provisions them on the healthy nodes if they are part of a replica set# kubernetes-in-depth
