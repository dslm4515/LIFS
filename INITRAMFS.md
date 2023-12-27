## Initramfs

Create a build directory and clone this repo.

```
mkdir -pv ~/initramfs_build && cd ~/initramfs_build
git clone https://github.com/dslm4515/LIFS
mkdir initramfs microcode
```

Download busybox and unpack it
```
wget https://www.busybox.net/downloads/busybox-1.36.1.tar.bz2
tar xf busybox-1.36.1.tar.bz2
cd busybox-1.36.1
```

Copy a config file to get started and check for any unresolved [config] symbols
```
cp -v ../LIFS/busybox-initramfs.config .config
make oldconfig
```

To customize, execute `make menuconfig`. This will run a tui to configure Busybox.
Otherwise, compile busybox.
```
make
```

Create a base directory tree for the initramfs
```
cd initramfs
mkdir -pv dev etc lib proc run bin sys mnt/root
mkdir -pv lib/{firmware,modules}
mkdir -pv etc/modprobe.d
mkdir -pv usr

# Create symlinks
ln -sv lib lib64
ln -sv bin sbin
ln -sv ../bin usr/bin
ln -sv ../bin usr/sbin

```

Copy any firmware blobs and/or kernel modules. 
For more information on what firmware blobs to add for GPU's, check Gentoo's guides for [Radeon](https://wiki.gentoo.org/wiki/Radeon) cards, [AMDGPU](https://wiki.gentoo.org/wiki/AMDGPU) cards and newer [Intel chipsets](https://wiki.gentoo.org/wiki/Intel#Firmware).`

Remember, copy kernel modules for the kernel that wil load this initramfs
```
cp -rv /lib/firmware/* lib/firmware/
cp -rv /lib/modules/* lib/modules/
```

Next, copy busybox and populate the `bin` directory with symlinks to busybox for utlities needed. This is can be skipped if just building an initramfs solely for early loading of CPU microcode and firmware blobs.
```
cp -v ../busybox-1.36.1/busybox bin/
for b in cat cp dd killall kmod ln ls mkdir \
         mknod mount rm sed sh sleep umount uname \
         basename readlink insmod lsmod mdev \
         switch_root findfs
do
  ln -sv  busybox bin/$b
done 
```

Create the `init` script. It must be executable and located at the root directory.
This is based off Gentoo's [guide](https://wiki.gentoo.org/wiki/Custom_Initramfs)
```
cat > init << "EOF"
#!/bin/busybox sh

# Mount the /proc and /sys filesystems.
mount -t proc none /proc
mount -t sysfs none /sys

# Mount the root filesystem.
mount -o ro /dev/sda1 /mnt/root

# Clean up.
umount /proc
umount /sys

# Boot the real thing.
exec switch_root /mnt/root /sbin/init
EOF

chmod -v a+x bin/init

```

Change ownership of the directories & files. Add nodes for udev/mdev to mount on.
```
sudo chown -R .
sudo mknod -m 600 dev/console c 5 1
sudo mknod -m 666 dev/null c 1 3
```

As root, create the initramfs image. Do not change directories until after image is made.

_May use cpio or libarchive's bsdcpio_
```
su
find . | bsdcpio -o -H newc | gzip -9  > ../initramfs.img
exit
cd ..
```

The `initramfs.img` is ready. If loading CPU microcode early, proceed below.

## Microcode (AMD/Intel)

For info on how to check & download micorcode, check the [Firmware section](https://www.linuxfromscratch.org/blfs/view/svn/postlfs/firmware.html) of BLFS.

Create the base directory:
```
cd microcode
mkdir -pv kernel/x86/microcode
```

Copy the microcode:
```
# For AMD:
cat /usr/lib/firmware/amd-ucode/microcode_amd*.bin > kernel/x86/microcode/AuthenticAMD.bin

# For Intel:
cat /usr/lib/firmware/intel-ucode/* > kernel/x86/microcode/GenuineIntel.bin
```

Create the microcode image
```
find . | cpio -o -H newc > ../microcode.img
cd ..
```

if desired, combine both microcode.img and initramfs.img into a single image
```
cat microcode.img initramfs.img > initrd.img
```


