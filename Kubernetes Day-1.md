# Kubernetes Day-1

### What is YAML ?
YAML is a digestible data serialization language often used to create configuration files with any programming language. It is widely used in Docker, Kubernetes etc.
- #### What is Data Serialization ?
	- Serialization is basically a process of converting data objects which is a combination of code + data into series of byte which saves the state of object in the form that is easily transmitted which can easily go into YAML File, Database or in Memory and after this Deserialization take place.
	- We can also use other Data Serialization languages such as **JSON** or **XML**.

![Pasted image 20240416065149](https://github.com/rohit-rajput1/PokeQuest/assets/76991475/192c0667-c027-429c-8307-93d0349b4734)

- Is **YAML Case-Sensitive** ? -> Yes it is Case Sensitive.
- YAML is made up of 4 basic things which are (simple english , :  , -, space)
- Best Advantage of YAML is that it is a Human Readable language which is used to represent data.

To make your YAML Syntactically Correct we use [YAML Lint](https://www.yamllint.com/)

![Pasted image 20240416072517](https://github.com/rohit-rajput1/PokeQuest/assets/76991475/1baae86e-e032-4e54-883c-bf51f4b68de3)

---
### Some Important Commands in Linux
- `ps aux` - It is used to list all the information related to running processes.
- `pstree` - It is used to display a tree diagram of process and visually represent the parent-child relationship between processes.

![Pasted image 20240416074133](https://github.com/rohit-rajput1/PokeQuest/assets/76991475/5a5d6946-881f-4db9-afe5-6124036c0ff9)

#### What is base64 in Linux ?
-   In Linux, Base64 is a method used to **encode binary data** (data that contains non-printable characters) into ASCII characters.
- This is very useful in Kubernetes Secret.
```bash
echo "rohit"  |  base64
# Output - > cm9oaXQK
```
- for decoding the string we use decode.
```bash
echo "cm9oaXQK" | base64 -d
# Output -> rohit
```

- `tcpdump` - It is a command-line packet analyzer which allows user to capture and analyze network traffic going through network interface.

![Pasted image 20240416074642](https://github.com/rohit-rajput1/PokeQuest/assets/76991475/008c9c0b-3844-4e24-9e5c-60aa6e63d2f2)

```bash
# To capture only limited amout of packet in tcpdump
sudo tcpdump -c 10
```

![Pasted image 20240416075016](https://github.com/rohit-rajput1/PokeQuest/assets/76991475/2ab1aaad-73ff-42ff-9c59-86f31ec4a01b)

- To save this `tcpdump` in particular file called rohit.txt and `-w ` is used for saving the previous output to the next.
```bash
sudo tcpdump -c 2 -w rohit.txt
```

- To execute you need to give your directory permission but be cautious in production environment.
	- To give a directory read, write, and execute (rwx) permissions in Unix-like systems, you can use the `chmod` command followed by the appropriate permission mode.

```bash
# chmod a+rwx /document/rohit.txt
chmod a+rwx rohit.txt
```

- To save our network interface packets in .txt file using **tcpdump**.

```bash
sudo tcpdump -c enp1s0 2 -w rohit.txt
# note: this enp1s0(basically network interface) can be different in different systems.
# sudo tcpdump -c enp1s0 2 > rohit.txt (Method 2)
```

#### Important Commands:

- `crictl` - It is a command which is used for interacting directly with contaibner runtime and supports Container Runtime Interface (CRI).
- `ctr` -  It is used only used for **containerd**.
```bash
# To list all process in container
crictl ps -a
# To see logs
crictl logs <container-id>
```

- `journalctl` - It is used for querying and displaying logs from the systemd journal and systemd is stored in **/etc/systemd**.
```bash
# To get entire Journal
sudo journalctl

# To get journal in one-pager format
sudo journalctl --no-pager

# To get logs from yesterday
sudo journalctl --since yesterday

# To jet journal for specific time period
journalctl --since "2024-04-15 00:00:00" --until "2024-04-16 00:00:00"

# To get realtime logging
journalctl -f

# Output in JSON format
journalctl -o json

# To get by using filter (e.g., emerg, alert, crit, err, warning, notice, info, debug).
journalctl -p err
```

![Pasted image 20240416111343](https://github.com/rohit-rajput1/PokeQuest/assets/76991475/57cc35b8-b8ad-47bf-826d-6a43fd9a9d7a)

![image](https://github.com/rohit-rajput1/PokeQuest/assets/76991475/00461084-16c6-46d6-9a66-63b874703918)

---
### Release Cycles of Kubernetes
- [Release Cycles of Companies](https://endoflife.date/)
---
### Installation of Kubernetes:

**Kubernetes Installation via Docker:**

![Pasted image 20240214190247](https://github.com/rohit-rajput1/PokeQuest/assets/76991475/6eea2b0b-d84f-409a-b038-0a6cacc12518)

**How to Start Minikube with more CPUs Compute Engine:**

```bash
minikube start --cpus=4
```

![Pasted image 20240214192040](https://github.com/rohit-rajput1/PokeQuest/assets/76991475/bf0d7b2b-5b7b-4c50-a5e6-af2778f96e1d)

**How to Increase memory allocation to cluster :**

```bash
minikube start --memory=4096
```

**How to stop Minikube Cluster :**

```bash
minikube stop
```

![Pasted image 20240214191101](https://github.com/rohit-rajput1/PokeQuest/assets/76991475/ebcb74bc-42a8-4ec7-a260-bf9e4dd7a12f)

**How to check no. of cpus in ubuntu for K8s :**
```
nproc

cat /proc/cpuinfo | grep "processor" | wc -l
```

![Pasted image 20240214191539](https://github.com/rohit-rajput1/PokeQuest/assets/76991475/61ed3b72-befd-4b18-a90f-b6a4ce82bdd6)

**How do we delete cluster in K8s:**

```bash
minikube delete
```

![Pasted image 20240214214629](https://github.com/rohit-rajput1/PokeQuest/assets/76991475/9bced533-2498-4865-ab31-a97c6c4e89ee)

**How do we start another minikube cluster:**

```bash
minikube start -p my-second-cluster

# this will list all the cluster
minikube profile list
```

![Screenshot from 2024-02-14 21-57-01](https://github.com/rohit-rajput1/PokeQuest/assets/76991475/a08d7826-01aa-4360-b54b-6b1ce155961b)

### If Error occuring while installation process:

If you get this error and still running cluster:

![Pasted image 20240214215214](https://github.com/rohit-rajput1/PokeQuest/assets/76991475/2ecd6fc8-12fe-4485-a02a-a650759da261)

**Solution of this:**

- Check Docker Context:
```bash
docker context ls
```

- Switch Context:
```bash
docker context use default
```

- If context not there , then create one:
```bash
docker context create default
```

- Restart Docker:
```bash
sudo systemctl restart docker
```

![Screenshot from 2024-02-14 21-50-49](https://github.com/rohit-rajput1/PokeQuest/assets/76991475/b4452971-1266-404a-bd17-4a90aef8eaaa)

---
#### According to Kubernetes, it is a Open-Source Container orchestration tool, but Kubernetes is all about Controllers.

**Due to following reason:**
- Kubernetes manages containerized applications, being an Open-Source Container orchestration tool.
- But, the real magic lies in Kubernetes Controllers. These act like control loops, constantly monitoring and adjusting the state of your applications to match your desired state. Imagine a thermostat for your applications - that's essentially a controller in action.

![image](https://github.com/rohit-rajput1/PokeQuest/assets/76991475/636fc215-6ed4-4cfd-901a-9a0abe7890f1)

---

`**Bonus:**` You can use [KillerKoda](https://killercoda.com/) for practicing Kubernetes.