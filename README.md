# Introduction

Ever looked at your computer's idle cores and wondered:

"Damn, I really wish I could use this spare processing power for something"

Or maybe you are just tired from dual booting your windows gaming system and your favorite Linux distro and thought to yourself:

"Gee, I see these Cloud services that are able to create virtualized machines according to hardware availability at will! How do they do it?! I wish I had something like that."

No? Well now you can wonder those exact things and may or may not use this guide (which documents the pain I went through) to get such a dream system up and running, where no longer would I have to shut down my computer to switch between operating systems. 

Originally, I thought about this when I started using remote access to train ML models, I found myself thinking about how cool it would be to relegate one of my GPUs for training, while using the other for gaming, because we all know how long some training tasks can be. However, if I were to simply run both tasks side by side, it would be hard to do all the balancing needed to ensure that one task doesn't hog up all of the resources (especially if you have a few tabs open too). 

Another reason was to simply have some more isolation going on in the case of an oopsy daisy, like swapping `+=` with `=+`. In a virtualized environment, if the guest (a running VM) crashes, that doesn't necessarily crash the whole system anymore, it's just another program. 

Using **libvirt** and **qemu**, alongside **KVM** and **VFIO** (a driver that allows for PCIe passthrough), a virtualization platform can be built that would eliminate the need for running OS's on bare-metal hardware. The exception being games that use kernel-level anti-cheat, which might soon be a thing of the past according to [this](https://www.notebookcheck.net/Microsoft-paves-the-way-for-Linux-gaming-success-with-plan-that-would-kill-kernel-level-anti-cheat.888345.0.html).

I'll start by listing the hardware specs I am working with, then move on to listing relevant sources that I used to get the basic set-up up and running. My main goal with this guide is to address the specific problems I faced in order to get the performance of my VMs to a point I was satisfied with. 

Specs:
* CPU - Ryzen 9 3900X 12 Cores 24 Threads
* MOBO - ASRock B550 Steel Legend
* RAM - Team TFORCE 32 GB (2 x 16 GB) DDR4 3200MHz
* Windows Guest Drive - Transcend 512 GB NVMe (don't remember exact model)
* Host OS - Patriot 128 GB NVMe (again, don't remember the exact model)
* Fedora Guest Drive - Team L3 EVO SSD 240GB
* GPU #1 - MSI RTX 4070 Ti SUPER
* GPU #2 - MSI GTX 1070 Ti

In my current configuration, I disabled hyperthreading (for a reason I will explain later) which leaves me with 12 cores, which needs to be split across 2 guests and the host machine. Currently, the hardware resources are split as follows:

* Windows Guest
	* 6 Cores, isolated and pinned
	* 12 GB RAM
	* GPU #1
	* 512 GB NVMe drive passthrough via VFIO
	* Keyboard, Mouse, and Sound Card passed through too
	* VirtIO NIC over Network Bridge
* Linux Guest
	* 4 Cores, not isolated nor pinned
	* 12 GB RAM
	* GPU #2
	* TEAM L3 SSD Block Device Path via VirtIO
	* Using synergy to share mouse and keyboard 
	* Audio not yet resolved
	* VirtIO NIC over Network Bridge
* Host
	* Whatever that's left (depends on workload since Linux guest cores are not isolated nor pinned)
	* OS - Manjaro Linux

Regarding the basic setup of virtualization using libvirt/qemu/KVM, there are many great resources that explain the setup procedure in great length and detail, and I will not try to copy what they have done. All props to the creators, and if I have something mentioned here that belongs to you and I haven't credited it, feel free to reach out. :)

# References

* https://github.com/mateussouzaweb/kvm-qemu-virtualization-guide
* https://wiki.archlinux.org/title/PCI_passthrough_via_OVMF
* https://www.heiko-sieger.info/running-windows-10-on-linux-using-kvm-with-vga-passthrough/
* https://github.com/bryansteiner/gpu-passthrough-tutorial
* https://docs.redhat.com/en/documentation/red_hat_enterprise_linux/7/html-single/virtualization_tuning_and_optimization_guide/index
