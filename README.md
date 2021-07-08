# gentoo-install
sudo setxkbmap us -variant colemak

sudo passwd

su root



mkdir -p /mnt/gentoo/proc


cd /mnt/gentoo/

cp /media/ubuntu/B2B0D5BEB0D5896D/stage3-amd64-20210630T214504Z.tar.xz /mnt/gentoo/

tar vxpf stage3-*.tar.xz --xattrs-include='.*' --numeric-owner

rm -f stage3-amd64-20210630T214504Z.tar.xz 



cp /media/ubuntu/B2B0D5BEB0D5896D/linux/gentoo/make.conf /mnt/gentoo/etc/portage/make.conf

mkdir -p -v /mnt/gentoo/etc/portage/repos.conf

cp /media/ubuntu/B2B0D5BEB0D5896D/linux/gentoo/gentoo.conf  /mnt/gentoo/etc/portage/repos.conf/gentoo.conf
cp --dereference /etc/resolv.conf /mnt/gentoo/etc/


nano /mnt/gentoo/etc/portage/make.conf



mount --types proc /proc /mnt/gentoo/proc
mount --rbind /sys /mnt/gentoo/sys
mount --rbind /dev /mnt/gentoo/dev

test -L /dev/shm && rm /dev/shm && mkdir /dev/shm
mount --types tmpfs --options nosuid,nodev,noexec shm /dev/shm 
chmod 1777 /dev/shm



mkfs.btrfs -f -L gentoo /dev/nvme0n1p6

mount /dev/nvme0n1p6 /mnt/gentoo


btrfs subvol create /mnt/gentoo/root
btrfs subvol create /mnt/gentoo/home
btrfs subvol create /mnt/gentoo/distfiles
btrfs subvol create /mnt/gentoo/snapshot

umount /mnt/gentoo





#-------------------------
chroot /mnt/gentoo /bin/bash 


source /etc/profile
export PS1="(chroot) ${PS1}"
 


mount -t btrfs -o rw,noatime,autodefrag,compress=zstd,space_cache,subvol=root /dev/nvme0n1p6 /

 #mkdir -pv /mnt/gentoo/{boot,home,.snapshot,var/cache/distfiles}
 
mount -t btrfs -o rw,noatime,autodefrag,compress=zstd,space_cache,subvol=home /dev/nvme0n1p6 /home
 mount -t btrfs -o rw,noatime,autodefrag,compress=zstd,space_cache,subvol=distfiles /dev/nvme0n1p6 /var/cache/distfiles
 
 mkdir /.snapshot
 
 mount -t btrfs -o rw,noatime,autodefrag,compress=zstd,space_cache,subvol=snapshot /dev/nvme0n1p6 /.snapshot

 mount -o defaults,noatime /dev/nvme0n1p1 /boot

  
 df -hT
 
emerge-webrsync
 

 emerge --sync
 
 rm /etc/portage/make.profile

ln -s /var/db/repos/gentoo/profiles/default/linux/amd64/17.1/ /etc/portage/make.profile





echo 'Asia/Shanghai' > /etc/timezone
emerge --config sys-libs/timezone-data

echo "en_US.UTF-8 UTF-8" >> /etc/locale.gen
echo "zh_CN.UTF-8 UTF-8" >> /etc/locale.gen
locale-gen
eselect locale list
eselect locale set en_US.utf8	

hwclock --systohc
date MMDDhhmmYY 

emerge --ask app-portage/cpuid2cpuflags
cpuid2cpuflags >> /etc/portage/make.conf
nano /etc/portage/make.conf

BTRFS_OPTS="rw,noatime,ssd,compress=zstd,space_cache,commit=60"
UEFI_UUID=$(blkid -s UUID -o value /dev/nvme0n1p1)
ROOT_UUID=$(blkid -s UUID -o value /dev/nvme0n1p6)


cat <<EOF > /etc/fstab
# <fs>		<mountpoint>	<type>	<opts>	 <dump/pass>
UUID=$ROOT_UUID /	btrfs	$BTRFS_OPTS,subvol=root	0 0
UUID=$ROOT_UUID /home	btrfs	$BTRFS_OPTS,subvol=home 0 0
UUID=$ROOT_UUID	/var/cache/distfiles	btrfs   $BTRFS_OPTS,subvol=distfiles	0 0
UUID=$ROOT_UUID /.snapshots btrfs	$BTRFS_OPTS,subvol=snapshots 0 0
UUID=$UEFI_UUID /boot	vfat	defaults,noatime 0 2
#tmpfs	/var/tmp/portage	tmpfs	size=4G,uid=portage,gid=portage,mode=755,noatime	0 0
EOF



emerge --ask --verbose --update --deep --newuse @world
#0---------------------------
 
 

