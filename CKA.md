# Certified Kubernetes Administrator

 
CKA Exam Topics Breakdown:
```
Core Concepts 19% - done
Security 12% - wip
Installation, Configuration & Validation 12%
Cluster Maintenance 11% - done
Networking 11%
Application Lifecycle Management 8% - done
Storage 7%
Scheduling 5% - done
Logging / Monitoring 5% - done
Troubleshooting 10%
```

Certified Kubernetes Administrator: https://www.cncf.io/certification/cka/

Exam Curriculum (Topics): https://github.com/cncf/curriculum

Candidate Handbook: https://www.cncf.io/certification/candidate-handbook

Exam Tips: http://training.linuxfoundation.org/go//Important-Tips-CKA-CKAD

https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#version
https://kubernetes.io/docs/reference/kubectl/cheatsheet/
 
K8s the hard way: https://github.com/mmumshad/kubernetes-the-hard-way
 
CORE CONCEPTS:
 
Cluster Architecture:

Control plane components
    - API-Server: stores info about cluster; a management component which manages all the below components
    - Controller-Manager:
       - node-controller: controls the nodes
       - replication-controller: controls the containers/replicasets
    - Scheduler: identifies the right node to schedule the nodes/pods/containers
    - ETCD cluster: highly available key-value store

kubelet contacts the api-server constantly, delivering the status of the node/containers

kube-proxy ensures necessary rules are in place to contact each other (nodes/containers); communication between services of different/same worker-nodes
 
ETCD:
1) Download the etcd binaries from official github page
2) Extract the downloaded tarball
3) Run the etcd service (./etcd) --> listens on 2379 by default
default client for etcd is etcdctl
 
options --advertise-client-urls is important because this is what etcd listens on; also this is what is configured on kube-apiserver
 
two types of setups:
1) manual - everything should be downloaded and installed from scratch
2) kubeadm install - kubeadm will install all the dns, etcd, kube-api-server & other components
 
kube-api-server:
In a kubeadm setup:
kube-api server yaml file location -> /etc/kubernetes/manifests/kube-apiserver.yaml --> when done using kubeadm setup
In a non-kubeadm setup:
we can inspect the service using the service: cat /etc/systemd/system/kube-apiserver.service
 
option --etcd-servers is where we specify the etcd server location in the form of a url
 
Controller-Manager: continuously monitors all the components in the system and bring them to desired state
1) watch status
2) remediate situation
Node-Controller: responsible for monitoring the status of nodes
node monitor period: 5 seconds ---> checks nodes every 5 seconds
node monitor grace period: 40 seconds ----> waits for 40 seconds before node is marked as down
pod eviction timeout: 5 minutes ----> waits for 5 minutes before scheduling the pods on bad node to a healthy node
 
Replication-Controller: same as above but schedules pods instead of nodes
 
all different type of controllers are packaged together into a single process called kubernetes-controller-manager
 
check for the option: -> controllers StringSlice to see which controllers are enabled
 
for kubeadm installs:
cat /etc/kubernetes/manifests/
for non-kubeadm installs:
view the configuration as a service located in /etc/systemd/system/service-name
 
we can also view the options by checking for the running process:
ps aux | grep -i controller-manager
 
Kube-Scheduler: schedulers only decide which pod goes into where, it is the kubelet who deploys the pods based on the input of scheduler
Scheduler uses the following rules to place a pod appropriately:
1) Filter nodes: check which nodes have sufficient resources
2) Rank Nodes: rank the nodes accordingly which has more resources after the pod is placed
 
Other concepts:
resource requirements and limits
taints and tolerations
node selectors/affinity
 
Kubelets:
A Kubelet  on worker-node does the following tasks:
a) registers the node
b) creates the pods as suggested by scheduler
c) maintain node and pods
d) report the status back to the kube-api-server
 
kubelet is not installed by kubeadm
 
Kube-proxy: is a process that runs on each node in k8s cluster which takes care of the services to direct traffic to backend pods  which are behind the services. It does so by using the iptables rules
In k8s architecture, a pod can communicate with another pod on a different node using a private-network which is pod-network.
It is a internal virtual network that spans across the cluster
 
Replicaset will consider the pods that are created before with the same label
 
selector is the difference between replicaset and replication-controller.
Selector is mandatory while specifying a replicaset
 
 
Namespaces:
default: by default all resources are created here
kube-system: namespace for kubernetes resources such as networking, dns, etc
kube-public: resources in this namespaces are made available to everyone
 
To connect to a service in another namespace, use the following:
mysql.connect("db-service.dev.svc.cluster.local")
 
syntax: service.namespace.subdomain.domain
 
cluster.local -> default domain of the cluster
svc -> subdomain for service
dev -> namespace
db-service -> service name
The reason we can access services in different namespaces is because whenever services are created, services will have their dns entries create at kubernetes global level
 
kubectl get pods --all-namespaces
kubectl get namespaces
kubectl get pods -n dev
kubectl get pods
 
create pods in different namespace
kubectl create -f pod-definition.yaml --namespace=dev
 
namespace attribute can be added in the metadata section of the pod definition, that way we dont have to add it on commandline
###################################
#namespace-dev.yaml
#create a namespace with name "dev"
apiVersion: v1
kind: Namespace
metadata:
            name: dev
###################################
 
#create namespace "dev" using commandline:
kubectl create namespace dev
 
we can set namespaces by default using context switching so that we dont have to mention namespaces all the time:
switching namespaces:
kubectl config set-context $(kubectl config current-context) --namespace=dev
 
