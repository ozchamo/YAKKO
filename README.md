# YAKKO 
## [Y]et [A]nother [K]VM [K]onfigurator for [O]penShift 

## WHAT IS YAKKO?
**The Short Explanation:** YAKKO installs OpenShift on your server running RHEL/Fedora.  

**The Long Explanation:** YAKKO is an "IPI-like" installer for Red Hat OpenShift (IPI = Installer Provisioned Infrastructure). What this means is that the installer also provisions everything! In this case, "everything" sits in one system, which is why there is no Red Hat provided installer for this non-enterpise situation. So, if you want to run up an OpenShift cluster with multiple nodes (1 or 3 masters/many workers) or why not, a Single Node Cluster ("SNC") and you have a big PC/small server with RHEL or Fedora, then YAKKO might be for you. 

There are plenty of cookbooks out there and they require that you do a lot of *manual* work. **YAKKO avoids it!** If you are a Linux tinkerer with a penchant for the command line and all things Red Hat/Fedora, YAKKO might just be that new friend you were looking for to play with Kubernetes.

---

## CURRENT VERSION: 5.02 (20230321.2053)
What's new? 

WHAT, 5.0?!!! WOWZA! WHY?
- This version rearranges the installation of required packages in preparation for a feature in BETA - the capability of building in disconnected fashion! 
  (This will require a registry properly configured.)
- Tested on RHEL 9.1 and Fedora 37 (F37 is all good now) 
- Added changing of OpenShift installer output verbosity (because sometimes there can be trouble...)
- Added links to your cluster in console.redhat.com on the 'yakko' text dashboard and on the http service
- Added a new option "yakko buildcluster" - this allows you to feed a cluster configuration file as the automatic base to build from (the file needs to be in the format that .lastyakkobuild creates after build a cluster). 
- Tested with OCP 4.12 (SNO and multi-master/multi-node configs)
- A few bugs cleaned up here and there

What's in the works already? 
- Disconnected installation (but you knew this already, see above, and wish me luck!)
- Looking at REMOTE nodes, yes! If only port 4789 wasn't UDP, I'd be done by now. If you want to participate on this, drop me a comment.

And some of the cool features that have been there a for a while...
- Adapt to changes in the IP address of the server (e.g. when changing wireless networks!)
- Support for single node/single master clusters since OpenShift 4.10
- Support for installing multiple clusters (but run one at a time!)
- Support for resizing (master/worker) node RAM - on the go!
- Setting up NFS shares for registry and for Namespace/Project storage on your server
- Assigning NFS shares for registry and for Namespace/Project storage 
- Purging existing downloaded OpenShift images on disk
- List services and files that are in use by a cluster
---
## INTRODUCTION
YAKKO was built around the concept of having ONE script/installer/manager that does it all, using the underlying operating system as the installation/operation platform and resource server/service. As a prime example, YAKKO depends on libvirt/KVM and so it will install and configure required packages on your server to build and run OpenShift VMs, just as it may be used as the DNS resource should you not have your own DNS. Because of this, YAKKO is a bit opinionated, but then again, it's not built for creating "production ready" clusters, and so it should suit most people with a passing need or interest in having an OpenShift cluster around (or... again!)

Why would you want to run your own (single-box) cluster, isn't that self defeating because there is no real resilience?
- A full cluster at your disposal lets you test full cluster functionality
- You can experiment with multiple node setups in an easy self-hosted lab fashion
- You can easily setup different versions of OpenShift and examine features and compare behaviour 
- No public cloud bill... or bill shock! 
- You can test your more complex apps on multiple worker nodes 'for real' 
- You might be a a fan of "Self hosting"
- OpenShift Pull Secrets from Red Hat have a lifetime of 60 days, so re-installing is kinda useful
- You can mess with different clusters on the same server (BUT NOTE! Only one cluster running at a time!)
- You can create cheap clusters for experimenting with Red Hat Advanced Cluster Manager (RHACM)
- LEARN without Fear or/of consequences!

In a nutshell, what does YAKKO do? 
- Sets up and installs any and all requirements/dependencies for you
- Installs OpenShift in a configuration of your choice:
  - Single-node cluster
  - Single-master cluster (with multiple workers)
  - 3-master cluster with or without worker nodes
  - Add worker nodes, on initial build or later
