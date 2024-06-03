### What is Kubernetes Cluster ?
- A Kubernetes Cluster is a group of computers (physical or virtual machines) working together to run containerized applications. It provides a platform for deploying, scaling and managing containerized workloads in a highly ahutomated and scalable way.
- There are two main types of Kubernetes Cluster are as follow:
	- **Single-Node-Cluster:**
		- Consists of a single machine that fulfills all the roles: running the control plane (API server, scheduler, controller manager) and worker node (hosting containerized applications in pods).
		- **Limitations:** Not Suitable for Production due to lack of scalability, fault tolerance, less high availbility and resource limitations. 
	- **Multi-Node-Cluster:**
		- Composed of **multiple machines**:
			- **Control Plane:** A set of dedicated machines running the Kubernetes control plane components. This ensures high availability and fault tolerance for critical cluster management tasks.
			- **Worker Nodes:** These machines run containerized applications packaged as pods. You can have multiple worker nodes to distribute the workload and scale your applications horizontally.
		- **Benefits:** It can easily scale the system, has the high availability and less faulty ensuring the continuity.

==**Note:**== Always the version of Worker Node should not be greater than Control Plane node due to backward compatibility issues can come.

| ==Feature==            | ==Single-Node Cluster==               | ==Multi-Node Cluster==        |
| ---------------------- | ------------------------------------- | ----------------------------- |
| **Nodes:**             | One node                              | Multiple nodes                |
| **Control Plane:**     | Runs on the single node               | Dedicated control plane nodes |
| **Worker Nodes:**      | The single node acts as a worker node | Multiple worker nodes         |
| **Scalability:**       | Limited                               | Highly scalable               |
| **High Availability:** | No                                    | Yes                           |
| **Fault Tolerance:**   | Limited                               | High                          |
| **Use Cases:**         | Development, testing                  | Production deployments        |
### What is Kubernetes Objects ?
- In Kubernetes, objects are the fundamental building blocks that defines and manage your applications. They acts as a record of intent, specifing the desired state of you containerized workloads.
- By creating Kubernetes objects, we essentially tell the Kubernetes system how we want our cluster to be configured. So the Kubernetes system then works automatically to make sure that cluster reaches that desired state.
- Types of Kubernetes Object are as follow:
	- **Pods:** The most basic unit in Kubernetes, representing a group of one or more containers that are deployed together on a shared underlying network namespace.
	- **Deployments:** Manage the lifecycle of your containerized applications. You specify the desired state (e.g., number of replicas) and Kubernetes automatically scales and updates the Pods.
	- **Services:** Provide a way to access your applications running on Pods across the cluster using a single network address and port.
	- **Namespaces:** Isolate groups of resources within a cluster, allowing for better organization and multi-tenancy.

### What are the tools which is used for setup and managing Kubernetes Cluster ?
- **kind:**
	- An open-source tool for creating local Kubernetes clusters in a runtime environment. kind uses Docker containers to simulate a multi-node Kubernetes cluster on a single machine. 
	- This provides a lightweight and portable way to experiment with Kubernetes locally without complex setups.
- **Minikube:**
	- A tool for running a single-node Kubernetes cluster locally. It's often considered a user-friendly option for beginners. 
	- Minikube provisions a single-node Kubernetes cluster on your local machine using a virtual machine or containerization technology like Docker.
- **Kubeadm:**
	- A tool for deploying production-grade multi-node Kubernetes clusters. It's an official Kubernetes component for cluster bootstrapping.
	- Kubeadm provides a command-line interface for initializing a Kubernetes cluster on bare-metal servers, cloud instances, or virtual machines. It configures the control plane components and prepares the worker nodes to join the cluster.

### What is Containerization and how its portable ?
- Containerization is a way of packaging software in a standardized unit that includes everything needed to run the code, regardless of the underlying computer system. It's like creating a self-contained shipping container for your application, with all its parts and instructions neatly packed inside.
- **Portability:**
	- **Standardized Container Images:** Container images are created using standardized formats like Dockerfile. These formats specify the instructions for building the container, including the application code, dependencies, and environment variables. This ensures consistent behavior across different systems.
	- **Container Runtime Engines:** Container runtime engines like Docker or containerd are responsible for running container images. These engines are available on most Linux distributions and cloud platforms.

![[Pasted image 20240424181634.png]]
### Why can't we use "registry tag" as "latest" ?
- For production deployments, we'll avoid using the "latest" tag for container images. This is because it's difficult to keep track of exactly which updates have been rolled out with "latest." Instead, we'll use specific version numbers from our registry to ensure we know precisely what code is running in our applications.
- Also backward compatibility is deminished.

### What are commands which makes layer  while making a Docker Image?
- A Dockerfile is a text file containing instructions that tell Docker how to build a container image. Each instruction you write in the Dockerfile contributes to the creation of a new layer in the final image.
- Instruction that created layers are **FROM**, **COPY**, **RUN**(If "run" runs more than 1 times then it created multiple layers) and **ADD**.
- Images with minimal layers are generally considered more efficient due to their smaller size and faster build times.

### What are distroless images ?
- A Distroless Images are images made by Google and made Open-Source for security purposes.
- **What Distroless Images Don't Include:**
	- **No package installer:** Distroless images don't have tools like APT or yum to install extra programs your application might need. Instead, they only include the program your application needs to run and the bare minimum to make it work.
	- **No command prompt:** Distroless images don't come with a command prompt (like bash or sh) that you can type commands into. They're meant to be run automatically, not used for interactive work.
