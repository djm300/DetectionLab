# Detection Lab Libvirt build

## Intro

This page contains the instruction to build DetectionLab for Qemu/LibVirt. This is the provider for you *if*:
* You are familiar with LibVirt, virt-manager and Qemu and prefer this software stack instead of VirtualBox
* You are willing to spend a bit more time thinkering with the build process as it is less hands-off than the official DetectionLab

## Prequisite
### LibVirt
Depending on your Linux distribution, the LibVirt packages needed may vary. For example, on Ubuntu/Debian, the `libvirt-dev` package is needed. A working setup on ArchLinux have the following LibVirt packages:
* libvirt
* libvirt-glib
* libvirt-python
* libvirt-python2  

A VNC and/or SPICE server is also recommended to view and interact with the VM. This is usually bundled with the distribution's `virt-manager` package.  

*Virt-Manager* is the recommended way to interact with VMs. It can be installed on the host, as well as remotely on Linux and Mac, and probably on Windows too. 

The `libvirt` and `virt-manager` installation walkthrough and documentation is out of scope of this project. To follow along, you need an already working installation of `libvirt`, `virt-manager`, and `QEMU+kvm`. 

### Vagrant
1. Install the necessary plugins:
* ` env TMPDIR=/media/storage_ssd/vm_hdd/detectionlab/ PACKER_LOG=1 PACKER_LOG_PATH="packer_build.log" packer build --only=qemu --on-error=abort windows_2016.json`

#### Notes: 
The libvirt builder is highly experimental. This sections describes the tradeoffs and the differences between the vanilla DetectionLab.

1. The boxes will have two network adapters
The vagrant-libvirt provider works by binding to a "management" network adapter IP addresses. The way vagrant finds the VM's IP address is by probing the dnsmasq lease file of libvirt's host. There's probably a better way, but this is the best I could do that just works (tm) so far. Here's what the configuration looks like:

    * Management Network: Isolated network, no NAT, no internet access, with DHCP.
    * Detectionlab Network: 192.168.38.0/24, with NAT, with internet access, with DHCP.

2. Libvirt needs additional options for VM storage
The qcow2 VM disks needs to be stored in LibVirt "storage pools". You can edit the `Vagrantfile` to match an existing storage pool configured in libvirt. My limited testing shows that both `libvirt.storage_pool_path` and `libvirt.storage_pool_name` needs to match an existing storage pool.

3. The synced folder is using an old, slow and buggy plugin. While this barely works, it's enough to push the provisioning scripts to the Windows instances. Any modifications to the `vm.synced_folder` in the libvirt provider will likely break the provisionning process

4. The graphical and input settings assume the use of virt-manager with the SPICE viewer on Windows and the VNC viewer on Linux (logger). The spice agent for copy/pasting and other quality of life improvement, like auto-resolution changes is *NOT* installed on the Windows hosts. 

### Packer

1.  The [Virtio drivers](https://docs.fedoraproject.org/en-US/quick-docs/creating-windows-virtual-machines-using-virtio-drivers/) ISO needs to be location in the `DetectionLab/Packer/` directory.   

    * This is a direct [link to the latest version of the virtio drivers ISO](https://fedorapeople.org/groups/virt/virtio-win/direct-downloads/latest-virtio/virtio-win.iso).   
    * There's also a "stable" version available [here](https://fedorapeople.org/groups/virt/virtio-win/direct-downloads/stable-virtio/virtio-win.iso).  

2. Edit the windows_X.json files
    * Make sure the following user-defined variables are pointing to the right thing:
         * `virtio_win_iso` : See [LibVirt](#LibVirt)
         * `packer_build_dir` : Where to output the QCOW2 images. It is different from where the `vagrant` post-processor will output it's .box file

3. Build the images
```env TMPDIR=/path/to/large/storage/ PACKER_LOG=1 PACKER_LOG_PATH="packer_build.log" packer build --only=qemu windows_2016.json    
env TMPDIR=/path/to/large/storage/ PACKER_LOG=1 PACKER_LOG_PATH="packer_build.log" packer build --only=qemu windows_10.json
```

### Vagrant

1. Edit the Vagrantfile with the appropriate paths and names for `libvirt.storage_pool_path` and `libvirt.storage_pool_name` (see [Vagrant Section of Prequisite](#Prequisite)
2. Vagrant up one box at a time or with `--no-parallel` 
* vagrant-libvirt supports parallel provisioning. While it's fun and all, there's really no way to support parallel provisioning without doing some dark Windows magic. Be patient, this can easily take an hour.
