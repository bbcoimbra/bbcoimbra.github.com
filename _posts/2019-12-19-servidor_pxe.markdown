---
layout: post
title: "Servidor PXE - Ubuntu 18.04"
---

Configuração do servidor PXE (TFTP, DHCP)
=========================================

* configurar IP Estático no servidor, i.e. 192.168.10.1 netmask 255.255.255.0
* criar diretórios necessários

```sh
# mkdir -p /netboot/{tftp,nfs,smb} 
# mkdir -p /netboot/tftp/pxelinux.cfg
```

* instalar dnsmasq `# apt-get install dnsmasq`
* configurar dnsmasq:

```config
interface=ens1p0
bind-interfaces
domain=pxeserver.local
 
dhcp-range=ens1p0,192.168.10.100,192.168.10.240,255.255.255.0,8h
dhcp-option=option:router,192.168.10.1
dhcp-option=option:dns-server,192.168.10.1
dhcp-option=option:dns-server,8.8.8.8
 
enable-tftp
tftp-root=/netboot/tftp
dhcp-boot=pxelinux.0,pxeserver.local,192.168.10.1
pxe-prompt="Press F8 for PXE Network boot.", 2
pxe-service=x86PC, "Install OS via PXE",pxelinux
```

* iniciar dnsmasq `# systemctl start dnsmasq`
* verificar se está rodandos  `# systemctl status dnsmasq`
* instalar servidor NFS, para imagens linux `# apt-get install nfs-kernel-server`
* configurar NFS no arquivo `/etc/exports`

```config
/netboot/nfs  *(ro,sync,no_wdelay,insecure_locks,no_root_squash,insecure,no_subtree_check)
```

* Tornar diretório disponivel, ativando compartilhamento `# exportfs -a`
* Instalar pacotes requeridos para o PXE  `# apt install -y syslinux pxelinux`
* Copiar arquivos requeridos para o diretório correto `/netboot/tftp`

```shell
# cp /usr/lib/PXELINUX/pxelinux.0 /netboot/tftp/
# cp /usr/lib/syslinux/modules/bios/{ldlinux.c32,libcom32.c32,libutil.c32,vesamenu.c32} /netboot/tftp
```

* Criar configuração de boot em `/netboot/tftp/pxelinux.cfg/`

```shell
# touch /netboot/tftp/pxelinux.cfg/default`
```

Adiciona imagem ubuntu 18.04
----------------------------

* Faz download da imagem `wget http://releases.ubuntu.com/18.04/ubuntu-18.04.2-desktop-amd64.iso`
* Monta em loopback `mount -o loop ubuntu-18.04.2-desktop-amd64.iso /mnt`
* Cria diretórios necessários `mkdir -v /netboot/{nfs,tftp}/ubuntu1804`
* Copia arquivos de instalação `cp -Rfv /mnt/* /netboot/nfs/ubuntu1804/`
* Copia kernel e initrd `cp /netboot/nfs/ubuntu1804/casper/{vmlinuz,initrd} /netboot/tftp/ubuntu1804`
* Acerta as permissões `chmod -Rfv 777 /netboot` (não use 777 nunca em produção).
* Desmonta loopback `umount /mnt`
* Remove imagem `ubuntu-18.04.2-desktop-amd64.iso`
* Adiciona entrada no menu de boot do PXE `/netboot/tftp/pxelinux.cfg/default`

```
default vesamenu.c32
 
label install1
menu label ^Install Ubuntu 18.04 LTS Desktop
menu default
kernel ubuntu1804/vmlinuz
append initrd=ubuntu1804/initrd boot=casper netboot=nfs nfsroot=192.168.50.1:/netboot/nfs/ubuntu1804/ splash toram ---
```

Adiciona imagem Windows 10
--------------------------

* instala pacotes necessários

```shell
# apt-get install -y samba genisoimage wimtools cabextract
```

* Link mkisofs `ln -s /usr/bin/genisoimage /usr/bin/mkisofs`
* Cria diretórios

```shell
mkdir -p /netboot/smb/windows10
```

* Preparando imagem de boot Windows PE

```shell
wget https://download.microsoft.com/download/8/E/9/8E9BBC64-E6F8-457C-9B8D-F6C9A16E6D6A/KB3AIK_EN.iso
mount KB3AIK_EN.iso /mnt
mkwinpeimg --iso --arch=amd64 --waik-dir=/mnt/waik /netboot/tftp/winpe.iso
umount /mnt
```

* Configuração do Samba `/etc/samba/smb.conf`

```
[global]
  workgroup = WORKGROUP
  map to guest = bad user
  usershare allow guests = yes

[windows10]
  browsable = true
  read only = yes
  guest ok = yes
  path = /netboot/smb/windows10
```

* Reinicia servidor samba `systemctl restart smbd`
* Prepara arquivos de instalação do Windows 10 no compartilhamento

```shell
mount <WINDOWS_ISO> /mnt
cp -R /mnt/* /netboot/smb/windows10/
```

* Configuração do PXE para disponibilizar o memdisk `ln -s /usr/lib/syslinux/memdisk /netboot/tftp`
* Adiciona opção do Windows no menu do PXE `/var/lib/tftpboot/pxelinux.cfg/default`

```
LABEL windows10
MENU LABEL Windows 10
KERNEL /memdisk
INITRD /winpe.iso
APPEND iso raw
```

Instruções no cliente PXE Windows
---------------------------------

* Boot pelo PXE
* Aguardar equipamento ser configurado pelo DHCP. Verificar com `ipconfig`
* Montar compartilhamento do samba `net use z: \\192.168.10.1\windows10`
* Executar binário de instalação `z:\setup.exe`
* Profit!

Referências:
------------

* [Configuração servidor PXE](https://linuxhint.com/pxe_boot_ubuntu_server/)
* [Adição da imagem Win10](https://docs.j7k6.org/windows-10-pxe-installation/)
* [Várias configurações de PXE com UEFI](https://github.com/tianocore/tianocore.github.io/wiki/Configuring-PXE-Boot-Servers-for-UEFI)
