# PFE-Demo-Guide :bulb:
In this guide, full instructions will be disposed to put in place the following steps : 

- Setting up an end-to-end pipelines to deploy microservices
- Implementing GitOps methodology using the argoCD tool
- Implementing Blue/Green and Canary deployments with argo rollouts 
- Defining network policies using the Cilium tool
- Defining namespace quotas
- Managing roles and role bindings as code 
- Enforcing Kyverno/OPA policies for security and compliance
- Integrate Keycloak for authentication to Kubernetes 
- Using external secrets operator to fetch secrets from Vault/Azure Key vault 
- Connecting OpenFeature with a running application Set up monitoring via Victoria Metrics

# Requirements
- An ubuntu physical / virtual machine

- Internet connection xd

In order to put in action all these steps, we need a kubernetes environment, since we can work on a local environment, we chose the kubernetes minikube cluster as an infrastructure.

### Setup Kubernetes (Minikube) : 

`curl -LO https://github.com/kubernetes/minikube/releases/latest/download/minikube-linux-amd64`

`sudo install minikube-linux-amd64 /usr/local/bin/minikube && rm minikube-linux-amd64`

### Start the cluster 
`minikube start`

### Check the minikube version
`minikube version`

### Get access to the minikube dashboard
`minikube dashboard`

### Deploy a service using minikube command (Advanced)
`Minikube service <service_name> `

Now that minikube is set up, we need kube CLI to execute kubernetes commands 

### KubeCLI Installation:

`curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"`


`sudo install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl`

### Check KubeCLI version

`kubectl version --client`

## 1. Setting up an end-to-end pipelines to deploy microservices
In this step, we will be using Jenkins as a CI tool, since its an open source Java tool, we need to set up *JDK* first

### JDK 11 Installation

#### Update the system

`sudo apt update`

#### Install the default JDK

`sudo apt install default-jdk`

In order to add the JDK to the environment variables, we should follow these steps:

1. Open System configuration file /etc/environment :
   
`sudo nano /etc/environment`

2. Add environment variable in /etc/environment :
   
`JAVA_HOME="/usr/lib/jvm/java-11-openjdk-amd64/"`

3. Apply modifications :
   
`source /etc/environment`

4. Verify if JAVA_HOME is configured correctly:
   
`echo $JAVA_HOME`

Now that we finished setting up the JDK, we will install Jenkins

### Jenkins Installation

`wget -q -O - https://pkg.jenkins.io/debian-stable/jenkins.io.key | sudo apt-key add -
sudo sh -c 'echo deb http://pkg.jenkins.io/debian-stable binary/ >
/etc/apt/sources.list.d/jenkins.list'`

`sudo apt-key adv --keyserver keyserver.ubuntu.com --recv-keys 5BA31D57EF5975CA`

`sudo apt update`

`sudo apt install jenkins`

`sudo systemctl start jenkins`

`sudo systemctl enable jenkins`

To access jenkins, you simply need to put your machine's ipv4 address with the following port number  IP@:8080 

The default username and password for Jenkins are : Admin/Admin

## 2. Implementing GitOps methodology using the argoCD tool

In this section, we will need to install the argoCD tool and access its UI

### Argo CD Installation

`kubectl create namespace argocd `

`kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml`

### Port Forwarding 

`kubectl port-forward svc/argocd-server -n argocd 8080:443`

### Get ArgoCD Password (Username : Admin)

`kubectl get secret argocd-initial-admin-secret -n argocd -o yaml`

`echo <argocd_pw> | base64 --decode`

## 3. Defining network policies using the Cilium tool

In this section, we will install the cilium tool as well the observability tool for cilium which is called Hubble

### Cilium Installation 

`cilium install`

You can check the cilium status with the wait flag to see the full process of cilium pods creation and activation

`cilium status --wait`


### Cilium CLI Installation

`CILIUM_CLI_VERSION=$(curl -s https://raw.githubusercontent.com/cilium/cilium-cli/main/stable.txt)
CLI_ARCH=amd64
if [ "$(uname -m)" = "aarch64" ]; then CLI_ARCH=arm64; fi
curl -L --fail --remote-name-all https://github.com/cilium/cilium-cli/releases/download/${CILIUM_CLI_VERSION}/cilium-linux-${CLI_ARCH}.tar.gz{,.sha256sum}
sha256sum --check cilium-linux-${CLI_ARCH}.tar.gz.sha256sum
sudo tar xzvfC cilium-linux-${CLI_ARCH}.tar.gz /usr/local/bin
rm cilium-linux-${CLI_ARCH}.tar.gz{,.sha256sum}`


### Hubble Client

`HUBBLE_VERSION=$(curl -s https://raw.githubusercontent.com/cilium/hubble/master/stable.txt)
HUBBLE_ARCH=amd64
if [ "$(uname -m)" = "aarch64" ]; then HUBBLE_ARCH=arm64; fi
curl -L --fail --remote-name-all https://github.com/cilium/hubble/releases/download/$HUBBLE_VERSION/hubble-linux-${HUBBLE_ARCH}.tar.gz{,.sha256sum}
sha256sum --check hubble-linux-${HUBBLE_ARCH}.tar.gz.sha256sum
sudo tar xzvfC hubble-linux-${HUBBLE_ARCH}.tar.gz /usr/local/bin
rm hubble-linux-${HUBBLE_ARCH}.tar.gz{,.sha256sum}`



### Hubble Enabling

You can simply enable hubble using the cilium CLI and then retype the status command as it will diplay hubble status in addition to the previous status

`cilium hubble enable`

`cilium hubble port-forward&`

`hubble status`

To access the hubble UI, 

`cilium hubble disable`

`cilium hubble enable --ui`

`cilium hubble ui`





