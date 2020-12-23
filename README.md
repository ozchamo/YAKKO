# yakko

(Y)et (A)nother (K)VM (K)onfigurator for (O)penShift  (tested with OCP 4.6!)
<BR>
INTRODUCTION:
If you want to run up an OpenShift cluster, and have a big PC/small server (mine is a i7-7700 w/64GB RAM) then this might be for you. There are plenty of cookbooks out there and they require that you do a lot of manual work. This avoids it!

Why would you want to run your own (single-box) cluster, isn't that self defeating because there is no real resilience?
- A full cluster at your disposal lets you test full cluster functionality
- You can experiment with multiple node setups in an easy self-hosted lab fashion
- You can create cheap clusters for experimenting with Red Hat Advanced Cluster Manager (RHACM)
- You can easily setup different versions of OpenShift and examine features and compare behaviour 
- No public cloud bill... or bill shock! 
- You might be a a fan of "Self hosting"
- OCP Pull Secret from Red Hat have a lifetime of 60 days, so re-installing is kinda useful
- LEARN LEARN LEARN!

In a nutshell, what does yakko do? 
- it installs OCP (latest if you want) automatically (if you want)
- by default, 3 masters + 2 workers (configurable - 3 masters only makes all nodes "schedulable")
- leverage the host as the bastion host, right?
- using a KVM network behind a NAT - 192.168.140.x 
- leverage the host as load balancer with HAproxy
- roll back individual failed stages so that you can fix if necessary and then keep going or just "yakko delete" what you've done so far and start afresh. It's scary how quickly it does away with a happily running cluster, so be careful...
- delete the entire cluster you've built, and unconfigure all the above (by running yakko delete <cluster-name>)
- some basic operational stuff - once you have the cluster up, it will hint you, using "yakko ops <command>"
  (add htpasswd, enable internal registry, open access to your network...)
- but don't worry - you can build again right? Automatically...
- What doesn't it do? I dunno yet. Tons of stuff I presume, but who doesn't want to have OCP in their study?

<BR>
REQUIREMENTS:
- a box with enough RAM such that you can build:
    - 3 node clusters in a server with 32GB RAM (I've succeeded on a box with 24GB but it's old and the CPU gets in the way :)
    - 5 node clusters on 48GB/64GB (3 masters + 2 workers) with plenty RAM to spare
- Project cockpit is a good (though hungry) friend
- access to the ol' internet
- Linux skills - if you are even attempting at using this, you must have some already!

<BR>
TESTED COMBINATIONS:
- RHEL 8.2, 8.3
- Fedora 32, 33
- OpenShift 4.3, 4.5, 4.6 
(Don't ask for support, but you can ask for help anytime!)

HOW TO - INSTALL or "DAY 1":<BR>

"yakko" will start the install process when there is no cluster defined, so no further parameters are necessary.
1) Get the bits as user "root":
    - You can clone the repo (ideally on /) OR  
    - download "yakko" and run it (https://github.com/ozchamo/YAKKO/raw/master/yakko) 
2) run "yakko" (again, as root) - the script will copy itself to /YAKKO (if it's not already there based on what you did in step (1)
3) follow instructions, my suggestion is that you run it manually until you get the hang of it
4) once you get the flow, it can build the cluster AUTOMATICALLY. I've built many in one week :)
5) depending on your hardware (mine's OK, not overfully powerful) you can have a cluster up and running in 30-50 minutes
6) Until there is no operational cluster, "yakko" will keep asking you to continue the install from where you left off
7) Once a cluster is operational, yakko on its reports something like this, anytime you run it without parameters:

     _______________________________________________________________________________________

      YAKKO: Yet Another KVM Konfigurator for Openshift
     _______________________________________________________________________________________

     Reading CLUSTER configuration file(s)...

     CLUSTER: prod

     The console and API server appear to be operational:

     Web console:  https://console-openshift-console.apps.prod.localdomain
     API server:   https://api.prod.localdomain:6443

     Cluster Administrator: kubeadmin
     Password:   qfeUr-vQNVH-gkAaz-tq4BS

     To use the OpenShift 'oc' command from hereon, run "source ocp-setup-env" in this shell.
     You can also call yakko with the following parameters: [ startup / shutdown / connect / delete / ops / backup ]

<BR>
HOW TO - OPS or "DAY 2" "ops":<BR>
Once you have created a cluster, "yakko" is not intended to do much more, after all the idea is to learn and experiment. However, I've automated a few "procedures" that are useful on a day to day basis:
- yakko connect <node>: let's you jump into a command line of a node (SSH login was automated on install)
- yakko shutdown [node]: let's you shutdown a node or THE ENTIRE CLUSTER
- yakko startup [node]: let's you startup a node or THE ENTIRE CLUSTER
- yakko delete <cluster-name>: let's you delete the lot. Yes, the lot, like it never happened. USE WITH CARE. It's the bomb, literally.
- yakko ops <OPTION> [parameters]:
    Options are:
    - htpasswd admin-username (deploy local password access and a new administrator)
    - useradd username (add a new user to local password DB)
    - userdelete username (delete an existing user from the local password DB)
    - localregistry (enable a local registry so you can actually use the cluster...)
    - openaccess (enable the cluster to be accessed/used by other machines in your network (via changing HA proxy)

<BR>
ACKNOWLEDGEMENTS: 
- I was inspired in automating this after reading https://github.com/eitchugo/openshift-libvirt. Thanks Hugo! 
It was "short" and after typing in all the looooong host kernel parameters I decided that this was worth investing time into. Thanks ;)
But there are a ton of cookbooks out there, they are all different. I didn't want to write another cookbook, I thought it would be more fun to write a bot-chef to cook for me. This is it.
- I needed a COVID confinement project. This was it!

<BR>
COMMITMENT:
I hereby plegde to test and update as new releases of OpenShift, RHEL and FEDORA come out... Until I don't, and then I will delete this section :)

<BR>
QUESTIONS YOU MAY HAVE, FOR FUN:
- Hey Daniel, why didn't you use Ansible? 
I could, I chose not to, because I would have to learn another TON of stuff. I actually pulled out a couple of lines where I did use it. I may some other day. I wanted this to be ONE script with no additional downloads for code, no dependencies of other scripts.
- Hey Daniel, what if I have two boxes and I want to spread the load?
I want to cook that too. I have two boxes, Large and medium. My dream is to turn the medium box into a CNV node - Well, I'm thinking about using a non-virtual network... But if I can use remote nodes on different networks, we'll see.
- Hey Daniel, what are the minimum requirements?
See above. I've happily succeeded with a 4 core/8 thread server from 2009, a Sun Ultra 27! It may well be the only Sun box in the Universe running OpenShift :) though on this box 
- Hey Daniel, is it AUTOMATIC?
Yes, after you master the basics. Who doesn't want to rebuild OCP all the time?