#creating ResourceQuota for namespaces
Compute-quota.yaml
######################################################################
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
######################################################################
 
Services:
 
 
SCHEDULING:
 
Manual Scheduling:
nodeName is the property which decides the pod where to go
nodeName is auto-assigned by scheduler during creation of the pod by creating a bind object to the node
If no scheduler is available, pods will be in pending state. To assign a pod we can manually assign pods to a node by add the nodeName in the pod definition file.
We cannot assign existing pods to new node is to creating a binding object and sending a curl post request to the binding api with the data set to the binding object in json format
######################################################################
apiVersion: v1
kind: Pod
metadata:
  name: nginx
spec:
  containers:
  -  image: nginx
     name: nginx
  nodeName: node01
######################################################################
 
Labels and Selectors:
Labels and Selectors are used to group & select k8s objects
Labels are added in the metadata section of the k8s object
Selectors are added in the spec section of the k8s object
 
Annotations: Annotations are used to record other details for informative purposes
example: name, build information, version
These are specified under metadata section of the replicaset/pod
 
kubectl get pods --selector env=dev
#get all k8s objects(pods, replicasets, deployments) with selector prod env
kubectl get all --selector env=prod
#get pods with selectors prod environment & finance business unit & frontend tier
kubectl get pods --selector env=prod,bu=finance,tier=frontend
 
 
Taints & Tolerations:
Taints are set on Nodes
Tolerations are set on Pods
 
syntax:
kubectl taint nodes node-name key=value:taint-effect
 
taint-effect options:
NoSchedule: no new pods will be scheduled
PreferNoSchedule: try not to schedule new pods but not guaranteed
NoExecute: no new pods will be scheduled and existing pods will be evicted if they dont tolerate the taint
 
Example:
kubectl taint nodes node1 app=blue:NoSchedule
 
Sample Tolerations yaml file:
######################################################################
apiVersion: v1
kind: Pod
metadata:
            name: myapp-pod
spec:
  containers:
  - name: nginx-container
    image: nginx
  tolerations:
  - key:"app"
    operator:"Equal"
            value:"blue"
            effect:"NoSchedule"
######################################################################
 
Removing taint on master:
master $ kubectl describe node master | grep -i taint
Taints:             node-role.kubernetes.io/master:NoSchedule
 
master $ kubectl taint nodes master node-role.kubernetes.io/master:NoSchedule-
node/master untainted
master $ kubectl describe node master | grep -i taint
Taints:             <none>
 
 
NodeSelectors:
######################################################################
apiVersion: v1
kind: Pod
metadata:
            name: myapp-pod
spec:
  containers:
  - name: nginx-container
    image: nginx
  nodeSelector:
            size: Large
######################################################################
 
Before applying nodeselectors, nodes should be properly labeled
 
how to label a node:
Syntax:
kubectl label nodes <node-name> <label-key>=<label-value>
 
Example:
kubectl label nodes node01 size=Large
 
If the requirement to place the pod is complex, nodeaffinity is a best fit to take a look
 
NodeAffinity:
######################################################################
apiVersion: v1
kind: Pod
metadata:
            name: myapp-pod
spec:
  containers:
  - name: nginx-container
    image: nginx
  affinity:
    nodeAffinity:
              requiredDuringSchedulingIgnoredDuringExecution:
                nodeSelectorTerms:
                        - matchExpressions:
                          - key: size
                            operator: In
                                    values:
                                    - Large
######################################################################
 
requiredDuringSchedulingIgnoredDuringExecution: pods must meet the nodeSelector during scheduling
preferredDuringSchedulingIgnoredDuringExecution: pods must meet the nodeSelector during scheduling
 
In both the above cases during execution (mean pods were running already), if the node changes the labels then pods are not evicted (they still keep running).
 
In order to get the pods scheduled on the expected/required nodes, we need to use a combination of Taints/Tolerations and NodeAffinity
 
Taints/Tolerations: this option cannot guarantee that the pods are alway placed on the tainted nodes, the pods may be placed on other nodes where there are no taints/tolerations are set
Node Affinity: this options cannot guarantee only the selected pods are always placed on the selected nodes, as other pods can also be placed on the selected nodes
 
Resource Requirements & Limits:
 
Default Limits:
1vCPU
512 Mi
 
If a pod tries to use more than its limit constantly, the pod will be terminated.
 
CPU: Throttle mode, cant use more than assigned CPU
Memory: Terminates Pod, More memory than the limit can be used but Pod is terminated if the pod does it constantly
 
Note: Limits & Requests are set for each container.
 
 
Daemon Sets:
Runs a copy of pod in each node
 
Example: kube-proxy, weave-net(networking agent) are deployed on each node as Daemonsets
 
Example Usecases: Monitoring Solution, Logs Viewer
 
kubectl get daemonsets
kubectl describe daemonsets daemon-set-1
 
Default Behavior till v.12 - using nodeName attribute set
From v1.2 - uses NodeAffinity and default scheduler
#######################################################################
# Creating a daemon set in kube-system namespace with elasticsearch image
#######################################################################
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: elasticsearch
  namespace: kube-system
spec:
  selector:
    matchLabels:
      app: monitoring-agent
  template:
    metadata:
      labels:
        app: monitoring-agent
    spec:
      containers:
      - name: elasticsearch
        image: k8s.gcr.io/fluentd-elasticsearch:1.20
######################################################################
 
 
Static Pods:
Dump the yaml files in the monitored directory, kubelet will pick up and deploy those on a cron basis
 
