# Create High Available Cluster With Some Raspberry Pi

<img src="https://user-images.githubusercontent.com/55381632/70054406-c9710280-15d7-11ea-9d49-e7a51e0fa6c2.png" width="100"><img src="https://user-images.githubusercontent.com/55381632/70060276-823c3f00-15e2-11ea-868f-4ac006b74642.png" width="100">


---
The objective of our project is to create a High Available (HA) cluster on some Raspberry Pi using k3s and the aim of this cluster to 
deploy a video streaming platform.

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
solution for that setup, please send us reports and feedbacks.

---

## Who are we?

This solution is proposed by a group of students in the CyberSecurity Classroom (2019)
of the Engineering School called "ISEN Yncrea Méditerranée" at Toulon (France).
The team is composed of BOSC Virginie, BOSSER Bastien, RIGAUX Thomas, SODA Théo and VALLAURI Mélissa.
This whole tutorial is the result of our project.

## Why Raspberry Pi?

"k3s is an open source system for managing containerized applications across
multiple hosts; providing basic mechanisms for deployment, maintenance, and
scaling of applications."

So, it is legitimate to think why should we use this setup on Raspberry Pi.
There are many reasons for that:
- The challenge
- It's experimental, so we learned a lot of things about how things work.
- If an application is enough optimized to work on Raspberry Pi (like WebApp or WebSite), the traffic can be managed with 
that solution, so you will not be forced to take a server immediatly.
- You can learn to manage k3s on a local solution, at a very low cost.

## Let's start!

### Requirements

- 5 Raspberry Pi (3 B+ or 4 B in our case) at least
- Same amount of micro-SD cards
- Hypriot OS (1.9 in our case): https://blog.hypriot.com/downloads/

### First Step: Format SD Card + Install OS

