---?image=https://i.imgur.com/8ghbf.jpg

# Minecraft on Kubernetes 

### It's not just for<br>-the kids-<br>anymore

---?image=https://cdn-images-1.medium.com/max/1600/1*9NG_meMVwlOwq30TWp5f_w.jpeg&opacity=20

### What are we going to cover

@ul[circles](false)
- Overview of Kubernetes
- Setup Requirements
- System Design
- Installation
- Minecraft YAML!
@ulend

---?image=https://i.kym-cdn.com/entries/icons/original/000/000/248/profit.jpg&opacity=30

### Why Kubernetes <br>and Minecraft
@ul[](false)
- Desire to learn Kubernetes
- Most guides miss middle ground
  - Minikube on desktop
  - Cloud scale
- Kids wanted a Minecraft server
@ulend

---?image=https://hackadaycom.files.wordpress.com/2016/08/yakshaving.jpg&opacity=30

@snap[north-west]
This is dumb - complete overkill
@snapend

@snap[west list-content-verbose span-100]
<br>
@ul[list-bullets-circles]
- Maintaining this will only lead to tears
- There are much easier ways to run a minecraft server
- Kubernetes is not one of them 
- There are really cheap public services are out there for dollars a month
@ulend
@snapend

---?image=https://cdn-images-1.medium.com/max/1600/1*9NG_meMVwlOwq30TWp5f_w.jpeg&opacity=20

@snap[north span-100]
Overview of Kubernetes
@snapend
@snap[west]
What is Kubernetes?
<p>
At is core Kubernetes is a desired state machine for containers
@snapend

@snap[south]
Give it YAML
<br>
Get back infrastructure
@snapend

---?image=https://wallpapercave.com/wp/i2JDlzW.jpg&opacity=40

### Kubernetes Inception

Deployments own replications sets which creates pods that contain containers that you can reach via services that connects to pods via endpoints of pods which can store persistent data in persistent volumes that it attaches to using persistent volume claims

---?image=https://storage.googleapis.com/cdn.thenewstack.io/media/2017/11/07751442-deployment.png&size=50% 100%&color=black

---

### The Ugly

- Everything is beta or alpha, get used to seeing that a lot
- Requirements and approved sofware change constently, look for current information
- RTFM and GitHub Issues is your best friend

---

@snap[north span-100]
System requirements
@snapend
@ul[list-bullets-circles](false)
- ESXi (or any other multi VM solution)
- 6GB free ram
- Ubuntu 18.04 ISO
- About 10Gb per VM to start
- Static IP address per VM
- NFS storage for persistance
@ulend

---