This monitored directory can be configured in two ways.
1) configure the path directly in the kubelet.service (using attribute --pod-manifest-path)
2) confgiure the path in a config.yaml file (as an attribute - staticPodPath) and add it as an attribute(--config=/etck8s/manifests) to the kubelet.service
 
Only pods can be created, replicasets/services/deployments cant be created
 
Static Pods can only viewed using docker ps command as they are not part of kubernetes cluster configuration. Also kubectl only works with kube-apiserver
 
name of the static pods get appended with the nodename
 
kubectl can still see the status of the static pods object as the kubelet returns the status of the static pods. Kubelet creates a mirrorobject of the static pods in the kube-apiserver.
However kubectl cannot delete the static pods as they are not created by kubectl.
The only way to delete them is to delete the pod definition files from the manifest folder of the respective node
 
Usecase: Deploying control plane components such as controller,apiserver, etcd as yaml files
Kube-scheduler ignores the pods created by static pods and daemonsets
 
Run the command ps -aux | grep kubelet and identify the config file - --config=/var/lib/kubelet/config.yaml. Then check in the config file for staticPodPath.
 
#Creating a static pod
kubectl run --restart=Never --image=busybox:1.28.4 static-busybox --dry-run -o yaml --command -- sleep 1000 > /etc/kubernetes/manifests/static-busybox.yaml
 
Multiple Schedulers:
 
In order to have multiple schedulers, we need to edit the following from properties
by making a copy of the existing scheduler
                        1) name(under metadata),scheduler-name,port(under spec/containers),
                        2) port (under livenessprobe),
                        3) change leader-elect to false
 
########################################################################
apiVersion: v1
kind: Pod
metadata:
  annotations:
    scheduler.alpha.kubernetes.io/critical-pod: ""
  creationTimestamp: null
  labels:
    component: kube-scheduler
    tier: control-plane
  name: my-scheduler
  namespace: kube-system
spec:
  containers:
  - command:
    - kube-scheduler
    - --address=127.0.0.1
    - --kubeconfig=/etc/kubernetes/scheduler.conf
    - --leader-elect=false
    - --scheduler-name=my-scheduler
    - --port=12053
    image: k8s.gcr.io/kube-scheduler-amd64:v1.11.3
    imagePullPolicy: IfNotPresent
    livenessProbe:
      failureThreshold: 8
      httpGet:
        host: 127.0.0.1
        path: /healthz
        port: 10253
        scheme: HTTP
      initialDelaySeconds: 15
      timeoutSeconds: 15
    name: kube-scheduler
    resources:
      requests:
        cpu: 100m
    volumeMounts:
    - mountPath: /etc/kubernetes/scheduler.conf
      name: kubeconfig
      readOnly: true
  hostNetwork: true
  priorityClassName: system-cluster-critical
  volumes:
  - hostPath:
      path: /etc/kubernetes/scheduler.conf
      type: FileOrCreate
    name: kubeconfig
########################################################################
 
 
########################################################################
# Using a custom scheduler in pod definition file
########################################################################
apiVersion: v1
kind: Pod
metadata:
  name: nginx
spec:
  containers:
  -  image: nginx
     name: nginx
  schedulerName: my-scheduler
########################################################################
 
 
LOGGING & MONITORING:
 
Monitoring:
opensource: metrics server, prometheus, and elastic stack
 
heapster - first projects to enable monitoring ;  is deprecated
metrics server(slimmed down version of heapster): in-memory solution (no historical data)
one metrics server per kubernetes cluster is allowed
cAdvisor(containerAdvisor): is a subcomponent of kubelet is responsible to retreive the metrics from pods and exposing them through kubelet api to make the metrics available to metrics server
 
 
propreitory:
datadog
dynatrace
 
clone the repo-metricsserver from the git and create the required resources using the yaml files
 
kubectl top node
kubectl top pod
 
Logging:
kubectl logs pod podname
kubectl logs pod podname containername
kubectl logs -f pod podname
kubectl logs -f pod podname containername
 
APPLICATION LIFECYCLE MANAGEMENT:
 
Rolling updates and Rollbacks:
Strategy:
RollingUpdate (default strategy) -> take down one pod & bring up one pod
Recreate -> take down all pods & bring up all pods up
 
Create:
kubectl create -f nginx.yaml --record=true
Get:
kubectl get deployments
Update:
kubectl apply -f nginx.yaml
kubectl set image deployment/nginx nginx=nginx:1.7.1
(Using above command will not update the deployment definiton file)
Status:
kubectl rollout status deployment/deployment-name
kubectl rollout history deployment/deployment-name
Rollback:
kubectl rollout undo deployment/deployment-name
 
Step 1) kubectl create deployment nginx --image=nginx --dry-run -o yaml > nginx.yaml
Step 2) kubectl create -f nginx.yaml --record=true
 
kubectl apply -f deployment-definition.yaml -> best way to edit the deployment and update the existing deployment
 
Commands & Arguments: Containers
 
Entrypoint vs CMD:
CMD will simple run the command, it cannot take any arguments
Entrypoint is nothing but the command that is run in the container but it can also take arguments
 
if we want to have a default argument value we have to use Entrypoint & Command both
########################
Dockerfile
########################
From Ubuntu
ENTRYPOINT ["sleep"]
CMD ["5"]
########################
Above values will give you sleep 5 seconds as default but can be overridden during runtime to the desired value of sleep time
docker build -t sleeper .
To modify the entrypoint altogether we can specify --entrypoint as argument during container creation
 
 
Commands & Arguments: K8s Pods
################################################
Args/Commands Pods Example
################################################
apiVersion: v1
kind: Pod
metadata:
            name: sleep-pod
