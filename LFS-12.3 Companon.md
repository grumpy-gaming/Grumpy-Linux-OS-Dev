LFS 12.3 Cut-and-Paste Installation Guide

This guide provides a condensed, cut-and-paste friendly installation procedure for Linux From Scratch (LFS) version 12.3. It assumes you have a working Linux host system with necessary tools and sufficient disk space.

Disclaimer: This is a simplified guide. For detailed explanations, troubleshooting, and best practices, always refer to the official LFS 12.3 book. This guide is intended for experienced users who understand the LFS process.
1. Preparation
1.1. Partitioning and Filesystem Creation

(Replace /dev/sdX with your target device, e.g., /dev/sda)

# Assuming /dev/sdX1 for LFS partition, /dev/sdX2 for swap
# Use fdisk or parted to create partitions:
#   - /dev/sdX1: Linux filesystem (e.g., 50GB+)
#   - /dev/sdX2: Linux swap (e.g., 2GB+)

# Create ext4 filesystem on LFS partition
mkfs -v -t ext4 /dev/sdX1

# Create swap space
mkswap /dev/sdX2

# Enable swap
/sbin/swapon /dev/sdX2

1.2. Mounting the LFS Partition

# Create the LFS mount point
mkdir -pv $LFS

# Mount the LFS partition
mount -v -t ext4 /dev/sdX1 $LFS

# Verify mount
df -h $LFS

1.3. Creating the LFS Directory Structure

mkdir -pv $LFS/sources
chmod -v a+wt $LFS/sources

1.4. Adding the LFS User

groupadd lfs
useradd -s /bin/bash -g lfs -m -k /dev/null lfs
passwd lfs # Set a password for lfs user

chown -v lfs $LFS/sources
chown -v lfs $LFS/tools # Will be created later

# Log in as lfs user for the rest of the process
su - lfs

1.5. Setting up the LFS Environment (as lfs user)

cat > ~/.bash_profile << "EOF"
exec env -i HOME=$HOME TERM=$TERM PS1='\u:\w\$ ' /bin/bash
EOF

cat > ~/.bashrc << "EOF"
set +h
umask 022
LFS=/mnt/lfs # Adjust if your LFS mount point is different
LC_ALL=POSIX
LFS_TGT=$(uname -m)-lfs-linux-gnu
PATH=/usr/bin:/bin
export LFS LC_ALL LFS_TGT PATH
export CONFIG_SITE=$LFS/usr/share/config.site
EOF

source ~/.bash_profile

2. Packages and Patches
2.1. Downloading LFS Packages and Patches

(Download these to $LFS/sources on your host system, or use wget as lfs user)

# Navigate to sources directory
cd $LFS/sources

