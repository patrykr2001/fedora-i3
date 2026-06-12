# Check your system capabilities

```bash
# Is IOMMU enabled?
sudo dmesg | grep -i iommu | head -20

# IOMMU groups - needed for passthrough
sudo find /sys/kernel/iommu_groups/ -type l | sort -V | head -40

# Hardware ID of GPU (needed for vfio)
rlspci -nn | grep -E "VGA|Display|3D|Audio" 

# Does the CPU support virtualization?
grep -E "vmx|svm" /proc/cpuinfo | head -5

# Active kernel's cmdline
cat /proc/cmdline
```

# Enable IOMMU in kernel

```bash
sudo nano /etc/default/grub
```

Find `GRUB_CMDLINE_LINUX` and add `amd_iommu=on iommu=pt`:
```bash
GRUB_CMDLINE_LINUX="... amd_iommu=on iommu=pt rd.driver.blacklist=nouveau,nova_core modprobe.blacklist=nouveau,nova_core"
```

Update grub:
```bash
sudo grub2-mkconfig -o /boot/grub2/grub.cfg
```

# Reserve GPU for VFIO before NVIDIA driver loads

You need ID of your GPU and audio device - you checked that in the steps above.
```bash
sudo nano /etc/modprobe.d/vfio.conf
```

Write:
```bash
options vfio-pci ids={your-gpu-id},{you-gpu-audio-id}
softdep nvidia pre: vfio-pci
```

# Load VFIO modules on start

```bash
sudo nano /etc/modules-load.d/vfio.conf
```

Write:
```bash
vfio
vfio_iommu_type1
vfio-pci
```

# Rebuild initramfs and reboot

```bash
sudo dracut --force
sudo reboot
```

After reboot check if VFIO got the GPU:
```bash
lspci -nnk | grep -A3 "01:00"
```

# Install QEMU/KVM and virt-manager

```bash
sudo dnf install @virtualization virt-manager
sudo systemctl enable --now libvirtd
sudo usermod -aG libvirt $USER
```

Logout and login.

# Create VM with Windows 11

```bash
virt-manager
```

When creating:
 1. **Firmware:** UEFI (OVMF) - required for Win11 and GPU passthrough
 1. **CPU:** minimum 4 cores, enable `host-passthrough`
 1. **RAM:** minimum 8GB
 1. **Drive:** minimum 60GB

Download Win11 ISO from Microsoft and VirtIO drivers:
```bash
wget https://fedorapeople.org/groups/virt/virtio-win/direct-downloads/stable-virtio/virtio-win.iso -P ~/Downloads/
```

# Add GPU to the VM

In virt-manager after creating VM, but before first start:
 1. Open VM settings
 1. **Add Hardware -> PCI Host Device**
 1. Add you GPU and GPU audio devices, e.g. `0000:01:00.0` and `0000:01:00.1`

Add Win11 ISO as CD-ROM device and do the same for VirtIO drivers.

# Managing VM from the cmdline

```bash
# List vm's
virsh --connect qemu:///system list --all

# Start vm
virsh --connect qemu:///system start win11

# Shutdown vm
virsh --connect qemu:///system shutdown win11

# Force close vm (destroy)
virsh --connect qemu:///system destroy win11
```

# After win11 instalation

Remove virtual display:
```bash
virsh --connect qemu:///system shutdown win11
virsh --connect:///system edit win11
```

Then find and remove:
```bash
<channel type='spicevmc'>
  <target type='virtio' name='com.redhat.spice.0'/>
  <address type='virtio-serial' controller='0' bus='0' port='1'/>
</channel>

<graphics type='spice' autoport='yes'>
  <listen type='address'/>
  <image compression='off'/>
</graphics>

<audio id='1' type='spice'/>

<video>
  <model type='qxl' ram='65536' vram='65536' vgamem='16384' heads='1' primary='yes'/>
  <address type='pci' domain='0x0000' bus='0x00' slot='0x01' function='0x0'/>
</video>

<redirdev bus='usb' type='spicevmc'>
  <address type='usb' bus='0' port='2'/>
</redirdev>
<redirdev bus='usb' type='spicevmc'>
  <address type='usb' bus='0' port='3'/>
</redirdev>
```

And add:
```bash
<video>
  <model type='none'/>
</video>
```

Then you can add devices that you want to be captured by the vm. E.g.
```bash
<input type='evdev'>
      <source dev='/dev/input/event5' grab='all' grabToggle='alt-alt' repeat='on'/>
    </input>
    <input type='evdev'>
      <source dev='/dev/input/event261'/>
    </input>
    <input type='evdev'>
      <source dev='/dev/input/event269'/>
    </input>
    <input type='evdev'>
      <source dev='/dev/input/event23'/>
    </input>
    <input type='evdev'>
      <source dev='/dev/input/event25'/>
    </input>
```