spec:
  containers:
  - name: sleep-container
    image: sleeper
    command: ["sleep2.0"] # this overrides entrypoint in dockerworld               = ENTRYPOINT ["sleep"]
                        args: ["10"]          # this overrides argument of the command in dockerworld  = CMD ["5"]
################################################
 
Environment Variables:
 
Environment Variables in Docker: docker -e color=pink simplewebapp
 
Environment Variables can be set in three ways
1) plain key/value pair as args of env
env:
            - name: color
                        value: pink
Example:
################################################
apiVersion: v1
kind: Pod
metadata:
            name: myapp-pod
spec:
  containers:
  - name: nginx-container
    image: nginx
    env:
                          - name: color
                                      value: pink
################################################
 
2) ConfigMaps:
Imperative:
kubectl create configmap appconfigmap --from-literal=app_color=pink --from-literal=app_mode=prod
 
# injecting single environment variable from a configmap
env:
            - name: color
                        valueFrom:
                          configMapKeyRef:
                                      name: app-config
                                                key: app_color
################################################
configmap.yaml # define configmap
################################################
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
data: #instead of spec for a configmap we have "data" section
  app_color: pink
  app_mode: prod
################################################
 
################################################
#inject configmap into pod-definiton file
################################################
apiVersion: v1
kind: Pod
metadata:
            name: myapp-pod
spec:
  containers:
  - name: nginx-container
    image: nginx
    envFrom:
                          - configMapRef:
                                        name: app-config
################################################
 
In the above pod-definiton file, all the data in configmap is loaded
 
volumes:
 - name: app-config-volume
   configMap:
               name: app-config
 
 
3) Secrets
env:
            - name: color
                        valueFrom:
                          secretKeyRef
 
Imperative:
kubectl create secret generic appsecret --from-literal=dbuser=dbadmin --from-literal=dbpwd=dbpass
 
################################################
secret.yaml # define secret
################################################
apiVersion: v1
kind: Secret
metadata:
  name: app-secret
data: #instead of spec for a secret we have "data" section
  dbusername: cFzgbc3dvcmQ=
  dbpassword: GcVdFzd43mQa
################################################
 
################################################
#inject secret into pod-definiton file
################################################
apiVersion: v1
kind: Pod
metadata:
            name: myapp-pod
spec:
  containers:
  - name: nginx-container
    image: nginx
    envFrom:
                          - secretRef:
                                        name: app-secret
################################################
 
In declarative way(yaml file), secrets should be in encoded format
on linux, "echo -n password | base64" give password in encoded format
kubectl get secrets app-secret -o yaml > this give the encoded value of the secrets
To decode the values:
on linux, "echo -n cFzgbc3dvcmQ= | base64 --decode" give encoded text in plaintext format
 
volumes:
 - name: app-secret-volume
   secret:
               name: app-secret
 
All secrets are created as individual files inside the Pod at path /opt/app-secret-volume
 
A note about Secrets!
Remember that secrets encode data in base64 format. Anyone with the base64 encoded secret can easily decode it. As such the secrets can be considered as not very safe.
The concept of safety of the Secrets is a bit confusing in Kubernetes. The kubernetes documentation page and a lot of blogs out there refer to secrets as a "safer option" to store sensitive data.
They are safer than storing in plain text as they reduce the risk of accidentally exposing passwords and other sensitive data. In my opinion it's not the secret itself that is safe, it is the practices around it.
Secrets are not encrypted, so it is not safer in that sense.
However, some best practices around using secrets make it safer. As in best practices like:
Not checking-in secret object definition files to source code repositories.
Enabling Encryption at Rest for Secrets so they are stored encrypted in ETCD.
 
Also the way kubernetes handles secrets. Such as:
A secret is only sent to a node if a pod on that node requires it.
Kubelet stores the secret into a tmpfs so that the secret is not written to disk storage.
Once the Pod that depends on the secret is deleted, kubelet will delete its local copy of the secret data as well.
Read about the protections and risks of using secrets here
 
Having said that, there are other better ways of handling sensitive data like passwords in Kubernetes, such as using tools like Helm Secrets, HashiCorp Vault.
I hope to make a lecture on these in the future.
 
Multi-container PODs:
Example:
################################################################################################
apiVersion: v1
kind: Pod
metadata:
            name: myapp-pod
spec:
  containers:
  - name: nginx-container
    image: nginx
    ports:
                          - containerPort: 8080
  - name: log-agent
              image: log-agent
################################################################################################
Multi-container PODs Design Patterns:
There are 3 common patterns, when it comes to designing multi-container PODs.
The first and what we just saw with the logging service example is known as a side car pattern.
The others are the adapter and the ambassador pattern.
 
InitContainers: https://kubernetes.io/docs/concepts/workloads/pods/init-containers/
 
Example:
################################################################################################
apiVersion: v1
kind: Pod
metadata:
  name: myapp-pod
  labels:
    app: myapp
spec:
  containers:
  - name: myapp-container
    image: busybox:1.28
    command: ['sh', '-c', 'echo The app is running! && sleep 3600']
  initContainers:
  - name: init-myservice
    image: busybox
    command: ['sh', '-c', 'git clone <some-repository-that-will-be-used-by-application> ; done;']
################################################################################################
 
