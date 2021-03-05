# yakko  ::  (Y)et (A)nother (K)VM (K)onfigurator for (O)penShift 


## INTRODUCTION
If you want to run up an OpenShift cluster, and have a big PC/small server (mine is a i7-7700 w/64GB RAM) then this might be for you. There are plenty of cookbooks out there and they require that you do a lot of *manual* work. **YAKKO avoids it!**

YAKKO was built around the concept of having ONE script/installer/manager that does it all, and no other dependencies. That one script is YAKKO, it's a bit opinionated, but then again, it's not built for producing "production ready" clusters, so that should suit most people with a passing need or interest in having an OpenShift cluster around.

Why would you want to run your own (single-box) cluster, isn't that self defeating because there is no real resilience?
- A full cluster at your disposal lets you test full cluster functionality
- You can experiment with multiple node setups in an easy self-hosted lab fashion
- You can create cheap clusters for experimenting with Red Hat Advanced Cluster Manager (RHACM)
- You can easily setup different versions of OpenShift and examine features and compare behaviour 
- No public cloud bill... or bill shock! 
- You can test your more complex apps on multiple worker nodes 'for real' 
- You might be a a fan of "Self hosting"
- OCP Pull Secret from Red Hat have a lifetime of 60 days, so re-installing is kinda useful
- LEARN LEARN LEARN!

In a nutshell, what does yakko do? 
- sets up and installs any and all requirements/dependencies for you
- installs OCP (latest if you want) automatically (if you want)
- leverages the host as the bastion host, right?
- eases the networking by using a KVM network behind NAT 
- leverages the host as load balancer with HAproxy
- rolls back individual failed stages so that you can fix if necessary and then keep going or just "yakko delete" what you've done so far and start afresh. It's scary how quickly it does away with a happily running cluster, so be careful...
- delete the entire cluster you've built, and unconfigure all the above (by running yakko delete <cluster-name>)
- it allows you to easily add and destroy worker and infra nodes to suit your use case 
- some basic operational stuff - once you have the cluster up, it will hint you, using "yakko [infra | ops] <command>"
- but don't worry - you can build again right? Automatically...
- What doesn't it do? I dunno yet. Tons of stuff I presume, but who doesn't want to have OCP in their study?

## WHAT YAKKO IS NOT
It is not a management tool for OpenShift. It has a small overlay of features to assist in the "automation" of getting things done that may otherwise be repetitive, but once your cluster is up, you can delete YAKKO for all you know, but since it can do a few things post install (see "Day 2 Ops)" you should keep it!

## REQUIREMENTS
A PC/server running RHEL8 or Fedora 32/33 with enough RAM such that you can build:
- 3 node clusters in a server with 32GB+ RAM (I've succeeded on a box with 24GB but it's old and the CPU gets in the way :)
- Many node clusters on 48GB/64GB (3 masters + many workers) with plenty RAM to spare
Tested combinations to date:
- RHEL 8.2, 8.3
- Fedora 32, 33
- OpenShift 4.3, 4.5, 4.6, 4.7
    
## NICE TO HAVES
- Project cockpit is a good (though hungry) friend
- access to the ol' internet
- Linux skills - if you are even attempting at using this, you must have some already!
- Your own DNS server that can handle wildcards (but YAKKO can otherwise assist)

## HOW TO - INSTALL or "DAY 1"
### [[Watch (an earlier vesion of) YAKKO in action building OpenShift]](https://youtu.be/hLsUp7dwxdQ)
1) Get the 'yakko' script as user "root": 
    - You can clone the repo (ideally on /) OR  
    - download it from https://github.com/ozchamo/YAKKO/raw/master/yakko  
2) run "yakko" as root - e.g. `[root@ocphost YAKKO]# ~/Downloads/yakko`
3) the script will copy itself to /YAKKO - if it's not already there based on what you did in step (1) - and ask you to re-run from there 
4) 'yakko' will start the install process when there is no cluster defined, so no further parameters are necessary.
5) follow instructions, my suggestion is that you run it manually until you get the hang of it.
6) once you get the flow, it can build the cluster AUTOMATICALLY. I've built many in one week :)
7) depending on your hardware (mine's OK, not overfully powerful) you can have a cluster up and running in 30-50 minutes
8) Until there is no operational cluster, "yakko" will keep asking you to continue the install from where you left off
9) Once a cluster is operational, YAKKO reports something like this, anytime you run it without parameters:

