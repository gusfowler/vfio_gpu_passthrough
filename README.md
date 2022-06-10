# VFIO_GPU_PASSTHROUGH

These are a series of configuration files and bash scripts that allow me to quickly switch between a single-GPU passthrough configuration, and a typical setup with some VFIO stuff to be left on.

# I AM NOT LIABLE FOR YOU BREAKING YOUR STUFF
This is pretty complicated and you can make your setup unusable if you do not know what you are doing. Be careful. This readme is not descriptive enough for you replicate a single-GPU passthrough setup on it's own, but at the bottom there will be links that are more descriptive.

# Context
Amongst other things, I run a Windows KVM over Proxmox Virtual Environment, configured to play videogames, with unnoticable lag. I also use another VM to run OpenWRT with some physical NICs passed through, some virtual-ones, all to have a relatively complicated home network. Problem is, if I mess something up in OpenWRT (which I do often), while I have my only GPU configured to pass into a virtual machine, I will have lost all physical access to my host machine. So, I created a relatively easy way to switch between GPU passed through and GPU not passed through.

# So what is what?
Inside the folder scripts, there are the bash scripts that will quickly switch between my goal states. They are both relatively simple.
* These scripts assume the contents of 'grub-configs' are located in /root/
Inside the folder 'modprobe-configs' are configuration files that can be placed in /etc/modprobe.d/ that will facilitate the GPU passthrough.
* Inside vfio.conf, you will see a spot to insert the device ids of the PCI-E device being passed through. The config files will not work without this information.
Inside the folder 'grub-configs', there are template grub files. The key line is GRUB_CMDLINE_LINUX_DEFAULT - #9. The majority of this line is specific to my set-up, but I will explain what some of them mean.
* 'amd_iommu=on' - this enables IOMMU for AMD systems.
* 'video=efifb:off' - this is key to disabling the EFI framebuffer so that all of the memory in the GPU is accessible by VFIO, and none of it held by anything. You will notice the no-passthrough version of this file does not contain this option. I assume "disabling the framebuffer" sounds like you are disabling access to the host system via any GPU. (if you have more than one GPU, you can use one for the host and pass the other through! but thats not detailed here)
* 'hugepagesz=1GB' and 'hugepages=20' - There are two possible hugepage sizes, 2 megabytes of 1 gigayte, some motherboards only support 2 megabytes. In order to get enough hugepage memory you need to divide the targeted amount of memory by the size selected. Bigger is better if you can. 
* * Hupepage memory is important for VFIO / PCI-E passthrough to VM. It requires high speeds / low latency in the CPU's RAM. Typically, the paging system of the Linux process scheduler has pieces of memory assigned to processes all over the memory address range; these pieces are called 'pages'. Virtual Machines are just processes in the eyes of the kernel scheduler. Hugepages puts larger sized pages in place in the memory, and does not move them around. Hugepages can also be used for any process, not just Virtual Machines.
* the rest of the arguments are specific to my system.

# Things I can think of off the top of my head that you should research if you are going to implent this in your system
* The IOMMU groups of your motherboards chipset
* If your CPU supports PCI passthrough
* If you are on older hardware it might be from the era of Code 43 of NVIDIA cards - NVIDIA recently started allowing consumer-grade cards to be passed through to VMs without much fussiness, however you used to have to extract the vBIOS of the card you were using, which sometimes was a wierd process, particularly if that vBIOS is encrypted / obfusicated inside something else.

# Links
Read this whole damn thing - should help even if you are just on debian / ubuntu too
* https://pve.proxmox.com/wiki/Pci_passthrough

# My Specs
* Ryzen 9 5900X
* NVIDIA RTX 3070
* Gigabyte Aorus X570 Master - great for this since each pci device is in it's own IOMMU group, however some of the software for this is infuriating, but if you don't care about it's gimmicy features you'll be fine.
* too much ram