# Example: Download a few common packages. Refer to the LFS book for the complete list.
# wget [https://www.kernel.org/pub/linux/kernel/v6.x/linux-6.7.4.tar.xz](https://www.kernel.org/pub/linux/kernel/v6.x/linux-6.7.4.tar.xz)
# wget [https://ftp.gnu.org/gnu/glibc/glibc-2.39.tar.xz](https://ftp.gnu.org/gnu/glibc/glibc-2.39.tar.xz)
# wget [https://ftp.gnu.org/gnu/binutils/binutils-2.42.tar.xz](https://ftp.gnu.org/gnu/binutils/binutils-2.42.tar.xz)
# wget [https://ftp.gnu.org/gnu/gcc/gcc-13.2.0/gcc-13.2.0.tar.xz](https://ftp.gnu.org/gnu/gcc/gcc-13.2.0/gcc-13.2.0.tar.xz)
# wget [https://ftp.gnu.org/gnu/make/make-4.4.1.tar.gz](https://ftp.gnu.org/gnu/make/make-4.4.1.tar.gz)
# wget [https://ftp.gnu.org/gnu/bash/bash-5.2.21.tar.gz](https://ftp.gnu.org/gnu/bash/bash-5.2.21.tar.gz)
# wget [https://ftp.gnu.org/gnu/coreutils/coreutils-9.4.tar.xz](https://ftp.gnu.org/gnu/coreutils/coreutils-9.4.tar.xz)
# wget [https://ftp.gnu.org/gnu/findutils/findutils-4.9.0.tar.xz](https://ftp.gnu.org/gnu/findutils/findutils-4.9.0.tar.xz)
# wget [https://ftp.gnu.org/gnu/gawk/gawk-5.3.0.tar.xz](https://ftp.gnu.org/gnu/gawk/gawk-5.3.0.tar.xz)
# wget [https://ftp.gnu.org/gnu/grep/grep-3.11.tar.xz](https://ftp.gnu.org/gnu/grep/grep-3.11.tar.xz)
# wget [https://ftp.gnu.org/gnu/gzip/gzip-1.13.tar.xz](https://ftp.gnu.org/gnu/gzip/gzip-1.13.tar.xz)
# wget [https://ftp.gnu.org/gnu/m4/m4-1.4.19.tar.xz](https://ftp.gnu.org/gnu/m4/m4-1.4.19.tar.xz)
# wget [https://ftp.gnu.org/gnu/patch/patch-2.7.6.tar.xz](https://ftp.gnu.org/gnu/patch/patch-2.7.6.tar.xz)
# wget [https://ftp.gnu.org/gnu/sed/sed-4.9.tar.xz](https://ftp.gnu.org/gnu/sed/sed-4.9.tar.xz)
# wget [https://ftp.gnu.org/gnu/tar/tar-1.35.tar.xz](https://ftp.gnu.org/gnu/tar/tar-1.35.tar.xz)
# wget [https://ftp.gnu.org/gnu/texinfo/texinfo-7.1.tar.xz](https://ftp.gnu.org/gnu/texinfo/texinfo-7.1.tar.xz)
# wget [https://ftp.gnu.org/gnu/util-linux/util-linux-2.39.3.tar.xz](https://ftp.gnu.org/gnu/util-linux/util-linux-2.39.3.tar.xz)
# wget [https://www.zlib.net/zlib-1.3.1.tar.xz](https://www.zlib.net/zlib-1.3.1.tar.xz)
# wget [https://www.python.org/ftp/python/3.12.2/Python-3.12.2.tar.xz](https://www.python.org/ftp/python/3.12.2/Python-3.12.2.tar.xz)
# wget [https://www.cpan.org/src/5.0/perl-5.38.2.tar.xz](https://www.cpan.org/src/5.0/perl-5.38.2.tar.xz)
# wget [https://github.com/mesonbuild/meson/releases/download/1.3.2/meson-1.3.2.tar.gz](https://github.com/mesonbuild/meson/releases/download/1.3.2/meson-1.3.2.tar.gz)
# wget [https://github.com/ninja-build/ninja/archive/v1.11.1/ninja-1.11.1.tar.gz](https://github.com/ninja-build/ninja/archive/v1.11.1/ninja-1.11.1.tar.gz)
# wget [https://www.openssl.org/source/openssl-3.2.1.tar.gz](https://www.openssl.org/source/openssl-3.2.1.tar.gz)
# wget [https://github.com/libffi/libffi/releases/download/v3.4.4/libffi-3.4.4.tar.gz](https://github.com/libffi/libffi/releases/download/v3.4.4/libffi-3.4.4.tar.gz)
# wget [https://www.sqlite.org/2023/sqlite-autoconf-3450000.tar.gz](https://www.sqlite.org/2023/sqlite-autoconf-3450000.tar.gz)

# Verify downloaded files (use md5sum or sha256sum from LFS book)
# md5sum *.tar.xz *.tar.gz

3. Building the Temporary System (Chapter 5)
3.1. Creating the Tools Directory

mkdir -pv $LFS/tools
ln -sv $LFS/tools /

3.2. Building Binutils - Pass 1

cd $LFS/sources
tar -xf binutils-2.42.tar.xz
cd binutils-2.42
mkdir -v build
cd build
../configure --prefix=$LFS/tools \
             --with-sysroot=$LFS \
             --target=$LFS_TGT \
             --disable-nls \
             --disable-werror
make
make install
cd $LFS/sources
rm -rf binutils-2.42

3.3. Building GCC - Pass 1

cd $LFS/sources
tar -xf gcc-13.2.0.tar.xz
cd gcc-13.2.0
# Apply patches if necessary (refer to LFS book)
# patch -Np1 -i ../gcc-13.2.0-fix_libstdcxx_linking-1.patch

