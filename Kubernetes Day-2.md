# Kubernetes Day-2

### What is HTTP 1.0 vs HTTP 1.1 vs HTTP 2.0 vs HTTP 3.0 ?
- **HTTP/1.0:**
	- It is released in 1996 and used simple request reponse model over TCP.
	- It is designed for a simple web environment and static web pages.
- **HTTP/1.1:**
	- It is released in 1999, it offers improvements over HTTP 1.0 such as persistent connections, pipelining, caching and chunked transfer encoding.
	- It enhances performance and reduces latency.
- **HTTP/2.0:**
	- Released in 2015, it introduces multiplexing, binary protocols, header compression, server push and priortization.
	- It resolves head-of-line blocking and improves efficiency.
- **HTTP/3.0:**
	- Released in 2022, it used QUIC protocol over UDP, eliminating head-of-line blocking and improvement performance.
	- It is designed for fast, reliable and secure web connection across all devices.

![Pasted image 20240418010952](https://github.com/rohit-rajput1/Kubernetes-learning/assets/76991475/59bb3cad-2b75-4f51-aa15-eab9cb586bb3)

### What is **API** and why it is used?
- An API(**Application Programming Interface**) is a standard way of talking between systems, allowing them to exchange information and functionality in a defined manner using client and server architecture model.
- An API is a middleman for software, letting apps talk to each other and share data easily. This saves developers time by providing pre-built functions they can integrate into their apps.

### What is gRPC ?
- gRPC stands for **gRPC Remote Procedure Calls**. It's an open-source framework that enables high-performance communication between applications.
- gRPC leverages the advantages of HTTP/2 for transport, like binary framing and multiplexing, but adds its own layer for defining services and data exchange procedures.
- **gRPC is based on two key components:**
	- **Remote Procedure Calls (RPC):** This is a networking paradigm that lets a client application call methods on a server as if they were local procedures. gRPC builds on this idea, making it appear like applications are talking directly to each other.
	- **Protocol Buffers or Protobuf:** This is a language-neutral way to define data structures. gRPC uses Protocol Buffers to efficiently serialize data being exchanged between applications. This ensures everything is understood clearly by both sides.
- In Kubernetes, Services talk with each other using **gRPC** and when we talk with Service we use **REST**.

![Pasted image 20240418014821](https://github.com/rohit-rajput1/Kubernetes-learning/assets/76991475/019f0878-4203-4c61-9b7b-775828e3918d)

# Kubernetes Architecture:

![Kubernetes excalidraw](https://github.com/rohit-rajput1/Kubernetes-learning/assets/76991475/b60832ca-2bbd-4f1a-a5b1-025dcdbeff35)

- In Kubernetes, there are 2 main component **Control Plane (Master Node)** and **Worker Plane (Slave Node)**.

## Master Node has 5 component are as follow:

### kube-api-server:
- **kube-api-server Architecture:** 

    ![Pasted image 20240418180404](https://github.com/rohit-rajput1/Kubernetes-learning/assets/76991475/f3920412-1af1-44da-862f-d79e71fdc922)

	- This is the most important component in the kubernetes, if this goes down whole cluster goes down and no activities are possible.
	- **let's say my master-node is down and my application is already deployed? does my application will be running or not ?**
		- -> The answer is Yes, because application is deployed on the worker node and we know master-node is for management. So application will be running.
		- **Authentication :** The kube-api-server verifies the identity of users or applications trying to access the API. It checks if they have the proper credentials using methods like client certificates, bearer tokens, or an authenticating proxy.
		- **Authorization :** After a user or application is authenticated, the kube-api-server doesn't just grant full access. It checks their authorization level using Role-Based Access Control (RBAC). This determines what actions they can perform (e.g., read, create, update, delete) on specific Kubernetes resources.
		- **Admission Control:** Admission control in Kubernetes is a process that happens **after** authentication and before resources are created or modified within the cluster. It acts as an additional layer of security and validation.
			- **Working:** After a user or application submits a request to create, update, or delete a Kubernetes resource (like a pod or deployment), the request goes through admission control. There are two controller in Admission Controller are as follow:
				- **Validate** the request to ensure it complies with defined policies (e.g., resource quotas, security best practices).
				- **Mutate (change)** the request to enforce specific configurations (e.g., injecting resource requests and limits).
			- If an admission controller rejects the request, the resource change won't be applied, and the user receives an error message.
			- So, admission control helps **maintain cluster health**, **security**, and **consistency** by ensuring only authorized and well-defined changes are made to Kubernetes resources.
		- **Admission Controller Architecture:** 

            ![Pasted image 20240418184937](https://github.com/rohit-rajput1/Kubernetes-learning/assets/76991475/98b63491-59af-46c1-be53-e1286c4f5c93)

			- How to check which webhook is enabled ? 
				- Go to terminal `cd /etc/kubernetes/manifests` this will shows all the control plane which runs as a static pods.
				- We can see the file related to **kube-apiserver** and we can check what plugins are enable to this file using `cat kube-apiserver | grep 'enable'`.
			- **What is CRD (Custom Resource Definition) and how kube-api-server leverages it ?**
				- A **CRD**(**Custom Resource Definition**), is not directly part of the kube-apiserver itself. It's a mechanism that extends the Kubernetes API to allow you to define and manage your own custom resources. Here's the relationship:
					- **CRD:** You create a CRD as a YAML or JSON file specifying the schema and behavior of your custom resources. This file is submitted to the kube-apiserver.
					- **kube-apiserver:** The kube-apiserver acts upon the submitted CRD definition. It understands how to handle requests for your custom resources based on the CRD information.
				- So, the kube-apiserver leverages the CRD definition to manage your custom resources, but the CRD itself is a separate entity that extends the capabilities of the kube-apiserver.
		- **Watch for Updates:** There are two main ways to watch for updates in Kubernetes in depending upon specific needs:
			- **Using watch API:** The Kubernetes API server provides a built-in functionality called the "watch" API. This allows you to establish a long-running connection and receive updates whenever a relevant resource changes within the cluster. This is ideal for scenarios where you need to react to changes in real-time, like automatically scaling deployments based on resource utilization.
			- **Periodic list with filter:** You can periodically query the API with a filter based on the resource version to get only updated resources. This is simpler but less real-time

### etcd:
- No etcd, no Kubernetes brain. It stores cluster data and coordinates actions, keeping everything in sync.
	- **etcd Architecture:** 
    
    ![Pasted image 20240418195027](https://github.com/rohit-rajput1/Kubernetes-learning/assets/76991475/162f2a66-e6c5-4ef5-b923-25abc5a77ecc)

	- Different components of **etcd** are as follow:
		- **Distributed Key-Value Store:** etcd itself is the distributed key-value store. It stores Kubernetes cluster data (pod definitions, deployments, etc.) with keys representing unique identifiers and values holding the configuration details. Multiple Kubernetes nodes can access and update this data simultaneously.
		- **Write-Ahead Log (WAL):** Before etcd commits a change to the key-value store, it first writes that change to the WAL. This ensures data consistency even during failures. Imagine etcd writing down the intended change in a log before updating the main key-value store.
		- **Protocol Buffers (fastest):** etcd leverages Protocol Buffers to define how data is formatted when stored or transmitted between nodes. This ensures all machines, regardless of programming language, understand the data structure. Think of it like a common language for all the machines working with etcd.
		- **Raft Consensus Algorithm:** Worlds top databases are based on Raft Consensus. This is the core of keeping all the distributed copies of the key-value store consistent. Raft ensures that all etcd nodes in the cluster agree on the latest data, even if some nodes fail or experience network issues. It's like a voting system where a majority of nodes agree on the final state of the data which has a election timeout of (150~300ms) [Check the Website For More](https://thesecretlivesofdata.com/) 

	- **How do we secure data in etcd ?**
		- **Data Encryption at Rest** is the primary method for securing data within etcd. It involves encrypting the data on the storage media itself using encryption keys. Even if someone gains access to the physical storage, the data will be scrambled and unreadable without the decryption key.
		- **Minimize Sensitive Data Storage** which allows us to limit the amount of highly sensitive information stored directly in etcd. This includes credentials like passwords, API keys, or tokens.
    
### kube-scheduler:
- Kube-scheduler is a crucial component in Kubernetes responsible for assigning pods (containers needing resources) to appropriate nodes (worker machines) within the cluster. It ensures efficient resource utilization and optimal placement of pods based on various factors.
	- **kube-scheduler architecture:** 

    ![Pasted image 20240419151139](https://github.com/rohit-rajput1/Kubernetes-learning/assets/76991475/c207edf4-fd70-4ee8-ab22-cc8faa52ed0c)

	- There are **`two cycles`** in kube-scheduler are as follow:
		- **Scheduling Cycle :** This phase focuses on selecting a suitable node for a new or pending pod.
			- **Queue Up:** A new pod enters the scheduling queue, waiting for its turn to be placed on a suitable node.
			- **Prefilter (Optional):** These lightning-fast plugins perform basic checks to exclude a large number of nodes quickly. They eliminate nodes with insufficient resources (CPU, memory, etc.) or those marked as unavailable due to taints (special labels indicating specific restrictions). Think of this as a preliminary scan to remove obvious mismatches.
			- **Filter:** These plugins conduct more in-depth examinations. They consider pod annotations (custom labels) or node labels to enforce specific placement requirements. For example, a filter plugin might ensure a pod requiring a GPU is only placed on nodes with GPUs available. It's like a more thorough screening based on defined criteria.
			- **Score:** Only nodes that pass the filter stage remain. Now, score plugins come into play. Each plugin assigns a score to each remaining node based on various factors. This might include available resources, current node utilization, or affinity/anti-affinity rules defined for the pod.
			- **Normalize Score (Optional):** Since different score plugins might use varying scales, these plugins (if enabled) adjust the scores to a common scale. This ensures scores from diverse plugins can be compared fairly in the final decision.
			- **Post Filter (Optional):** These plugins act as a final safety net before selecting a node. They can perform additional checks based on the combined scores or other factors. For example, a post-filter might ensure a minimum score threshold is met before considering a node and kicks out the low priority pods.
			- **Permit (Optional):** These plugins provide an additional layer of authorization. They can enforce security policies or business logic before a pod is bound to a node.
			- **Binding Cycle:** The binding cycle in Kube-scheduler is another crucial step after the scheduling cycle selects a suitable node for a pod.
			- **Pre-Bind (Optional):** Reserving resources on the node to prevent conflicts with other pods and checking the node health or readliness for additional safety measures.
			- **Bind:** This is the core of the binding cycle. Here, the kube-scheduler sends a request to the kubelet (agent running on the chosen node) to bind the pod to the node. The kubelet updates its internal state to reflect the incoming pod and allocates necessary resources.
			- **Post-Bind (Optional):** - Updating pod object metadata with the assigned node information and triggers notifications or events to signal successful pod binding.
			- **Wait-on-Permit (Optional):** This phase involves waiting for an external approval (permit) before proceeding. This could be useful for integrating with security systems or external authorization mechanisms.
    
### kube-controller-manager:
- The kube-controller-manager is a vital component in Kubernetes responsible for running multiple controllers in the background. These controllers continuously monitor the state of the cluster and take actions to ensure the desired state of your applications is maintained.
	- **kube-controller-manager architecture:** 

    ![Pasted image 20240419160500](https://github.com/rohit-rajput1/Kubernetes-learning/assets/76991475/c11035df-d131-4898-966c-ceedc03906a1)

	- The described core component are as follow:
		- **Node Controller:** This controller manages worker nodes (machines running containerized workloads). It monitors node health, detects failures, and attempts to automatically restart them or cordon/drain unhealthy nodes (prevent pod scheduling/evict existing pods) to avoid impacting healthy workloads.
		- **Service Controller:** This controller watches over Service objects and ensures they are translated into Kubernetes networking resources like Endpoints (mapping pods to a service) or LoadBalancers (providing external access). It keeps service configurations in sync with the underlying network infrastructure.
		- **Namespace Controller:** This controller manages namespaces, which provide a way to isolate resources within a cluster. It ensures namespaces are created/deleted as needed and enforces resource quotas within a namespace.
		- **DaemonSet Controller:** This controller ensures a daemonset (a special pod meant to run on all or a subset of nodes) has its desired number of pod replicas running on each node in the cluster. It creates or deletes pods as needed to maintain the specified state.
		- **CronJob Controller:** This controller manages CronJobs, which are Kubernetes objects that allow you to schedule tasks to run on a defined schedule (e.g., every hour, daily, etc.). The CronJob controller monitors these schedules and creates pods at the designated times to execute the desired tasks.
		- **Custom Controllers:** The beauty of Kubernetes is its extensibility. You can develop your own custom controllers to manage specific resources or automate tasks tailored to your application needs.
    
### cloud-controller-manager:
- In Kubernetes, the cloud-controller-manager is a component that comes into play **specifically when you're using a cloud provider** to manage your Kubernetes cluster. It acts as an intermediary between the Kubernetes control plane and your cloud provider's API.
 

## Worker Node has 3 component are as follow:
- **Worker Node Internal Working Architecture:** 

![Kubernetes2](https://github.com/rohit-rajput1/Kubernetes-learning/assets/76991475/6082a293-e642-41eb-a191-2a2ea8ffde29)

### pod:
- A pod is the fundamental unit of deployment in Kubernetes. It represents a group of one or more containers that are meant to be deployed together on a shared underlying infrastructure. Think of it as a containerized application or service encapsulated within a single unit.
	- The Components of **`Pods`** are as follow:
		- **Containers:** A pod can contain one or more containerized applications. These containers share the pod's storage, network resources, and lifecycle. Imagine each container within a pod as a microservice contributing to an overall application.
		- **Shared Storage:** Pods have access to a shared storage volume mounted at a specific path within each container. This allows containers within the pod to share data and collaborate effectively.
		- **Pod Spec:** This is the blueprint that defines the pod's configuration, including the container images to be used, resource requests and limits, environment variables, and storage requirements.

### kube-proxy:
- Kube-proxy is installed by default on all worker nodes in a kubernetes cluster. Its primary function is to map Service objects to actual network rules on each node.
	- **Watches for Service Changes:** Kube-proxy keeps a close eye on the Kubernetes API server for any updates to Service objects. These Services define how network traffic should be directed to your pods based on labels or selectors.
	- **Translation to Network Rules:** Whenever a Service changes, Kube-proxy steps in and translates the Service definition into concrete network rules specific to the node's operating system. The chosen mode (iptables or ipvs for Linux, kernelspace for Windows) determines the translation method.
	- **Network Rule Implementation:** Based on the translated rules, Kube-proxy modifies firewall settings (iptables or similar) or load balancing configurations on the worker node. These rules ensure that traffic targeting a Service is effectively routed to the appropriate pods within the cluster, considering selector criteria defined in the Service. 
    
### kubelet:
- Kubelet, short for **Kubernetes Node Agent**, is a critical component that runs on each worker node within a Kubernetes cluster. It acts as the bridge between the Kubernetes control plane and the i*ndividual nodes, playing a central role in managing container lifecycles and ensuring the smooth operation of your pods.
	- **Kubelet Cycle:** 

    ![Pasted image 20240420193032](https://github.com/rohit-rajput1/Kubernetes-learning/assets/76991475/3eb368d6-78c4-4a72-99a9-739c5e331200)

	- **Node Registration:** Kubelet registers the worker node with the Kubernetes API server using its hostname or a specific cloud provider integration. This allows the control plane to be aware of available resources and schedule pods accordingly.
	- **Pod Management:** Kubelet receives pod specifications from the API server. It then translates these specifications into actionable steps for the container runtime engine (like Docker or containerd) on the node. Kubelet ensures pods are created, started, stopped, and deleted based on instructions from the control plane.
	- **Health Monitoring:** Kubelet continuously monitors the health of running pods and the overall health of the node itself. It reports this information back to the API server, allowing the control plane to take corrective actions if necessary (e.g., restarting unhealthy containers or rescheduling pods to healthy nodes).
	- **Resource Management:** Kubelet monitors resource utilization (CPU, memory, etc.) on the node. It enforces pod resource limits and requests defined in the pod specifications, ensuring fair resource allocation and preventing pods from consuming more than their allocated share.
	- **Secret and ConfigMap Management:** Kubelet fetches secrets and ConfigMaps required by pods from the API server and makes them accessible to containers within the pod. This allows pods to access sensitive information or configuration data securely.
	- There are **3 interfaces in Kubelet** are as follow: 
		- **Container Runtime Interface:** Kubelet interacts with the container runtime engine (CRI) on the node. The CRI is responsible for the low-level tasks of creating, managing, and running containers. Kubelet acts as an abstraction layer, shielding the Kubernetes control plane from the specifics of the underlying container runtime.
		- **CNI (Container Network Interface):** Provides a standardized way for Kubelet to configure networking for pods on worker nodes using different CNI plugins (e.g., overlay networks, bridge networking).
		- **CSI (Container Storage Interface):** Provides a standardized way for Kubelet to interact with various storage providers (cloud, local) through CSI drivers to provision and manage persistent storage volumes for pods. 

### Key Metrics in Kubernetes which is used in production:

- **etcd_server_leader :** etcd-server-leader is the single elected server in the cluster responsible for **processing writes** and **maintaining consistency** across all nodes.
- **etcd_server_leader_changes_seen_total :** It tracks the number of times the leader node has changed within an etcd cluster.
- **etcd_network_peer_round_trip_time_seconds_bucket :** It provides insights into the communication speed between etcd cluster members which includes network performance, time taken to travel from one etcd server to another and back again.
- **workqueue_adds_total :** It is a metric that tracks the total number of items added to a specific workqueue. A Workqueue is essentially a queue used by controller in kubernetes to manage tasks.
- **workqueue_depth :** It is a metric that guages the current number of items waiting to be processed in a specific workqueue.
- **kubelet_running_pods :** It is a way to check the number of running pods on a specific node within your kubernetes cluster.
- **kubelet_pod_start_duration_seconds_count :** This metric tracks the **total number of times** a pod has gone from a pending state to a running state on a specific node within a Kubernetes cluster.
- **rule_sync :** This ensures that all nodes have the same set of rules and operate consistently.
- **apiserver_admission_controller_duration_seconds :** It tracks the **time taken by admission controllers** in the Kubernetes API server to process requests.
---

**Blog:** [**How OpenAI does the scaling Kubernetes to 7500 Nodes by OpenAI**](https://openai.com/research/scaling-kubernetes-to-7500-nodes)