- Builds automatically or in stages
- Leverages the host as bastion / HAProxy / DNS / image/storage server 
- Simplifies the networking by using a KVM network behind NAT 
- Rolls back individual failed stages so that you can fix if necessary and then keep going or just delete everything you've done so far and start afresh. 
  (Be careful - It's scary how quickly it does away with a happily running cluster!)
- Adds and destroys worker and infra nodes easily to suit your use case 
- Lets you create additional clusters (not just one, BUT you can only run one at a time)
- Overlays basic operational stuff - once you have the cluster up, it will hint you, using 'yakko [infra | ops]'
- Deletes the entire operational cluster you've built, and unconfigure all the above 
- But don't worry - you can build again right? Automatically... (try 'yakko rebuildcluster' for this!) 
- All of this very tidily - if you stop (or delete) a cluster, it will retire all associated system services files

---
## WHAT YAKKO IS NOT
It is not a management tool for OpenShift. It has a small overlay of features to assist in the "automation" of getting things done that may otherwise be repetitive, but once your cluster is up, you can delete YAKKO for all you know, however, since it can do a few things post install (see "Day 2 Ops)" as well as allow you to delete all VMs and the configuration in your system, you should keep it!

---
## REQUIREMENTS
**A single PC/server with:**
- Access to the internet
- RHEL/Fedora as the base installed operating system ("Server with GUI" and then YAKKO gives you a working cluster)
- Ports 80 and 443 available (to pass through to your applications in OpenShift)
- 16GB+ RAM for a single node cluster (good luck though and note that your ability to run apps will be impaired, recommended is 32GB. I've succeed with as little as 12GB, used for a Single Node Cluster setup)
- 32GB+ RAM for a 3 master cluster, likely no workers (I've succeeded on a box with 24GB but it's old and the CPU gets in the way :)
- 48GB+ for multi-node clusters (3 masters + a worker or three :) 
- 2.5GB of disk space for the install files (YAKKO will accumulate older OpenShift versions so keep an eye on the "images" directory within the directory where it resides)
- SSD class storage with capacity as follows:
    - 3 masters require 90GB (30GB each)
    - worker nodes require 20GB each
    - you can tweak the disk sizes if you must - edit YAKKO and look for MASTERDISKSIZE and WORKERDISKSIZE

**Tested combinations to date with this release:**
- OpenShift 4.12
- RHEL 9.1 and Fedora 37 

**What's the test bed**  
The testbed to build and test YAKKO is an Alienware Aurora R6 with an Intel i7-7700 (4c/8t @ 3.6GHz, ~2017) w/64GB RAM and one m.2 512GB SSD. For fun, the largest cluster I have built on it had 6 worker nodes. This machine has seen the build of more than 300 OpenShift clusters with YAKKO! This system has Fedora 37, which is still undergoing issues with some operators not fully coming up. 
The current "prod" system is an Intel NUC with a i9 8-core CPU, 64GB RAM, running RHEL 9.1
And my "RHEL 9 dev" system? A laptop! (It's a sweet Lenovo Thinkpad P1 Gen-3 with 64GB RAM and 8c/16t). And no, I have never used spinning disk, if you do, I wish you luck.
  
**Nice to Haves**
- Linux skills - if you are even attempting at using this, you must have some already...
- Project cockpit is a good (though hungry) friend
- Your own DNS server that can handle wildcards (but YAKKO can handle this responsibility)
---
## HOW TO - INSTALL or "DAY 1"
#### ➜ [Watch YAKKO 4.20 build a 3 master/worker cluster"](https://asciinema.org/a/497235)
#### ➜ [Watch YAKKO 4.01 build a Single Node OpenShift cluster "SNO"](https://asciinema.org/a/2DJYgTFn1R9wLHVYDuCi5isaf)
#### ➜ [Watch YAKKO 4.01 build a 3m+2w Cluster](https://asciinema.org/a/NloEXfUHUdXVH6NIUOdcSkHqF)
#### ➜ [Watch (an earlier vesion of) YAKKO in action building OpenShift (video with voiceover)](https://youtu.be/hLsUp7dwxdQ)
#### STEPS
1) Get the 'yakko' script as user "root": 
   - You can clone the repo (ideally on /) OR  
   - download it from https://github.com/ozchamo/YAKKO/raw/master/yakko  
2) Run 'yakko' as root (always!) - e.g. `[root@ocphost YAKKO]# ~/Downloads/yakko`
3) Choose a destination home directory for YAKKO - **usually /YAKKO** - you will be asked to re-run from there. Quite typically you will:
   - cd /YAKKO   # (if this is where you installed it)
   - ./yakko     # (to run it from the directory you're in. Surely you knew this ;)
4) 'yakko' will start the OpenShift install process when there is no cluster defined, so no further parameters are necessary
5) Follow instructions, my suggestion is that you run it manually until you get the hang of it
6) Once you get the flow, it can build a cluster AUTOMATICALLY. I've built many in one week
7) Depending on your hardware and desired cluster size (mine's OK, not overly powerful) you can have a cluster up and running in 20-50 minutes
8) Until there is no operational cluster, YAKKO will keep asking you to continue the install from where you left off
9) Once a cluster is operational, YAKKO reports something like this, anytime you run it without parameters:

```
__________________________________________________________________________

 YAKKO: Yet Another KVM Konfigurator for Openshift (Ver. 5.0)
__________________________________________________________________________

 CLUSTER: cluster.testdomain  (ID: 4c43f198-91ee-424c-a746-c6f73434cb72)
 Version: 4.12.4  (Built: 26-Feb-2023@14:27:53)
 @RedHat: https://console.redhat.com/openshift/details/4c43f198-91ee-424c-a746-c6f73434cb72)

               state      
 Web Console:  [ ✔ ]  https://console-openshift-console.apps.cluster.testdomain
 API Service:  [ ✔ ]  https://api.cluster.testdomain:6443

 Active Masters:   3/3
 Active Nodes:     2/2 (workers/infra)
 Active Operators: 33/33 (1 progressing, 0 degraded)

 Administrator: kubeadmin
 Password:      DVTGk-Wa85w-yrFQc-cTK6n

 Registry configuration: local

 External access: ENABLED (to change: yakko infra changeaccess)

 - See yakko command usage --------> yakko usage
 - Make infrastructure changes ----> yakko infra <options>
 - Make operational changes -------> yakko ops <options>
 - Use OpenShift's 'oc' command ---> source ocp-setup-env  (in this shell)
 - Basic cluster info at ----------> http://192.168.100.2:8080
 - Access cluster externally ------> Add [192.168.100.2] as a DNS server in your clients
   (This provides an alternative for when configuring DNS in your network is not possible)

 NOTE: You did not have KUBECONFIG set when you invoked YAKKO.
       Remember to use 'source ocp-setup-env' or adjust your environment accordingly!
```
---
## GENERAL USAGE
The best and quickest way to understand the YAKKO idiosyncrasy is by reading the output of 'yakko usage':

`[root@ocphost YAKKO]# ./yakko usage`

```
__________________________________________________________________________

 YAKKO: Yet Another KVM Konfigurator for Openshift (Ver. 4.42)
__________________________________________________________________________

When NO CLUSTER is configured you can call: 
    yakko  (no params)    -> build a new cluster
    yakko rebuildcluster  -> recreate the last cluster built

When a CLUSTER IS CONFIGURED you can call:
    yakko  (no params)    -> show running cluster state and configuration
    yakko startcluster    -> startup the cluster (same as yakko infra startcluster)
    yakko stopcluster     -> shutdown the cluster (same as yakko infra stopcluster)
    yakko deletecluster   -> delete the running or stopped cluster (same as yakko infra deletecluster)
    yakko infra <options> -> make infrastructure changes offered by yakko (see below)
    yakko ops <options>   -> make operational changes offered by yakko (see below)

At any time, you can call:
    yakko addcluster      -> add a new cluster (this will setup YAKKO in a new directory)
    yakko purgedownloads  -> delete downloaded OpenShift images

Calling 'yakko infra' (with no option) will always remind you of the following:
USAGE: yakko infra <OPTION> [parameters]

(... SEE BELOW - DAY 2 OPS: yakko infra ...)

Calling 'yakko ops' (with no optiona) will always remind you of the following:
USAGE: yakko ops <OPTION> [parameters]

(... SEE BELOW - DAY 2 OPS: yakko ops ...)

```
---
## DAY 2 OPERATIONS - Infrastructure ("infra") and OpenShift ("ops") features
Once you have created a cluster, YAKKO is not intended to do much more, after all the idea is to learn and experiment, however, it automates a few "procedures" that are useful on a day to day basis. 

These are called through passing a parameter to YAKKO, either "infra" or "ops", as explained above. Hopefully each function is self explanatory through the usage shown.

Admittedly, the most important and frequently used operations are **'startcluster'**, **'stopcluster'** and **'deletecluster'** as these operations not only modify the state and existence of the cluster, they also leave your system in a proper state for running or stopping a cluster, and cleaning up when you do a delete. Unless you really get under the covers, you should keep the YAKKO script until you don't need OpenShift as delivered by YAKKO anymore and ideally after you have done a full cluster delete should you do away with YAKKO.
<br>

