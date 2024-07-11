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
  - Kube-proxy ensures necessary rules are in place on the worker nodes to allow the containers running on them to reach each other. Eg. An application in node1 being able to reach the database in node2.
  - It looks for new services and each time a new service is created, it creates the appropriate rules on each node to forward traffic from those services to the back-end pod eg. ip-tables rule… in this case, it creates an IP table rule on each node in the cluster to forward traffic heading to the IP of the service to the IP of the actual pod

- __Pods__
  - Smallest building block in k8s. It is the smallest object we can create in Kubernetes
  - It is a single instance of an application
  - There is a 1:1 relationship between pods and containers except where you have helper containers.
  - Helper containers are destroyed at the same time as the pods they are associated with. The life of a helper container is tied to the pod lifecycle.
  - The helper container and the app container communicate with each other using local host since they belong to the same network.

- Yaml Instructions:
  - You can only have name and labels and other Kubernetes acceptable information under metadata, but you can enter as many key:value descriptors under labels
  - Name and labels are children of metadata and should have the same indentation as they are siblings, but more indented than the parent

 
- __ReplicaSets__
  - Controllers are the processes that monitor kubernetes objects and respond accordingly.
  - Replication controller helps us to run multiple instances of a single pod in the k8s cluster, providing high availability
  - You can still use a replication controller with one pod because then the replication controller can help by automatically bringing up a new pod when the old one fails, thus ensuring that the specified number of pods is running at all times.
  - Replication controllers also help us create multiple pods to share the load across them, on different nodes as well as scale our application when the demand increases
  - Replication Controller vs. ReplicaSet
    - Both have the same purpose but they are not the same
    - RC is the older technology being replaced by RS
    - ReplicaSet requires a selector definition.
      - The selector section helps the ReplicaSet identify what pods fall under it
      - This is because RS can also manage and monitor pods that were not created as part of the RS creation. Eg. pods created before the RS that match labels specified in the selector, the replicaset will also take those pods into cosideration when creating replicas.
      - RS can be used to monitor pods that are already existing. In case they were not yet created, the Replicaset will create them for you
    - ** *Note:* RS requires a selector section, while the RC doesn’t. The RC still has one available but does not require it. It just assumes it is the same as the labels provided in the pod definition file
      - In RS, the selector has matchLabels which matches the labels specified under it to the labels on the pod
      - The RS selector also provides more options for matching labels than are available in the RC
    - RS can be used to monitor pods that are already existing. In case they were not yet created, the Replicaset will create them for you.
      - The role of the replicaset is to monitor the pods and if any of them were to fail, to deploy new ones. The Replicaset is a process that monitors the pods.
      - The labels are used as a filter for the replicaset. Under the selector section, the match labels filter has the same labels used while creating the pods, so that the RS knows which pods to monitor
  - Scaling a ReplicaSet
    - Update the number of replicas in the definition file and run `kubectl replace -f definition.yaml` OR  simply run `kubectl edit rs replicaSetName`, edit and save.
    - NB: when a replicaset definition file is updated, the replicaset does not automatically create new pods with the new updates. The existing pods must be deleted before the replicaset can recreate new pods with the updates. To delete all the pods, run `kubectl delete pods --all`
    - `kubectl scale --replicas=## -f definition.yaml`
    - `kubectl scale --replicas=[replicas desired] replicaset [name of RS]`- to scale on the CLI
    - NB: using the filename as an input will not automatically update the number of replicas in the definition file
      - <https://kubernetes.io/docs/reference/kubectl/conventions/>

- __Deployments__
  - Deployments are a kubernetes object that come higher in the hierarchy that replica sets.
  - It provides us with the capability to upgrade the underlying instances seamlessly ie scale/update/replace pods with rolling update, rollbacks, pause and resume options
  - The yaml file is the same as the for the RS except for the kind
  - Once a deployment is created, it automatically creates a replicaset which in turn creates the desired number of pods as defined by the number of replicas
    - `kubectl get deploy`: display deployments created
    - `kubectl get replicaset`: to get the replicaset that is automatically created by the deployment
    - `kubectl get pods`: because the replicaset creates pods
    - `kubectl get all`  to view everything that was created at once