Disable autostart if you want:
```bash
sudo virsh --connect qemu:///system autostart --disable win11
```

If you want you can adjust CPU setup.
Find or add static vcpu placement. Adjust for you desired threads.
```bash
<vcpu placement='static'>12</vcpu>
```

And change what guest OS can see as CPU, by default it will be propably 2 CPU's and 2 cores.
```bash
<cpu mode='host-passthrough' check='none' migratable='on'>
  <topology sockets='1' dies='1' cores='6' threads='2'/>
</cpu>
```

You can also pass entire USB device instead of sharing it with host.
Find you device:
```bash
lsusb
```

Note vendor ID and Product ID, e.g. for `Bus 001 Device 023: ID 1532:0257 Razer USA, Ltd Razer Huntsman Mini` it is `1532` and `0257`.
Add before `</devices>`:
```bash
<hostdev mode='subsystem' type='usb' managed='yes'>
      <source>
        <vendor id='0x1532'/>
        <product id='0x0257'/>
      </source>
    </hostdev>
```

# Different kernel arguments for normal usage and for VFIO

If you want to be able to boot into normal system with NVIDIA GPU or to VFIO setup you can add two separate kernel entries with different arguments in grub.

Check your kernel entries:
```bash
sudo ls /boot/loader/entries/
```

Then based on it create new entries, e.g. entry is:
```bash
sudo cat /boot/loader/entries/9435cc60df664a45a48b2914bff7b215-7.0.11-200.fc44.x86_64.conf
title Fedora Linux (7.0.11-200.fc44.x86_64) 44 (i3)
version 7.0.11-200.fc44.x86_64
linux /vmlinuz-7.0.11-200.fc44.x86_64
initrd /initramfs-7.0.11-200.fc44.x86_64.img
options root=UUID=efa0aea7-e3c2-45df-955f-328dd24e4aad ro rootflags=subvol=root rhgb quiet amd_iommu=on iommu=pt rd.driver.blacklist=nouveau,nova_core modprobe.blacklist=nouveau,nova_core
grub_users $grub_users
grub_arg --unrestricted
grub_class fedora
```

Copy it to separate entry. This one will be for VFIO.
```bash
sudo tee /boot/loader/entries/9435cc60df664a45a48b2914bff7b215-7.0.11-200.fc44.x86_64-vfio.conf << 'EOF'
title Fedora Linux — VFIO (VM with GPU)
version 7.0.11-200.fc44.x86_64
linux /vmlinuz-7.0.11-200.fc44.x86_64
initrd /initramfs-7.0.11-200.fc44.x86_64.img
options root=UUID=efa0aea7-e3c2-45df-955f-328dd24e4aad ro rootflags=subvol=root rhgb quiet amd_iommu=on iommu=pt rd.driver.blacklist=nouveau,nova_core modprobe.blacklist=nouveau,nova_core vfio-pci.ids=10de:249d,10de:228b
grub_users $grub_users
grub_arg --unrestricted
grub_class fedora
EOF
```

And change the old one to be normal without VFIO:
```bash
sudo tee /boot/loader/entries/9435cc60df664a45a48b2914bff7b215-7.0.11-200.fc44.x86_64.conf << 'EOF'
title Fedora Linux — NVIDIA (without VM)
version 7.0.11-200.fc44.x86_64
linux /vmlinuz-7.0.11-200.fc44.x86_64
initrd /initramfs-7.0.11-200.fc44.x86_64.img
options root=UUID=efa0aea7-e3c2-45df-955f-328dd24e4aad ro rootflags=subvol=root rhgb quiet amd_iommu=on iommu=pt rd.driver.blacklist=nouveau,nova_core modprobe.blacklist=nouveau,nova_core
grub_users $grub_users
grub_arg --unrestricted
grub_class fedora
EOF
```

Updated grub config:
```bash
sudo grub2-mkconfig -o /boot/grub2/grub.cfg
```

And restart. After restart check GRUB and you should see two separate entries.

There is another step - we need to remove vfio.conf - because it always blocks NVIDIA driver from grabbing GPU.
```bash
# 1. Remove vfio.conf - we moved ids into kernel parameters
sudo rm /etc/modprobe.d/vfio.conf

# 2. Rebuild initramfs
sudo dracut --force

# 3. Update original entry - only NVIDIA without vfio
sudo nano /boot/loader/entries/
# 2. Rebuild initramfs
sudo dracut --force

# 3. Update original entry - only NVIDIA without vfio
sudo nano /boot/loader/entries/9435cc60df664a45a48b2914bff7b215-7.0.11-200.fc44.x86_64.conf
```