### System design
@ul[](false)
- Three nodes (VM's)
  - One master
  - Two workers
- All VM's on the same subnet
  - Do NOT add a seperate network for cluster traffic unless you like fixing routing tables
- Partition layout
  - 10GB OS
  - /var/lib/docker on separate filesystem
@ulend

--- 

#### Install System and Software
@ul[](false)
- Default minimal install of Ubuntu, add SSH
- Create home for Docker images
    - mkdir /var/lib/docker
    - format and mount second disk, add to fstab
- Install Docker
    - Add repo for Docker
    - Install and lock version based on K8n CRI doc
    - Add your user to the docker group
- Install Kubernetes
    - Add repo for Kubernetes
    - Install kubeadm
@ulend

---

### Bringing up the cluster 
On the master run
```
$ sudo -s
# kubeadm init
.....
   Lots of checks and messages
.....
Your Kubernetes master has initialized successfully!

 kubeadm join --token <token> <master-ip>:<master-port> --discovery-token-ca-cert-hash sha256:<hash>
```
To administer as a normal user
```
$ mkdir -p $HOME/.kube
$ sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
$ sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

---

### Lets talk about the CNI <br> Container Network Interface
@ul[](false)
- Lots of different options
- L2, L3, VXLAN, BGP, OSPF
- CRD's and etcd for configs
- Encryption options
- I am not going to cover all of them
- Were picking Weave Net
@ulend
```
$ kubectl apply -f "https://cloud.weave.works/k8s/net?k8s-version=$(kubectl version | base64 | tr -d '\n')"
```

---

### Add the worker nodes
SSH into the other nodes in the network
```
# sudo su -
Use the join token from before
# kubeadm join --token <token> <master-ip>:<master-port> --discovery-token-ca-cert-hash sha256:<hash>
```
Disconnect, we are done with the workers

---?image=https://wallpaper.wiki/wp-content/uploads/2017/04/wallpaper.wiki-Binary-Code-Background-Full-HD-PIC-WPB003927.jpg

### Creating a Pod for Minecraft

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: dc612
  labels:
    app: dc612
spec:
  containers:
  - name: minecraft
    image: itzg/minecraft-server
    env:
    - name: EULA
      value: "TRUE"
    - name: VERSION
      value: "1.13.2"
    ports:
    - containerPort: 25565
    imagePullPolicy: Always
```
@[1](All YAML starts with a version)
@[2](What are we creating)
@[3-6](Name of the pod and label selector)
@[9](Name of the pod in the container)
@[10](What are we pulling)
@[11-15](Enviroment Variables to pass in)
@[16-17](Container ports we wish to expose)
@[18](When are we pulling the image)

---?image=https://wallpaper.wiki/wp-content/uploads/2017/04/wallpaper.wiki-Binary-Code-Background-Full-HD-PIC-WPB003927.jpg

### Test in Prod

```
# kubectl apply -f dc612-pod.yaml
pod/dc612 created

# kubectl get po
NAME                     READY   STATUS    RESTARTS   AGE
dc612                    1/1     Running   0          11m

# kubectl describe po dc612
Name:               dc612
Namespace:          default
Priority:           0
PriorityClassName:  <none>
Node:               kubework1/192.168.10.102
Start Time:         Wed, 07 Nov 2018 21:43:59 -0600
Labels:             app=dc612
Annotations:        kubectl.kubernetes.io/last-applied-configuration:
                      {"apiVersion":"v1","kind":"Pod","metadata":{"annotations":{},"labels":{"app":"dc612"},"name":"dc612","namespace":"default"},"spec":{"conta...
Status:             Running
IP:                 172.17.0.5
Containers:
  minecraft:
    Container ID:   docker://18ebf66eb572f5314ff555e65aa65601c82c7b69c842f137d4e89e4662d1e820
    Image:          itzg/minecraft-server
    Image ID:       docker-pullable://itzg/minecraft-server@sha256:c486fecafb0d16b112d6de317ffc0bcbab5f6376aec85afe032cfa01c763dff4
    Port:           25565/TCP
    Host Port:      0/TCP
    State:          Running
      Started:      Wed, 07 Nov 2018 21:44:05 -0600
    Ready:          True
    Restart Count:  0
    Environment:
      EULA:     TRUE
      VERSION:  1.13.2
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from default-token-9t2tz (ro)
Conditions:
  Type              Status
  Initialized       True
  Ready             True
  ContainersReady   True
  PodScheduled      True
Volumes:
  default-token-9t2tz:
    Type:        Secret (a volume populated by a Secret)
    SecretName:  default-token-9t2tz
    Optional:    false
QoS Class:       BestEffort
Node-Selectors:  <none>
Tolerations:     node.kubernetes.io/not-ready:NoExecute for 300s
                 node.kubernetes.io/unreachable:NoExecute for 300s
Events:
  Type    Reason     Age   From                Message
  ----    ------     ----  ----                -------
  Normal  Scheduled  12m   default-scheduler   Successfully assigned default/dc612 to kubework1
  Normal  Pulling    12m   kubelet, kubework1  pulling image "itzg/minecraft-server"
  Normal  Pulled     12m   kubelet, kubework1  Successfully pulled image "itzg/minecraft-server"
  Normal  Created    12m   kubelet, kubework1  Created container
  Normal  Started    12m   kubelet, kubework1  Started container

```

---

### Make it stable

Replication controllers are the <br>first level of the state machine
@ul[](false)
- Ensures the requested number of pods are always running
- If there are extras it kills them
- If a pod dies it starts new ones
@ulend
Clean up and lets add a replication controller
```
# kubectl delete -f dc612-pod.yaml
```

---?image=https://wallpaper.wiki/wp-content/uploads/2017/04/wallpaper.wiki-Binary-Code-Background-Full-HD-PIC-WPB003927.jpg

### Replication controller for Minecraft

```yaml
apiVersion: v1
kind: ReplicationController
metadata:
  name: dc612-rc
spec:
  replicas: 1
  selector:
    app: dc612
  template:
    metadata:
      labels:
        app: dc612
    spec:
      containers:
      - name: minecraft        image: itzg/minecraft-server
        env:
        - name: EULA
          value: "TRUE"
        - name: VERSION
          value: "1.13.2"
        imagePullPolicy: Always
        ports:
        - containerPort: 25565
```
@[2](Creating a replication controller)
@[6](How many replicas)
@[7-8](What is it controlling)
@[11-12](Matching selector)
@[13-23](Same pod as before)

---

### How do we get to it?

There are three classic types of services
@ul[](false)
- Cluster port
  - Stable IP address but inside of the k8n only
- Node port
  - Use any node for access
  - Ports 30000–32767 without work
  - One service per port
- Load Balancer
  - Only on a real cloud
  - There are options though, see references

---?image=https://wallpaper.wiki/wp-content/uploads/2017/04/wallpaper.wiki-Binary-Code-Background-Full-HD-PIC-WPB003927.jpg

### Node Port Service

```yaml
apiVersion: v1
kind: Service
metadata:
  name: dc612-svc
  labels:
    app: dc612
spec:
  type: NodePort
  ports:
  - port: 25565
    nodePort: 31565
    protocol: TCP
  selector:
    app: dc612
```
@[2](Making a service)
@[8](A Node port service)
@[10](Container port we are connecting to)
@[11](Port we are exposing on all nodes)

---

### Persistent Storage in K8n

@ul[](false)
- Sucks if you are not in the cloud
- Auto provisions is there but require effort
- NFS is the easiest I have found
  - Requires configuration per mount
  - Auto provision is there but prereqs are helm/tiller
@ulend

---

### Creating a home on the NAS
```
# showmount -e 192.168.100.170
Export list for 192.168.100.170:
/vmware/docker           *

# mount 192.168.100.170:/vmware/docker /mnt
# mkdir /mnt/dc612
# umount /mnt
```
@[1-3](Get the list of exports from the nas)
@[5-7](Mount the NAS, make a folder, unmount)

---?image=https://wallpaper.wiki/wp-content/uploads/2017/04/wallpaper.wiki-Binary-Code-Background-Full-HD-PIC-WPB003927.jpg

### Make the Persistent volume
```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: dc612
  labels:
    app: dc612
spec:
  capacity:
    storage: 1Mi
  accessModes:
    - ReadWriteOnce
  nfs:
    server: 192.168.100.170
    path: "/vmware/docker/dc612"
```
@[8-9](Quota size for the volume)
@[10-11](Access Restriction)
@[13](NAS IP address)
@[14](Export path + folder)

---?image=https://wallpaper.wiki/wp-content/uploads/2017/04/wallpaper.wiki-Binary-Code-Background-Full-HD-PIC-WPB003927.jpg

### Make the Persistent Claim

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: dc612
spec:
  accessModes:
    - ReadWriteMany
  storageClassName: ""
  resources:
    requests:
      storage: 1Mi
```
@[11](Must match volume spec)

---?image=https://wallpaper.wiki/wp-content/uploads/2017/04/wallpaper.wiki-Binary-Code-Background-Full-HD-PIC-WPB003927.jpg

### New replication controller with persistence

```yaml
apiVersion: v1
kind: ReplicationController
metadata:
  name: dc612-rc
spec:
  replicas: 1
  selector:
    app: dc612
  template:
    metadata:
      labels:
        app: dc612
    spec:
      containers:
      - name: minecraft        image: itzg/minecraft-server
        env:
        - name: EULA
          value: "TRUE"
        - name: VERSION
          value: "1.13.2"
        imagePullPolicy: Always
        ports:
        - containerPort: 25565
        volumeMount:
        - name: dc612-data
          mountPath: "/data"
      volumes:
      - name: dc612-data
        persistentVolumeClaim:
          claimName: dc612
```
@[24-26](Volume mount in container)
@[25](Name of the volume we are mounting)
@[26](Where to mount inside of the container)
@[27-30](Volume list to fullfull requests)
@[28](Matching volume name)
@[29-30](How to fufill the claim)
---


### References 

Install Kubeadm
https://kubernetes.io/docs/setup/independent/install-kubeadm/

Install CRI
https://kubernetes.io/docs/setup/cri/

CNI Overview
https://chrislovecnm.com/kubernetes/cni/choosing-a-cni-provider/

Service Types
https://medium.com/google-cloud/kubernetes-nodeport-vs-loadbalancer-vs-ingress-when-should-i-use-what-922f010849e0

Metal LB
https://metallb.universe.tf/

Traefik
https://traefik.io/

NFS Provisioner
https://github.com/helm/charts/tree/master/stable/nfs-client-provisioner