- __Services__
  - They enable comunication between various components within and outside the application
  - They help connect applications together with other applications or users
  - They enable connection between the users and the front-end applications, the front-end apps and backend apps, backend apps and external databases, etc., thus enabling loose coupling within our application
  - Because pods are ephemeral and sometimes get recreated on a different IP, we use a service, which is a permanent IP, so that pods do not lose communication if a pod fails and is recreated.
  - Services help to make applications discoverable within and outside of the cluster.
  - Service is a k8s object. It listens to a port on the node and forwards the requests to a port on the pod running the web application, and maps requests to the node
  - Types of service:
    - __NodePort:__ 
      - Outside of the cluster.  It listens to a port on the node and forwards requests to the pods. It uses the node IP and a static port to expose the container.
      - NOTE: The port on the pod where the container/application is running is known as the TargetPort while the port on the Service is simply known as the Port. 
      - The Service also has its own IP address known as the ClusterIP and the port on the node itself used to access applications in the cluster externally is known as the NodePort (range 30000-32767).
      - In case we have multiple instances of the application (pods):
        - NodePort will forward the requests to all the pods with the matching labels and selectors defined by using a *random algorithm*. 
        - As such it acts as a built-in load balancer to distribute loads across different pods on the same node and across different pods in the cluster
          ```
          Algorithm: Random
          SessionAffinity: Yes
          ```
        - K8s also creates a service that spans all the nodes in the cluster and maps the target port to the same node port on all the nodes in the cluster, making it possible to access your application using the IP of any node in the cluster, using the same port number.
        - When pods are removed/added, the service is automatically updated, making it highly flexible and adaptive.
    - __ClusterIP:__
      - Inside the cluster. The service creates a virtual service within the cluster to enable communication between the services within the cluster. Eg. front-end and back-end
      - It acts as a single interface to group and connect the pods within the cluster
      - The service can be accessed by other pods using the ClusterIP or using the service name
    - __LoadBalancer:__ 
      - Outside of the cluster. 
      - Cloud provides a front-end to our service. Kubernetes has support for integrating with native load balancers from various cloud providers
      - Only on supported clouds like Azure, GCP, AWS
      - If set to LoadBalancer on an unsupported platform, like VirtualBox, then it would have the same effect as setting it to nodePort; it will not do any external load balancer configuration

- __Namespaces__
  - In Kubernetes, this is a logical isolation for resources, or virtual clusters within your k8s cluster so that resources in one isolation do not modify the resources in another
  - Kubernetes creates three namespaces by default:
    - Kube-system: for k8s system for resources that should not be deleted by mistake or interfered with. Scheduling pods on this space could mess up the whole cluster
    - Default: You can create objects in this space but other namespaces are recommended in the production env
    - Kube-public: for resources that should be made available to all users
  - The above three systems should be reserved for system usage
  - Namespace on the CLI:
    `kubectl create ns [name of namespace]`
    `kubectl delete ns [name of namespace]`
    `kubectl get pods --namespace=kube-system` or `kubectl get pods -ns kube-system` ⇒ to list pods in the kube-system namespace
  - A namespace can also be created under the pod metadata section in the manifest file to make sure it gets created under the specified namespace all the time without having to do it on the CLI
  - Creating Namespace object via definition/yaml file:
    ```   
      apiVersion: v1
      kind: Namespace
      metadata:
        namespace: dev
    ```

  - Namespaces can be used to define who can work on certain resources, the resource limits
    - Resources within the same ns can refer to each other by their hostnames
      - Eg. `mysql.connect("db-service")`
    - To connect resources in different namespaces, the namespace name must be appended to the name of the service ie `servicename.namespace.svc.cluster.local` format
      - Eg. `mysql.connect(“db-service.dev.svc.cluster.local”)`
      - `db-service` = Service Name
      - `dev` = Namespace
      - `svc` = subdomain for service
      - `cluster.local` = domain (default domain name of the k8s cluster)
    - When a service is created, a DNS is created automatically in above format
    - In order to move into a namespace permanently, eg. to dev namespace, and not have specify the namespace everytime on the CLI:
      - `kubectl config set-context $(kubectl config current-context) --namespace=dev` or use `kubens <namespace>`
        - *Contexts* are used to manage multiple clusters and multiple environments from the same management system
      - `kubectl get pods --all-namespaces`: displays all the pods in all the namespaces
  - Creating a resource quota (a resource quota helps define/allocate/limit resources in a namespace)
    - Create the resource quota and the namespace you want the RQ created in and then set the limits under spec:
        ```
        apiVersion: v1
        kind: ResourceQuota
        metadata:
          name: compute-quota
          namespace: dev
        spec:
          hard:
            pods: "10"
            requests.cpu: "4"
            requests.memory: 5Gi
            limits.cpu: "10"
            limits.memory: 10Gi
        ```
- __Imperative vs. Declarative__
  - Imperative: Gives step by step directions on how to reach the desired state
    - Run, create, expose, edit, etc 
      - Create objects
        - `kubectl run --image=nginx nginx`
        - `kubectl create deployment --image=nginx nginx`
        - `kubectl expose deployment nginx --port 80`
      - Update objects
        - `kubectl edit deployment nginx`
          - This command generates a yaml file from within the kubernetes memory, similar to the definition file used to create the object but with some additional fields such as the “status” field. You can make and save changes to this file and these changes will be applied to the live object. 
          - However, these changes will not be recorded in the original definition file(hence not recorded anywhere). 
          - Therefore, the kubectl edit command is recommended for use only when you are not going to rely on the object configuration file in the future because any subsequent changes to the object definition file will erase all changes made using the `kubectl edit` command.
          - A better approach is to first edit the object configuration file with the required changes and run `kubectl replace -f definition.yaml`. This approach ensures that the changes made are recorded and can be tracked as part of the change review process.
  - Declarative: This is done in yaml files and then with a single apply command, Kubernetes reads the configuration file and builds the infrastructure to the desired state, either to create, update, scale or bring down the infrastructure.
    - `kubectl apply -f nginx.yaml` OR `kubectl apply -f /path/to/config-files`
  - The `kubectl apply` command:
    - This will create an object if it does not already exist. If there are multiple configuration files in a directory, we can specify the directory path instead of a single file so all the configuration files are applied at once.
    - When the `kubectl apply` command is used to create a k8s object, the manifest file is converted into a json file and stored as the `Last Applied Configuration`. To make any updates on the object, the Local file (original manifest file), Last Applied Configuration (json file) and Live Object Configuration (generated using kubectl edit) are compared to identify what changes are to be made on the live object.
    - The Last Applied Configuration is stored on the Live Object Configuration file on the k8s cluster as an annotation named `kubectl.kubernetes.io/last-applied-configuration`. This section is only generated when an object is created using `kubectl apply`.

## Scheduling
- __Manual Scheduling__
  - Every Pod has a field called the `nodeName` that is not set by default. 
  - This field is automatically added by kubernetes when creating a Pod. 
  - The scheduler is then responsible for scheduling the Pods that do not have this field set.
  - Manual Scheduling: 
    - The scheduler initially goes through the pods to identify those that do not have a “nodeName” indicated(set).
    - These are the pods that need to be scheduled. 
    - The scheduler then runs the scheduling algorithm to find the right node for the pod. 
    - Once it finds the node, it schedules the pod on the node by setting the `nodeName` property to the name of the node - creating a binding object. 
  - Without a scheduler or if there are no nodes with enough resources to support the pod, the pods will remain in a pending state.
    - In that case, the easiest option would be to set the `nodeName` in the manifest file. 
    - This can only be done at creation time and K8s does not allow modification of a `nodeName` property of a pod.
  - If the pod is already created, and you want to manually assign the pod to a node: create a binding object and then send the post request to the pod’s binding API with the data set to the binding object in a json format (mimicking what the pod scheduler does)
  ```
    apiVersion: v1
    kind: Binding
    metadata:
        name: nginx
    target:
        apiVersion: v1
        kind: Node
        name: node02
  ```

  - `curl –header :Content-Type:application/json” --request POST --data ‘{“apiVersion” : “v1”, “kind”: “Binding“ ….  } http://$SERVER/api/v1/namespaces/default/pods/$PODNAME/binding`

- __Labels and Selectors:__ 
  - These are used to group and select objects. 
  - _Labels_ are defined under “metadata” in the manifest file in a key value pair format. 
  - A pod with a specific label can be selected by executing:
    - `kubectl get pods --selector [condition] app=App1` OR `kubectl get pods -A --selector=app=App1`
    - `kubectl get pods --selector env=dev --no-headers`
    - Kubernetes uses labels and selectors internally to connect different objects together eg. replicaSets and pods, services and pods etc
      - `kubectl get all --selector env=prod,bu=finance,tier=frontend`
  - NB: `kubectl get pods --selector env=dev | wc -l` will output the number of pods with selector `env=dev` (including the headers)
  - `kubectl get pods --selector env=dev --no-headers | wc -l` 

  - _Annotations:_ These are used to record other details for informative purposes e.g. tool details like name, build version, build information, contact details like phone numbers, emails, etc that might be used for some kind of integration purpose.
  
  - Attaching metadata to objects
    - You can use either labels or annotations to attach metadata to Kubernetes objects. 
    - Labels can be used to select objects and to find collections of objects that satisfy certain conditions. 
    - In contrast, annotations are not used to identify and select objects. 
    - The metadata in an annotation can be small or large, structured or unstructured, and can include characters not permitted by labels.
    - Annotations, like labels, are key/value maps:
      ```
      "metadata": {
        "annotations": {
          "key1" : "value1",
          "key2" : "value2"
        }
      }
      ```
    - Note: The keys and the values in the map must be strings. In other words, you cannot use numeric, boolean, list or other types for either the keys or the values.
      - https://kubernetes.io/docs/concepts/overview/working-with-objects/annotations/
  
- __Taints and Tolerations__
  - 



