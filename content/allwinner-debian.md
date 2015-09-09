Title: ȫ־ Allwinner A20 ������ˢ��ԭ�� Debian
Date: 2015-09-09 13:09:27
Authors: yang
Category: Tips
Tags: debian
Slug: allwinner-debian

���� 60 ��������һ̨�������Ӿ��� 3 ��׿�����С�������Ŀ�ľ���Ϊ��ˢ��ԭ���� Debian ���������а汾��

<!-- PELICAN_END_SUMMARY -->

�鿴��һ�¾����Ӳ����ȫ־ Allwinner A20 ˫�� CPU��Cortex-A7 �ܹ����ڴ� 1G������ 4G������ USB��һ�� HDMI��һ�� AV�����忴��һ�� Sunxi �� Wiki�����ֿ��԰������µİ취��ˢ�롣�����һ�� ttl ����������ӵ� UART �ӿڲ鿴�����Ϣ��

�����ǲ������裺

**��һ���֣�**

����Ĭ�ϵİ�׿ϵͳ��ͨ�� ttl��ʹ�� root �˻�ֱ�ӹ��� nanda ��������ȡ���е� script.bin�������û�� ttl �ߣ������ȳ����� adb ���Ӻ��ӣ�Ȼ���� root ��ʦ����ȡ root Ȩ�ޣ�֮��˳��������ɣ��ȰѺ��������� Wifi��Ȼ�� `adb connect IPADDRESS`��root����֮������Ϊ�˻�ȡ script.bin��

��ȡ script.bin������

    # mkdir /sdcard/nanda
    # mount -t vfat /dev/block/nanda /sdcard/nanda
    # exit
    # adb pull /sdcard/nanda/script.bin

ȡ�� script.bin �����Ҫ�޸����еĽڵ㣬��ô��Ҫ sunxi-tools��

    # git clone https://github.com/linux-sunxi/sunxi-tools
    # make
    ./bin2fex script.bin script.fex

�༭ fex �ļ����༭���������ɶ������ļ���

    ./fex2bin script.fex script.bin

script.bin �ļ��� fex �ļ��Ķ�����ʵ�֣�fex �ļ����� SoC ����ι����ģ������� GPIO ���Ų����� DRAM����ʾ���� HDMI��VGA���ֱ��ʣ��Ȳ�����

**�ڶ����֣�**

1������ uboot

��ߵı��뻷��Ϊ Linux version 3.16.0-4-686-pae (debian-kernel@lists.debian.org) (gcc version 4.8.4 (Debian 4.8.4-1) ) #1 SMP Debian 3.16.7-ckt11-1+deb8u3 (2015-08-04)��Ĭ�ϵı��빤��Ϊ gcc-arm-linux-gnueabihf���ڡ�deb http://emdebian.org/tools/debian/ jessie main��Դ�п����ҵ���

��Ϊ�����û��Ҳ�Ҳ������ӵ� uboot Դ�룬�ҳ������� cubieboard2 �� uboot Դ�룬������������ʹ�á�

    git clone https://github.com/linux-sunxi/u-boot-sunxi -b wip/a20
    make cubieboard2 ARCH=arm CROSS_COMPILE=arm-linux-gnueabihf-

��һ�� SD ��������������ȫ־����Ĭ��Ϊ SD ���������� fdisk �� SD ��������������һ��Ϊ fat���ڶ���Ϊ ext4 ��ʽ�����岻��׸���������ҵ��� sdb1��sdb2��

������õ� uboot д�뵽 sdcard��

    # dd if=spl/sunxi-spl.bin of=/dev/sdb bs=1024 seek=8
    # dd if=u-boot.bin of=/dev/sdb bs=1024 seek=32

�½�һ�� boot.cmd �ļ��������������ݣ�

    setenv bootargs console=ttyS0,115200 root=/dev/mmcblk0p2 rootwait
    panic=10 ${extra}
    fatload mmc 0 0x48000000 uImage
    bootm 0x48000000

ʹ�� cmd �ļ������� scr �ļ���

    mkimage -C none -A arm -T script -d boot.cmd boot.scr 

2�������ں�

����ʹ�� cubieboard2 ���ںˣ���Ϊ��ʹ�� Sunxi ���ں˱�����޷����������˲��������˼���û�����ˡ�ֱ��ʹ�� cubieboard2 ���ں˿���������������Ҫ��Ӻ��ӵ� PHY �����������������Ӿ��� 3 �� PHY Ϊ ICplus оƬ�����²�����

    # git clone https://github.com/cubieboard2/linux-sunxi
    # make ARCH=arm CROSS_COMPILE=arm-linux-gnueabihf- sun7i_defconfig
    # make ARCH=arm CROSS_COMPILE=arm-linux-gnueabihf- menuconfig

���� menuconfig ״̬����� ICplus ������֧�֣�

    # make -j$(nproc) ARCH=arm CROSS_COMPILE=arm-linux-gnueabihf- uImage modules
    # make ARCH=arm CROSS_COMPILE=arm-linux-gnueabihf- INSTALL_MOD_PATH=output modules_install

���ɵ��ں˺�ģ��·����

    arch/arm/boot/uImage
    output/lib/

���� Debian rootfs��

    # debootstrap --verbose --arch=armhf --foreign jessie debian http://ftp.cn.debian.org/debian
    # cd debian
    # cp /usr/bin/qemu-arm-static usr/bin/
    # LC_ALL=C LANGUAGE=C LANG=C chroot . /debootstrap/debootstrap --second-stage
    # LC_ALL=C LANGUAGE=C LANG=C chroot . dpkg --configure -a

chroot ������ rootfs��

    passwd
    echo "a20" > etc/hostname
    echo "127.0.0.1 a20" >> etc/hostname
    echo T0:2345:respawn:/sbin/getty -L ttyS0 115200 vt100 >> etc/inittab
    echo deb http://ftp.cn.debian.org/debian/ jessie main contrib non-free > etc/apt/sources.list
    echo deb http://security.debian.org/ jessie/updates main contrib non-free >> etc/apt/sources.list
    apt-get update
    apt-get dist-upgrade
    apt-get install openssh-server
    apt-get install locales
    echo "en_US.UTF-8 UTF-8" > etc/locale.gen
    echo "zh_CN.UTF-8 UTF-8" >> etc/locale.gen
    locale-gen

��Ҫ�޸� rootfs �µ������ļ� /etc/network/interfaces �� /etc/ssh/sshd_config��������̬ ip ��ַ��֧�� root ��¼��

���е���������ˣ����濽���ļ��� SD ����Ӧ������

������ sdb1 �µ��ļ���

uImage script.bin boot.scr

Ȼ�� Debian �� rootfs �ļ������� sdb2���ں�ģ�鿽���� /lib �¡�

���� SD �������ӣ�ͨ���ȴ�Ƭ�̼����� ssh ��¼�����ˣ�һ������ԭ���� Debian ϵͳ�������ˡ�

Ŀǰ���о������ʹ�� /dev/fb0 Ӧ�ÿ��Լ������� Xorg������ LXDE Ӧ��ûʲô���⡣

Ŀǰ���ڵ����⣺�����ϵ����� USB �ӿ��޷�ʹ�ã���Ϊ�����õ� cubieboard2 ��Դ��� uboot�����嵽���� script.bin ��Ե�ʻ���Դ���Ե�ʣ��������˼��죬��Ȼ�㲻����ϣ���о�ͨǶ��ʽ�����Ѱ��ҿ����ܷ��������⣬лл��ң�
