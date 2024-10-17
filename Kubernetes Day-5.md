# Kubernetes Day-5

## Labels in K8s

### What are labels ?
- In Kubernetes, **labels** are key-value pairs assigned to objects (like Pods, Nodes, or Services) to help organize, categorize, and select resources based on meaningful attributes.

- **If by default we don't have label what will be the label ?** 
	- Kubernetes does not assign a default key-value pair like **`run:<pod-name>`** unless explicitly defined by the user.
	- However, we use **`kubectl run <pod-name>`** to create a pod, kubernetes automatically assigns the following labels to it **Key: `run`** and **Value: `<pod-name>`**  and this happens only with specific command:

```bash
kubectl run my-app --image=nginx
```

**Generated Output:**

```yaml
metadata:
  labels:
    run: my-app
```
- **Can we add more than one label in Kubernetes ?**
	- Yes, you can definitely add more than one label to resources in Kubernetes. In fact, using multiple labels is a common and recommended practice for better organization and management of your Kubernetes resources.

```yaml
selector:
  matchLabels:
    environment: production
    tier: frontend
```

- This selector would match resources that have both the **`environment: production`** and **`tier: frontend`** labels. Remember, while you can add many labels, it's good practice to keep your labeling strategy consistent and meaningful across your cluster.

### Key Points about Labels:
- **Key-Value Pairs:** Labels consist of a unique key and an associated values. **Eg.** **`app: Frontend`** , **`env: production`**.
- **Metadata:** Labels are part of an object's metadata, meaning they don't directly affect the object's behavior.
- **Selectors:** Labels enable label selectors to filter or group objects (eg. select all pods with **`app=frontend`**) which is useful in Deployments, ReplicaSets and Services.
- **Immutable Keys:** Once assigned, the key of a label cannot be changed, but values can be updated.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: my-pod
  labels:
    app: frontend
    env: production
spec:
  containers:
    - name: nginx
      image: nginx
```

### Change labels using command line :

1.  **Add or Update a label :** This commands **adds or update** the **environment label** with the value production on the pod named **`my-pod`**.

```bash
# kubectl label <resource-type> <resource-name> <key>=<value> --overwrite
kubectl label pod my-pod environment=production --overwrite
```

2. **Remove a Label :** This removes the **`environment`** label from the pod named **`my-pod`**. Adding a **-**
after the key removes the label.

```bash
# kubectl label <resource-type> <resource-name> <key>-
kubectl label pod my-pod environment-
```

3. **Verify Labels :** After applying the changes, you can verify the labels using commands.

```bash
# kubectl get <resource-type> <resource-name> --show-labels
kubectl get pod my-pod --show-labels
```

4. **See labels :** This command will display the labels directly in the output.

```bash
kubectl get pods --show-labels
```

![Pasted image 20241015190551](https://github.com/user-attachments/assets/3eb4a263-074e-4eda-982c-25fbeff6a77f)

#### **Q**. What happens if we delete cluster does everything inside it gets deleted?

- When you delete a Kubernetes Cluster resources like **Pods, Services, Deployments** will be permanently deleted when the cluster is deleted.
- Persistent Volumes (PVs) in **Kubernetes** are not deleted automatically when the cluster is removed, ensuring that **data is not lost by default**.
- So, when a pod or even the cluster is deleted, the PV remains unless it **Reclaim Policy** is explicitly set to **Delete**. This is useful to avoid unintended data loss during cluster teardown.

### Step-by-Step: Using Private Repositories in Kubernetes

1. **Create a Docker Config File for Authentication** where we create a **`~/.docker/config.json`** file containing the credentials, this more secure because it doesn't requires you to directly input your credentials in the Kubectl command, which could be logged or visible in your shell history.

```bash
docker login <registry-url> -u <username> -p <password>
```

2. **Create a Kubernetes Secret** where we will store the credentials inside a secret.

```bash
kubectl create secret generic regcred \
  --from-file=.dockerconfigjson=~/.docker/config.json \
  --type=kubernetes.io/dockerconfigjson