mkdir -v build
cd build
../configure --target=$LFS_TGT \
             --prefix=$LFS/tools \
             --with-glibc-version=2.39 \
             --with-sysroot=$LFS \
             --with-newlib \
             --without-headers \
             --enable-initfini-array \
             --disable-nls \
             --disable-shared \
             --disable-multilib \
             --disable-decimal-float \
             --disable-threads \
             --disable-libatomic \
             --disable-libgomp \
             --disable-libquadmath \
             --disable-libssp \
             --disable-libvtv \
             --disable-libstdcxx \
             --enable-languages=c,c++
make
make install
cd $LFS/sources
rm -rf gcc-13.2.0

3.4. Building Linux API Headers

cd $LFS/sources
tar -xf linux-6.7.4.tar.xz
cd linux-6.7.4
make mrproper
make headers
find usr/include -name '*.h' -exec cp -v {} $LFS/usr/include \;
cd $LFS/sources
rm -rf linux-6.7.4

3.5. Building Glibc

cd $LFS/sources
tar -xf glibc-2.39.tar.xz
cd glibc-2.39
patch -Np1 -i ../glibc-2.39-fhs-1.patch # Apply patches if necessary
mkdir -v build
cd build
../configure --prefix=/usr \
             --host=$LFS_TGT \
             --build=$(../scripts/config.guess) \
             --enable-kernel=6.7.4 \
             --with-headers=$LFS/usr/include \
             libc_cv_slibdir=/lib
make
make DESTDIR=$LFS install
cd $LFS/sources
rm -rf glibc-2.39

3.6. Building Libstdc++ from GCC

cd $LFS/sources
tar -xf gcc-13.2.0.tar.xz # Re-extract GCC
cd gcc-13.2.0
mkdir -v build
cd build
../libstdc++-v3/configure \
    --host=$LFS_TGT \
    --prefix=/usr \
    --disable-multilib \
    --disable-nls \
    --disable-libstdcxx-pch \
    --with-gxx-include-dir=/tools/$LFS_TGT/include/c++/13.2.0
make
make DESTDIR=$LFS install
cd $LFS/sources
rm -rf gcc-13.2.0

3.7. Building Binutils - Pass 2

cd $LFS/sources
tar -xf binutils-2.42.tar.xz # Re-extract Binutils
cd binutils-2.42
mkdir -v build
cd build
../configure --prefix=/usr \
             --enable-gold \
             --enable-ld=default \
             --enable-plugins \
             --enable-shared \
             --disable-werror \
             --with-sysroot=$LFS \
             --target=$LFS_TGT
make
make DESTDIR=$LFS install
cd $LFS/sources
rm -rf binutils-2.42

3.8. Building GCC - Pass 2

cd $LFS/sources
tar -xf gcc-13.2.0.tar.xz # Re-extract GCC
cd gcc-13.2.0
mkdir -v build
cd build
../configure --prefix=/usr \
             --build=$(../config.guess) \
             --host=$LFS_TGT \
             --disable-nls \
             --enable-languages=c,c++ \
             --disable-multilib \
             --disable-bootstrap \
             --with-system-zlib
make
make DESTDIR=$LFS install
ln -sv gcc $LFS/usr/bin/cc
cd $LFS/sources
rm -rf gcc-13.2.0

4. Entering the Chroot Environment (Chapter 6)
4.1. Preparing the Virtual Kernel File Systems

# As root (or from host system before chroot)
mount -v --bind /dev $LFS/dev
mount -v --bind /dev/pts $LFS/dev/pts
mount -v -t proc proc $LFS/proc
mount -v -t sysfs sysfs $LFS/sys
mount -v -t tmpfs tmpfs $LFS/run

# If you have /dev/shm mounted on host, bind it
# mount -v --bind /dev/shm $LFS/dev/shm

4.2. Chroot into the LFS Environment

chroot "$LFS" /usr/bin/env -i   \
    HOME=/root                  \
    TERM="$TERM"                \
    PS1='(lfs chroot) \u:\w\$ ' \
    PATH=/usr/bin:/bin          \
    /bin/bash --login