- Size Comparision of Images:
	- Distroless < Strach < Alpine (Greater in Sizes)
- Making Distroless Images means simplifying them by removing as many layers as possible, which helps them become smaller and faster.

### What is Multistage ?
- In Multistage Production Grade Environment we use 2 Images one of which is Alpine for base and from that only we create our Final Image.
- **Using Alpine for a Base Image:**
    - **Alpine Linux** is a popular choice for base images in multi-stage builds due to its:
    - **Small Size:** Alpine is a lightweight Linux distribution, leading to smaller final images.
    - **Package Management:** It uses apk, a package manager known for efficiency.
    - **Security Focus:** Alpine prioritizes security with regular updates.
        - **==Multi-Stage Build Process:==**
        - **Stage 1: Build Environment:**
            - You create a Dockerfile with a base image like `alpine:latest`.
            - In this stage, you install all the development tools and dependencies needed to build your application code. These tools might include compilers, libraries, and build utilities.
            - This stage can be large because it includes the build environment.
            - **Crucially, you don't copy the application code itself into this stage.**
        - **Stage 2: Final Image:**
            - You create another stage in the Dockerfile that uses a minimal image as its base, often referred to as a "scratch" image (`FROM scratch`). This ensures the final image has only the necessary components.
            - You copy the compiled application code (artifacts) from the build stage (stage 1) into this final image.
            - You also copy any essential runtime dependencies your application needs to function.
            - This final image is much smaller than the build stage because it excludes the bulky build tools.
#### To Run Images we have another abstraction named "Containers".

### What is a Container ?
- Containers are a way to package software in a standardized unit that includes everything needed to run the code, regardless of the underlying computer system. They are like self-contained shipping containers for your applications.
- Also Don't Confuse yourself with Container as Docker, as Docker is a tool used for Containerization.
- There are **two types of containers application** are as follow:
	- **Stateless Application:** They don't remember anything about past interactions with users. Each request is treated independently, like a new customer at a store each time or Forgotful Application.
	- **Stateful Application:** They keep track of user interactions and preferences across multiple requests. They "remember" you, like a store with a loyalty program or Memorable Application.

### What is Kubectl ?
- Kubectl is the command-line tool or utility for interacting with Kubernetes clusters or we can use `alias k=kubectl` to save time but don't use in production environment.
- So, whatever command we write and with addition if we write `-o wide` which will be very helpful in debugging
- To views all the resources of Kubernetes Services `kubectl api-resources`.
1. **Deployment Management**:
    - `kubectl create deployment <deployment-name> --image=<image-name>`: Creates a new deployment with a specified image.
    - `kubectl get deployments`: Lists all deployments in your cluster.
    - `kubectl scale deployment <deployment-name> --replicas=<number-of-replicas>`: Scales the deployment to a desired number of replicas (pods).
    - `kubectl delete deployment <deployment-name>`: Deletes a deployment.
2. **Pod Management**:
    - `kubectl get pods`: Lists all pods in your cluster.
    - `kubectl describe pod <pod-name>`: Shows detailed information about a specific pod.
    - `kubectl exec -it <pod-name> bash`: Opens an interactive shell session within a running pod.
    - `kubectl delete pod <pod-name>`: Deletes a pod.
3. **Service Management**:
    - `kubectl create service <service-type> <service-name> --selector=<label-selector> --port=<port>`: Creates a service of a specific type (e.g., NodePort, LoadBalancer) to expose your application.
    - `kubectl get services`: Lists all services in your cluster.
    - `kubectl describe service <service-name>`: Shows detailed information about a specific service.
    - `kubectl delete service <service-name>`: Deletes a service.
4. **Resource Management**:
    - `kubectl get all`: Lists all resources (pods, deployments, services, etc.) in your cluster.
    - `kubectl get nodes`: Lists all worker nodes in your cluster.
    - `kubectl get namespaces`: Lists all namespaces in your cluster.
    - `kubectl delete resource <resource-type> <resource-name>`: Deletes a specific resource (e.g., deleting a deployment named "myapp").
5. **Viewing Logs**:
    - `kubectl logs <pod-name>`: Shows logs generated by a specific pod.
    - `kubectl logs -f <pod-name>`: Follows the logs of a pod in real-time.
6. **Events:** 
	- `kubectl get events`: It will list all the events from all namespaces in your cluster.
	- `kubectl get events -n <namespace-name>`: List events only for having a specific namespace
	- `kubectl get events --since=10m`: Show events that occurred after a specific duration
	- `kubectl describe event <event-name>`: This command provides detailed information about a specific event, including its reason, message, involved object, and source.
	- `kubectl get events -w -n <namespace-name>`: This command continuously monitors and displays new events as they occur in your cluster.

[**K9s**](https://k9scli.io/) is a dashboard for the resources we have in our kubernetes cluster or K8s.

### Does Pods are disposable and Ephemeral ?
- Yes, pods in Kubernetes are designed to be **disposable** and **ephemeral**. Here's a breakdown of the concept:
	- **Disposable:** Pods are meant to be short-lived and can be created, scheduled, and terminated as needed. They are not intended to be long-running processes like traditional virtual machines.
	- **Ephemeral:** This means pods are temporary and may not always be guaranteed to persist. Events like node failures or scaling actions can lead to pod restarts or terminations.