```

- **regcred** : The name of the secret.
- **~/.docker/config.json** : Docker Authentication file. 

3. Reference the **Secret** in Deployment YAML file where we put it as value in **`imagePullSecrets`**.

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-app
spec:
  replicas: 1
  selector:
    matchLabels:
      app: my-app
  template:
    metadata:
      labels:
        app: my-app
    spec:
      containers:
      - name: my-app-container
        image: <registry-url>/<namespace>/<image>:<tag>  # Example: ghcr.io/user/my-app:latest
      imagePullSecrets:
      - name: regcred  # Name of the secret
```

4. After applying the YAML file, Kubernetes will use the **secret to authenticate** with the **Private Registry**.

```bash
kubectl apply -f deployment.yml
kubectl get pods
```

#### **Q**. How do we specify which pod will run on which node ?

- In Kubernetes, we have several methods to control which pods run on which nodes:
	1. **Node Selectors:** This is the simplest method. We add labels to nodes and use nodeSelector in the pod spec to match those labels.
	2. **Node Affinity and Anti-Affinity:** These provide more flexible node selection. They allow complex matching rules and can express preferences rather than hard requirements.
	3. **Taints and Tolerations:** Taints are applied to nodes to repel certain pods, while tolerations allow pods to schedule onto nodes with matching taints.
	4. **Pod Affinity and Anti-Affinity:** These allow us to schedule pods based on the labels of other pods running on the node.

- **Example of Node Affinity:** This would ensure the pod runs on nodes labeled with either 'us-west-1a' or 'us-west-1b' for the 'zone' key.

```yaml
nodeAffinity:
  requiredDuringSchedulingIgnoredDuringExecution:
    nodeSelectorTerms:
    - matchExpressions:
      - key: zone
        operator: In
        values:
        - us-west-1a
        - us-west-1b
```
### Probes

- In Kubernetes, **probes** are mechanisms used to check the **health** and **availability** of conatiners to ensure they are fucntioning properly. There are three types of probes:
	- **Liveness Probe :**
		- It ensures that the container is **alive** and responsive. If the container fails the liveness probe, Kubernetes will restart it.
		- **Where to use:** It is used to detect and fix stuck or deadlock situations where the app stops responding. 
	- **Readiness Probe :**
		- It ensures that the containers is **ready** to accept traffic. If the readiness probe fails, kubernetes will **remove the pod from the service's endpoints**, preventing traffic from being routed to it.
		- **Where to use:** It is useful for containers that need some **initialization time** before they can serve requests.
	- **Startup Probe :**
		- It ensures that the applications has successfully started. If the startup probe fails, kubernetes will **kill and restart the container**.
		- **Where to use:** It is useful for apps that takes a long time to start. The startup probe disable the liveness probe disables the liveness probe untill the container is ready.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: my-app-pod
spec:
  containers:
  - name: my-app
    image: my-app-image:latest
    ports:
    - containerPort: 8080
    livenessProbe:
      httpGet:
        path: /healthz
        port: 8080
      initialDelaySeconds: 3 
      periodSeconds: 3
    readinessProbe:
      httpGet:
        path: /ready
        port: 8080
      periodSeconds: 5
    startupProbe:
      httpGet:
        path: /healthz
        port: 8080
      failureThreshold: 30
      periodSeconds: 10