Self Healing Applications:
Kubernetes supports self-healing applications through ReplicaSets and Replication Controllers.
The replication controller helps in ensuring that a POD is re-created automatically when the application within the POD crashes.
It helps in ensuring enough replicas of the application are running at all times.
 
Kubernetes provides additional support to check the health of applications running within PODs and take necessary actions through Liveness and Readiness Probes.
However these are not required for the CKA exam and as such they are not covered here. These are topics for the Certified Kubernetes Application Developers (CKAD) exam and are covered in the CKAD course.
 
 
CLUSTER MAINTENANCE:
If node is down for more than 5 minutes, k8s considers the node as dead
 
pod-eviction timeout: The time it waits for a pod to come back online is called pod eviction timeout
set on controller-manager with a default value of 5 min
 
If the node comes back online after the pod eviction timeout then the node will be blank (no pods)
Upgrade Strategies:
Strategy-1: upgrade all nodes at once, downtime involved
Strategy-2: upgrade one node at a time, no downtime involved
Strategy-3: provision new nodes with new version, migrate workload, delete the old node
 
Upgrade process:
1) Cordon a node -> makes a node unschedulable
2) Drain a node -> moves pods to other nodes
3) Uncordon a node -> makes a node schedulable again
``` 
Example:
kubectl cordon node-01
kubectl drain node-01
kubectl uncordon node-01
``` 
Note: If there are pods that are not part of replicaset, deployment, daemonset, job,  statefulset are on the node that is taken down for maintenance, the pod will be lost forever. Also the node needs to be drained forcefully using following command:
 
kubectl drain <nodename> --ignore-daemonsets --force
```
Kubernetes Software Versions:
 
MAJOR.MINOR.PATCH
Minor-> Features, Functionalities
Patch-> BugFixes
 
alpha-release: bug fixes & improvements, features are disabled by default and are buggy
beta-release: code tested and new features turned on
After beta-release, its merged to a stable release
 
Control plane components with same version: api-server, controller, scheduler, proxy, kubelet, kubectl
ETCD cluster & CoreDNS have their own versions as they are separate projects
https://kubernetes.io/docs/concepts/overview/kubernetes-api/
 
Here is a link to kubernetes documentation if you want to learn more about this topic (You don't need it for the exam though):
 
https://github.com/kubernetes/community/blob/master/contributors/devel/sig-architecture/api-conventions.md
 
https://github.com/kubernetes/community/blob/master/contributors/devel/sig-architecture/api_changes.md
```
```
kube-apiserver > controller & scheduler > kubelet & kube-proxy
    X                 X-1                       X-2
 (1.10)           (v1.9 or v1.10)      (v1.8 or v1.9 or v1.10)
 ```
kubectl can be higher than the kube-apiserver
 
kubernetes only supports upto latest 3 minor versions
Lets say you are on 1.10 and a new version 1.11 & 1.12 are available.
K8s will support only 1.10, 1.11 and 1.12. This means when K8s releases 1.13, 1.10 will be unsupported
 
Recommended way to upgrade the cluster is to upgrade to one minor version at a time.
1.10 -> 1.11 -> 1.12 -> 1.13
 
kubeadm upgrade plan -> lists out current and latest k8s version
apt-get upgrade -y kubeadm=1.12.0-00
kubeadm upgrade apply plan v1.12.0
apt-get upgrade -y kubelet=1.12.0-00
 
Node-upgrade:
kubectl drain node-01
apt-get upgrade -y kubeadm=1.12.0-00
apt-get upgrade -y kubelet=1.12.0-00
kubeadm upgrade node config --kubelet-version v.12.0
systemctl restart kubelet
kubectl uncordon node-01
 
Note: Version shown in kubectl get nodes is the version of the kubectl
 
Backup & Restore:
Backup Candidates:
ETCD & Persistent volumes
Resource Configuration:
kubectl get all --all-namespaces -o yaml > all-deploy-services.yaml
ETCD Cluster:
check etcd.service for value of data-dir value which is the location of the data
etcdctl snapshot save snapshot.db
etcdctl snapshot status snapshot.db
 
Restore ETCD Cluster:
ETCDCTL_API=3 etcdctl snapshot save snapshot.db
ETCDCTL_API=3 etcdctl snapshot status snapshot.db
service kube-apiserver stop
ETCDCTL_API=3 etcdctl snapshot restore snapshot.db \
            --data-dir= /var/lib/etcd-backup\
            --initial-cluster master-1=https://192.168.1.22:2308,master-2=https://192.168.1.22:2308 \
            --initial-cluster-token etcd-cluster-1 \
            --initial-advertise-peer-urls https://${internal_ip}:2380
change the cluster-token and data-dir values in the etcd.service
systemctl daemon-reload
service etcd restart
service kube-apiserver start
 
when etcd restores from backup it creates a initializes a new cluster configuration and
configures the members of the etcd as new members of the new cluster.
 
This prevents the new members from accidentally joining the existing cluster
 
Also specify the following certs for authentication:
ETCDCTL_API=3 etcdctl snapshot save snapshot.db \
            --endpoints=https://127.0.0.1:2379 \
            --cacert=/etc/etcd/cacert.crt \
            --cert=/etc/etcd/etcd-server.crt \
            --key=/etc/etcd/etcd-server.key
 
 
ETCDCTL_API=3 etcdctl snapshot save /tmp/snapshot-pre-boot.db --cacert /etc/kubernetes/pki/etcd/ca.crt --cert /etc/kubernetes/pki/etcd/server.crt --key /etc/kubernetes/pki/etcd/server.key
 
