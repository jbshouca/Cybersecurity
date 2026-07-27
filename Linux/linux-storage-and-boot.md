# Linux Storage & Boot
 
Disks, partitions, mounts, filesystem recovery, GRUB, chroot, and Python virtual environments.
 
---
 
## Partitioning: parted and gparted
 
Both manage disk partitions — creating, resizing, deleting, organizing.
 
- `parted` — command-line
- `gparted` — GUI version
Use them for setting up a new disk, dual-booting, or forensic work examining disk images from compromised machines.
 
### Basic parted workflow
 
```bash
# Open parted on the new disk
sudo parted /dev/sdb
```
 
Inside interactive mode:
```
print                             # view current layout
mklabel gpt                       # create partition table
mkpart primary ext4 0% 50%        # create first partition
mkpart primary ext4 50% 100%      # second partition, remaining space
rm 2                              # delete partition 2
quit                              # exit
```
 
### Format and mount
 
```bash
# Format partitions
sudo mkfs.ext4 /dev/sdb1
sudo mkfs.ext4 /dev/sdb2
 
# Create mount points
sudo mkdir /mnt/disk1 /mnt/disk2
 
# Mount
sudo mount /dev/sdb1 /mnt/disk1
sudo mount /dev/sdb2 /mnt/disk2
 
# Verify
df -h | grep sdb
lsblk
 
# Test
echo "test file" | sudo tee /mnt/disk1/test.txt
ls /mnt/disk1
```
 
---
 
## fstab (persistent mounts)
 
`/etc/fstab` (file system table) tells the system what to mount and where on every boot. Without it, you'd manually mount everything after every reboot.
 
### Adding a remote Windows SMB share
 
**Step 1 — verify manual mounting works first.** A broken fstab entry can hang boot.
 
```bash
sudo apt install cifs-utils -y
sudo mkdir -p /mnt/windows_share
 
# Test manual mount
sudo mount -t cifs //192.168.170.135/SharedFolder /mnt/windows_share \
    -o username=winuser,password=winpass
 
ls /mnt/windows_share
sudo umount /mnt/windows_share
```
 
**Step 2 — credentials file** (don't put passwords in fstab; it's world-readable).
 
```bash
sudo vim /root/.smb_credentials
```
Contents:
```
username=your_windows_username
password=your_windows_password
domain=WORKGROUP
```
Lock it down:
```bash
sudo chmod 600 /root/.smb_credentials
```
 
**Step 3 — add fstab entry.**
 
```bash
sudo vim /etc/fstab
```
```
//192.168.170.135/SharedFolder  /mnt/windows_share  cifs  credentials=/root/.smb_credentials,_netdev,nofail,uid=1000,gid=1000,file_mode=0664,dir_mode=0775  0  0
```
 
Options explained:
- `credentials=` — credentials file location
- `_netdev` — this is a network mount; wait for network to be up before mounting
- `nofail` — if the Windows machine is off, don't hang boot
- `uid`/`gid` — ownership on the Linux side
- `file_mode` / `dir_mode` — permissions on files/dirs in the mount
**Step 4 — test without rebooting.**
 
```bash
sudo mount -a
```
`-a` reads fstab and mounts everything not already mounted. If it works without errors here, it'll work on boot.
 
---
 
## Mounting a Windows share (one-off)
 
```bash
sudo mkdir /mnt/windows_share
 
# Folder share
sudo mount -t cifs //192.168.170.135/SharedFolder /mnt/windows_share \
    -o credentials=/root/.smb_credentials
 
# Whole drive (admin share)
sudo mount -t cifs //192.168.170.135/C$ /mnt/windows_share \
    -o username=your_windows_username,password=your_windows_password
```
 
---
 
## Mounting an ISO with a one-liner for loop
 
Search the filesystem for an ISO, then mount the first one found:
 
```bash
for iso in $(find / -name "*.iso" -type f 2>/dev/null); do
    echo "Found: $iso"
    sudo mkdir -p /mnt/iso
    sudo mount -o loop "$iso" /mnt/iso && echo "Mounted $iso at /mnt/iso"
    break
done
```
 
Broken down:
- `find / -name "*.iso" -type f 2>/dev/null` — search entire filesystem for `.iso` files, suppressing permission-denied errors
- `-o loop` — treat a file as if it were a disk device (ISOs are files, not physical disks)
- `break` — stop after the first ISO; without this it would try to mount every ISO to the same mount point
---
 
## fsck (filesystem check)
 
Scans a filesystem for errors and repairs them — corrupted files, bad blocks, broken directory structures, orphaned inodes.
 
**NEVER run fsck on a mounted filesystem.** It reads and writes directly to the raw disk. Running it against a mounted FS while the OS is actively using it will cause data corruption.
 
```bash
# Check if it's mounted
mount | grep sdb1
 
# Unmount first
sudo umount /dev/sdb1
 
# Then check
sudo fsck /dev/sdb1
```
 
---
 
## chroot
 
`chroot` (change root) changes what a process thinks the root directory (`/`) is. When you chroot into a directory, that directory becomes `/` for everything running inside it — the process can't see or access anything outside.
 
### Basic usage
 
```bash
sudo chroot /path/to/new/root
```
 
Now everything after that runs as if `/path/to/new/root` were `/`.
 
### Use case 1: Fixing a broken system
 
Your system won't boot — GRUB is broken, fstab is bad, kernel update killed it. Boot from a live USB/ISO and chroot in to repair from the outside.
 