4.3. Post-Chroot Environment Setup (inside chroot)

cat > /etc/bash.bashrc << "EOF"
set +h
umask 022
PS1='(lfs chroot) \u:\w\$ '
BUILD64=$(uname -m)
LFS_TGT=$BUILD64-lfs-linux-gnu
LC_ALL=POSIX
PATH=/usr/bin:/bin:/usr/sbin:/sbin
export LFS_TGT LC_ALL PATH
EOF

cat > /etc/profile << "EOF"
export PATH=/usr/bin:/bin:/usr/sbin:/sbin
export LANG=C.UTF-8
EOF

export PATH=/usr/bin:/bin:/usr/sbin:/sbin
export LANG=C.UTF-8

5. Building the LFS System (Chapter 7)

(Inside the chroot environment, as root or lfs user - lfs user is recommended for building, then root for installation)
5.1. Managing Directories and Ownership

mkdir -pv /{boot,home,mnt,opt,srv}
mkdir -pv /etc/{opt,sysconfig}
mkdir -pv /lib/firmware
mkdir -pv /media/{floppy,cdrom}
mkdir -pv /usr/{,local/}{bin,include,lib,sbin,share,src}
mkdir -pv /usr/local/share/{doc,info,locale,man}
mkdir -pv /usr/local/share/man/man{1..8}
mkdir -pv /var/{cache,lib,local,log,mail,opt,spool}
mkdir -pv /var/lib/{color,hwclock,misc,locate}
mkdir -pv /var/spool/mail
mkdir -pv /var/run
mkdir -pv /tmp /var/tmp
chmod -v a+wt /tmp /var/tmp

ln -sv /run /var/run
ln -sv /proc/self/mounts /etc/mtab

5.2. Building Essential Packages (Example order, refer to book for full list)

# Example: Man-pages
cd /sources
tar -xf man-pages-6.06.tar.xz
cd man-pages-6.06
make prefix=/usr install
cd /sources
rm -rf man-pages-6.06

# Example: Glibc (re-installing for the final system)
cd /sources
tar -xf glibc-2.39.tar.xz
cd glibc-2.39
patch -Np1 -i ../glibc-2.39-fhs-1.patch # Apply patches if necessary
mkdir -v build
cd build
../configure --prefix=/usr \
             --host=$LFS_TGT \
             --build=$(../scripts/config.guess) \
             --enable-kernel=6.7.4 \
             --with-headers=/usr/include \
             libc_cv_slibdir=/lib
make
make install
cd /sources
rm -rf glibc-2.39

# Example: Zlib
cd /sources
tar -xf zlib-1.3.1.tar.xz
cd zlib-1.3.1
./configure --prefix=/usr
make
make install
cd /sources
rm -rf zlib-1.3.1

# ... Continue with all other packages as per LFS book (e.g., Binutils, GCC, Bash, Coreutils, etc.)
# Each package follows a similar pattern:
# cd /sources
# tar -xf <package>.tar.xz
# cd <package>
# ./configure --prefix=/usr ... (or other options)
# make
# make install
# cd /sources
# rm -rf <package>

6. Configuring the System (Chapter 8)
6.1. Creating /etc/fstab

cat > /etc/fstab << "EOF"
# Begin /etc/fstab

# file system  mount-point  type     options         dump  fsck
#                                                            order

/dev/sdX1      /            ext4     defaults        1     1
/dev/sdX2      swap         swap     pri=1           0     0
proc           /proc        proc     nosuid,noexec,nodev 0     0
sysfs          /sys         sysfs    nosuid,noexec,nodev 0     0
devpts         /dev/pts     devpts   gid=5,mode=620  0     0
tmpfs          /dev/shm     tmpfs    defaults        0     0
tmpfs          /run         tmpfs    defaults        0     0
devtmpfs       /dev         devtmpfs mode=0755,nosuid 0     0

# End /etc/fstab
EOF

6.2. Configuring the Network

# For DHCP
cat > /etc/sysconfig/ifconfig.eth0 << "EOF"
ONBOOT=yes
IFACE=eth0
SERVICE=ipv4-dhcp
EOF