```

- Probes can use different mechanisms to check container health:
	1. **HTTP GET:** Performs an HTTP GET request and have specified endpoints which expects a **`2xx`** or **`3xx`** status.
	2. **TCP Socket:** Attempts to open a socket connection of specific port.
	3. **Exec:** Executes a command inside the container. If the command returns a **0 status code**, probe is successful.

---

## Replicas in K8s

### What are Replicas ?
- In Kubernetes, replicas refers to the number of identical instances (Pods) of an application running at the same time to ensure **high availability**, **load balancing** and **fault tolerance**. Replicas are managed by **ReplicaSet** or **Deployment**.

### Why there is need of Replicas?

- **High Availability :**
	- Ensure uptime by running multiple instances (Pods) of the application. If one pod crashes, other continue to serve traffic without downtime.
	- **Example:** If a web application runs with 3 replicas, the service stays available even if one pod fails.
- **Load Balancing :** 
	- Replicas distribute the workload across multiple instances, preventing any single pod from being overwhelmed. Kubernetes Services automatically balance traffic between replicas to improve performance.
	- **Example:** If 1000 users acces the app, traffic will be spread across mutiple pods to handle requests efficiently.
- **Fault Tolerance :** 
	- Kubernetes monitors replicas and replace failed pods to maintain the desired number. This ensures the system recovers automatically from failures without manual interventions.
- **Scalability :**
	- You can increase or decrease replicas based on the demand (horizontal scaling). During peal hours, you can scale up the replicas and scale down when demand reduces, optimizing resources usage.
	- **Example:** E-Commerce apps can scale from 2 replicas to 10 during a sale, ensuring they can handle the increased load.

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-app
spec:
  replicas: 3  # Three replicas
  selector:
    matchLabels:
      app: my-app
  template:
    metadata:
      labels:
        app: my-app
    spec:
      containers:
      - name: nginx
        image: nginx
```

- **Rolling Updates & Zero Downtime Deployments :**
	- With replicas, Kubernetes can **update the application gradually** (rolling updates) to avoid downtime.
	- New Pods are created while old ones are terminated, ensuring continuous service availability.

### How do we define an availability of website ?

- The **availability chart** for cloud services, showing how downtime translates to **different availability percentages** over a given time period:

| **Availability %**   | **Downtime per Year**         | **Downtime per Month** | **Downtime per Week** | **Downtime per Day**   |
| -------------------- | ----------------------------- | ---------------------- | --------------------- | ---------------------- |
| 99.999% (Five Nines) | 5 minutes 15 seconds          | ~26 seconds            | ~6 seconds            | ~0.86 seconds          |
| 99.99%               | 52 minutes 36 seconds         | ~4 minutes 23 seconds  | ~1 minute             | ~8.6 seconds           |
| 99.9%                | 8 hours 45 minutes 36 seconds | ~43 minutes 12 seconds | ~10 minutes           | ~1 minute 26 seconds   |
| 99%                  | 3 days 15 hours 36 minutes    | 7 hours 12 minutes     | ~1 hour 41 minutes    | ~14 minutes 24 seconds |

- **Five nines (99.999%) availability** is the standard for **emergency response systems**, allowing only about **5 minutes and 15 seconds** of downtime per year.
- Achieving such high availability requires:
    - **Redundant infrastructure** (software, hardware, networks)
    - **Fault-tolerant systems** with automatic failover mechanisms
    - **Continuous monitoring** to detect and resolve issues instantly

### What kinds of pods can replica set runs ?
- Pods are of two types in terms of **Homogeneous** and **Hetrogeneous pods**, where a **ReplicaSet** is designed to manage only **Homogeneous pod**, meaning **all pods it runs are identical**.

#### 1. Homogeneous Pods (Allowed)
- Pods within a replica set that have identical specifications, including container images, resource requests, and environment variables.
- **Where to use :** It is used in stateless applicatio where any pod can handle the same workload which ensures uniformity and load balancing across all replicas.

**Example:**  A ReplicaSet managing **three Nginx Pods** that all use the same image and configuration.

```yaml
spec:
  replicas: 3
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
        - name: nginx
          image: nginx:latest
```

#### 2. Heterogeneous Pods (Not Allowed)

- Pods within a replica set that have different specifications, such as varying container images, resource requests, or environment variables.
- **Where to use :** It is used in microservice architecture where one pod acts as a **frontend** and another as a **backend**.

#### How we use Heterogeneous Pods using ReplicaSet (Tweaks) :

