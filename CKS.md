# Certified Kubernetes Security Specialist

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
Node (Node Authorizer), ABAC, RBAC, Webhook, [AlwaysAllow, AlwaysDeny]

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
