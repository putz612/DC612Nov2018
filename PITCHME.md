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

---

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

---
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