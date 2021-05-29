# Certified Kubernetes Security Specialist

CKS Exam Topics Breakdown:
```
Cluster Setup - 10%
Cluster Hardening - 15%
System Hardening - 15%
Minimize Microservice Vulnerabilities - 20%
Supply Chain Security - 20% 
Monitoring, Logging and Runtime Security - 20%
```

#### 4 C's of Cloud Native Security:
- Cloud: Cloud Security - Infrastructure that hosted in the cloud (firewalls could have been avoided the attack)
- Cluster: Cluster Security - K8s API, Docker Daemon and Dashboard, others were not secured
- Container: Container Security - Privileged Containers, Run any Image
- Code: Code Security - hardcoding credentials, passing using environment variables, exposing without tls

### Cluster Setup and Hardening

#### TLS in Kubernetes:
Certificates API is managed by Controller-Manager(on master node). 
It has CSR-Signing & CSR-Approving controllers that are responsible for signing & approving certificates 
It also has the ca public & private keys that are used to sign certificates.

To send a CSR for signing, a yaml file is generated(which includes the CSR) & submitted 
kubectl get csr -> view CSRs 
kubectl certificate approve jane -> approve certificate 
kubectl get csr jane -o yaml -> view approved certificate 
kubectl deny csr agent-smith -> deny csr 
kubectl delete csr agent-smith -> delete csr

Note: the CSR K8s object should include the the csr in base64 encoded format
Also, once the csr is approved, the approved certificate is in the form of base64 encoded certificate in the command output (kubectl get csr jane -o yaml)

Kube Config: context is the combination of user and cluster name. 
Each config file has a default context in the name of current-context 
kubectl config view -> points to default config which is /root/.kube/config 
kubectl config view –kubeconfig=/root/my-kube-config 
kubectl config –kubeconfig=/root/my-kube-config use-context research -> change context

#### API Groups:
/api - core (all core functionality) - /v1 - namespaces, pods, rc, events, nodes, pv, pvc etc
/apis - named (new features) - /apps, /extensions, /networking.k8s.io, /authentication.k8s.io, /certificates.k8s.io

All the above named objects are called API Groups.

/apps   /v1     /deployments –> list, get, create, update, delete, watch /replicasets /statefulsets
Above deployments, replicasets, stateful sets are called Resources & each resoource has actions called Verbs (such as list, get, create, update, delete, watch) 

#### Authorization: Authorization modes as specified as a flag on the kube-apiserver

Heirarchy: NODE -> RBAC -> WEBHOOK

Types:
- Node (Node Authorizer)
- ABAC 
- RBAC
- Webhook
- AlwaysAllow, AlwaysDeny

ABAC: Attribute Based Access Control
{"kind": "Policy", "spec": {"user": "dev-user", "namespace": "*", "resources": "pods", "apiGroup": "*"}}
{"kind": "Policy", "spec": {"user": "security-user", "namespace": "*", "resources": "csr", "apiGroup": "*"}}

Everytime, add or change in security such as csr, the policy need to be edited manually and kube-api server needs to be restarted
For this reason, we go for RBAC

Webhook:
Open Policy Agent is a 3rd party tool that help AdmissionControlAuthorization
Whenever a user request comes in, kube-apiserver makes a call to OPA to check if the user can be allowed or not

#### RBAC: kubectl get roles kubectl get rolebindings
Create a Role: creates the role with necessary operations 
################################################################ 
apiVersion: rbac.authorization.k8s.io/v1 
kind: Role 
metadata:   
    name: developer 
rules:
- apiGroups: [""] # for core group, this can be blank, for anything else it should be specified
  resources: ["pods"]
  verbs: [“list”, “create”, “get”, “update”, “delete”]
- apiGroups: [””]   
  resources: [“ConfigMap”]   
  verbs: [“create”] 
################################################################
  Create a RoleBinding: ties the user to the role

apiVersion: rbac.authorization.k8s.io/v1 
kind: RoleBinding 
metadata:   
    name: devuser-developer-binding