**"yakko infra \<parameter\>"**:  
Relates to the mundane tasks related to the infrastructure of the cluster and the resources running on your host, outside of the purview of OpenShift:
   
`[root@ocphost YAKKO]# ./yakko infra`
```
USAGE: yakko infra <OPTION> [parameters]  
OPTION is one of:  
    - startcluster    ->  Start up an existing cluster  
    - stopcluster     ->  Shutdown an existing cluster  
    - addnode         ->  Grow the cluster compute capacity by adding a new compute/infra node  
    - deletenode      ->  Remove a running node from the cluster  
    - nodelogs        ->  Display the logs of a particular node  
    - sshtonode       ->  Provide terminal access to an individual cluster node  
    - changeaccess    ->  Enable/disable OpenShift access by other clients in your network  
    - restartservices ->  Restart supporting services for cluster (virt network/HAproxy/libvirtd)
    - listresources   ->  Print a summary of services and files in use by the (YAKKO) cluster
    - describehw      ->  Describe the harware supporting the installation
    - resizeram       ->  Change the RAM size of a node
    - purgedownloads  ->  Delete all downloaded OpenShift images on disk
    - nfsshare        ->  Setup a directory as NFS share for creating a PVC for registry or NS store
    - installcomplete ->  Mark a cluster build as completed even if the installer refuses to say it is
    - deletecluster   ->  Delete entire cluster and all infrastructure  
```
<br>
  
**"yakko ops \<parameter\>"**:  
Relates to the higher level Kubernetes/OpenShift stuff that make life more easy going. These are all functions that ultimately you can figure out by yourself and experiment to achieve, but when all else fails and time is of the essence, yakko can do them for you. These came from a need of scripting OpenShift 'stuff' that I repeated over and over for testing and for my own use, and also as placeholders to remind me of information sources and how to get things done. They eventually got productised:

`[root@ocphost YAKKO]# ./yakko ops`

```
USAGE: yakko ops <OPTION> [parameters]  
OPTION is one of:  
    - htpasswd        ->  Deploy local password access and a new administrator  
    - useradd         ->  Add a new user to local password DB  
    - userdelete      ->  Delete an existing user from the local password DB  
    - mastersched     ->  Enable/disable master scheduling  
    - nodelabel       ->  Change the label of a node between worker <-> infra  
    - localregistry   ->  Enable a local registry so you can actually use the cluster  
    - nfsregistry     ->  Enable an existing NFS share as registry (persistent)
    - nfsmap          ->  Map an existing NFS share to a namespace
    - ingresscert     ->  Install an existing wildcard certificate
    - approvecsrs     ->  Approve any outstanding CSRs (Certificate Signing Requests)
    - yakkotest       ->  Deploy the 'yakkotest' app on your cluster, to test the lot!!  
```
---
## WHAT DOES THE YAKKO ARCHITECTURE LOOK LIKE?
![YAKKO - Architecture](YAKKO-architecture.png)

Interested in how this all works? Here goes a little explaining of the YAKKO building blocks, from bottom to top when looking at the architecture above.
- The **big grey box** is your server, with CPU and disk
- Next,the **operating system** that it all sits on is either Red Hat Enterprise Linux or Fedora
- The **big white box** represents all that YAKKO will provision... See next points!
- **HAProxy:** Used to route all traffic to the cluster. Yakko will rebuild the HAProxy as you scale the cluster up and down. HAProxy is fundamental, as it will act as the transparent bridge between the server and the virtualised clusted nodes, as well as provide access to the cluster for any systems coming in over the public network. Simply put, if you are running a YAKKO cluster on your laptop, HAProxy will allow you to interact transparently with it from your laptop, while permitting other users on external machines (e.g. other laptops on the same network) to interact with the cluster in exactly the same way. 
  - Ports 80 and 443 - to worker nodes running 'your apps'. This includes the OpenShift console.
  - Ports 6443 and 22623 - to master nodes, for the API server and the machine config server
- **Apache Web Server (HTTPD)**: Used for two purposes:
  - It is the primary delivery mechanism of RHCOS images for the nodes to PXE boot over HTTP
  - When a cluster is up, it provides the user some basic cluster information via port 8080