- **Environmental Variables or ConfigMaps:**
	- Inject dynamic configuration via **environment variables**, **configMaps** or **Secrets** based on Pod name or labels. This allows different behavior for each Pod, though the Pod templates are technically identical.
	- **Example:** Using **`POD_NAME`** environment variable for conditional logic within the container.

```yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: heterogeneous-app
spec:
  replicas: 3
  selector:
    matchLabels:
      app: heterogeneous-app
  template:
    metadata:
      labels:
        app: heterogeneous-app
    spec:
      containers:
      - name: app-container
        image: myapp:latest
        env:
        - name: POD_NAME
          valueFrom:
            fieldRef:
              fieldPath: metadata.name
        - name: POD_IP
          valueFrom:
            fieldRef:
              fieldPath: status.podIP
        - name: NODE_NAME
          valueFrom:
            fieldRef:
              fieldPath: spec.nodeName
        # Use a ConfigMap for additional configuration
        envFrom:
        - configMapRef:
            name: app-config
```

- **Mounted Volumes for Different Behavior :**
	- Mount ConfigMaps or Secrets as volumes containing Pod-specific configurations. Each Pod can mount a different configuration file based on its name or label.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: my-app
spec:
  containers:
    - name: nginx
      image: nginx:latest
      volumeMounts:
        - name: config-volume
          mountPath: /etc/config
  volumes:
    - name: config-volume
      configMap:
        name: app-config
```

**Note :** The ConfigMap is mounted at `/etc/config`. Each Pod gets access to the same configuration.

- **Pod Affinity and Anti-Affinity :** 
	- This YAML demonstrates **affinity** (Pods prefer to run together) and **anti-affinity** (Pods prefer to run on different nodes).
	- **PodAffinity :** Tries to schedule Pods on the same node based on the `kubernetes.io/hostname` key. 
	- **PodAntiAffinity :** Tries to **avoid placing Pods** on the same node, distributing them across nodes for fault tolerance.

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-app
spec:
  replicas: 3
  selector:
    matchLabels:
      app: my-app
  template:
    metadata:
      labels:
        app: my-app
    spec:
      affinity:
        podAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
            - labelSelector:
                matchLabels:
                  app: my-app
              topologyKey: "kubernetes.io/hostname"
        podAntiAffinity:
          preferredDuringSchedulingIgnoredDuringExecution:
            - weight: 1
              podAffinityTerm:
                labelSelector:
                  matchLabels:
                    app: my-app
                topologyKey: "kubernetes.io/hostname"
      containers:
        - name: nginx
          image: nginx:latest
```

- **Adding Tolerance/Node Selectors :**
	- Modify node scheduling behavior based on environment variables or external configurations. Pods can exhibits different behavior depending on the node they're scheduled on.
	- Here the pod will **tolerate nodes** with a taint `key1=value1:NoSchedule`, meaning it can be scheduled on such nodes despite the taint.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: toleration-example
spec:
  containers:
    - name: nginx
      image: nginx:latest
  tolerations:
    - key: "key1"
      operator: "Equal"
      value: "value1"
      effect: "NoSchedule"
```

### How does ReplicaSet Scale-Up or Scale-Down?

- **Scale-Up:**
    - When scaling **up**, the ReplicaSet controller creates new Pods to meet the increased replica count.
- **Scale-Down:**
    - When scaling **down**, the controller decides which Pods to delete using the following **priority order**:
#### ReplicaSet Scale-Down Algorithm:
1. **Pending (and Unschedulable) Pods are Deleted First**
    - Any Pods that are **pending** (e.g., stuck in scheduling or waiting for resources) are prioritized for deletion.
2. **`controller.kubernetes.io/pod-deletion-cost` Annotation**
    - If the **`pod-deletion-cost`** annotation is present, the Pod with the **lower value** is deleted first. 
    - **Pods with higher values** are more expensive to delete and are kept longer.

```yaml
metadata:
  annotations:
    controller.kubernetes.io/pod-deletion-cost: "-10"