subjects:
- kind: User   
  name: dev-user   
  apiGroup: rbac.authorization.k8s.io 
roleRef:   
- kind: Role   
  name: developer   
  apiGroup: rbac.authorization.k8s.io 
################################################################
Check Access: kubectl auth can-i create deployments output: yes/no 
kubectl auth can-i delete nodes no 
We can also impersonate a user to check whether access was granted correctly or not: 
kubectl auth can-i create deployments –as dev-user yes 
kubectl auth can-i create pods –as dev-user yes 
kubectl auth can-i create pods –as dev-user –namespace test no 

We can also give access to specific resources using resourceNames attribute: 
resourceNames: ["blue","orange"]
This will grant access only to blue and orange pods

kubectl create role developer –verb=list,create –resource=pods –dry-run -o yaml > developer.yaml 
kubectl create rolebinding dev-user-binding –role=developer –user=dev-user –dry-run -o yaml > developerrb.yaml

ClusterRoles & ClusterRoleBindings: resources are scoped as namespaced or cluster-scoped

Namespaced Scope: pods, rs, jobs, deployments, services, secrets, role, rolebindings, pvc, configmaps 
Issue the following command to get all the resources that are namespace scope: kubectl api-resources –namespaced=true

Cluster Scope: nodes, PV, clusterrole, clusterrolebinding, namespaces, certificatesigningrequests 
Issue the following command to get all the resources that are cluster scope: kubectl api-resources –namespaced=false

Sample:
kind: ClusterRole apiVersion: rbac.authorization.k8s.io/v1 metadata:   name: node-admin rules:

apiGroups: [””]   resources: [“nodes”]   verbs: [“get”, “watch”, “list”, “create”, “delete”]
kind: ClusterRoleBinding apiVersion: rbac.authorization.k8s.io/v1 metadata:   name: michelle-binding subjects:

kind: User   name: michelle   apiGroup: rbac.authorization.k8s.io roleRef:   kind: ClusterRole   name: node-admin   apiGroup: rbac.authorization.k8s.io
kubectl create clusterrole storage-admin –-verb=get,watch,create,list,delete –-resource=persistentvolumes,storageclasses –dry-run -o yaml > storageadmin.yaml 
kubectl create clusterrolebinding michelle-storage-admin –-clusterrole=storage-admin  –-user=michelle  –dry-run -o yaml > storageadminrb.yaml


#### Kubelet Security: 
kubelet configuration file -> kubelet-config.yaml/config.yaml
Note: kubeadm does not deploy the Kubelet

issues:
a) anonymous authentication
b) authentication using certificates
c) authorization mode
d) read-only-port

10250 - serves API that allows full access
10255 - serves API that allows unauthenticated read-only access

--anonymous-auth=false or kubelet-config.yaml authentication->anonymous-> enabled:false

supported authentication: Certificates and API Bearer Tokens

Certificates:
Authentication:
Without any authentication, we can access the kubelet API using:
curl -sk https://localhost:10250/pods 
--client-ca-file=/path/to/ca.crt or kubelet-config.yaml: authentication->x509->clientCAFile
Once the above setting is in place, we have to pass in the key and cert to access the kubelet API
curl -sk https://localhost:10250/pods -key kubelet-key.pem -cert kubelet-cert.pem

Also the kube-apiserver needs the kubelet-client-certificate and kubelet-client-key certs to talk to kubelet

If any of the above mechanism reject a request, the default behavior of kubelet is to allow the request as anonymous and unauthenticated

Authorization:
Default authorization mode is set to AlwaysAllow
Set the authorization mode to Webhook -> this will make a call to Kube-apiserver to decide the authorization of the request

On port 10255, this is set a read-only-port on kubelet.service/kubelet-config.yaml
To change, set this port to 0 in the kubelet.service/kubelet-config.yaml