# For static IP (example)
# cat > /etc/sysconfig/ifconfig.eth0 << "EOF"
# ONBOOT=yes
# IFACE=eth0
# SERVICE=ipv4-static
# IP=192.168.1.100
# GATEWAY=192.168.1.1
# PREFIX=24
# BROADCAST=192.168.1.255
# EOF

cat > /etc/resolv.conf << "EOF"
# Begin /etc/resolv.conf
nameserver 8.8.8.8
nameserver 8.8.4.4
# End /etc/resolv.conf
EOF

echo "lfs-system" > /etc/hostname # Replace with your desired hostname

cat > /etc/hosts << "EOF"
# Begin /etc/hosts
127.0.0.1 localhost.localdomain localhost
192.168.1.100 lfs-system.example.com lfs-system # Replace with your static IP and hostname if applicable
# End /etc/hosts
EOF

6.3. Configuring System Clock

# Adjust for your timezone
ln -sf /usr/share/zoneinfo/America/Chicago /etc/localtime # Example: Chicago

6.4. Installing the Linux Kernel

cd /sources
tar -xf linux-6.7.4.tar.xz
cd linux-6.7.4
make mrproper
make headers # Already done, but harmless to repeat
make defconfig # Or use make menuconfig/xconfig for custom config
make -j$(nproc)
make modules_install
cp -iv arch/x86/boot/bzImage /boot/vmlinuz-6.7.4-lfs-12.3
cp -iv System.map /boot/System.map-6.7.4
cp -iv .config /boot/config-6.7.4
install -d /usr/share/doc/linux-6.7.4
cp -r Documentation/* /usr/share/doc/linux-6.7.4
cd /sources
rm -rf linux-6.7.4

6.5. Configuring GRUB Bootloader

grub-install /dev/sdX # Install GRUB to MBR (replace /dev/sdX)

cat > /boot/grub/grub.cfg << "EOF"
# Begin /boot/grub/grub.cfg
set default=0
set timeout=5

insmod ext2
set root=(hd0,msdos1) # Adjust to your LFS partition

menuentry "GNU/Linux, Linux 6.7.4-lfs-12.3" {
    linux   /boot/vmlinuz-6.7.4-lfs-12.3 root=/dev/sdX1 ro
}
# End /boot/grub/grub.cfg
EOF

7. Final Touches and Reboot (Chapter 9)
7.1. Creating the /etc/inputrc file

cat > /etc/inputrc << "EOF"
# Begin /etc/inputrc
# Allow the command-line editing to be done with the vim editor.
# set editing-mode vi

# Allow the command-line editing to be done with the emacs editor.
set editing-mode emacs

# Do not bell on tab-completion
set bell-style none

# All of the following will enable the use of the arrow keys
# on the command line.
"\e[1~": beginning-of-line
"\e[4~": end-of-line
"\e[5~": history-search-backward
"\e[6~": history-search-forward
"\e[3~": delete-char
"\e[2~": quoted-insert
"\e[5C": forward-word
"\e[5D": backward-word
"\e[M": paste-from-clipboard
"\e[F": end-of-line
"\e[H": beginning-of-line
EOF

7.2. Creating the /etc/shells file

cat > /etc/shells << "EOF"
# Begin /etc/shells
/bin/sh
/bin/bash
# End /etc/shells
EOF

7.3. Cleaning Up and Exiting Chroot

# Clean up sources
rm -rf /sources/*

# Unmount virtual filesystems (inside chroot)
umount -v /dev/pts
umount -v /dev/shm
umount -v /run
umount -v /proc
umount -v /sys
umount -v /dev

# Exit chroot
exit

# Unmount LFS partition (as root on host system)
umount -v $LFS

# Reboot the system
reboot

Important Notes:

    Read the Official LFS Book: This guide is a summary. The official LFS 12.3 book provides crucial details, explanations, and troubleshooting steps for every command.

    Error Handling: If any command fails, do not proceed. Consult the LFS book or community forums for solutions.

    Package Versions: Ensure the package versions you download match those specified in the LFS 12.3 book.

    Host System: Your host system must meet the minimum requirements (e.g., sufficient disk space, necessary build tools).

    Patience: LFS is a long and meticulous process. Take your time and double-check every step.

Good luck with your LFS build!