```

3. **Pods on Nodes with More Replicas**
    - Pods running on **nodes with more replicas** are deleted before those on nodes with fewer replicas.
    - This ensures **better distribution** of Pods across nodes.
4. **Pod Creation Time (Logarithmic Scaling)**
    - If Pods have different **creation times**, **recently created Pods** are deleted before older ones.
    - When the **`LogarithmicScaleDown` feature gate** is enabled, the creation times are **bucketed on a logarithmic scale** to handle large clusters efficiently.


### How to Annotate a Pod ?

- There are 2 ways to Annotate a Pod in Kubernetes, **Add Annotations** during Pod Creation and **Modify Annotations** on an existing Pod using **`Kubectl`**.

- **Add Annotation During Pod Creation :**

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: example-pod
  annotations:
    environment: "production"
    owner: "dev-team"
spec:
  containers:
    - name: nginx
      image: nginx:latest
```

- **Add or Update Annotations on an Existing Pod :**

```bash
# kubectl annotate pod <pod-name> key1=value1 key2=value2
kubectl annotate pod example-pod environment=production owner=dev-team
```

- **Remove an Annotation :** To **remove an annotation**, set it to a blank value or use the **`--remove`** flag.

```bash
kubectl annotate pod example-pod environment-
```

- **Verify Annotations :** Following command is used to check the annotation on a Pod.

```bash
kubectl describe pod <pod-name> | grep Annotations
```

---

## Deployment in K8s

### What is a Deployment in Kubernetes ?

- A Deployment is a Kubernetes controller that manages a ReplicaSet to ensure the desired state of an application. It helps automate he **creation**, **scaling** and **updates** of Pods. 
- A Deployment ensures that a specific number of identical pods are running at all times and provides features like **rolling updates** and **rollbacks**.

### Key Function of Deployment :

- **Scaling :** Scale up or down the number of Pods.
- **Rolling Updates :** Gradually update Pods to a new version.
- **Self-healing :** Recreate Pods if they fails or are deleted.
- **Rollback :** Revert to a previous version if a new update fails.

### Deployment Strategies in Kubernetes

1. **Recreate Strategy :** Here all existing pods are deleted first and then new pods are created. It is used when downtime is acceptable or the new version is incompatible with the previous one.

```yaml
strategy:
  type: Recreate
```

2. **Rolling Update Strategy (Default) :** Pods are updated incrementally. A few old pods are terminated, and new one are started untill the entire application is updated. It is ued when the update must happen without downtime.

```yaml
strategy:
  type: RollingUpdate
  rollingUpdate:
    maxUnavailable: 1 
    maxSurge: 1
```

- **maxUnavailable :** Number of pods that can be unavailable during the update.
- **maxSurge :** Number of extra pods created temporarily during the update.

### Why use a Deployment Instead of a ReplicaSet ?

- **Automated Rolling Updates and Rollbacks :**
	- Deployment supports **rolling updates**, ensuring that the application is updated **without downtime**. It also allows **rollback** to previous versions if an update fails.
	- **ReplicaSet** alone does not provide these features for that we need to manage updates manually.
- **Example :** With Deployment, we can update the app version like this , which will gradually replace old pods with new ones, ensuring uptime.
```bash
kubectl set image deployment/my-app nginx=nginx:1.19.10
```

- **Version History Management :**
	- Deployments keeps track of **previous ReplicaSets** (for rollback purposes).
	- With ReplicaSets alone, we need to manually track versions to revert if something goes wrong.

- **Self-Healing :**
	- While both **ReplicaSets** and **Deployments** can restart failed Pods, Deployments manage **multiple ReplicaSets** and ensure the **desired state** is achieved across updates.

- **Declarative Management**
	- With Deployments, you just **declare the desired state** (e.g., number of replicas, version of the app), and Kubernetes takes care of the rest.
	- Using ReplicaSets alone would require more **manual management** (e.g., scaling up/down ReplicaSets and deleting old ones).

