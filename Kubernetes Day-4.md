# Kubernetes Day-4

let's take an scenario which describes that we have an image which is build layer-by-layer and one of the layer has **"CVE(Common Vulnerability and Exposures)"** and can act as a BackDoor to Hackers.
So for this we need to make sure that our image is secured and for that we use a tool called **"Trivy"**.

### What is Trivy ?
- Trivy is a popular **open-source scanner** designed to **identify vulnerabilities in software**. It's particularly useful in the DevSecOps (development, security, operations) space, where it can be integrated into your CI/CD pipeline to find issues early in the development process.
- **Specific Use of Trivy:**
	- **Vulnerability Scanning:** Trivy can scan container images, code repositories, and cloud environments for known security vulnerabilities.
	- **Misconfiguration Detection:** It can identify misconfigurations in infrastructure as code (IaC) files.
	- **Secret Detection:** Trivy can uncover sensitive information like passwords and API keys accidentally embedded in code or configurations.
	- **SBOM Discovery:** It can help create a Software Bill of Materials (SBOM) which is a list of components used to build software, including their licenses and known vulnerabilities.
	- [**Trivy GitHub Repository**](https://github.com/aquasecurity/trivy) for exploration.

### Installation
 - Trivy Installation on **Debain/Ubuntu**:
```bash
sudo apt-get install apt-transport-https
wget -qO - https://aquasecurity.github.io/trivy-repo/deb/public.key | sudo apt-key add -
echo deb https://aquasecurity.github.io/trivy-repo/deb [CODE_NAME] main | sudo tee -a /etc/apt/sources.list
sudo apt-get update
sudo apt-get install trivy
```

**Note:** *CODE_NAME* - wheezy, jessie, stretch, buster, trusty, xenial, bionic, jammy
- **wheezy:** This is the code name for Debian 7, released in 2013.
- **jessie:** This is the code name for Debian 8, released in 2015.
- **stretch:** This is the code name for Debian 9, released in 2017.
- **buster:** This is the code name for Debian 10, released in 2019.
- **trusty:** This is the code name for Ubuntu 14.04 LTS (Long Term Support), released in 2014. LTS releases are supported for a longer period, typically 3-5 years, making them popular choices for enterprise environments.
- **xenial:** This is the code name for Ubuntu 16.04 LTS, released in 2016.
- **bionic:** This is the code name for Ubuntu 18.04 LTS, released in 2018.
- **jammy:** This is the code name for Ubuntu 22.04 LTS, released in 2022.

---

### How do we scan an particular image ?
- For Scanning a particular image we have a syntax `trivy image <image_name>`. Replace `<image_name>` with the name of the image you want to scan. The image can be a local image name or one stored in a container registry.
```shell
trivy image ubuntu:latest
```

#### Filtering Results:
- **`--severity <severity>`:** This option allows you to filter vulnerabilities by their severity level(critical, high, medium, low).
```bash
# This will only show "critical" vulnerabilities found in the "ubuntu:latest" image.
trivy image --severity critical ubuntu:latest
```

- **`--ignore-unfixed`**: This option instructs Trivy to exclude vulnerabilities without known fixes from the scan results.
```bash
# Here, the output will only show vulnerabilities with available patches or updates. This can be helpful to focus on "actively addressable security issues".
trivy image --ignore-unfixed ubuntu:22.04

# This command scans the current directory (`.`) for vulnerabilities and displays all results, including unfixed ones, but only for vulnerabilities with a severity level of "critical" or "high".
# fs - File system
# --ignore-unfixed=false : explicitly tells Trivy to include unfixed vulnerabilities
trivy fs --ignore-unfixed=false --severity critical,high .
```

#### Output Formatting:
- **`--format <format>`**: This option specifies the output format for the scan results. Trivy supports various formats like JSON, SARIF, and table (default).
```bash
# This will display the scanned result in JSON Format.
trivy image --format json ubuntu:latest
```

#### Additional Options:
- **`--help, -h`**: Shows the help message with all available options and flags.
- **`--version, -v`**: Prints the Trivy version information.
- **`--cache-dir <cache_dir>`**: Specifies a custom location for the Trivy vulnerability database cache.
```bash
# This command scans the `ubuntu:latest` image, but stores the vulnerability database cache in the `/shared/trivy-cache` directory.
trivy image --cache-dir /shared/trivy-cache ubuntu:latest
```
- **`--offline-scan`**: Performs the scan without downloading updates for the vulnerability database.
```bash
# This command scans the current directory (`.`) for vulnerabilities in an offline mode and displays only critical ones. This helps identify the most severe security issues even without updating the vulnerability database.
trivy fs --offline-scan --severity critical .
```

---

### Kind Installation in Kubernetes
- Kind is a tool for running local Kubernetes clusters using Docker container “nodes”. It was primarily designed for testing Kubernetes itself, but may be used for local development or CI.

[**Kind Installation for DIfferent OS**](https://kind.sigs.k8s.io/docs/user/quick-start/#installation)

- For Linux Binary Installation:

```bash
# For AMD64 / x86_64
[ $(uname -m) = x86_64 ] && curl -Lo ./kind https://kind.sigs.k8s.io/dl/v0.22.0/kind-linux-amd64
# For ARM64
[ $(uname -m) = aarch64 ] && curl -Lo ./kind https://kind.sigs.k8s.io/dl/v0.22.0/kind-linux-arm64
chmod +x ./kind
sudo mv ./kind /usr/local/bin/kind
```

- Cluster Creation using `kind`.

```bash
kind create cluster --name rohit
```

![Pasted image 20240504220756](https://github.com/user-attachments/assets/1f82e58a-d937-410e-b58f-b3c98be1977b)

- To see the created Cluster.
```bash
kind get clusters
```

### Helm Installation on Ubuntu

- Download the [**Helm Package**](https://github.com/helm/helm/releases) from the Website.
- After that extract the downloaded package.
```bash
tar xvf helm-v3.14.4-linux-arm64.tar.gz
```

- Move the **`linux-amd64/helm`** file to the **`/usr/local/bin`** directory:
```bash
sudo mv linux-amd64/helm /usr/local/bin
```

- Finally, verify you have successfully installed Helm by checking the version of the software:
```bash
helm version
```
### Helm Uninstallation from Ubuntu
- Remove the downloaded file using the following command:
```bash
rm helm-v3.4.1-linux-amd64.tar.gz
```

- Remove the **`linux-amd64`** directory to clean up space by running:
```bash
rm -rf linux-amd64
```

### Kyverno Installation using Helm

- What is Kyverno ?
	- Kyverno is used for enforcing security and compliance policies within your Kubernetes clusters [**Kyverno Documentation**](https://kyverno.io/docs/).

- To install Kyverno with Helm, first add the Kyverno Helm Repository.
```bash
helm repo add kyverno https://kyverno.github.io/kyverno/
```

- After this we will scan the new repository for charts.
```bash
helm repo update
```


- Now we will install Kyverno (policy engine) with 1 replica in a newly created **`kyverno`** namespace using helm. 
```bash
heml install kyverno kyverno/kyverno -n kyverno --create-namespace --set replicaCount=1
```

![Pasted image 20240504231934](https://github.com/user-attachments/assets/6f7284e4-b0ab-44c3-b7e5-566fd2d51147)

- [**kyverno Policies**](https://kyverno.io/policies/) are collections of rules that define how resources in your Kubernetes cluster should be managed and configured. They act as a safeguard to ensure your cluster adheres to security best practices and compliance requirements.
### Kubectl Installation
-  Let's first install snap on the system.
```bash
sudo apt install snap
```

- Now we will install the Kubectl CLI for the system.
```bash
sudo snap install kubectl --classic
```

- Kubernetes Pod creation using **`Kubectl`**.
```bash
kubectl run before --image nginx -- sleep 1d
```

- To see the created pods in the terminal.
```bash
kubectl get pods
```

- To see the pod all time running.
```bash
kubectl get pods --watch
```

- To describe the pod in kubectl.
```bash
#kubectl describe pod <pod-name>
kubectl describe pod before
```

- We will add the poilcy to the pod by assigning the resources to it for example assignment of memory and storage policy for the pod. 
```bash 
kubectl apply -f policy.yaml
```

- So, after this any pod I created it will assign memory and storage to it.
```bash
# Policy of adding memory and Storage is added automatically due to the policy we have created earlier.
kubectl run after --image nginx --sleep 2d
```

### Why do we prefer validating webhooks over mutating webhooks ?

| Topics                            | Validating Webhooks                                                                                                                                                                                                                     | Mutating Webhooks                                                                                                                                                                                                                                                              |
| --------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| *Predictibility and Transparency* | These webhooks **reject** requests that violate policies before any changes are made to the cluster. This provides clear feedback to the user about why their request was denied and helps maintain a predictable state in the cluster. | These webhooks **modify** requests to comply with policies. While convenient, unexpected modifications can be confusing and potentially lead to unintended consequences.                                                                                                       |
| *Error Handling and Debugging*    | When a request is rejected, the user receives an error message explaining the violation. This allows them to easily diagnose the issue and resubmit a compliant request.                                                                | In case of errors during mutation, debugging can be more challenging. You might need to analyze the webhook's logs to understand why the mutation failed. Additionally, unexpected mutations could introduce cascading issues if dependent resources are modified incorrectly. |
| *Security Considerations*         | These webhooks operate on the original request data, minimizing the attack surface. Malicious actors cannot exploit vulnerabilities in the webhook to modify cluster state.                                                             | These webhooks have more access and control, potentially introducing a security risk if the webhook itself has vulnerabilities. An attacker could exploit these vulnerabilities to inject malicious code or manipulate the cluster state.                                      |
| *Flexibility and Maintainability* | Policies defined in validating webhooks are often simpler and easier to understand. They focus on rejecting non-compliant requests rather than modifying them, leading to cleaner and more maintainable code.                           | Mutating webhooks can become complex, especially when handling edge cases or dealing with multiple policies. This complexity can make them harder to maintain and debug over time.
                                                                                             
### What is Kubelinter ?

- KubeLinter is an open-source command-line tool specifically designed to analyze Kubernetes configurations for potential issues. It acts as a static code analysis tool, meaning it examines your code (in this case, Kubernetes YAML files and Helm charts) without actually running it.
- It acts as a spellchecker for your Kubernetes configurations.
	- **Static Analysis:** KubeLinter scans your Kubernetes YAML files and Helm charts to identify misconfigurations, security vulnerabilities, and potential best practice violations.
	- **Early Detection:** By catching these issues early in the development phase, KubeLinter helps you prevent problems from propagating to production environments.
	- **DevSecOps Integration:** It can be integrated into your CI/CD pipeline to automatically check your Kubernetes configurations before deployment, ensuring a proactive approach to security and best practices.
	- **Customization:** KubeLinter comes with a set of built-in checks, but you can also configure it to create custom checks tailored to your specific needs and organization's policies.
- So overall this is beneficial in improving the security, applying the best practices and streamlining development.
- It suggest the potential error in the YAML file you give to it and suggest the changes regarding to it.

### Installation Guide

#### First install the HomeBrew for installation of Kubelinter.

[**Kube-linter Documentation**](https://docs.kubelinter.io/#/using-kubelinter)
- First we will install the package called as `build-essentials`.
```bash
sudo apt install build-essential
```

- Checking of the compiler availablility on the local system.
```bash
which make
```

- Now we install the latest homebrew for our kubelinter installation.
```bash
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
```

- Installation of Kubelinter.
```bash
brew install kube-linter
```

**How to Use Kube-linter:**
- For example, we have an **yaml file** to which we can **linter** to find the vulnerabilities.
```bash
# kube-linter lint <file-name>
kube-linter lint sample.yml
```

### What is Kube-bench ?
- KubeBench is an open-source tool designed to assess the security of your Kubernetes clusters. This also scans the cluster and provides the solution in **remediation master**.

- **Installation for (Ubuntu/Debian):**
```bash
curl -L https://github.com/aquasecurity/kube-bench/releases/download/v0.6.2/kube-bench_0.6.2_linux_amd64.deb -o kube-bench_0.6.2_linux_amd64.deb

sudo apt install ./kube-bench_0.6.2_linux_amd64.deb -f
```

**After this run kube-bench directly:**

```bash
kube-bench
```

**Binary installation where sudo does't support:**

```bash
curl -L https://github.com/aquasecurity/kube-bench/releases/download/v0.6.2/kube-bench_0.6.2_linux_amd64.tar.gz -o kube-bench_0.6.2_linux_amd64.tar.gz

tar -xvf kube-bench_0.6.2_linux_amd64.tar.gz

./kube-bench --config-dir `pwd`/cfg --config `pwd`/cfg/config.yaml 
```

It does this by:
- **Checking against CIS Kubernetes Benchmarks:** These benchmarks are a set of best practices and security recommendations developed by the Center for Internet Security (CIS). KubeBench compares your cluster configuration to these benchmarks to identify any deviations.
- **Automating Security Checks:** It automates the process of running these checks, saving you time and effort compared to manual configuration reviews.
- **Identifying Security Risks:** By highlighting areas where your cluster configuration doesn't align with CIS benchmarks, KubeBench helps you identify potential security vulnerabilities and areas for improvement.

### What is Back-off Algorithm in k8s?

- Back-off Algorithm is applied to every components of kubernetes. The backoff algorithm in Kubernetes helps manage the retry behavior for pods and jobs, preventing constant, rapid retries that could overwhelm the system. 
- By introducing exponential delays between retries, Kubernetes ensures a more stable and manageable approach to handling pod failures.
- The delay lookalike 10sec, 20sec, 40sec, 80sec, 160sec ... till 600sec.

---
