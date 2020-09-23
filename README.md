# yakko

Yet another KVM/Konfigurator for OpenShift

SHORT INTRODUCTION

If you want to run up OpenShift, and have a big PC/small server (mine is a i7-7700 w/64GB RAM) then this might be for you.

Requirements:
- a box with 32GB+ RAM (I haven't tried it with 32GB, but 24GB could not handle it) 
- RHEL8 or FEDORA 31 (well, latest is what I have used) - project cockpit is a good (hungry) friend. 
- access to the ol' internet of course

How to use it?
- download "setup-openshift-on-kvm-host" and run it
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
It was "short" and after typing in all the host kernel parameters I decided that this was worth investing time into. Thanks ;)
But there are a ton of cookbooks out there, they are all different. I didn't want to write another cookbook, I thought it would be more fun to write a bot-chef to cook for me. This is it.

Questions I like to answer, for fun
- Hey Ozchamo, why didn't you use Ansible? 
I could, I chose not to. I actually pulled out a couple of lines where I did use it. I may some other day.
- Hey Ozchamo, what if I have two boxes and I want to spread the load?
I want to cook that too. I have two boxes, Large and medium. My dream is to turn the medium box into a CNV node.
- Hey Ozchamo, what are the minimum requirements?
I'm guessing you need 48GB to be somewhat comfy for a full blown 3+2 node cluster. I have installed a 1+2 on 24GB easily but that was with OCP 4.3. 4.5 doesn't like 2+1 out of the box, further experimentation required.
- Hey Ozchamo, is it AUTOMATIC?
It seems to be after you master the basics. Who doesn't want to rebuild OCP all the time?