https://kubernetes.io/docs/tasks/administer-cluster/configure-upgrade-etcd/#backing-up-an-etcd-cluster
 
https://github.com/etcd-io/etcd/blob/master/Documentation/op-guide/recovery.md
 
https://www.youtube.com/watch?v=qRPNuT080Hk
 
SECURITY:
asymmetric encryption is used to distribute the symmetric key for ssl/tls encryption

ca certs (public keys of CA) -> trusted ca tab in the browser

if your org is using a private CA, you need to install the public keys of private CA in your browser (this will validate the authenticity of private CA)

.crt and .pem -> public keys
.key and -key.pem -> private keys

Server components in Kubernetes cluster:
Kube-api server: api server exposes http service that other components use to manage k8s cluster
ETCD server: talks to kube-api server & stores all the k8s state information
kubelet server: talks to kube-api server

Client components:
admin: admin uses kubectl/rest-api to authenticate to kube-api server
kube-scheduler: talk to api-server to schedule pods
kube-controller manager: talks to api-server to monitor and bring up new pods according to desired config
kube-proxy: talks to api-server
--------------------------------------------------------
kubelet: kubelet is a client to kube-apiServer
kube-apiserver: kube-apiServer is a client to kubelet
kubelet/kube-apiserver can use the same keys generated for server certs but they can also generate
a separate pair of certs just for this exclusive communication

we also need a CA to sign all the above certificates (ca.crt, ca.key)
Creating a CA Cert:
openssl genrsa -out ca.key 2048
openssl req -new -key ca.key -subj "/CN=KUBERNETES-CA" -OUT ca.csr
openssl x509 -req -in ca.csr -signkey ca.key -out ca.crt

Signing client certificates using the CA key generated above:
openssl genrsa -out admin.key 2048
openssl req -new -key admin.key -subj "/CN=kube-admin/O=system:masters" -OUT admin.csr
openssl x509 -req -in admin.csr -CA ca.crt -CAkey ca.key -out admin.crt

All the system components such as scheduler, controller-manager CN name should be prefixed with
system:KUBE-SCHEDULER, system:KUBE-CONTROLLER-MANAGER

For ETCD Server, certs should be generated as etcd(master),etcdpeer1, etcdpeer2 and add the certs in the etcd.service
For Kube-apiServer, requires the following to have name in the Alternative Names field:
kubernetes
kubernetes.default
kubernetes.default.svc
kubernetes.default.svc.cluster.local
IPAddress

For Kubelet, server certs should be generated as node name as node01, node02, node03
For Kubelet, client certs are needed as it needs to talk to the kube-apiServer
Client certs should be generated as system:node:node01 as they are system components such as kube-scheduler
Also, the client certs should mention the group name as system:nodes to get the right privileges

Once the certs are setup we can access the kube-api server using kubectl or by calling api-server-rest
curl http://kube-apiserver:6443/api/v1/pods/ -key admin.key --cert admin.crt --cacrt ca.crt

for kubectl specify the certs file path in kube-config.yaml:
######################################################
apiVersion: v1
clusters:
- cluster:
    certificate-authority: ca.crt
		server: https://kube-apiserver:6443
	name: kubernetes
kind: Config
Users:
- name: kubernetes-admin
  user:
	  client-certificate: admin.crt
		client-key: admin.key
######################################################

inspect service logs:
if k8s setup was from scratch:
journalctl -u etcd.service -l
if k8s setup was by kubedam:
kubectl logs etcd-master
if kube-api server is down, kubectl commands wont work. In that case dig down to docker commands
docker ps -a to see which pod belongs to etcd & use docker logs etcd-container

Certificates API is managed by Controller-Manager(on master node).
It has CSR-Signing & CSR-Approving controllers that are responsible for signing & approving certificates
It also has the ca public & private keys that are used to sign certificates.

To send a CSR for signing, a yaml file is generated(which includes the CSR) & submitted
kubectl get csr -> view CSRs
kubectl certificate approve jane -> approve certificate
kubectl get csr jane -o yaml -> view approved certificate
kubectl deny csr agent-smith -> deny csr
kubectl delete csr agent-smith -> delete csr

Kube Config:
context is the combination of user and cluster name. Each config file has a default context in the name of current-context
kubectl config view -> points to default config which is /root/.kube/config
kubectl config view --kubeconfig=/root/my-kube-config
kubectl config --kubeconfig=/root/my-kube-config use-context research -> change context


API Groups:

/api - core (all core functionality) - /v1 - namespaces, pods, rc, events, nodes, pv, pvc etc
/apis - named (new features) - /apps, /extensions, /networking.k8s.io, /authentication.k8s.io, /certificates.k8s.io

All the above named objects are called API Groups,

/apps
  /v1
    /deployments --> list, get, create, update, delete, watch
	/replicasets
	/statefulsets
Above deployments, replicasets, stateful sets are called Resources & each resoource has actions called Verbs (such as list, get, create, update, delete, watch)
RBAC:
kubectl get roles
kubectl get rolebindings

Create a Role: creates the role with necessary operations
################################################################
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: developer
rules:
- apiGroups: [""] # for core group, this can be blank, for anything else it should be specified
  resources: ["pods"]
  verbs: ["list", "create", "get", "update", "delete"]
- apiGroups: [""]
  resources: ["ConfigMap"]
  verbs: ["create"]
################################################################

 
Create a RoleBinding: ties the user to the role

