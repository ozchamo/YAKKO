# yakko

Yet another KVM/Konfigurator for OpenShift

Introduction:
If you want to run up OpenShift, and have a big PC/small server (mine is a i7-7700 w/64GB RAM) then this might be for you.

Requirements:
- a box with enough RAM 
    - 3 node clusters in a server with 24GB RAM with 2-4GB to spare 
    - 5 node clusters (3 masters + 2 workers) with plenty RAM to spare
- RHEL8 or FEDORA 31 (well, latest is what I have used) - project cockpit is a good (hungry) friend. 
- access to the ol' internet of course

How to use it?
- download "yakko" and run it
- the script will copy itself to /OCP-SETUP
- run it from there as root 
- follow instructions, my suggestion is that you run it manually until you get the hang of it
- once you get the flow, it can build the cluster AUTOMATICALLY. I've built many in one week :)

What can it do?
- it installs OCP (latest if you want) automatically
- by default, 3 masters + 2 workers
- using a KVM network behind a NAT - 192.168.140.x
- leverages the host as load balancer with HAproxy
- leverages the host as the bastion host, right?
- delete the entire cluster you've built, and unconfigure all the above
- but don't worry - you can build again right? AUTOMATICALLY

What can't it do?
- I dunno yet. Tons of stuff I presume, but who doesn't want to have OCP in their study?

Acknowledgements
I was inspired in automating this after reading https://github.com/eitchugo/openshift-libvirt. Thanks Hugo! 
It was "short" and after typing in all the looooong host kernel parameters I decided that this was worth investing time into. Thanks ;)
But there are a ton of cookbooks out there, they are all different. I didn't want to write another cookbook, I thought it would be more fun to write a bot-chef to cook for me. This is it.

Questions I like to answer, for fun

- Hey Daniel, why didn't you use Ansible? 
I could, I chose not to. I actually pulled out a couple of lines where I did use it. I may some other day.

- Hey Daniel, what if I have two boxes and I want to spread the load?
I want to cook that too. I have two boxes, Large and medium. My dream is to turn the medium box into a CNV node.

- Hey Daniel, what are the minimum requirements?
See above. I've happily succeeded with a 4 core/8 thread server from 2010, a Sun Ultra 27! It may well be the only Sun box in the Universe running OpenShift :)

- Hey Daniel, is it AUTOMATIC?
It seems to be after you master the basics. Who doesn't want to rebuild OCP all the time?

