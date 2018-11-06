---?image=https://i.imgur.com/8ghbf.jpg

# Minecraft on Kubernetes 

### It's not just for<br>-the kids-<br>anymore

---?image=https://cdn-images-1.medium.com/max/1600/1*9NG_meMVwlOwq30TWp5f_w.jpeg&opacity=20

### What are we going to cover

@ul[circles]
- Overview of Kubernetes
- Setup Requirements
- System Design
- Instalation 
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

---

### Kubernetes Inception

Deployments own replications sets which creates pods that contain containers that you can reach via services that connects to pods via endpoints of pods which can store persistant data in presistant volumes that it attaches to using persistant volume claims

---
@snap[north span-100]
System requirements
@snapend
@ul[list-bullets-circles]
- ESXi (or any other multi VM solution)
- 6GB free ram
- Ubuntu 18.04 ISO
- About 30Gb of datastore for VM's
- 3 static IP address
- NFS storage for persistance
@ulend

---

### The Ugly

- Everything is beta or alpha, get used to seeing that a lot
- Requirements and approved sofware change constently, get the latest
- RTFM and GitHub Issues is your best friend

---

### Sytem design

@ul[](false)
- Three nodes
  - One master
  - Two workers
@ulend