################################################################
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: devuser-developer-binding
subjects:
- kind: User
  name: dev-user
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: Role
  name: developer
  apiGroup: rbac.authorization.k8s.io
################################################################

Check Access:
kubectl auth can-i create deployments
output: yes/no
kubectl auth can-i delete nodes
no
We can also impersonate a user to check whether access was granted correctly or not:
kubectl auth can-i create deployments --as dev-user
yes
kubectl auth can-i create pods --as dev-user
yes
kubectl auth can-i create pods --as dev-user --namespace test
no
We can also give access to specific resources using resourceNames attribute

kubectl create role developer --verb=list,create --resource=pods --dry-run -o yaml > developer.yaml
kubectl create rolebinding dev-user-binding --role=developer --user=dev-user --dry-run -o yaml > developerrb.yaml

ClusterRoles & ClusterRoleBindings:
resources are scoped as namespaced or cluster-scoped

Namespaced Scope: pods, rs, jobs, deployments, services, secrets, role, rolebindings, pvc, configmaps
Issue the following command to get all the resources that are namespace scope:
kubectl api-resources --namespaced=true

Cluster Scope: nodes, PV, clusterrole, clusterrolebinding, namespaces, certificatesigningrequests
Issue the following command to get all the resources that are cluster scope:
kubectl api-resources --namespaced=false

Sample:
---
kind: ClusterRole
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: node-admin
rules:
- apiGroups: [""]
  resources: ["nodes"]
  verbs: ["get", "watch", "list", "create", "delete"]

---
kind: ClusterRoleBinding
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: michelle-binding
subjects:
- kind: User
  name: michelle
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: ClusterRole
  name: node-admin
  apiGroup: rbac.authorization.k8s.io


kubectl create clusterrole storage-admin --verb=get,watch,create,list,delete --resource=persistentvolumes,storageclasses --dry-run -o yaml > storageadmin.yaml
kubectl create clusterrolebinding michelle-storage-admin --clusterrole=storage-admin  --user=michelle  --dry-run -o yaml > storageadminrb.yaml


kubectl api-resources -> gets all the api resources
kubectl api-resources -o wide -> gives detailed information invluding the verbs on api resources

Image Security:
For all the private registries, a k8s secret needs to be generated first and passed as a secret in the
pod definition to pull an image from the private registry.
Step 1: kubectl create secret docker-registry regcred \
                --docker-server=private-repo.io \
								--docker-username=admin \
								--docker-password=Server \
								--docker-email=admin@server.com
Step 2: inject the secret into the pod definition file
######################################################
apiVersion: v1
kind: Pod
metadata:
  name: web-pod
spec:
  containers:
	  - name: ubuntu
		  image: private.io/ubuntu
  imagePullSecrets:
	- name: regcred
######################################################
SecurityContexts:
docker run --user=1001 ubuntu sleep 3600
docker run --cap-add MAC_ADMIN ubuntu
SecurityContext applied at pod level replicates to all the containers in the pod.
If its applied at both pod and container level, container will override pod level values.

kubectl exec ubuntu-sleeper whoami # run command on pod to see which user is tied
######################################################
apiVersion: v1
kind: Pod
metadata:
  name: web-pod
spec:
  securityContext:  # pod-level setting
	  runAsUser: 1001
	containers:
	  - name: ubuntu
		  image: ubuntu
			command: ['sleep', '3600']
			securityContext:    # container level setting
				runAsUser: 1001
      capabilities:       # capabilities are only supported at container level
			  add: ["MAC_ADMIN"]
######################################################

Network Policy:

kubectl get networkpolicy

sample: allowing ingress traffic only from api pod to db pod
######################################################
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: db-policy
spec:
  podSelector:
    matchLabels:
	  role: db
  policyTypes:
  - Ingress
  ingress:
  - from:
    - podSelector:
	    matchLabels:
		  name: api-pod
	ports:
	- protocol: TCP
	  port: 3306
######################################################

Sample Networking policy to allow egress traffic to mysql & payroll pods from internal pods
######################################################
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: internal-policy
  namespace: default
spec:
  podSelector:
    matchLabels:
      name: internal
  policyTypes:
  - Egress
  - Ingress
  ingress:
    - {}
  egress:
  - to:
    - podSelector:
        matchLabels:
          name: mysql
    ports:
    - protocol: TCP
      port: 3306

  - to:
    - podSelector:
        matchLabels:
          name: payroll
    ports:
    - protocol: TCP
      port: 8080
######################################################

Solutions that support network policies:
kube-router, calico, romana, weave-net

Flannel does not support network policies

We can use the solutions that does not support network policies, but the restrictions wont work once enforced. 
STORAGE:
 
PV & PVC should match the attributes for a successful claim (such as storage, accessModes etc)
When a PVC is deleted, the PV is not deleted but also its not available (unreleased)
because of the default claim policy - Retain; it has to be manually deleted by Administrator.
It can be automatically deleted when the "PersistentVolumeReclaimPolicy: Delete"
 
PersistentVolumeReclaimPolicy: Retain
PersistentVolumeReclaimPolicy: Recycle -> Data will be scrubbed before making it available
PersistentVolumeReclaimPolicy: Delete -> PV gets deleted automatically
 
PersistentVolumes:
AccessModes: ReadOnlyMany; ReadWriteOnce, ReadWriteMany
 
Kubernetes binds the persistent volumes to claims based on the requests and  properties set on the volume.
We can also use labels and selectors to bind the  right volumes.
Every persistentVolumeClaim is bound to a single PersistentVolume.
 Note: a smaller claim may get a larger volume if all the criteria are matched if there are no better options