#### Kubectl proxy:
Using kubectl proxy, we can access the kube-apiserver locally on port 8001
kubectl proxy
    Starting to serve on 127.0.0.1:8001

#### Kubectl port-forward:
kubectl port-forward pod/service localhost_port:pod/service_port
kubectl port-forward service/nginx 28080:80
service/nginx is running on 80 as ClusterIP
we can access it using http://localhost:28080/ using the above port forward command

#### Kubernetes Dashboard:
Securing Dashboard: Limit the serviceaccount that uses the kubernetes dashboard to specific namespace or tie it to a view clusterrole

#### Verify Downloaded Kubernetes Binaries:
use checksum to verify
To download curl https://dl.k8s.io/v1.21.1/kubernetes.tar.gz -L -o kubernetes.tar.gz
on Mac: shasum -a 512 kubernetes.tar.gz
On Linux: sha512sum kubernetes.tar.gz

#### Kubernetes Software Versions:
1.21.3
Major.Minor.Patch

Minor -> Features/Functionalities
Patch -> Bug Fixes

alpha-release: bug fixes & improvements, features are disabled by default and are buggy 
beta-release: code tested and new features turned on After beta-release, its merged to a stable release

Control plane components with same version: api-server, controller, scheduler, proxy, kubelet, kubectl 
ETCD cluster & CoreDNS have their own versions as they are separate projects 

https://kubernetes.io/docs/concepts/overview/kubernetes-api/

kube-apiserver > controller & scheduler > kubelet & kube-proxy
  X                 X-1                       X-2
(1.10)           (v1.9 or v1.10)      (v1.8 or v1.9 or v1.10)
kube-bench is cis benchmark tool from aqua security

kubectl can be higher than the kube-apiserver

kubernetes only supports upto latest 3 minor versions Lets say you are on 1.10 and a new version 1.11 & 1.12 are available. K8s will support only 1.10, 1.11 and 1.12. This means when K8s releases 1.13, 1.10 will be unsupported

#### Cluster upgrade process using kubeadm:
Master Node Upgrade Steps:
kubeadm upgrade plan -> lists out current and latest k8s version
upgrade kubeadm
  apt-get upgrade -y kubeadm=1.12.0-00
upgrade kubernetes control components using upgraded kubeadm
  kubeadm upgrade apply plan v1.12.0
upgrade kubelets
  apt-get upgrade -y kubelet=1.12.0-00

Worker Node Upgrade Steps:
kubectl drain node-01 
apt-get upgrade -y kubeadm=1.12.0-00 
apt-get upgrade -y kubelet=1.12.0-00 
kubeadm upgrade node config –kubelet-version v.12.0 
systemctl restart kubelet 
kubectl uncordon node-01

Lab Commands: apt update, apt install kubeadm=1.19.0-00 and then kubeadm upgrade apply v1.19.0 and then apt install kubelet=1.19.0-00

Note: Version shown in kubectl get nodes is the version of the kubelet present on the nodes  


#### Network Policy:

Supported Selectors: podSelector, namespaceSelector, ipBlock

#### Ingress: L7 Load Balancer


#### Securing Docker Daemon:
we can start docker in debug mode for troubleshooting: dockerd --debug
By Default Docker daemon is not exposed, if there is a need to expose we need to protect it
By Default when docker daemon is started, its opens a unix socket at /var/run/docker.sock
A unix socket is an IPC(Inter process communication), that is used for communication between different processes on same host.
This means Docker Daemon is accessible on same host and Docker CLI is configured to talk to the dockerdaemon via this socket.
Enable TLS Encryption followed by Authentication (enable tlsverify and tlscacert)
Once Authentication is enabled, all clients need to have a key-pair along with cacert to access the docker Daemon
Also, clients need to use tls options (DOCKER_HOST,--tls, --tlscert, --tlskey, --tlscacert) to connect to remote Docker Daemon or we can drop the certs into .docker under users home directory

2375 -> unencrypted
2376 -> encrypted