```
_______________________________________________________________________________________
YAKKO: Yet Another KVM Konfigurator for Openshift
_______________________________________________________________________________________

CLUSTER: prod.localdomain

Active Masters:   3
Active Nodes:     2 (workers/infra)
Active Operators: 31/31

The console and API server appear to be operational:

Web console:   https://console-openshift-console.apps.prod.localdomain
API server:    https://api.prod.localdomain:6443

Administrator: kubeadmin
Password:      ZefST-hvBBY-fR39z-73ghN

- To use OpenShift's 'oc' command run: "source ocp-setup-env" in this shell.
- To make infrastructure changes use:  "yakko infra <options>"  
- To make operational changes use:     "yakko ops <options>" 
_______________________________________________________________________________________

```

## HOW TO - OPS or "DAY 2" "ops"
Once you have created a cluster, "yakko" is not intended to do much more, after all the idea is to learn and experiment. However, I've automated a few "procedures" that are useful on a day to day basis.

These are called through passing a parameter to YAKKO, either "infra" or "ops".

"yakko infra <parameter>" relates to the mundane tasks related to the infrastructure of the cluster:
   
`[root@ocphost YAKKO]# ./yakko ops`
```
USAGE: yakko infra <OPTION> [parameters]  
OPTION is one of:  
    - startcluster  -> start up an existing cluster  
    - stopcluster   -> shutdown an existing cluster  
    - addnode       -> grow the cluster compute capacity by adding a new compute/infra node  
    - deletenode    -> remove a running node from the cluster  
    - nodelogs      -> display the logs of a particular node  
    - sshtonode     -> provide terminal access to an individual cluster node  
    - openaccess    -> enable OpenShift access by other clients in your network  
    - deletecluster -> delete entire cluster and all infrastructure  
```
"yakko ops" relates to the higher level kubernetes/OCP stuff that makes life more easy going:
`[root@ocphost YAKKO]# ./yakko ops`

```
USAGE: yakko ops <OPTION> [parameters]  
OPTION is one of:  
    - htpasswd      -> deploy local password access and a new administrator  
    - useradd       -> add a new user to local password DB  
    - userdelete    -> delete an existing user from the local password DB  
    - mastersched   -> enable/disable master scheduling  
    - nodelabel     -> Change the label of a node between worker <-> infra  
    - localregistry -> enable a local registry so you can actually use the cluster  
    - yakkotest     -> deploy the 'yakkotest' app on your cluster, to test the lot!!  
```

## ACKNOWLEDGEMENTS
- I was inspired in automating this after reading https://github.com/eitchugo/openshift-libvirt. Thanks Hugo! 
It was "short" and after typing in all the looooong host kernel parameters I decided that this was worth investing time into. Thanks ;)
But there are a ton of cookbooks out there, they are all different. I didn't want to write another cookbook, I thought it would be more fun to write a bot-chef to cook for me. This is it!
- I needed a COVID confinement project. This is it!

## COMMITMENT
- I hereby plegde to test and update as new releases of OpenShift, RHEL and FEDORA come out... Until I don't, and then I will delete this section :)

## QUESTIONS YOU MAY HAVE, FOR FUN
- Why didn't you use Ansible? 
I could, I chose not to, because I would have had to learn another TON of stuff. I actually pulled out a couple of lines where I did use it. I may some other day. I wanted this to be ONE script with no additional downloads for code, no dependencies of other scripts.
- What if I have two boxes and I want to spread the load?
I want to cook that too. I have two boxes, Large and medium. My dream is to turn the medium box into a CNV node - Well, I'm thinking about using a non-virtual network... But if I can use remote nodes on different networks, we'll see.
- What are the minimum requirements?
See above. I've happily succeeded with a 4 core/8 thread server from 2009, a Sun Ultra 27! It may well be the only Sun box in the Universe running OpenShift :)  
- Is it AUTOMATIC?
Yes, after you master the basics. Who doesn't want to rebuild OCP all the time?

## FINAL NOTE: WHAT IS IT MISSING?  
Short of this being a backlog...
- BYO network (i.e. don't depend on a virtual network) - this should be mostly easy, but I am yet to build a business case
- Setting up Chrony 
- Adding nodes from other physical machines and moving virtual nodes around (may well defeat YAKKO's own purpose)
- Lots of stuff I haven't though off, so leave your comments!

