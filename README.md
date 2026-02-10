# Complete Workstation Setup Guide
**University of Richmond - Academic Research Computing**

**Last Updated:** February 2026  
**Based on:** Rocky Linux 10.x / RHEL 10.x

---

## Table of Contents

1. [Building a Workstation from Hardware Components](#1-building-a-workstation-from-hardware-components)
2. [Preparing for OS Installation](#2-preparing-for-os-installation)
3. [Installing Rocky Linux](#3-installing-rocky-linux)
4. [Initial System Configuration](#4-initial-system-configuration)
5. [Authentication Setup](#5-authentication-setup)
6. [Software Installation](#6-software-installation)
7. [GPU and CUDA Setup](#7-gpu-and-cuda-setup)
8. [User Management](#8-user-management)
9. [Maintenance Tasks](#9-maintenance-tasks)
10. [Appendices](#appendices)

---

## 1. Building a Workstation from Hardware Components

*Skip this section if you're working with a pre-assembled machine.*

### 1.1 Before You Start

**Useful Resources:**
- Motherboard manual (specific to your model)
- Video tutorial: How to build a PC (search YouTube for recent guides)
- ESD-safe workspace

**Expected Time:** 2-4 hours for first build

### 1.2 Inventory and Components

**Case:**
- Remove protective films
- Remove black panel behind front door for easier cable access
- Remove 3rd and 4th PCIe slot covers from top (for GPU)
- Remove one HDD mounting block if you have only 2 drives
- Remove PCIe slot covers as needed

**Fans (3 included with case):**
- Install one fan on each side of the case
- **Important:** Fans should face OUTWARD to push air inside
- GPU fans will push air out, maintaining positive pressure

**Power Supply (PSU):**
1. Test power cord condition
2. Connect PSU to outlet (nothing should happen)
3. Test fan operation by switching PSU on/off
4. Connect cables:
   - CPU power (8-pin or 4+4-pin)
   - 24-pin motherboard power
   - SATA power cables
5. Mount PSU in bottom corner with 4 screws

### 1.3 Motherboard Installation

**Preparation:**
1. Place motherboard on box it came in
2. Remove all plastic protective covers
3. Install components in this order: CPU → RAM → CPU Cooler

**Motherboard Installation:**
1. Place case with glass side facing up
2. Open glass door and support it
3. Align motherboard with standoffs in case
4. Secure with 8-9 screws
5. Connect all cables to motherboard:
   - CPU power (8-pin, top-left usually)
   - 24-pin main power (right side)
   - SATA data cables
   - Case fans (check manual for fan header locations)
   - Front panel connectors (see below)

### 1.4 Memory (RAM) Installation

1. Push RAM slot protectors/latches outward to open
2. Check motherboard manual for correct slot configuration
   - Usually use slots 2 and 4 for dual-channel (A2/B2)
3. Align notch in RAM with notch in slot
4. Press down firmly until latches click closed
5. **Do not force** - if it doesn't fit, check orientation

### 1.5 CPU Installation

**Critical - Handle with extreme care:**

1. Locate CPU socket lever, push right and lift
2. **Never touch CPU pins/contacts** - hold by edges only
3. Find golden triangle on CPU and socket (alignment markers)
4. Gently drop CPU into socket (should fall into place)
5. Lower lever - may require firm pressure
6. Black plastic protector will pop off (save for storage)

### 1.6 CPU Cooler Installation

**AMD Installation:**
1. Remove factory mounting brackets from motherboard
2. Use AMD mounting hardware from cooler box
3. Install new mounting stand per manual instructions
4. Apply thermal paste:
   - Use small drop (rice grain size) or X pattern
   - **Do not over-apply** - excess is wasteful and messy
5. Place cooler on CPU
6. Remove top thumb screws by hand
7. Secure cooler to mounting bracket
8. Replace thumb screws
9. Install secondary fan (if included)
10. Connect fan cables to motherboard CPU_FAN headers

### 1.7 Storage Drives

**Installation:**
1. If available, test drives first: `smartctl -a /dev/sdX`
2. Mount drives in drive cages
3. Connect SATA data cable from each drive to motherboard
4. Connect SATA power cables from PSU
5. **Tip:** Use one SATA power cable with multiple connectors for all drives

**Recommended Layout:**
- SSD (NVMe M.2): OS and applications
- HDD 1: `/home` (user data)
- HDD 2: `/scratch` (temporary computation data)

### 1.8 GPU Installation

**Wait until most other components are installed**

1. Remove PCIe slot covers (3-4 slots usually needed)
2. Carefully remove GPU from anti-static packaging
3. Insert into top PCIe x16 slot (closest to CPU)
4. Press down firmly until retention clip engages
5. Secure GPU bracket to case with screws
6. Connect PCIe power cables from PSU (6-pin, 8-pin, or both)
7. **Do not force** - check for obstructions if GPU doesn't seat

### 1.9 Front Panel Connectors

**Location:** Lower-right corner of motherboard (usually labeled JFP1)

**Connectors to attach:**
- **PWR_SW** (Power button) - polarity doesn't matter
- **RESET_SW** (Reset button) - polarity doesn't matter  
- **PWR_LED** (Power indicator) - polarity matters (+ and -)
- **HDD_LED** (Activity light) - polarity matters

**Check motherboard manual** for exact pinout - it varies by manufacturer.

### 1.10 First Power-On

**Pre-flight Check:**
- [ ] All power cables connected
- [ ] RAM fully seated (latches closed)
- [ ] CPU cooler connected to CPU_FAN header
- [ ] Front panel connectors attached
- [ ] GPU power cables connected (if GPU installed)
- [ ] No loose screws or metal in case
- [ ] Monitor connected to GPU (not motherboard)

**Power On:**
1. Connect power cord to PSU and outlet
2. Switch PSU power switch to ON (I position)
3. Press case power button
4. **Watch for:**
   - Motherboard lights illuminate
   - Fans start spinning
   - POST codes on motherboard (if LED display present)
   - Video output to monitor

**Troubleshooting No Boot:**
- Check RAM seating (most common issue)
- Verify CPU power connected
- Check front panel power button connection
- Reseat GPU if no video output
- Consult motherboard manual POST code meanings

---

## 2. Preparing for OS Installation

### 2.1 Before You Begin an Upgrade

**If upgrading an existing workstation:**

1. **Backup `/home`:**
```bash
   /root/dailybackup.sh  # If this script exists
   # Or manual backup to NAS
```

2. **Backup crontab:**
```bash
   crontab -l > /root/crontab.backup.txt
   # Copy this file to another location
```

3. **Backup `/root` customizations** (optional but recommended)

4. **Document current setup:**
```bash
   lspci > /root/lspci.txt
   lsblk > /root/lsblk.txt
   ip addr > /root/network.txt
```

### 2.2 Create Installation Media

**Requirements:**
- USB drive: 32GB minimum (64GB recommended)
- Rocky Linux 10.x ISO image

**Download ISO:**
```bash
# From Rocky Linux website
wget https://download.rockylinux.org/pub/rocky/10/isos/x86_64/Rocky-10.x-x86_64-dvd.iso

# Verify checksum
sha256sum Rocky-10.x-x86_64-dvd.iso
```

**Create bootable USB (Linux):**
```bash
# Find USB device
lsblk

# Create bootable USB (replace /dev/sdX with your USB device)
sudo dd if=Rocky-10.x-x86_64-dvd.iso of=/dev/sdX bs=4M status=progress conv=fsync
```

### 2.3 BIOS/UEFI Setup

**Boot into BIOS:**
- Usually press `DEL` or `Delete` during boot
- Some systems use `F2`, `F10`, or `F12`

**Configure:**
1. **Boot Order:**
   - Move USB devices above internal drives
   - Or specifically select USB device as first boot

2. **Disable Secure Boot** (if present)
   - Required for some NVIDIA drivers
   - Usually in Security or Boot section

3. **Enable Virtualization** (if available)
   - Intel: VT-x
   - AMD: AMD-V
   - Helpful for containerization

4. **Save and Exit**

### 2.4 Network Information Gathering

**You'll need:**
- **Hostname:** Choose an appropriate hostname
- **MAC Address:** From network interface card
```bash
  # If existing system
  ip link show
  # Look for link/ether XX:XX:XX:XX:XX:XX
  
  # Or from BIOS/UEFI system information
```

- **Submit to IT:**
  - Hostname and MAC address must be registered
  - Required for campus network access

---

## 3. Installing Rocky Linux

### 3.1 Boot from USB

1. Insert USB installation media
2. Power on/reboot computer
3. Select USB drive from boot menu
4. Optional: Test installation media (recommended for first use)

### 3.2 Installation Welcome Screen

**Language Selection:**
- Choose language preference (US English default)
- Additional languages can be added later

### 3.3 Installation Summary Screen

**Critical settings to configure before proceeding:**

#### Root Password
- Set a strong temporary password
- **Write it down** - cannot recover without reinstall
- Will change later to key-based authentication

#### Language Support
- US English pre-selected
- Add additional languages if needed
- Useful for testing multi-language software

#### Installation Source
- Should show "Local media" (your USB drive)
- **Do not change** unless expert user

#### Software Selection
**Choose: "Server with GUI"**

This provides:
- Graphical desktop environment (GNOME)
- Essential server tools
- Better starting point than minimal install

Additional packages will be installed via network after installation.

#### Network & Hostname

**Configure network interface:**
1. Click network interface (usually `enpXsY` or `eth0`)
2. Toggle switch to **ON**
3. Click "Configure" button
4. **General tab:**
   - ☑ Connect automatically
5. **IPv4 Settings:** (if static IP)
   - Method: Manual
   - Add static IP, netmask, gateway
   - DNS: 141.166.35.156 (UR DNS)
6. **Set hostname:** Your approved hostname

**Example:**
```
Hostname: adam.richmond.edu
or: adam  (domain added automatically)
```

#### Installation Destination (Disk Partitioning)

**This is the most critical step.**

**Parish Lab Standard Partitioning Scheme:**

##### For Machines with SSD + HDD:

**SSD (1TB NVMe - OS):**
- `/boot/efi`: 10 GB (EFI System Partition)
- `/boot`: 10 GB (XFS)
- `swap`: 4 GB
- `/` (root): Remaining space (~976 GB) (XFS)

**HDD 1 (User data):**
- `/home`: Entire disk (XFS)

**HDD 2 (Computation scratch):**
- `/scratch`: Entire disk (XFS)

**Configuration Steps:**

1. Select "Custom" storage configuration
2. Click "Done"
3. Click "+" to add mount point:
```
Mount Point: /boot/efi
Capacity: 10 GiB
Device Type: Standard Partition
File System: EFI System Partition
Device: Select SSD (nvme0n1)

Mount Point: /boot
Capacity: 10 GiB
Device Type: Standard Partition
File System: xfs
Device: Select SSD (nvme0n1)

Mount Point: (swap)
Capacity: 4 GiB
Device Type: Standard Partition
File System: swap
Device: Select SSD (nvme0n1)

Mount Point: /
Capacity: (remaining SSD space, ~976 GiB)
Device Type: Standard Partition
File System: xfs
Device: Select SSD (nvme0n1)

Mount Point: /home
Capacity: (entire disk)
Device Type: Standard Partition
File System: xfs
Device: Select first HDD (sda)

Mount Point: /scratch
Capacity: (entire disk)
Device Type: Standard Partition
File System: xfs
Device: Select second HDD (sdb)
```

4. Click "Done"
5. Review and accept changes

**Important Notes:**
- Separate `/scratch` prevents computation data from filling `/home`
- Each filesystem is on its own physical disk for maximum isolation
- XFS is recommended for large files (better for HPC)
- No LVM - using standard partitions for simplicity and performance

### 3.4 Begin Installation

1. Click "Begin Installation"
2. Installation will proceed (20-45 minutes depending on media speed)
3. **Set root password** during installation
4. Optionally create additional user (not necessary - will add users later)

### 3.5 Complete Installation

1. Click "Reboot" when installation completes
2. Remove USB drive when prompted
3. System will boot into new installation

---

## 4. Initial System Configuration

### 4.1 First Boot Configuration

**On first login (graphical):**
1. Accept license agreement
2. Skip user creation (will add via command line)
3. Disable location services (optional)
4. Complete setup wizard

### 4.2 Prevent Non-Root Logins During Setup

**Prevents users from logging in until machine is fully configured:**
```bash
sudo touch /etc/nologin
```

Remove this file when setup is complete:
```bash
sudo rm /etc/nologin
```

### 4.3 Configure sudo Access

**Allow wheel group to use sudo without password:**
```bash
sudo visudo
```

Find these lines:
```
## Allows people in group wheel to run all commands
%wheel  ALL=(ALL)       ALL

## Same thing without a password
# %wheel        ALL=(ALL)       NOPASSWD: ALL
```

Change to:
```
## Allows people in group wheel to run all commands
# %wheel  ALL=(ALL)       ALL

## Same thing without a password
%wheel        ALL=(ALL)       NOPASSWD: ALL
```

**Add your administrative user to wheel group:**
```bash
sudo usermod -aG wheel yourusername
```

**For remaining operations, switch to root:**
```bash
su root
```

### 4.4 Disable SELinux

**Why:** SELinux interferes with scientific software packages. Workstations are VPN/campus-access only, so security risk is minimal.

**Immediate disable (current session):**
```bash
setenforce 0
grubby --update-kernel ALL --args selinux=0
```

**Permanent disable:**
```bash
vi /etc/selinux/config
```

Change:
```
SELINUX=enforcing
```

To:
```
SELINUX=disabled
```

### 4.5 Remove FIPS Boot Parameter (if present)

**Check if FIPS is enabled:**
```bash
grep fips /proc/cmdline
```

**If present, remove it:**
```bash
grubby --update-kernel ALL --remove-args fips=1
grubby --info=ALL | grep args | head -1
```

**Reboot for changes to take effect.**

### 4.6 System Update

**Update all packages to latest versions:**
```bash
dnf -y update
```

This may take 20-60 minutes depending on connection speed and updates available.

**Reboot after major updates:**
```bash
shutdown -r now
```

### 4.7 Fix .bashrc for scp/rsync Compatibility

**Problem:** Default .bashrc outputs text during non-interactive sessions, breaking scp/rsync.

**Fix for root:**
```bash
sed -i '1i # Exit if not running interactively\n[ -z "$PS1" ] && return\n' /root/.bashrc
```

**Fix template for new users:**
```bash
sed -i '1i # Exit if not running interactively\n[ -z "$PS1" ] && return\n' /etc/skel/.bashrc
```

### 4.8 Configure Shared Filesystems

**Set up /scratch for multi-user access:**
```bash
# Set sticky bit + world-writable permissions
# 1777 = sticky bit (users can only delete their own files) + rwxrwxrwx
chmod 1777 /scratch

# Verify permissions
ls -ld /scratch
# Should show: drwxrwxrwt 2 root root ... /scratch
```

**The sticky bit (1777) ensures:**
- All users can create files/directories in `/scratch`
- Users can only delete their own files
- Prevents accidental deletion of other users' computation data

**Optional: Set up quota warnings** (if disk quotas are needed):
```bash
# Create a README for users
cat > /scratch/README << 'EOF'
/scratch - Temporary Computation Space

- This is for temporary computation data only
- Files older than 30 days may be automatically deleted
- Back up important results to your home directory
- DO NOT store final results here permanently
EOF

chmod 644 /scratch/README
```

---

## 5. Authentication Setup

### 5.1 Required Packages
```bash
dnf -y install krb5-workstation
dnf -y install sssd-tools
dnf -y install authselect
```

### 5.2 Kerberos Configuration

**Get the `/etc/krb5.conf` from another workstation:**

**Set permissions:**
```bash
chown root:root /etc/krb5.conf
chmod 0644 /etc/krb5.conf
```

### 5.3 SSSD Configuration

**Get the `/etc/sssd/sssd.conf` from another workstation:**

**Set permissions:**
```bash
mkdir -p /etc/sssd/{conf.d,pki}
chown -R root:sssd /etc/sssd
chmod 750 /etc/sssd
chmod 751 /etc/sssd/conf.d
chmod 751 /etc/sssd/pki
chmod 640 /etc/sssd/sssd.conf
```

### 5.4 Update Cryptographic Policies

**Set legacy policy for compatibility:**
```bash
update-crypto-policies --set LEGACY
update-ca-trust
```

### 5.5 Enable Authentication
```bash
authselect select sssd --force
systemctl enable sssd
systemctl start sssd
```

**Verify authentication is working:**
```bash
systemctl --no-pager status sssd

# Should show "active (running)"
# Test authentication:
id yourusername
# Should return UID, GID, and groups from LDAP
```

---

## 6. Software Installation

### 6.1 Enable Additional Repositories
```bash
# EPEL (Extra Packages for Enterprise Linux)
dnf -y install epel-release

# RPM Fusion (free and non-free)
dnf -y install https://download1.rpmfusion.org/free/el/rpmfusion-free-release-10.noarch.rpm
dnf -y install https://download1.rpmfusion.org/nonfree/el/rpmfusion-nonfree-release-10.noarch.rpm

# Enable extras and powertools
dnf config-manager --enable extras
dnf config-manager --enable powertools
```

### 6.2 Essential System Packages

**Development tools:**
```bash
dnf -y groupinstall "Development Tools"
dnf -y install gcc gcc-gfortran gcc-c++
dnf -y install make cmake
dnf -y install binutils binutils-devel
dnf -y install kernel-headers kernel-devel
```

**Libraries:**
```bash
dnf -y install libcrypt\*
dnf -y install libgfortran\*
dnf -y install libGLU\*
dnf -y install libXmu\*
dnf -y install libffi\*
dnf -y install libXt-devel libX11-devel libXext-devel
dnf -y install ncurses\*
dnf -y install zlib\*
dnf -y install bzip2 bzip2-devel
```

**Utilities:**
```bash
dnf -y install htop
dnf -y install tree
dnf -y install wget
dnf -y install unzip
dnf -y install screen
dnf -y install bc
dnf -y install expect
dnf -y install kitty  # Modern terminal emulator
```

**Scientific computing:**
```bash
dnf -y install mpich mpich-devel
dnf -y install openmpi\*
dnf -y install netcdf-devel openmpi-devel fftw-devel
```

**Python tools:**
```bash
dnf -y install python3-devel
dnf -y install python3-dnf-plugin-versionlock
```

### 6.3 Environment Modules

**Why:** Allows switching between different software versions (e.g., CUDA 11.7 vs 12.0)

**Install:**
```bash
dnf -y install environment-modules
dnf -y install tcl tcl-devel
```

**Configure:**
```bash
# Modules are managed via /usr/local/ur/modulefiles
# This directory will be populated from NAS mount (see next section)
```

### 6.4 Mount NAS Shares

**UR Chemistry software and Intel tools are on NAS:**
```bash
# Create mount points
mkdir -p /usr/local/chem.sw
mkdir -p /opt/intel
```

**Add to `/etc/fstab`:**
```bash
cat >> /etc/fstab << 'EOF'
141.166.186.35:/mnt/usrlocal/8          /usr/local/chem.sw  nfs  ro,nosuid,nofail,_netdev,bg,timeo=10,retrans=2  0 0
141.166.186.35:/mnt/usrlocal/intel-tools  /opt/intel       nfs  ro,nosuid,nofail,_netdev,bg,timeo=10,retrans=2  0 0
EOF
```

**Mount now:**
```bash
mount -av
```

**Create symlinks in `/usr/local`:**
```bash
cd /usr/local

# Remove default directories (they're empty on new install)
rm -fr bin etc games include lib lib64 libexec sbin src

# Create symlinks to NAS software
for f in $(ls -1 chem.sw); do 
    ln -s "chem.sw/$f" "$f"
done
```

**Columbus chemistry software symlink:**
```bash
mkdir -p /usr/local/columbus/Col7.2.2_2023-09-06_linux64.ifc_bin
cd /usr/local/columbus/Col7.2.2_2023-09-06_linux64.ifc_bin
ln -s /usr/local/chem.sw/Columbus Columbus
```

### 6.5 Legacy Library Compatibility

**Some older software requires legacy libraries.**

**Create symlinks as needed:**
```bash
cd /usr/lib64

# Example: If software needs libffi.so.6 but only libffi.so.8 exists:
ln -s libffi.so.8 libffi.so.6
ln -s libffi.so.8 libffi.so.6.0
```

**Check what's available:**
```bash
ls -la /usr/lib64/libffi*
```

---

## 7. GPU and CUDA Setup

### 7.1 Identify GPU Hardware
```bash
lspci | grep -i nvidia
```

**Example output:**
```
01:00.0 VGA compatible controller: NVIDIA Corporation AD102 [GeForce RTX 4090] (rev a1)
```

**Common UR GPUs:**
- RTX 6000 Ada (arachne nodes)
- RTX 4090 (newer workstations)
- RTX 3090 (mid-age workstations)
- RTX A6000 (older workstations)

### 7.2 NVIDIA Repository Setup
```bash
# Add NVIDIA CUDA repository
dnf -y config-manager --add-repo https://developer.download.nvidia.com/compute/cuda/repos/rhel10/x86_64/cuda-rhel10.repo

# Install NVIDIA base repositories (copy from another machine or download)
# These RPMs contain repository configurations
dnf -y install nvidia-main-local-repo-25.11-11.el10.x86_64.rpm
dnf -y install nvidia-driver-local-repo-rhel10-580.105.08-1.0-1.x86_64.rpm

# Or download directly:
dnf install -y https://repo.download.nvidia.com/baseos/el/el-files/10/nvidia-repositories-25.09-5.el10.x86_64.rpm
```

### 7.3 Disable Nouveau Driver

**Nouveau is the default open-source NVIDIA driver - must be disabled.**

**Edit `/etc/default/grub`:**
```bash
vi /etc/default/grub
```

Find line starting with `GRUB_CMDLINE_LINUX="..."`

Add at the end (inside quotes):
```
modprobe.blacklist=nouveau
```

**Create blacklist files:**
```bash
cat > /etc/modprobe.d/blacklist-nouveau.conf << 'EOF'
blacklist nouveau
options nouveau modeset=0
EOF

cat > /etc/modprobe.d/denylist.conf << 'EOF'
blacklist nouveau
options nouveau modeset=0
EOF
```

**Rebuild boot image:**
```bash
dracut --force
```

**Update GRUB configuration:**
```bash
# Check if EFI boot:
if [ -d /sys/firmware/efi ]; then
    grub2-mkconfig -o /boot/efi/EFI/rocky/grub.cfg
else
    grub2-mkconfig -o /boot/grub2/grub.cfg
fi
```

### 7.4 Disable GUI for Driver Installation

**NVIDIA driver cannot be installed with GUI running:**
```bash
systemctl disable gdm
systemctl set-default multi-user.target
shutdown -r now
```

**System will reboot to text mode.**

### 7.5 Install Required Build Tools
```bash
# Disable problematic CRB repo
dnf config-manager --set-disabled crb

# Install build essentials
dnf install -y dkms kernel-devel kernel-modules-extra kernel-headers
dnf install -y gcc make acpid
dnf install -y libglvnd-glx libglvnd-opengl libglvnd-devel
dnf install -y vulkan-devel elfutils-libelf-devel
dnf install -y pkgconfig
```

### 7.6 Install NVIDIA Driver

**If previous installation failed, clean it first:**
```bash
nvidia-uninstall 2>/dev/null
dnf remove '*nvidia*' -y    
rm -rf /var/lib/dkms/nvidia* /usr/src/nvidia-* /lib/modules/$(uname -r)/extra/*nvidia*
dnf clean all
```

**Install NVIDIA driver (choose version based on GPU):**
```bash
# For RTX 4090, RTX 6000 Ada (latest GPUs):
dnf install -y nvidia-driver-580.82.07 \
    nvidia-driver-libs-580.82.07 \
    nvidia-driver-cuda-580.82.07 \
    nvidia-driver-cuda-libs-580.82.07 \
    nvidia-settings-580.82.07 \
    nvidia-persistenced-580.82.07 \
    kmod-nvidia-latest-dkms-580.82.07

# Rebuild cache
dnf clean all && dnf makecache
```

**For older GPUs (RTX 3090, RTX A6000):**
```bash
# Use driver version 550.x or 535.x
dnf install -y nvidia-driver-550.xxx [etc...]
```

### 7.7 Install CUDA Toolkit
```bash
dnf install -y cuda-toolkit
```

**Verify installation:**
```bash
/usr/local/cuda/bin/nvcc --version
```

**Expected output:**
```
nvcc: NVIDIA (R) Cuda compiler driver
Copyright (c) 2005-2025 NVIDIA Corporation
Built on Wed_Aug_20_01:58:59_PM_PDT_2025
Cuda compilation tools, release 13.0, V13.0.88
Build cuda_13.0.r13.0/compiler.36424714_0
```

### 7.8 GPU Direct Storage (GDS)

**For NVMe drives in CPU-direct slot (significant performance boost):**
```bash
dnf install -y nvidia-gds --skip-broken
```

**Check GDS configuration:**
```bash
/usr/local/cuda/gds/tools/gdscheck -p
```

**Test GDS performance (optional):**

Create test file `gdsread.cu`:
```c
#include <fcntl.h>
#include <stdlib.h>
#include <cuda_runtime.h>
#include <cufile.h>

int main(int argc, char **argv) {
    const char *path = argv[1]; 
    size_t sz = strtoull(argv[2], NULL, 10);
    int fd = open(path, O_RDONLY | O_DIRECT);
    cuFileDriverOpen();
    CUfileDescr_t d = {}; 
    d.handle.fd = fd; 
    d.type = CU_FILE_HANDLE_TYPE_OPAQUE_FD;
    CUfileHandle_t h; 
    cuFileHandleRegister(&h, &d);
    void *dptr; 
    cudaMalloc(&dptr, sz);
    ssize_t n = cuFileRead(h, dptr, sz, 0, 0); 
    cudaDeviceSynchronize();
    cuFileHandleDeregister(h); 
    cuFileDriverClose(); 
    close(fd); 
    cudaFree(dptr);
    return (n < 0);
}
```

Compile and test:
```bash
nvcc -O2 -o gdsread gdsread.cu -lcufile

# Create test file on NVMe
dd if=/dev/zero of=/mnt/nvme0/bigfile bs=1G count=32

# Test GDS read
FILE=/mnt/nvme0/bigfile
SIZE=$((32 * 1024 * 1024 * 1024))
/usr/bin/time -v ./gdsread "$FILE" "$SIZE"

# Compare to CPU read
/usr/bin/time -v dd if="$FILE" of=/dev/null bs=4M status=none
```

**GDS should be 2-3x faster than CPU read.**

### 7.9 Multiple CUDA Versions (Optional)

**Some software requires older CUDA versions (e.g., Amber24 needs CUDA 12.0):**
```bash
cd /tmp

# Download CUDA 12.0 installer
wget https://developer.download.nvidia.com/compute/cuda/12.0.0/local_installers/cuda_12.0.0_525.60.13_linux.run

# Install to alternate location
sudo sh /tmp/cuda_12.0.0_525.60.13_linux.run \
  --silent \
  --toolkit \
  --toolkitpath=/usr/local/cuda-12.0 \
  --no-opengl-libs \
  --no-drm \
  --no-man-page \
  --override

# Verify installation
ls -la /usr/local/cuda-12.0/
/usr/local/cuda-12.0/bin/nvcc --version

# Check required libraries
ls -la /usr/local/cuda-12.0/lib64/libcufft.so.11
ls -la /usr/local/cuda-12.0/lib64/libcublas.so.12
```

**Use environment modules to switch between versions:**
```bash
module load cuda/12.0
# or
module load cuda/13.0
```

### 7.10 Re-enable GUI
```bash
systemctl enable gdm
systemctl set-default graphical.target
systemctl start gdm
shutdown -r now
```

**System will reboot with GUI.**

**Verify GPU is working:**
```bash
nvidia-smi
```

**Expected output:**
```
+-----------------------------------------------------------------------------+
| NVIDIA-SMI 580.82.07    Driver Version: 580.82.07    CUDA Version: 13.0   |
|-------------------------------+----------------------+----------------------+
| GPU  Name        Persistence-M| Bus-Id        Disp.A | Volatile Uncorr. ECC |
| Fan  Temp  Perf  Pwr:Usage/Cap|         Memory-Usage | GPU-Util  Compute M. |
|===============================+======================+======================|
|   0  NVIDIA GeForce ...  Off  | 00000000:01:00.0  On |                  N/A |
| 30%   45C    P8    25W / 450W |    523MiB / 24564MiB |      2%      Default |
+-------------------------------+----------------------+----------------------+
```

---

## 8. User Management

### 8.1 Restore User Data from Backup

**If rebuilding existing workstation:**
```bash
# Install screen for background jobs
dnf install -y screen
mkdir -p /run/screen
chmod 777 /run/screen

# Get hostname
HOSTNAME=$(hostname -s)

# Start restore in screen session
screen -dmS restore bash -c "rsync -avhP \
  --exclude='scr/' \
  --exclude='nohup.out' \
  --exclude='*_macro' \
  franksinatra:/mnt/everything/backup-testing/${HOSTNAME}/ \
  /home/ > /var/log/${HOSTNAME}_restore.log 2>&1"

# Monitor progress
tail -f /var/log/${HOSTNAME}_restore.log

# Or reattach to screen
screen -r restore
```

**Important:** This restores to `/home/` directory structure. Verify data landed in correct user directories.

### 8.2 Add Users to nogroup

**After rsync completes:**
```bash
# Create nogroup (GID 65533)
groupadd -g 65533 nogroup

# Add all users to nogroup
for user in /home/*; do
    user=$(basename "$user")
    
    # Skip non-user directories
    [ "$user" = "scratch" ] && continue
    [ "$user" = "dnf-cache" ] && continue
    [ "$user" = "lost+found" ] && continue
    
    # Add user to nogroup if not already member
    if id "$user" &>/dev/null && ! id "$user" | grep -q nogroup; then
        usermod -aG nogroup "$user" && echo "Added: $user"
    fi
done
```

### 8.3 Create New Users

**Using wstools (recommended):**
```bash
# Clone wstools repository
cd /root
git clone https://github.com/georgeflanagin/wstools
cd wstools

# Source the functions
source wstools.bash

# Add to .bashrc for permanent availability
echo "source /root/wstools/wstools.bash" >> /root/.bashrc

# Define your workstations
export my_computers="adam eve cain abel"

# Add single user to this machine
newuser netid

# Add multiple users to this machine
newusers netid1 netid2 netid3

# Add users to remote machine
newusers_remote hostname netid1 netid2

# Add users to all machines
newusers_remote all netid1 netid2
```

**Manual user creation:**
```bash
# Get user info from LDAP
id netid

# Create user with correct UID
useradd -u UID -s /bin/csh netid

# Set home directory permissions
chown -R netid:users /home/netid
chmod 2755 /home/netid
find /home/netid -type d -exec chmod 2755 {} \;
find /home/netid -type f -exec chmod 0644 {} \;
```

### 8.4 User Removal (Graduated Students)

**For student lab machines:**
```bash
# List inactive users
for user in /home/*; do
    user=$(basename "$user")
    last -1 "$user" 2>/dev/null | head -1
done

# Remove user (keep home directory for archive)
userdel username  # Does NOT remove /home/username

# Archive home directory
tar czf /root/archive/username-$(date +%Y%m%d).tar.gz /home/username
mv /home/username /home/.archived-username
```

---

## 9. Maintenance Tasks

### 9.1 System Updates

**Regular updates:**
```bash
# Update all packages
dnf update -y

# Reboot if kernel updated
dnf needs-restarting -r
# If exit code 1, reboot needed:
shutdown -r now
```

**Automatic updates (optional):**
```bash
dnf install -y dnf-automatic

# Edit /etc/dnf/automatic.conf
# Set: apply_updates = yes

systemctl enable --now dnf-automatic.timer
```

### 9.2 NVIDIA Driver Updates

**After kernel updates, NVIDIA driver may need rebuilding:**

**Check if driver loaded:**
```bash
nvidia-smi
```

**If error, rebuild driver:**
```bash
# Stop GUI
systemctl stop gdm

# Reinstall driver (DKMS will rebuild for new kernel)
dnf reinstall -y kmod-nvidia-latest-dkms

# Or install kernel-devel for new kernel
dnf install -y kernel-devel-$(uname -r) kernel-headers-$(uname -r)

# Rebuild DKMS modules
dkms autoinstall

# Restart GUI
systemctl start gdm
```

### 9.3 Disk Space Management

**Monitor disk usage:**
```bash
df -h
```

**Common issues:**

#### /boot partition full:
```bash
# Remove old kernels (keeps current + 1 previous)
dnf remove --oldinstallonly kernel

# List installed kernels
rpm -q kernel

# Current kernel
uname -r

# Manually remove specific old kernel
dnf remove kernel-5.14.0-xxx.el10.x86_64
```

#### /scratch full:
```bash
# Find large files
du -sh /scratch/* | sort -h | tail -20

# Notify users to clean up
# Or implement automatic cleanup policy:

# Delete files older than 30 days
find /scratch -type f -mtime +30 -delete
```

#### /home full:
```bash
# Check per-user usage
du -sh /home/* | sort -h

# Common culprits:
du -sh /home/*/.cache
du -sh /home/*/.local/share/Trash
du -sh /home/*/Downloads

# Users should clean:
rm -rf ~/.cache/*
rm -rf ~/.local/share/Trash/*
```

### 9.4 NAS Mount Issues

**If NAS mounts fail to mount at boot:**
```bash
# Check if mounted
df -h | grep 141.166.186.35

# Manual mount
mount -a

# Check fstab entry
grep 141.166.186 /etc/fstab

# Test mount
mount -v -t nfs 141.166.186.35:/mnt/usrlocal/8 /usr/local/chem.sw
```

**Permanent fix if mount fails at boot:**
```bash
# Add to /etc/fstab if not already present:
# bg,timeo=10,retrans=2 allows boot to continue if NAS unavailable
```

### 9.5 Authentication Issues

**If users cannot login:**
```bash
# Check SSSD status
systemctl status sssd

# Restart SSSD
systemctl restart sssd

# Test authentication
id username

# Check logs
tail -50 /var/log/sssd/sssd.log
tail -50 /var/log/sssd/sssd_default.log
```

**If LDAP connection fails:**
```bash
# Test LDAP server
ldapsearch -x -H ldap://ldap.richmond.edu -b "dc=richmond,dc=edu"

# Test Kerberos
kinit username@GONDOR.RICHMOND.EDU
klist
```

### 9.6 GPU Troubleshooting

**No GUI after driver install:**
```bash
# Check if driver loaded
lsmod | grep nvidia

# Check for errors
dmesg | grep -i nvidia

# Verify nouveau is disabled
lsmod | grep nouveau  # Should be empty

# Rebuild initramfs
dracut --force
reboot
```

**GPU not detected:**
```bash
# Check hardware
lspci | grep -i nvidia

# Reseat GPU (power off first!)

# Check PCIe slot in BIOS
# Some motherboards disable slots when others are used
```

### 9.7 Performance Monitoring

**CPU and memory:**
```bash
htop

# Or continuous monitoring
watch -n 2 'cat /proc/loadavg; free -h'
```

**GPU monitoring:**
```bash
# Real-time GPU usage
watch -n 1 nvidia-smi

# Or install gpustat
pip3 install gpustat
gpustat -i 1
```

**Disk I/O:**
```bash
iostat -x 2

# Per-process I/O
iotop
```

### 9.8 Backup Procedures

**Daily automated backup:**
```bash
# Backup script should exist at /root/dailybackup.sh
# Run manually:
/root/dailybackup.sh

# Or add to crontab:
crontab -e

# Add line:
0 2 * * * /root/dailybackup.sh > /var/log/backup.log 2>&1
```

**Manual backup to NAS:**
```bash
# Backup specific user
rsync -avhP --delete /home/username/ franksinatra:/mnt/everything/backup-testing/$(hostname -s)/username/

# Backup all users
for user in /home/*; do
    username=$(basename "$user")
    [ -d "$user" ] && rsync -avhP --delete "$user/" \
        franksinatra:/mnt/everything/backup-testing/$(hostname -s)/$username/
done
```

### 9.9 Log Management

**Important logs:**
```bash
# System logs
journalctl -xe
journalctl -u sssd
journalctl -u gdm

# Authentication
/var/log/sssd/
/var/log/secure

# NVIDIA
dmesg | grep -i nvidia
```

**Rotate logs to prevent disk filling:**
```bash
# Logrotate is configured by default
# Check configuration:
cat /etc/logrotate.conf
ls /etc/logrotate.d/
```

### 9.10 Enable /etc/nologin removal

**Once setup is complete, allow users to login:**
```bash
rm /etc/nologin
```

---

## Appendices

### Appendix A: Quick Reference Commands

**System Information:**
```bash
# Hostname
hostname -f

# OS version
cat /etc/redhat-release

# Kernel version
uname -r

# CPU info
lscpu

# Memory
free -h

# Disk info
lsblk
df -h

# Network interfaces
ip addr show

# PCI devices
lspci

# USB devices
lsusb
```

**Package Management:**
```bash
# Search for package
dnf search packagename

# Install package
dnf install packagename

# Remove package
dnf remove packagename

# Update all packages
dnf update

# List installed packages
dnf list installed

# Show package info
dnf info packagename

# List files in package
rpm -ql packagename

# Find which package provides file
dnf provides */filename
```

**Service Management:**
```bash
# Start service
systemctl start servicename

# Stop service
systemctl stop servicename

# Restart service
systemctl restart servicename

# Enable service (start at boot)
systemctl enable servicename

# Disable service
systemctl disable servicename

# Check service status
systemctl status servicename

# List all services
systemctl list-units --type=service
```

**User Management:**
```bash
# Add user
useradd username

# Delete user
userdel username

# Modify user
usermod -aG groupname username

# Change password
passwd username

# List users
cat /etc/passwd

# List groups
cat /etc/group

# Check user info
id username
```

---

### Appendix B: Standard Directory Structure
```
/
├── boot/           # Boot files
│   ├── efi/        # EFI boot (on EFI systems)
│   └── grub2/      # GRUB bootloader
├── home/           # User home directories
│   └── username/   # Individual user directories
├── opt/            # Optional/vendor software
│   └── intel/      # Intel tools (NFS mount)
├── root/           # Root user home
├── usr/
│   └── local/      # Local software installations
│       ├── bin/    # Local binaries (symlinks to NAS)
│       ├── lib/    # Local libraries (symlinks to NAS)
│       ├── chem.sw/ # NAS mount point for chemistry software
│       └── ur/     # UR-specific installations
│           └── modulefiles/  # Environment module files
├── var/            # Variable data (directory, not separate mount point)
│   └── log/        # Log files
└── scratch/        # Computation scratch space (should be separate filesystem)
```

**Best Practice:**
- `/home` and `/scratch` should be on separate filesystems
- `/var` should remain on root filesystem (not separate mount)
- `/usr/local` symlinks to NAS-mounted software
- `/opt/intel` is NFS-mounted from NAS

---

### Appendix C: Common Software Locations

**Installed via dnf:**
- `/usr/bin/` - System binaries
- `/usr/lib64/` - System libraries
- `/usr/share/` - Shared data

**Locally compiled:**
- `/usr/local/bin/` - Local binaries
- `/usr/local/lib/` - Local libraries

**NAS-mounted chemistry software:**
- `/usr/local/chem.sw/` - All chemistry packages
- Symlinked to `/usr/local/packagename`

**CUDA:**
- `/usr/local/cuda/` - Symlink to current version
- `/usr/local/cuda-12.0/` - Specific CUDA versions
- `/usr/local/cuda-13.0/`

**NVIDIA drivers:**
- `/usr/lib/modules/$(uname -r)/extra/` - Kernel modules
- `/usr/lib64/libnvidia-*.so` - NVIDIA libraries

**Environment modules:**
- `/usr/local/ur/modulefiles/` - Module files
- Load with: `module load packagename`

---

### Appendix D: Troubleshooting Flowcharts

#### Can't Login (GUI)
```
Can't login → Check if user exists → id username
                ↓ No                    ↓ Yes
         Add user                Check SSSD → systemctl status sssd
                                        ↓ Not running
                                  Start SSSD → systemctl start sssd
                                        ↓ Still fails
                                  Check logs → /var/log/sssd/
```

#### No GUI After Boot
```
No GUI → Check if GDM running → systemctl status gdm
            ↓ Not running           ↓ Running but no display
       Start GDM              Check NVIDIA driver
            ↓                        ↓
   systemctl start gdm         nvidia-smi fails?
            ↓                        ↓ Yes
       Still fails?            Reinstall driver
            ↓                        ↓
   Check default target      Check nouveau disabled
            ↓                        ↓
   systemctl get-default     lsmod | grep nouveau
            ↓                   (should be empty)
   multi-user.target?
            ↓ Yes
   systemctl set-default graphical.target
```

#### NVIDIA Driver Won't Load
```
nvidia-smi fails
    ↓
Check if nouveau loaded → lsmod | grep nouveau
    ↓ Nouveau present
Nouveau still active → Rebuild initramfs
    ↓                      ↓
    |                dracut --force
    |                      ↓
    |                   Reboot
    ↓
Check DKMS build → dkms status
    ↓ No nvidia module
Rebuild DKMS → dnf reinstall kmod-nvidia*
    ↓
Check kernel-devel → dnf install kernel-devel-$(uname -r)
```

---

### Appendix E: Contact Information

**University of Richmond IT Support:**
- IT Help Desk: 804-287-6400
- Email: hpc@richmond.edu

**Academic Research Computing:**
- João Riva Tonini: jtonini@richmond.edu
- Office: Gottwald Science Center

---

### Appendix F: Revision History

| Date | Changes |
|------|---------|
| Feb 2023 | Initial workstation setup guide (George Flanagin) |
| Jun 2023 | Added hardware assembly section |
| Jan 2026 | Updated for Rocky Linux 10.x, modern NVIDIA drivers, current authentication |
| Feb 2026 | Combined all documentation into comprehensive guide |

---

## Notes

**Images referenced in original documents:**
- Motherboard installation diagrams
- Cable routing photos
- BIOS screenshots
- NVIDIA driver download screenshots
- Boot partition listings

These images should be referenced from the original PDF documents when this guide is used.

**Best practices:**
- Always test on one machine before deploying to fleet
- Document any deviations from standard procedure
- Keep this guide updated as software versions change
- Backup before making major changes

---

*End of Guide*
