# YAKKO 
## [Y]et [A]nother [K]VM [K]onfigurator for [O]penShift 

## CURRENT VERSION: 2.0  (14-Aug-2021)
Support for OCP 4.8 and single node clusters, 'openaccess' is default, and a major cleanup with small improvements!

## INTRODUCTION
If you want to run up an OpenShift cluster, and have a big PC/small server then this might be for you. There are plenty of cookbooks out there and they require that you do a lot of *manual* work. **YAKKO avoids it!**

YAKKO was built around the concept of having ONE script/installer/manager that does it all, using the underlying operating system as the installation/operation platform and resource server/service. As a prime example, YAKKO depends on libvirt/KVM and so it will install and configure required packages on your server to build and run OpenShift VMs, just as it may be used as the DNS resource should you not have your own DNS. Because of this, YAKKO is a bit opinionated, but then again, it's not built for creating "production ready" clusters, and so it should suit most people with a passing need or interest in having an OpenShift cluster around (or again.

Why would you want to run your own (single-box) cluster, isn't that self defeating because there is no real resilience?
- A full cluster at your disposal lets you test full cluster functionality
- You can experiment with multiple node setups in an easy self-hosted lab fashion
- You can create cheap clusters for experimenting with Red Hat Advanced Cluster Manager (RHACM)
- You can easily setup different versions of OpenShift and examine features and compare behaviour 
- No public cloud bill... or bill shock! 
- You can test your more complex apps on multiple worker nodes 'for real' 
- You might be a a fan of "Self hosting"
- OCP Pull Secret from Red Hat have a lifetime of 60 days, so re-installing is kinda useful
- LEARN without Fear or Consequences!

In a nutshell, what does yakko do? 
- sets up and installs any and all requirements/dependencies for you
- installs OCP (latest if you want) automatically (if you want)
- leverages the host as the bastion host, right?
- eases the networking by using a KVM network behind NAT 
- leverages the host as load balancer with HAproxy
- rolls back individual failed stages so that you can fix if necessary and then keep going or just delete everything you've done so far and start afresh. It's scary how quickly it does away with a happily running cluster, so be careful...
- delete the entire operational cluster you've built, and unconfigure all the above 
- allows you to easily add and destroy worker and infra nodes to suit your use case 
- some basic operational stuff - once you have the cluster up, it will hint you, using "yakko [infra | ops]"
- but don't worry - you can build again right? Automatically...
- What doesn't it do? I dunno yet. Tons of stuff I presume, but who doesn't want to have OCP in their study?

## WHAT YAKKO IS NOT
It is not a management tool for OpenShift. It has a small overlay of features to assist in the "automation" of getting things done that may otherwise be repetitive, but once your cluster is up, you can delete YAKKO for all you know, but since it can do a few things post install (see "Day 2 Ops)" as well as allow you to delete all VMs and the configuration in your system, you should keep it!

## REQUIREMENTS
Access to the internet!

A single PC/server with:
- RHEL or Fedora as the base installed operating system ("Server with GUI" and then YAKKO gives you a working cluster)
- A fixed IP address to your network
- 24GB+ RAM for a single node cluster (easily)
- 32GB+ RAM for a 3 master cluster, likely no workers (I've succeeded on a box with 24GB but it's old and the CPU gets in the way :)
- 48GB+ for multi-node clusters (3 masters + many workers) with plenty RAM to spare
- 2.5GB of disk space for the install files (yakko will accumulate older OCP versions so keep an eye on the "images" directory within /YAKKO)
- SSD class storage, spinning disk has never been tested:
    - 3 masters required 60GB (30GB each)
    - worker nodes require 30GB each
    - you can tweak the disk sizes if you must - edit YAKKO and look for MASTERDISKSIZE and WORKERDISKSIZE

Tested combinations to date with this release:
- RHEL 8.4 
- Fedora 34
- OpenShift 4.7 and 4.8 

The testbed used to build and test 'yakko' is an Alienware Aurora R6 with an Intel i7-7700 (4c/8t @ 3.6GHz, ~2017) w/64GB RAM and one m.2 512GB SSD. For fun, the largest cluster I have built on it had 6 worker nodes.
    
## NICE TO HAVES
- Project cockpit is a good (though hungry) friend
- Linux skills - if you are even attempting at using this, you must have some already!
- Your own DNS server that can handle wildcards (but YAKKO assists)

## HOW TO - INSTALL or "DAY 1"
#### [[Watch (an earlier vesion of) YAKKO in action building OpenShift]](https://youtu.be/hLsUp7dwxdQ)
1) Get the 'yakko' script as user "root": 
    - You can clone the repo (ideally on /) OR  
    - download it from https://github.com/ozchamo/YAKKO/raw/master/yakko  
2) Run "yakko" as root (always!) - e.g. `[root@ocphost YAKKO]# ~/Downloads/yakko`
3) Choose a destination home directory for yakko **usually /YAKKO** - you will be asked to re-run from there
4) 'yakko' will start the OpenShift install process when there is no cluster defined, so no further parameters are necessary
5) Follow instructions, my suggestion is that you run it manually until you get the hang of it
6) Once you get the flow, it can build the cluster AUTOMATICALLY. I've built many in one week, and since keeping a tally, 200+ clusters...
7) Depending on your hardware (mine's OK, not overfully powerful) you can have a cluster up and running in 25-50 minutes
8) Until there is no operational cluster, "yakko" will keep asking you to continue the install from where you left off
9) Once a cluster is operational, YAKKO reports something like this, anytime you run it without parameters:

```
__________________________________________________________________________

 YAKKO: Yet Another KVM Konfigurator for Openshift
__________________________________________________________________________

 CLUSTER: prod.pichus.net  (Ver: 4.7.19  Built: 20-Jul-2021@01:15:57)

 Active Masters:   3/3
 Active Nodes:     2/2 (workers/infra)
 Active Operators: 31/31

              state      
 Web Console: [ ✔ ]  https://console-openshift-console.apps.myopenshiftcluster.localdomain
 API Server:  [ ✔ ]  https://api.myopenshiftcluster.localdomain:6443

 Administrator: kubeadmin
 Password:      M9jhk-znNif-UCiCE-NMwII

 External access: ENABLED (to change: yakko infra changeaccess)

 - To use OpenShift's 'oc' command --> source ocp-setup-env  (in this shell)
 - To make infrastructure changes ---> yakko infra <options>
 - To make operational changes ------> yakko ops <options>

```

## HOW TO - OPS or "DAY 2" "ops"
Once you have created a cluster, "yakko" is not intended to do much more, after all the idea is to learn and experiment. However, I've automated a few "procedures" that are useful on a day to day basis.

These are called through passing a parameter to YAKKO, either "infra" or "ops".

"yakko infra <parameter>" relates to the mundane tasks related to the infrastructure of the cluster:
   
`[root@ocphost YAKKO]# ./yakko ops`
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
    - deletecluster   ->  Delete entire cluster and all infrastructure  
```
"yakko ops" relates to the higher level kubernetes/OCP stuff that makes life more easy going:
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
    - ingresscert     ->  Install an existing wildcard certificate
    - approvecsrs     ->  Approve any outstanding CSRs (Certificate Signing Requests)
    - yakkotest       ->  Deploy the 'yakkotest' app on your cluster, to test the lot!!  
```

Oh, and one final little back door, when you are recreating the same cluster often, you can run, no questions asked: 

`     yakko rebuildcluster`

## WHAT IS YAKKO MISSING?  (Backlog of sorts?)
Short of this being a backlog...
- BYO network (i.e. don't depend on a virtual network) - this should be mostly easy, but I am yet to build a business case
- Setting up Chrony as an "ops" function (though this seems to be covered in 4.7+)
- Adding nodes from other physical machines and moving virtual nodes around (which may well defeat YAKKO's own purpose)
- Many 'certainty' principles of higher level systems administration, this said, it tries to keep your firewall on, your SELinux running etc etc.

## COMMITMENT and ACKNOWLEDGEMENTS
- I hereby pledge to test and update as new releases of OpenShift, RHEL and FEDORA come out... Until I don't, and then I will delete this section :)
- I was inspired in automating this after reading https://github.com/eitchugo/openshift-libvirt. Thanks Hugo! 
It was "short" and after typing in all the looooong host kernel parameters I decided that this was worth investing time into. 
But there are a ton of cookbooks out there, they are all different. I didn't want to write another cookbook, I thought it would be more fun to write a bot-chef to cook for me. 
- I needed a COVID confinement project. This is it!

## QUESTIONS YOU MAY HAVE, FOR FUN
- Why didn't you use Ansible? 
I could, I chose not to, because I would have had to learn another TON of stuff. I actually pulled out a couple of lines where I did use it. I may some other day. I wanted this to be ONE script with no additional downloads for code, no dependencies of other scripts.
- What if I have two boxes and I want to spread the load?
I want to cook that too. I have two boxes, Large and medium. My dream is to turn the medium box into a CNV node - Well, I'm thinking about using a non-virtual network... But if I can use remote nodes on different networks, we'll see.
- What are the minimum requirements?
See above. I've happily succeeded with a 4 core/8 thread server from 2009, a Sun Ultra 27! It may well be the only Sun box in the Universe running OpenShift :)  
- Is it AUTOMATIC?
Yes, after you master the basics. Who doesn't want to rebuild OCP all the time?
- Lots of stuff I haven't thought off, so leave your comments!

## MY OWN EXPERIENCE AND FINAL WORDS
- YAKKO is a big undertaking. It began as an exercise in curiosity and developed into a fledgling almost-product
- As I created clusters and then tried things I didn't know how to do, I quicky appreaciated having the ease of rebuilding from scratch. I can see my customers taking the same approach!
- Kubernetes, with all the ultra-modern design principles of distributed computing and software delivery it promotes, is replacing the face of everything we took for granted in computing: hardware becomes even more commodity (anything goes, including Raspebrry Pies), virtualisation is turned on its head to become a second (or even third) class citizen (CNV), dependencies are a thing of the past, resilience is a given, and immutability and statelessness are here to stay for the good of mankind. Kubernetes is the new Operating System (of the datatacentre) and Containers are Linux. Time to learn again
- There will always be people, like me, who hold "self-hosted" as a guiding principle to learn from and... Enjoy!



