# ByteOS - Custom Minimal Linux-Based OS

ByteOS is a minimal Linux-based operating system that boots with the Linux kernel, GRUB bootloader, and a lightweight init system. It is designed to be run in QEMU.

## 📌 Prerequisites
Ensure you have the following tools installed on your Linux system:

```bash
sudo apt update
sudo apt install -y build-essential qemu-system-x86 grub-pc-bin grub-common grub2-common mtools fdisk cpio xz-utils
```

## 📂 File Structure
```
ByteOS/
├── linux-6.x.y  # Linux kernel source (already compiled)
├── rootfs       # Root filesystem
│   ├── bin/
│   ├── boot/
│   ├── dev/
│   ├── etc/
│   ├── lib/
│   ├── sbin/
│   ├── usr/
│   ├── var/
│   └── ...
└── ByteOS.img   # Disk image (created later)
```

---

## 📥 1. Create a Bootable Disk Image

```bash
# Create a 512MB raw disk image
dd if=/dev/zero of=ByteOS.img bs=1M count=512

# Create a partition table and partition
sudo fdisk ByteOS.img <<EOF
n
p
1


w
EOF
```

Format the first partition:
```bash
sudo losetup -fP --show ByteOS.img   # Attach image as loop device
sudo mkfs.ext4 /dev/loop0p1          # Format partition as ext4
```

Mount the partition:
```bash
mkdir -p /mnt/ByteOS
sudo mount /dev/loop0p1 /mnt/ByteOS
```

Copy the root filesystem:
```bash
sudo cp -r rootfs/* /mnt/ByteOS
```

---

## 🔥 2. Install GRUB Bootloader

Ensure `boot` and `grub` directories exist:
```bash
sudo mkdir -p /mnt/ByteOS/boot/grub
```

Install GRUB:
```bash
sudo grub-install --target=i386-pc --boot-directory=/mnt/ByteOS/boot --no-floppy /dev/loop0
```

Create the GRUB configuration file:
```bash
cat <<EOF | sudo tee /mnt/ByteOS/boot/grub/grub.cfg
set timeout=5
set default=0

menuentry "ByteOS" {
    set root='(hd0,1)'
    linux /boot/vmlinuz root=/dev/sda1 rw
    initrd /boot/initrd.img
}
EOF
```

---

## ⚙ 3. Create `initrd.img`

Prepare the initramfs structure:
```bash
mkdir -p initramfs/{bin,sbin,etc,proc,sys,dev}
cp -a rootfs/bin/* initramfs/bin/
cp -a rootfs/sbin/* initramfs/sbin/
cp -a rootfs/etc/* initramfs/etc/
```

Create the init script:
```bash
cat <<EOF > initramfs/init
#!/bin/sh
mount -t proc proc /proc
mount -t sysfs sys /sys
mount -t devtmpfs devtmpfs /dev
echo "ByteOS Initramfs Loaded!"
exec /bin/bash
EOF
chmod +x initramfs/init
```

Create `initrd.img`:
```bash
cd initramfs
find . | cpio -o -H newc | gzip > ../rootfs/boot/initrd.img
```

Verify the file:
```bash
ls -lh ../rootfs/boot/initrd.img
```

---

## 🚀 4. Final Steps & Boot ByteOS in QEMU

Unmount and detach the disk:
```bash
sudo umount /mnt/ByteOS
sudo losetup -d /dev/loop0
```

Now, run ByteOS in **QEMU**:
```bash
qemu-system-x86_64 -drive file=ByteOS.img,format=raw -m 512M -smp 2 -serial mon:stdio
```

If everything is set up correctly, **ByteOS will boot into the shell!** 🎉

---

## 📌 Troubleshooting
**GRUB Not Found?**
```bash
sudo grub-install --target=i386-pc --boot-directory=/mnt/ByteOS/boot --no-floppy /dev/loop0
```

**initrd.img Not Found?**
```bash
ls -lh rootfs/boot/initrd.img  # Check if it exists
```

---

## 🎯 What's Next?
- Add networking support (e.g., `dhcpcd`)
- Add a package manager
- Add GUI support (Wayland/Xorg)

ByteOS is now **bootable and functional**. Enjoy hacking on it! 😎🚀