```bash
# Identify the broken system's partition
lsblk
 
# Mount it
sudo mount /dev/sda1 /mnt
 
# Bind-mount the special filesystems chroot needs
sudo mount --bind /dev  /mnt/dev
sudo mount --bind /dev/pts /mnt/dev/pts
sudo mount --bind /proc /mnt/proc
sudo mount --bind /sys  /mnt/sys
sudo mount --bind /run  /mnt/run
 
# chroot in
sudo chroot /mnt
 
# Now / IS the broken system's root — fix whatever's wrong
```
 
### Use case 2: Resetting a forgotten password
 
Same setup, then:
 
```bash
passwd root
passwd <username>
exit
sudo umount -R /mnt
sudo reboot
```
 
### Use case 3: Sandboxing a process
 
Run a program inside chroot so it sees only a minimal filesystem. If the program is compromised, the attacker is trapped in the chroot.
 
```bash
# Create the directory structure
sudo mkdir -p /opt/jail/{bin,lib,lib64}
 
# Copy bash
sudo cp /bin/bash /opt/jail/bin/
 
# Copy the libraries bash needs (find with ldd)
ldd /bin/bash
 
# Copy each listed library
sudo cp /lib/x86_64-linux-gnu/libtinfo.so.6 /opt/jail/lib/
sudo cp /lib/x86_64-linux-gnu/libc.so.6     /opt/jail/lib/
sudo cp /lib64/ld-linux-x86-64.so.2         /opt/jail/lib64/
 
# Enter
sudo chroot /opt/jail /bin/bash
```
 
### chroot into an encrypted filesystem (LUKS)
 
```bash
# Step 1: Boot from a Linux ISO
 
# Step 2: Find the encrypted partition
lsblk
 
# Step 3: Confirm LUKS
sudo cryptsetup isLuks /dev/sda2 && echo "Yes, it's LUKS"
 
# Step 4: Decrypt
sudo cryptsetup luksOpen /dev/sda2 decrypted
# Unlocked partition appears at /dev/mapper/decrypted
 
# LVM on LUKS (common) — scan for LVM volumes
sudo vgchange -ay
sudo lvs
 
# Step 5: Mount
# Without LVM:
sudo mount /dev/mapper/decrypted /mnt
# With LVM:
sudo mount /dev/<vg_name>/root /mnt
 
# Step 6: Mount /boot separately (not encrypted)
sudo mount /dev/sda1 /mnt/boot
 
# Step 7: Bind mounts + chroot
sudo mount --bind /dev  /mnt/dev
sudo mount --bind /dev/pts /mnt/dev/pts
sudo mount --bind /proc /mnt/proc
sudo mount --bind /sys  /mnt/sys
sudo mount --bind /run  /mnt/run
sudo chroot /mnt
```
 
---
 
## GRUB (bootloader)
 
GRUB (GRand Unified Bootloader) runs before the OS loads. It presents a menu of operating systems / kernel versions and boots your pick.
 
### View boot menu entries
 
```bash
grep -E "^menuentry|^submenu" /boot/grub/grub.cfg
```
 
### Change the default entry
 
Never edit `/boot/grub/grub.cfg` directly — it gets regenerated on every GRUB update. Edit the source config instead:
 
```bash
sudo nano /etc/default/grub
```
 
To boot the second entry inside the "Advanced options" submenu:
```
GRUB_DEFAULT="1>2"
```
(Submenu 1, entry 2.)
 
Apply:
```bash
sudo update-grub
sudo reboot
```
 
---
 
## Python virtual environments
 
A Python venv is an isolated copy of Python with its own packages, separate from the system Python. Think of it like chroot for Python — packages inside don't affect the system.
 
Use it for any project that imports third-party packages.
 
```bash
# Install venv
sudo apt install python3-venv -y
 
# Create the venv
mkdir ~/projects && cd ~/projects
python3 -m venv myenv
 
# Activate
source myenv/bin/activate
 
# You're now inside the venv — pip installs stay here
pip install requests
 
# Deactivate
deactivate
```
 
---
 
## Compression and archives
 
Compression reduces file size by finding patterns and encoding them more efficiently. Different algorithms trade compression ratio for speed.
 
### tar
 
`tar` bundles multiple files/directories into a single **tarball**. It doesn't compress by itself — you layer gzip, bzip2, or xz on top.
 
```bash
# Bundle without compression
tar -cvf archive.tar /home/user/myproject
 
# Bundle + gzip (most common)
tar -czvf archive.tar.gz /home/user/myproject
 
# Extract gzip tarball to current directory
tar -xzvf archive.tar.gz
 
# Extract to a specific directory
tar -xzvf archive.tar.gz -C /tmp/extracted/
```
 
Flags:
- `c` — create
- `x` — extract
- `v` — verbose
- `f` — file
- `z` — gzip
- `j` — bzip2
- `J` — xz
### xz
 
Best compression ratio of the standard tools — use when size matters more than speed.
 
```bash
xz file.txt           # compresses, replaces original with file.txt.xz
xz -d file.txt.xz     # decompress
```
 
### gzip / gunzip
 
```bash
gzip file.txt         # → file.txt.gz
gunzip file.txt.gz    # → file.txt
```
 
### bzip2
 
Middle ground on compression vs. speed.
```bash
bzip2 file.txt
bunzip2 file.txt.bz2
```
 
### zip / unzip
 
Cross-platform (Windows compatibility).
```bash
zip archive.zip file1 file2 dir/
unzip archive.zip
```
 
### 7z
 
Best-in-class ratio for many workloads; supports encryption.
```bash
sudo apt install p7zip-full -y
7z a archive.7z file1 dir/
7z x archive.7z          # extract
```
