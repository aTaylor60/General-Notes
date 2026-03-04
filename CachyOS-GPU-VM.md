Solid guide to implement mostly:
https://gist.github.com/safwyls/96b6cf4b49e04af2668b7a77502e5ff2
Others referenced:
https://gist.github.com/r0kk0/5fc69b0685dda898845597e9b8259014
https://github.com/bryansteiner/gpu-passthrough-tutorial

**To allow internet traffic through cirtual network 'default': NAT**

UFW Rules to add
sudo ufw route allow in on virbr0
sudo ufw route allow out on virbr0
sudo ufw allow in on virbr0

Bridged network does not have the same firewall problems. Bridge is basically like an internal switch.
1. Delete existing basic wired connection
2. Create bridge and attach ethernet adapter to it (nmtui just worked, plasma nm not so much)
3. In NIC config of VM choose bridge and eneter the name of the established bridge

**Edit VM Drive size**
1. stop the VM
2. run qemu-img resize vmdisk.img +10G to increase image size by 10Gb
3. start the VM, resize the partitions and LVM structure within it normally. mini-tool.com free partition manager can reolocate the recovery partition

**Non-comprehensive list of files edited or added during setup**

/etc/libvirt/hooks/kvm.conf

    ## Virsh devices
    VIRSH_GPU_VIDEO=pci_0000_06_00_0
    VIRSH_GPU_AUDIO=pci_0000_06_00_1


home/andrew/.looking-glass-client.ini
OpenGL renderer seems much more stable than EGL (no crashes of client, stable output when not the window in focus)

    [app]
    renderer=OpenGL
    
    [win]
    title=looking-glass-client
    size=2560x1440
    keepAspect=yes
    borderless=no
    fullScreen=yes
    uiFont=pango:Iosevka
    uiSize=14
    maximize=yes
    showFPS=yes
    
    #[egl]
    #vsync=yes
    #multisample=yes
    #scale=0
    
    [wayland]
    warpSupport=yes
    fractionScale=yes
    
    [input]
    escapeKey=70
    autoCapture=yes



etc/udev/rules.d/99-kbmfr.rules

    SUBSYSTEM=="kvmfr", OWNER="libvirt-qemu", GROUP="kvm", MODE="0666"


etc/libvirt/qeum.conf
Uncomment to following block and add the last line "/dev/kvmfr0"

    cgroup_device_acl = [
        "/dev/null", "/dev/full", "/dev/zero",
        "/dev/random", "/dev/urandom",
        "/dev/ptmx", "/dev/userfaultfd",
        "/dev/kvmfr0"
    ]


etc/modules-load.d/kvmfr.conf

    # 3. KVMFR Looking Glass module
    kvmfr


etc/modprobe.d/kvmfr.conf

    #KVMFR Looking Glass module
    options kvmfr static_size_mb=64


etc/tmpfiles.d/10-looking-glass.conf

    # Type Path               Mode UID  GID Age Argument

    f /dev/shm/looking-glass 0660 andrew kvm -


/etc/libvirt/network.conf
Add the following at the bottom. This may be commented out at system update.

    firewall_backend = "nftables"
