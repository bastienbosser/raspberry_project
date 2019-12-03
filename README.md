# Create High Available Cluster With Some Raspberry Pi

<img src="https://user-images.githubusercontent.com/55381632/70054406-c9710280-15d7-11ea-9d49-e7a51e0fa6c2.png" width="100">

---
The objective of our project is to create a High Available cluster on some Raspberry Pi with k3s and use this cluster to 
deploy a streaming platform.

This tutorial is made for all people who want to try what k3s can do on a cluster of X Raspberry Pi. 
High Available means that we have multiple master and load balancer. 
Here, we used 15 Raspberry Pi 3 Model B+ and 10 Raspberry Pi 4 Model B.

As it's written on the kubernetes documentation, please notice that:
"Your cluster must run k3s version 1.0.0.
You should also be aware that setting up High Available clusters with xinetd and haproxy is
still experimental. You might encounter issues with upgrading your clusters,
for example. We encourage you to try either approach, and provide feedback."

Our method is probably not the best, but it works properly. We are not perfect, we
made a lot of mistakes to come to a decent result. If you have a better
solution for that setup, please make us reports and feedbacks.

---

## Who are we?

This solution is proposed by a group of students in the CyberSecurity Classroom (2019)
of the Engineering School called "ISEN Yncrea Méditerranée" at Toulon (France).
The team is composed of BOSC Virginie, BOSSER Bastien, RIGAUX Thomas, SODA Théo and VALLAURI Mélissa.
All this tutorial is a result of our project.

## Why Raspberry Pi?

"k3s is an open source system for managing containerized applications across
multiple hosts; providing basic mechanisms for deployment, maintenance, and
scaling of applications."

So, the question is legitimate: Why a Setup on Raspberry Pi ?
There are many reasons for that:
- The challenge
- It's experimental, so we learned lot of things about how things work.
- If an application is enough optimize to work on Raspberry Pi (like WebApp or WebSite), the traffic can be manage with 
that solution, so you will not be forced to take a server immediatly
- You can learn to manage k3s on local solution, with a very low cost.

## Let's start!

### Requirements

- 5 Raspberry Pi (3 B+ or 4B in our case) at least
- Same amount of micro-SD cards
- Hypriot OS (1.9 in our case): https://blog.hypriot.com/downloads/

### First Step: Format SD Card + Install OS

If your cards are not new thought to format them.
If you have a write permission problem when formatting SD cards, you can use this [link](https://bit.ly/37Y37AC).

After this, flash all your cards with Hypriot. You can use Linux commands to flash your cards.
If you have Windows or if you prefer graphical stuff, please look at solutions like [Etcher](https://www.balena.io/etcher/) 
or go on your own.


Modify the hostname of each pi by something which can be use to identify it.  
For our solution, we have 25 Raspberry Pi with these hostnames: rasp00,rasp01...rasp25.

To modify the default hostname, you should change the line “hostname: black-pearl” by “hostname: myname” in file user-data.
It is important to realise this step after having flashed SD cards and before booting Raspberry Pi for the first time.

By default, ssh is fully open on Hypriot installs.
You can connect your pi via:

    ssh pirate@X.X.X.X (Replace X.X.X.X by Raspberry's IP)  
    Default password: hypriot 

Notice that, if you don't know Raspberry IP, you have many solutions:  
- Try to get a graphic access via HDMI
- Analyse your network with tools like [Angry Ip Scanner](https://angryip.org/) (By default, their hostname is black-pearl)
- You can fix Raspberry's IP address by adding “ip=X.X.X.X” in file cmdline.txt (Replace X.X.X.X by an IP address of your 
network)

On each PI you will now need to install Kubernetes.
If you have too many, write a script or use Ansible can be a good idea.
Please now connect on each pi, and do the following commands:

    sudo apt-get update && sudo apt-get install -y apt-transport-https
    curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add -
    echo "deb https://apt.kubernetes.io/ kubernetes-xenial main" | sudo tee -a /etc/apt/sources.list.d/kubernetes.list
    sudo apt-get update
    sudo apt-get install -y kubectl kubelet kubeadm

### Second Step: Define your server

Our team have 16 Raspberry Pi. Our design follow these rules:

- All our Pi have fixed IP from 10.10.50.194 to 10.10.50.219
- Our master will be .198, .203, .208, .214 and .219
- They have all an hostname following their IP: rasp00 = .194 rasp25= .219

So, now, deploy the master .198 with k3s:

    sudo curl -sfL https://get.k3s.io | sh -

Retrieve the TOKEN that will allow to deploy the slaves with the command:

    sudo cat /var/lib/rancher/k3s/server/node-token
    
Now, deploy slaves .194,..., .197 with k3s:

    curl -sfL https://get.k3s.io | K3S_URL=https://myserver:6443 K3S_TOKEN=XXX sh - (Replace myserver by the IP of master and XXX by token retrieve)
    sudo k3s agent --server https://myserver:6443 --token NODE_TOKEN (Replace myserver by the IP of master and NODE_TOKEN by token retrieve)

Verified that all Raspberry are ready by executing the following command on the master:

    sudo k3s kubectl get nodes

Repeat this operation for each master and their slaves.

### Third step: Define your Load Balancer