And change `options` line to:
```bash
options root=UUID=efa0aea7-e3c2-45df-955f-328dd24e4aad ro rootflags=subvol=root rhgb quiet amd_iommu=on iommu=pt rd.driver.blacklist=nouveau,nova_core modprobe.blacklist=nouveau,nova_core
```

Then update VFIO kernel entry:
```bash
# 4. VFIO entry with ids in kernel
sudo tee /boot/loader/entries/9435cc60df664a45a48b2914bff7b215-7.0.11-200.fc44.x86_64-vfio.conf << 'EOF'
title Fedora Linux — VFIO (VM z GPU)
version 7.0.11-200.fc44.x86_64
linux /vmlinuz-7.0.11-200.fc44.x86_64
initrd /initramfs-7.0.11-200.fc44.x86_64.img
options root=UUID=efa0aea7-e3c2-45df-955f-328dd24e4aad ro rootflags=subvol=root rhgb quiet amd_iommu=on iommu=pt rd.driver.blacklist=nouveau,nova_core modprobe.blacklist=nouveau,nova_core vfio-pci.ids=10de:249d,10de:228b
grub_users $grub_users
grub_arg --unrestricted
grub_class fedora
EOF
```

Update grub:
```bash
# 5. Update grub
sudo grub2-mkconfig -o /boot/grub2/grub.cfg
```

# Automating creation of custom grub entries for VFIO after kernel updates

Edit `/usr/local/bin/setup-kernel` and write:
```bash
#!/bin/bash

KERNEL=$(uname -r)
MACHINE_ID=$(cat /etc/machine-id)
ENTRY_DIR="/boot/loader/entries"
BASE_OPTIONS="root=UUID=efa0aea7-e3c2-45df-955f-328dd24e4aad ro rootflags=subvol=root rhgb quiet amd_iommu=on iommu=pt rd.driver.blacklist=nouveau,nova_core modprobe.blacklist=nouveau,nova_core"
VFIO_IDS="vfio-pci.ids=10de:249d,10de:228b"

echo "=== Setup for kernel: $KERNEL ==="

# Step 1: Check if NVIDIA modules are build
echo "[1/4] Checking NVIDIA modules..."
if ! ls /usr/lib/modules/$KERNEL/extra/nvidia/nvidia.ko.xz &>/dev/null; then
    echo "No NVIDIA modules found —  building..."
    sudo akmods --force --rebuild
    sudo dracut --force
else
    echo "OK —  NVIDIA modules are present"
fi

# Step 2: Check signing
echo "[2/4] Checking if modules are signed..."
sudo modprobe nvidia 2>/dev/null && echo "OK —  modules load corectly" || echo "WARNING —  issue with signing, check MOK"

# Step 3: Create kernels VFIO entry
echo "[3/4] Creating VFIO entry in bootloader..."
VFIO_ENTRY="$ENTRY_DIR/${MACHINE_ID}-${KERNEL}-vfio.conf"

if [ -f "$VFIO_ENTRY" ]; then
    echo "OK —  VFIO entry already present"
else
    cat > "$VFIO_ENTRY" << EOF
title Fedora Linux —  VFIO (VM with GPU) ($KERNEL)
version $KERNEL
linux /vmlinuz-$KERNEL
initrd /initramfs-$KERNEL.img
options $BASE_OPTIONS $VFIO_IDS
grub_users \$grub_users
grub_arg --unrestricted
grub_class fedora
EOF
    echo "OK —  created $VFIO_ENTRY"
fi

# Step 4: Update GRUB
echo "[4/4] Updating GRUB..."
sudo grub2-mkconfig -o /boot/grub2/grub.cfg

echo ""
echo "=== Done! ==="
echo "Bootloader entries:"
ls $ENTRY_DIR/ | grep $KERNEL
```

Then:
```bash
sudo chmod +x /usr/local/bin/setup-kernel
```

And create hook that will launch the script automatically after kernel update. Edit `/etc/kernel/postinst.d/99-setup-nvidia-vfio`:
```bash
#!/bin/bash
/usr/local/bin/setup-kernel
```

Then:
```bash
sudo chmod +x /etc/kernel/postinst.d/99-setup-nvidia-vfio
```

Manual usage after kernel update:
```bash
setup-kernel
```