No other claims can utilize the remaining capacity in the volume as there is a one-to-one relationship between PV & PVC
 
If no other volumes are available, PVC will be in a pending state.
 
Certification TIPS & TRICKS:
 
Lecture 28:
Certification Tip!
Here's a tip!
 
As you might have seen already, it is a bit difficult to create and edit YAML files. Especially in the CLI. During the exam, you might find it difficult to copy and paste YAML files from browser to terminal. Using the kubectl run command can help in generating a YAML template. And sometimes, you can even get away with just the kubectl run command without having to create a YAML file at all. For example, if you were asked to create a pod or deployment with specific name and image you can simply run the kubectl run command.
 
Use the below set of commands and try the previous practice tests again, but this time try to use the below commands instead of YAML files. Try to use these as much as you can going forward in all exercises
 
Reference (Bookmark this page for exam. It will be very handy):
 
https://kubernetes.io/docs/reference/kubectl/conventions/
 
Create an NGINX Pod
kubectl run --generator=run-pod/v1 nginx --image=nginx
 
Generate POD Manifest YAML file (-o yaml). Don't create it(--dry-run)
kubectl run --generator=run-pod/v1 nginx --image=nginx --dry-run -o yaml
 
Create a deployment
kubectl run --generator=deployment/v1beta1 nginx --image=nginx
 
Generate Deployment YAML file (-o yaml). Don't create it(--dry-run)
kubectl run --generator=deployment/v1beta1 nginx --image=nginx --dry-run -o yaml
 
Generate Deployment YAML file (-o yaml). Don't create it(--dry-run) with 4 Replicas (--replicas=4)
kubectl run --generator=deployment/v1beta1 nginx --image=nginx --dry-run --replicas=4 -o yaml
 
Save it to a file - (If you need to modify or add some other details before actually creating it)
kubectl run --generator=deployment/v1beta1 nginx --image=nginx --dry-run --replicas=4 -o yaml > nginx-deployment.yaml
 
Lecture 35:
Certification Tips - Imperative Commands with Kubectl
While you would be working mostly the declarative way - using definition files, imperative commands can help in getting one time tasks done quickly, as well as generate a definition template easily. This would help save considerable amount of time during your exams.
 
Before we begin, familiarize with the two options that can come in handy while working with the below commands:
 
--dry-run: By default as soon as the command is run, the resource will be created. If you simply want to test your command , use the --dry-run option. This will not create the resource, instead, tell you weather the resource can be created and if your command is right.
 
-o yaml: This will output the resource definition in YAML format on screen.
 
 
 
Use the above two in combination to generate a resource definition file quickly, that you can then modify and create resources as required, instead of creating the files from scratch.
 
 
 
POD
Create an NGINX Pod
 
kubectl run --generator=run-pod/v1 nginx --image=nginx
 
 
 
Generate POD Manifest YAML file (-o yaml). Don't create it(--dry-run)
 
kubectl run --generator=run-pod/v1 nginx --image=nginx --dry-run -o yaml
 
 
 
Deployment
Create a deployment
 
kubectl run --generator=deployment/v1beta1 nginx --image=nginx
 
Or the newer recommended way:
 
kubectl create deployment --image=nginx nginx
 
 
 
Generate Deployment YAML file (-o yaml). Don't create it(--dry-run)
 
kubectl run --generator=deployment/v1beta1 nginx --image=nginx --dry-run -o yaml
 
Or
 
kubectl create deployment --image=nginx nginx --dry-run -o yaml
 
 
 
Generate Deployment YAML file (-o yaml). Don't create it(--dry-run) with 4 Replicas (--replicas=4)
 
kubectl run --generator=deployment/v1beta1 nginx --image=nginx --dry-run --replicas=4 -o yaml
 
kubectl create deployment does not have a --replicas option. You could first create it and then scale it using the kubectl scale command.
 
 
 
Save it to a file - (If you need to modify or add some other details)
 
kubectl run --generator=deployment/v1beta1 nginx --image=nginx --dry-run --replicas=4 -o yaml > nginx-deployment.yaml
 
 
 
Service
Create a Service named redis-service of type ClusterIP to expose pod redis on port 6379
 
kubectl expose pod redis --port=6379 --name redis-service --dry-run -o yaml
 
(This will automatically use the pod's labels as selectors)
 
Or
 
kubectl create service clusterip redis --tcp=6379:6379 --dry-run -o yaml  (This will not use the pods labels as selectors, instead it will assume selectors as app=redis. You cannot pass in selectors as an option. So it does not work very well if your pod has a different label set. So generate the file and modify the selectors before creating the service)
 
 
 
Create a Service named nginx of type NodePort to expose pod nginx's port 80 on port 30080 on the nodes:
 
kubectl expose pod nginx --port=80 --name nginx-service --dry-run -o yaml
 
(This will automatically use the pod's labels as selectors, but you cannot specify the node port. You have to generate a definition file and then add the node port in manually before creating the service with the pod.)
 
Or
 
kubectl create service nodeport nginx --tcp=80:80 --node-port=30080 --dry-run -o yaml
 
(This will not use the pods labels as selectors)
 
Both the above commands have their own challenges. While one of it cannot accept a selector the other cannot accept a node port. I would recommend going with the `kubectl expose` command. If you need to specify a node port, generate a definition file using the same command and manually input the nodeport before creating the service.
 
 
 
Reference:
 
https://kubernetes.io/docs/reference/kubectl/conventions/
