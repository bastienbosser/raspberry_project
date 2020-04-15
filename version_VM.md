# Version with Virtual Machine

### Setup Local VMs K8s Cluster Multi-masters

First, you should create a NatNetwork for the cluster. In VirtualBox, go to Preferences >> Network >> create NatNetwork 
<img src="https://user-images.githubusercontent.com/55381632/79360902-dbc75980-7f44-11ea-9f42-e49a391e01f9.png" width="25">

You should have something like this:

<img src="https://user-images.githubusercontent.com/55381632/79360904-dbc75980-7f44-11ea-9146-a657f3e1fe1d.png">

You can change the name if you want, to do this just right click on your NatNetwork and on Properties.

Then Download Debian-iso: https://cdimage.debian.org/debian-cd/current/amd64/iso-dvd/

Now, create 3 VM in VirtualBox: Master1, Master2 and Worker.

<img src="https://user-images.githubusercontent.com/55381632/79360899-da962c80-7f44-11ea-91ad-fe55b5d3985a.png">

#### Configurations Recommendations

- I recommend at least 2048MB of RAM for the Master 1, it should be nice for Master 2 but 1024 should do the trick and 
for the worker 1024 should be enough.
- Two CPU cores for Master1 and Master2 and one for Worker.
- You can Install Master1 with Desktop for convenience, but it is not necessary and server type for Master2 and Worker 
will be sufficient.
- For the storage it depends on your needs, I personally recommend 15GB,10GB and 10GB.
- Go to settings in each VM and change the network settings to:

<img src="https://user-images.githubusercontent.com/55381632/79360893-d964ff80-7f44-11ea-8b53-71cdfd8c600a.png">

- I advise to use the same password for the 3 users if you want to use ansible.

Now, for convenience you can do repeat this step on each VM.

As root,

    sudo adduser master1 sudo

Then, be Sure to have the config that follow in /etc/apt/sources.list

    deb http://deb.debian.org/debian buster main
    deb-src http://deb.debian.org/debian buster main

    deb http://security.debian.org/debian-security buster/updates main contrib
    deb-src http://security.debian.org/debian-security buster/updates main contrib

    deb http://deb.debian.org/debian buster-updates main
    deb-src http://deb.debian.org/debian buster-updates main
    
Go Back to virtual box preferences >> Network > rightclick k8sNetwork edit > Port Forwarding and you can setup something like 
that to communicate with the host machine, there I have enable ssh for the 3 VM





