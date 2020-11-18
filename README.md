# yakko

Yet another KVM/Konfigurator for OpenShift

Introduction:
- If you want to run up OpenShift, and have a big PC/small server (mine is a i7-7700 w/64GB RAM) then this might be for you
- It will use a Pull Secret from Red Hat which has a lifetime of 60 days, so re-installing is kinda useful...

Requirements:
- a box with enough RAM such that you can build:
    - 3 node clusters in a server with 32GB RAM (I've succeeded on a box with 24GB but it's old and the CPU gets in the way :)
    - 5 node clusters on 48GB/64GB (3 masters + 2 workers) with plenty RAM to spare
- RHEL8 or FEDORA 31 (well, latest is what I have used) - project cockpit is a good (hungry) friend
  (RHEL 8.3 and Fedora 33 have been tested)
- access to the ol' internet of course

How do I use this?
1) Get the bits as user "root":
    - You can clone the repo (ideally on /) OR  
    - download "yakko" and run it (https://github.com/ozchamo/YAKKO/raw/master/yakko) 
2) run "yakko" (again, as root) - the script will copy itself to /YAKKO (if it's not already there based on what you did in step (1)
3) follow instructions, my suggestion is that you run it manually until you get the hang of it
4) once you get the flow, it can build the cluster AUTOMATICALLY. I've built many in one week :)

What can it do?
- it installs OCP (latest if you want) automatically
- by default, 3 masters + 2 workers (configurable - 3 masters only makes all nodes "schedulable")
- using a KVM network behind a NAT - 192.168.140.x (configurable by running "yakko defaults")
- leverages the host as load balancer with HAproxy
- leverages the host as the bastion host, right?
- it will roll back individual failed stages so that you can fix if necessary and then keep going, or just "yakko delete" what you've done so far and start afresh. It's scary how quickly it does away with a happily running cluster
- delete the entire cluster you've built, and unconfigure all the above (by running yakko delete <cluster-name>)
- some basic operational stuff - once you have the cluster up, it will hint you 
  (add htpasswd, enable internal registry, open access to your network...)
- but don't worry - you can build again right? Automatically...

What can't it do?
- I dunno yet. Tons of stuff I presume, but who doesn't want to have OCP in their study?
- Well, I'm working on using a non-virtual network...

Acknowledgements:
- I was inspired in automating this after reading https://github.com/eitchugo/openshift-libvirt. Thanks Hugo! 
It was "short" and after typing in all the looooong host kernel parameters I decided that this was worth investing time into. Thanks ;)
But there are a ton of cookbooks out there, they are all different. I didn't want to write another cookbook, I thought it would be more fun to write a bot-chef to cook for me. This is it.
- I needed a COVID confinement project. This was it!

Questions I like to answer, for fun

- Hey Daniel, why didn't you use Ansible? 
I could, I chose not to, because I would have to learn another TON of stuff. I actually pulled out a couple of lines where I did use it. I may some other day.

- Hey Daniel, what if I have two boxes and I want to spread the load?
I want to cook that too. I have two boxes, Large and medium. My dream is to turn the medium box into a CNV node.

- Hey Daniel, what are the minimum requirements?
See above. I've happily succeeded with a 4 core/8 thread server from 2009, a Sun Ultra 27! It may well be the only Sun box in the Universe running OpenShift :)

- Hey Daniel, is it AUTOMATIC?
It seems to be after you master the basics. Who doesn't want to rebuild OCP all the time?