All the above options can be specified in /etc/docker/daemon.json or passed to the docker daemon while startup as flags
With Authentication:
{
  "hosts": ["tcp://192.168.1.10:2376"]
  "tlscert": "/var/docker/server.pem",
  "tlskey": "/var/docker/serverkey.pem",
  "tlsverify":"true"
  "tlscacert":"/var/docker/caserver.pem"
}

Without Authentication:
{
  "hosts": ["tcp://192.168.1.10:2376"]
  "tlscert": "/var/docker/server.pem",
  "tlskey": "/var/docker/serverkey.pem",
  "tls":"true"
}

### System Hardening:

Limit Node Access\
RBAC Access\
Remove Obsolete Packages & services\
Restrict Network Access\
Restrict Obsolete Kernel Modules\
Identify and Fix Open Ports\

Limit Node Access:
- Create cluster in private network and allow access to Cluster via VPN
- Restrict a particular range of IPs using firewalls

Types of Accounts:
```
User Account
SuperUser Account (root)
System Account (mail, ssh)
Service Account (nginx, http)
```

Commands:
id, who, last - last logged in of a user

Access Control Files:
```
/etc/passwd: has user info
/etc/shadow: has password
/etc/group: has all group info
```

Remove user :  
update a default shell to nologin shell:\
usermod -s /bin/nologin michael -> disables shell login for michael\
usermod -s /usr/sbin/nologin himanshi -> disables shell login for himanshi\
userdel michael\

Remove a user from a group:  
userdel michael admin -> removes michael from 'admin' group

SSH Hardening:

- Use SSH Key pair, ssh public key is installed in users home directory under .ssh/authorized_keys
- Disable ssh for root account (this will allow to login with users own account)

Disable rootlogin and password authentication:\
- Edit /etc/ssh/sshd_config with following parameters
  - PermitRootLogin no 
  - PasswordAuthentication no
- Save the file and restart sshd service ->systemctl restart sshd

User Privilege Escalation:
Preferred way to run commands is to use sudo

Sudoers file can be edited in two ways:
- use visudo
- vi /etc/sudoers 

user/group/ hostname/user:group the command to be run / command
jim     ALL=(ALL:ALL) ALL -> sudo with password
jim     ALL=(ALL:ALL) NOPASSWD:ALL -> sudo without password
%admin  ALL=(ALL:ALL) ALL -> all users in admin group will have sudoers access

Remove Obsolete Packages and Services:\
- Keep the system as lean as possible by making sure that only the required software is installed
- Update the installed packages constantly to address security fixes.

```
systemctl list-units --type service
systemctl stop apache2
systemctl disable apache2

remove unit file of the nginx service:  
rm /lib/systemd/system/nginx.service
```

Restrict kernel modules:  
```
list kernel modules:  
lsmod

load a kernel module: (needs to run as root user)
modprobe pcspkr (loads pckspkr kernel module)

blacklist a kernel module:
/etc/modprobe.d/blacklist.conf
add this line: blacklist evbug
restart the system (shutdown -r now)
list kernel modules & search for the blacklisted kernel module to verify (blacklisted kernel module should be absent)
```
check for ports in services / port-service mappings:  
cat /etc/services | grep -i 53

UFW: Uncomplicated Firewall. 
ufw is a wrapper for iptables
```
check ufw status:  
ufw status

enable/disable/reset firewall:  
ufw enable
ufw disable
ufw reset - to reset firewall rules

allow ssh connections from a a jump service:  
ufw allow from 172.16.238.5 to any port 22 proto tcp

allows traffic on port 22:  
ufw allow 22

denies traffic on port 80:  
ufw deny 80

delete a rule using number (5 being the line number on the rules (from ufw status command)):  
ufw delete 5 

show firewall rules with numbers:  
ufw status numbered

allow a tcp port range between 1000 and 2000 in ufw:  
ufw allow 1000:2000/tcp
```
Linux Syscalls:  
strace touch /tmp/error.log
strace -c touch /tmp/error.log
strace -p 3596