If your cards are not new, think about formatting them.
If you have a write permission problem when formatting SD cards, you can use this [link](https://bit.ly/37Y37AC).

Then, flash all your cards with Hypriot. You can use Linux commands to flash your cards.
If you have Windows or if you prefer graphical stuff, please look at some solutions like [Etcher](https://www.balena.io/etcher/) 
or go on your own.


Modify the hostname of each Pi by something which can be use to identify it easily.  
Concerning our project, we have 25 Raspberry Pi with these hostnames: rasp00,rasp01...rasp25.

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
If you have too many, write a script or using Ansible can be a good idea.
Please now connect on each pi, and do the following commands:

    sudo apt-get update && sudo apt-get install -y apt-transport-https
    curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add -
    echo "deb https://apt.kubernetes.io/ kubernetes-xenial main" | sudo tee -a /etc/apt/sources.list.d/kubernetes.list
    sudo apt-get update
    sudo apt-get install -y kubectl kubelet kubeadm

### Second Step: Define your server

Our team have 25 Raspberry Pi. Our design follow these rules:

- All our Pi have fixed IP from 10.10.50.194 to 10.10.50.219
- Our master will be .198, .203, .208, .214 and .219
- They have all an hostname following their IP: rasp00 = .194 rasp25= .219

So, now, deploy the master .198 with k3s:

    sudo curl -sfL https://get.k3s.io | sh -

Retrieve the TOKEN that will allow you to deploy the slaves using the following command:

    sudo cat /var/lib/rancher/k3s/server/node-token
    
Now, deploy slaves .194,..., .197 using k3s:

    curl -sfL https://get.k3s.io | K3S_URL=https://myserver:6443 K3S_TOKEN=XXX sh - (Replace myserver by the IP of master and XXX by token retrieve)
    sudo k3s agent --server https://myserver:6443 --token NODE_TOKEN (Replace myserver by the IP of master and NODE_TOKEN by token retrieve)

Verified that all Raspberry are ready executing the following command on the master:

    sudo k3s kubectl get nodes

Repeat this operation for each master and their slaves.

### Third step: Define your Load Balancer

HAProxy is a TCP/HTTP relay offering easy integration in a highly available environment. Indeed, he is capable:
- to perform static routing defined by cookies;
- to perform load balancing with cookie creation to ensure session persistence;
- to provide external visibility of one's health status;
- to stop smoothly without sudden loss of service;
- to modify/add/delete headers in the request and response;
- to prohibit requests that verify certain conditions;
- to use backup servers when the main servers are out of order;
- to maintain clients on the right application server based on application cookies;
- Provide HTML status reports to authenticated users, through intercepted application URIs.

It requires few resources, and its single-process event architecture allows it to easily manage several thousand simultaneous connections on several relays without collapsing the system, so we will focus mainly on load balancing, at least initially.

The Pi .204 will be our load balancer.

#### Configuring your haproxy server

So, now, install haproxy on Pi .204 and telnet:

    sudo apt-get install haproxy
    sudo apt-get install telnet

Create a backup of the basic configuration: 

    sudo cp /etc/haproxy/haproxy.cfg /etc/haproxy/haproxy.cfg.old

Edit the file /etc/haproxy/haproxy.cfg to obtain the configuration below:

    global
    log 127.0.0.1 local2
    chroot /var/lib/haproxy
    pidfile /var/run/haproxy.pid
    maxconn 4000
    user haproxy
    group haproxy
    daemon

    # turn on stats unix socket

    stats socket /var/lib/haproxy/stats

    defaults
    mode tcp
    log global
    option dontlognull
    option http-server-close
    #option forwardfor except 127.0.0.0/8
    option redispatch
    retries 3
    timeout http-request 10s
    timeout queue 1m
    timeout connect 10s
    timeout client 1m
    timeout server 1m
    timeout http-keep-alive 10s
    timeout check 10s
    maxconn 3000

    listen haproxy_servers

    # bind is the listening port of your HAPROXY

    bind *:6443
    mode tcp
    option tcplog
    timeout client 10800s
    timeout server 10800s
    #balance leastconn
    default-server inter 10s downinter 5s rise 2 fall 2 slowstart 60s maxconn 200 maxqueue 250 weight 100 error-limit 10 on-error mark-down on-marked-down shutdown-sessions agent-port 9707 agent-inter 30s
    server rasp04 10.10.50.198:6443 check agent-check observe layer4
    server rasp09 10.10.50.203:6443 check agent-check observe layer4
    server rasp14 10.10.50.208:6443 check agent-check observe layer4
    server rasp20 10.10.50.214:6443 check agent-check observe layer4
    server rasp25 10.10.50.219:6443 check agent-check observe layer4
    server localhost 127.0.0.1:6443 maxconn 500 backup weight 1

    backend servers

    mode tcp
    option tcplog
    #balance roundrobin
    server rasp04 10.10.50.198:6443
    server rasp09 10.10.50.203:6443
    server rasp14 10.10.50.208:6443
    server rasp20 10.10.50.214:6443
    server rasp25 10.10.50.219:6443
 
#### Installing the agent on the host machines (.198, .203, .208, .214, .219)

Create a file /usr/local/bin/haproxy-agent-check and insert the script below:

    #!/bin/bash
    LMAX=90

    load=$(uptime | grep -E -o 'load average[s:][: ].*' | sed 's/,//g' | cut -d' ' -f3-5)
    cpus=$(grep processor /proc/cpuinfo | wc -l)

    while read -r l1 l5 l15; do {
    l5util=$(echo "$l5/$cpus*100" | bc -l | cut -d"." -f1);
    [[ $l5util -lt $LMAX ]] && echo "up 100%" && exit 0;
    [[ $l5util -gt $LMAX ]] && [[ $l5util -lt 100 ]] && echo "up 50%" && exit 0;
    echo "down#CPU overload";
    }; done < <(echo $load)

    exit 0

Adjust the permissions so that the script is executable:

    sudo chmod +x /usr/local/bin/haproxy-agent-check

Install xinetd, bc and telnet:

    sudo apt-get install xinetd
    sudo apt-get install bc
    sudo apt-get install telnet

Edit the file /etc/services and add the line below:

    haproxy-agent-check 9707/tcp                # haproxy-agent-check

Create the file /etc/xinetd.d/haproxy-agent-check with the following content:

    # default: on
    # description: haproxy-agent-check
    service haproxy-agent-check
    {
    disable         = no
    flags           = REUSE
    socket_type     = stream
    port            = 9707
    wait            = no
    user            = nobody
    server          = /usr/local/bin/haproxy-agent-check
    log_on_failure  += USERID
    per_source      = UNLIMITED
    }

Adjust the rights:

    sudo chmod +x /etc/xinetd.d/haproxy-agent-check
    
Restart the xinetd service:

    sudo service xinetd restart

Once you have done this on each of your machines, test the connectivity between your HAProxy server and your hosts via port 9707:

    sudo telnet 10.10.50.198 9707
    
Repeat this command for Pi .203, .208, .214 and .219.

Then restart the HaProxy service on the machine provided for this purpose:

    sudo service haproxy restart
    
Finally, from your HAproxy machine, check that your agents are responding correctly with the following command:

    sudo echo "show stat" | sudo socat stdio unix-connect:/var/lib/haproxy/stats  | sudo cut -d ',' -f1,2,18,19 | sudo grep haproxy
    
You should get a something like this:

    sudo kubectl apply -f livefox.yml
    
    sudo kubectl get -all --all-namespaces
    
    sudo kubectl delete -f livefox.yml

    

    

    
    
    