### When Would You Use a ReplicaSet Directly?

- If you don’t need the **rolling update or rollback** functionality (e.g., **batch jobs** or simple workloads).
- In some rare cases, you may want **full control** over the Pods without any automatic management, but this is uncommon.

---

## Canary Deployment in K8s

### What is Canary Deployment ?
- A Canary deployment is a strategy for gradually rolling out a **new version of an application** to a small subset of users or traffic, while the rest of the users continue to access the existing version.
- It ensures **new features are tested in production** without disrupting the entire system. If everything works as expected, traffic is shifted progressively to the new version; if not, it can be **rolled back** easily.

![Pasted image 20241017104243](https://github.com/user-attachments/assets/61dcd41d-683d-4885-8dc7-5404bcf62656)

### Step-by-Step Guide for Canary Deployment

- **Step 1 :** Pull the Docker Image of Nginx and verify the downloaded image.

```bash
docker pull nginx
docker image ls
```

- **Step 2 :** Create the Initial Deployment **(Version 1.0)**. Here we creating for Nginx **`nginx-deployment.yaml`**.

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
        version: "1.0"
    spec:
      containers:
        - name: nginx
          image: nginx:alpine
          resources:
            limits:
              memory: "128Mi"
              cpu: "50m"
          ports:
            - containerPort: 80
          volumeMounts:
            - mountPath: /usr/share/nginx/html
              name: index.html
      volumes:
        - name: index.html
          hostPath:
            path: /path/to/v1
```

- Apply and verify the deployment of **(Version 1.0)**.

```bash
kubectl apply -f nginx-deployment.yaml
kuebctl get pods -o wide
```

- **Step 3 :** Create a Service for the Deployment for our **(Version 1.0)** which is **`nginx-deployment-service.yaml`**.

```yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx-service
spec:
  type: LoadBalancer
  selector:
    app: nginx
    version: "1.0"
  ports:
    - port: 8888
      targetPort: 80
```

- Apply and verify the Service of **(Version 1.0)**.

```yaml
kubectl apply -f nginx-deployment-service.yaml
kubectl get service
```

- **Step 4 :** Now we will create Deployment for **Canary Version (Version 2.0)** which is **canary-deployment.yaml**.

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-canary-deployment
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
        version: "2.0"
    spec:
      containers:
        - name: nginx
          image: nginx:alpine
          resources:
            limits:
              memory: "128Mi"
              cpu: "50m"
          ports:
            - containerPort: 80
          volumeMounts:
            - mountPath: /usr/share/nginx/html
              name: index.html
      volumes:
        - name: index.html
          hostPath:
            path: /path/to/v2
```

- Deploy and verify the canary pods of **(Version 2.0)** of **`canary-deployment.yaml`**.

```yaml
kubectl apply -f nginx-canary-deployment.yaml
kubectl get pods -o wide
```

- **Step 5 :** Modify the Service to Route Traffic to Canary Pods for that we need to edit the service to route part of the traffic to **Canary (Version 2.0)**.

```yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx-service
spec:
  type: LoadBalancer
  selector:
    app: nginx
    version: "2.0"
  ports:
    - port: 8888
      targetPort: 80
```

- Apply the Updated Service and test out the deployment by refreshing the webpage to see responses from both **Version 1** and **Version 2**.

```bash
kubectl apply -f nginx-deployment-service.yaml
```

**Step 6 :** Roll Back or Roll Out the Deployment
- **Roll Back** :  If **(Version 2.0)** isn’t working correctly, delete the canary deployment.

```bash
kubectl delete deployment.apps/nginx-canary-deployment
```
- **Roll Out** : If (Version 2.0) works fine, update the service to route all traffic to version 2 and **delete the old version’s deployment** by keeping the new one.

```bash
kubectl delete deployment.apps/nginx
```

---