- **DNSmasq:** Used for wildcard DNS into your OpenShift cluster, if you don't have such a service readily available. Note that DNSmasq is operated under NetworkManager - YAKKO will not provision a separate DNSmasq service nor will it interfere with an existing one. Running DNSmasq will also allow external users leveraging the YAKKO cluster as a nameserver to see the cluster with its own FQDN, no matter what local domain you set it up with!
- **KVM:** This is the virtualisation foundation for the cluster running on your server. It will be used to create all virtual machines that perform master/worker nodes as well as provision a virtual network to hide the cluster nodes' internal communication from the outside world.

When yakko starts a cluster, it will add the following files to your system and pull them when the cluster is subsequently shutdown as long as you use 'yakko startcluster' and 'yakko stop cluster':
- **HAProxy**: /etc/haproxy/conf.d/yakko-CLUSTERNAME-haproxy.cfg
- **HTTPD**: /etc/httpd/conf.d/yakko-prod-httpd.conf
- **NetworkManager**: /etc/NetworkManager/conf.d/yakko-CLUSTERNAME-NetworkManager.conf
- **DNSmasq**: /etc/NetworkManager/dnsmasq.d/yakko-CLUSTERNAME-dnsmasq.conf 
- **systemd-resolved**: /etc/systemd/resolved.conf.d/yakko-CLUSTERNAME-resolved.conf (only if systemd/resolved is in use)
---
## WHAT IS YAKKO MISSING?  (Backlog of sorts?)
Short of this being a backlog...
- BYO network (i.e. don't depend on a virtual network) - this should be mostly easy, but the business case for adding this is not very clear!
- Adding nodes from other physical machines and moving virtual nodes around (which may well defeat YAKKO's own purpose)
- Possibly a few 'certainty' principles of higher level systems administration, this said, it tries to keep your firewall on, your SELinux running etc etc.
---
## QUESTIONS YOU MAY HAVE, FOR FUN
- Why didn't I use Ansible? 
I could, I chose not to, because I would have had to learn another TON of stuff. I actually pulled out a couple of lines where I did use it. I wanted this to be ONE script with no additional downloads for code, no dependencies of other scripts. YAKKO is not big scale automation anyway.
- What if I have two boxes and I want to spread the load?
I want to cook that too. I have two boxes, Large and medium. My dream is to turn the medium box into a CNV node. Now that remote nodes are supported, this is in the short term roadmap, success looks promising but not guaranteed!
- What are the minimum requirements?
See above. I've happily succeeded with a 4 core/8 thread server from 2009, a Sun Ultra 27! It may well be the only Sun box in the Universe running OpenShift :)  (note - Single Node Cluster only on this hardware)
- Is it AUTOMATIC
Yes, after you master the basics. Who doesn't want to rebuild OpenShift all the time?
- Do you have to really deploy HAProxy on the box? 
It's more than a convenience. HAProxy bridges the virtual network so that both the OpenShift host and other hosts on your network can talk to the cluster in the same way and through the 'public' network.
- Lots of other stuff I haven't thought off, so leave your comments!
---
## COMMITMENT and ACKNOWLEDGEMENTS
- I hereby pledge to test and update as new releases of OpenShift, RHEL and FEDORA come out... Until I don't, and then I will delete this section :)
- I was inspired in automating this after receiving a certification in "Advanced Red Hat OpenShift Container Platform Deployment and Management" and later reading https://github.com/eitchugo/openshift-libvirt. Thanks Hugo! It was "short" and after typing in all the looooong host kernel parameters required for each VM to boot, I decided that this was worth investing time into. But there are a ton of cookbooks out there, they are all different. I didn't want to write another cookbook, I thought it would be more fun to write a bot-chef to cook for me. 
- I needed a COVID confinement project. This is it!
---
## MY OWN EXPERIENCE AND FINAL WORDS
- YAKKO is a big undertaking. It began as an exercise in curiosity and developed into a fledgling almost-product
- As I created clusters and then tried things I didn't know how to do, I quicky appreciated having the ease of rebuilding from scratch. I can see YAKKO users taking the same approach! (but be careful of the destructive power of "yakko deletecluster force"!)
- Kubernetes, with all the ultra-modern design principles of distributed computing and software delivery it promotes, is replacing the face of everything we took for granted in computing: hardware becomes even more commoditised (anything goes, including Raspberry Pies), virtualisation is turned on its head to become a second (or even third) class citizen (due to CNV - Container Native Virtualisation), dependencies are a thing of the past, resilience is a given, and immutability and statelessness are here to stay for the good of mankind. Kubernetes is the new Operating System (of the datatacentre) and Containers are Linux. Time to learn again.
- There will always be people, like me, who hold "self-hosted" as a guiding principle to learn from and... Enjoy!



