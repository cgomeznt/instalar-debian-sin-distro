### Instalar Debian sin la distro

Algo de teoría para saber que es una distro, lo siguiente es un extracto de https://es.wikipedia.org/wiki/Distribuci%C3%B3n_Linux

Una distribución Linux (coloquialmente llamada distro) es una distribución de software basada en el núcleo Linux que incluye determinados paquetes de software para satisfacer las necesidades de un grupo específico de usuarios, dando así origen a ediciones domésticas, empresariales y para servidores. Por lo general están compuestas, total o mayoritariamente, de software libre, aunque a menudo incorporan aplicaciones o controladores propietarios.

En pocas palabras una distribución es un instalador de un sistema operativo que permite seleccionar paquetes y/o configuraciones en el proceso de instalación, para obtener al final un sistema operativo instalado en algún medio físico.

En este laboratorio vamos instalar el sistema operativo universal Debian en un pendrive, pero sin utilizar la distro de Debian, es decir, lo vamos instalar paso a paso como si nosotros fuéramos la herramienta de distribución.

* Tener un medio físico, cualquier cosa que sirva de almacenamiento permanente y que se pueda conectar por USB(disco duro, dispositivos de bloques, pendrive, etc)

* Particionar (opcional).

* Formatearlo en ext4 (opcional, pero es buena idea).

* Instalamos los paquetes requeridos.

* Herramienta para MBR (Solo para pendrive).

* Ejecutar el debootstrap.

* Hacer chroot (Conocido como jaula).

* Algunas configuraciones básicas.

* Instalar el Kernel.

* Instalar Grub 2.

* Preparar el fstab.

* Instalar firmware.

* Instalar entorno gráfico de escritorio.



### Particionar (opcional)

Particionamos nuestro dispositivo de almacenamiento permanente.

```
	# fdisk -l

	Disco /dev/sda: 465,8 GiB, 500107862016 bytes, 976773168 sectores
	Unidades: sectores de 1 * 512 = 512 bytes
	Tamaño de sector (lógico/físico): 512 bytes / 512 bytes
	Tamaño de E/S (mínimo/óptimo): 512 bytes / 512 bytes
	Tipo de etiqueta de disco: dos
	Identificador del disco: 0xae265c89

	Device     Boot     Start       End   Sectors   Size Id Type
	/dev/sda1  *         2048    206847    204800   100M  7 HPFS/NTFS/exFAT
	/dev/sda2          206848 122882047 122675200  58,5G  7 HPFS/NTFS/exFAT
	/dev/sda3       122884094 976771071 853886978 407,2G  5 Extended
	/dev/sda5       122884096 976771071 853886976 407,2G 8e Linux LVM

	Device     Boot Start     End Sectors  Size Id Type
	/dev/sdb1  *       64 2709119 2709056  111,3G 17 Hidden HPFS/NTFS
```

Vemos el disco /dev/sdb1 que es de solo 111,3G y esta en NTFS, lo particionaremos

```
	# fdisk /dev/sdb

	Bienvenido a fdisk (util-linux 2.25.2).
	Los cambios solo permanecerán en la memoria, hasta que decida escribirlos.
	Tenga cuidado antes de utilizar la orden de escritura.


	Orden (m para obtener ayuda): m

	Ayuda:

	  DOS (MBR)
	   a   conmuta el indicador de iniciable
	   b   modifica la etiqueta de disco BSD anidada
	   c   conmuta el indicador de compatibilidad con DOS

	  Generic
	   d   borra una partición
	   l   lista los tipos de particiones conocidos
	   n   añade una nueva partición
	   p   muestra la tabla de particiones
	   t   cambia el tipo de una partición
	   v   verifica la tabla de particiones

	  Miscelánea
	   m   muestra este menú
	   u   cambia las unidades de visualización/entrada
	   x   funciones adicionales (sólo para usuarios avanzados)

	  Guardar y Salir
	   w   escribe la tabla en el disco y sale
	   q   sale sin guardar los cambios

	  Crea una nueva etiqueta
	   g   crea una nueva tabla de particiones GPT vacía
	   G   crea una nueva tabla de particiones SGI (IRIX) vacía
	   o   crea una nueva tabla de particiones DOS vacía
	   s   crea una nueva tabla de particiones Sun vacía


	Orden (m para obtener ayuda): p
	Disco /dev/sdb: 111,8 GiB, 4013948928 bytes, 7839744 sectores
	Unidades: sectores de 1 * 512 = 512 bytes
	Tamaño de sector (lógico/físico): 512 bytes / 512 bytes
	Tamaño de E/S (mínimo/óptimo): 512 bytes / 512 bytes
	Tipo de etiqueta de disco: dos
	Identificador del disco: 0x46c0dd74

	Device     Boot Start     End Sectors  Size Id Type
	/dev/sdb1  *       64 2709119 2709056  111,3G 17 Hidden HPFS/NTFS


	Orden (m para obtener ayuda): d
	Se ha seleccionado la partición 1
	Se ha borrado la partición 1.

	Orden (m para obtener ayuda): w
	Se ha modificado la tabla de particiones.
	Llamando a ioctl() para volver a leer la tabla de particiones.
	Se están sincronizando los discos.
```

Extraemos el disco y lo conectamos nuevamente, monitoreando con dmesg, ejecutamos nuevamente fdisk para crear la partición apropiada.

```
	# fdisk /dev/sdb

	Bienvenido a fdisk (util-linux 2.25.2).
	Los cambios solo permanecerán en la memoria, hasta que decida escribirlos.
	Tenga cuidado antes de utilizar la orden de escritura.


	Orden (m para obtener ayuda): m

	Ayuda:

	  DOS (MBR)
	   a   conmuta el indicador de iniciable
	   b   modifica la etiqueta de disco BSD anidada
	   c   conmuta el indicador de compatibilidad con DOS

	  Generic
	   d   borra una partición
	   l   lista los tipos de particiones conocidos
	   n   añade una nueva partición
	   p   muestra la tabla de particiones
	   t   cambia el tipo de una partición
	   v   verifica la tabla de particiones

	  Miscelánea
	   m   muestra este menú
	   u   cambia las unidades de visualización/entrada
	   x   funciones adicionales (sólo para usuarios avanzados)

	  Guardar y Salir
	   w   escribe la tabla en el disco y sale
	   q   sale sin guardar los cambios

	  Crea una nueva etiqueta
	   g   crea una nueva tabla de particiones GPT vacía
	   G   crea una nueva tabla de particiones SGI (IRIX) vacía
	   o   crea una nueva tabla de particiones DOS vacía
	   s   crea una nueva tabla de particiones Sun vacía


	Orden (m para obtener ayuda): p
	Disco /dev/sdb: 111,8 GiB, 4013948928 bytes, 7839744 sectores
	Unidades: sectores de 1 * 512 = 512 bytes
	Tamaño de sector (lógico/físico): 512 bytes / 512 bytes
	Tamaño de E/S (mínimo/óptimo): 512 bytes / 512 bytes
	Tipo de etiqueta de disco: dos
	Identificador del disco: 0x46c0dd74



	Orden (m para obtener ayuda): n
	Tipo de partición
	   p   primaria (0 primarias, 0 extendidas, 4 libres)
	   e   extendida (contenedor para particiones lógicas)
	Seleccionar (valor predeterminado p): p
	Número de partición (1-4, valor predeterminado 1): 
	Primer sector (2048-7839743, valor predeterminado 2048): 
	Último sector, +sectores o +tamaño{K,M,G,T,P} (2048-7839743, valor predeterminado 7839743): 

	Crea una nueva partición 1 de tipo 'Linux' y de tamaño 111,8 GiB.

	Orden (m para obtener ayuda): p
	Disco /dev/sdb: 111,8 GiB, 4013948928 bytes, 7839744 sectores
	Unidades: sectores de 1 * 512 = 512 bytes
	Tamaño de sector (lógico/físico): 512 bytes / 512 bytes
	Tamaño de E/S (mínimo/óptimo): 512 bytes / 512 bytes
	Tipo de etiqueta de disco: dos
	Identificador del disco: 0x46c0dd74

	Device     Boot Start     End Sectors  Size Id Type
	/dev/sdb1        2048 7839743 7837696  3,8G 83 Linux


	Orden (m para obtener ayuda): w
	Se ha modificado la tabla de particiones.
	Llamando a ioctl() para volver a leer la tabla de particiones.
	Fallo al leer de nuevo la tabla de particiones.: Argumento inválido

	El núcleo todavía usa la tabla antigua. La nueva tabla se usará en el próximo reinicio o después de que usted ejecute partprobe(8) o kpartx(8).
```

Desconectar y volver a conectar.

### Formatear en ext4

```
	# mkfs.ext4 -L Debian /dev/sdb1
	mke2fs 1.42.12 (29-Aug-2014)
	Se está creando El sistema de ficheros con 979712 4k bloques y 245280 nodos-i

	UUID del sistema de ficheros: 26afebdf-b96f-4c90-add9-66b36be0aacc
	Respaldo del superbloque guardado en los bloques: 
		32768, 98304, 163840, 229376, 294912, 819200, 884736

	Reservando las tablas de grupo: hecho
	Escribiendo las tablas de nodos-i: hecho
	Creando el fichero de transacciones (16384 bloques): hecho
	Escribiendo superbloques y la información contable del sistema de ficheros: hecho
```

### Instalar los paquetes requeridos

```
	# apt-get install debootstrap coreutils -y
```

### Herramienta para MBR (Solo para pendrive)

https://www.debian.org/releases/jessie/amd64/ch04s03.html.en#usb-copy-flexible

```
	# apt-get install mbr

	# install-mbr /dev/sdb
```

### Ejecutar el debootstrap

Ejecutamos el debootstrap para que descargue desde un repositorio oficial de Debian un sistema básico de Debian, esto tendrá un aproximado de 294M. (Esto es muy utilizado para las Jaulas, aquí trabajaremos con jaulas). Para más ayuda man debootstrap.
El dispositivo que formateamos debe estar montado.

```
	# mkdir /media/jaula
	# mount /dev/sdb1 /media/jaula

	# mount
	[ ... ]
	/dev/sdb1 on /media/jaula type ext4 (rw,relatime,data=ordered)
```

Al tener identificado donde esta montado procedemos con debootstrap

```
	# j1=/media/jaula
	# echo $j1
	/media/jaula

	# debootstrap --arch=amd64 jessie $j1 http://ftp.debian.org/debian
	I: Retrieving Release 
	I: Retrieving Release.gpg 
	I: Checking Release signature
	I: Valid Release signature (key id 75DDC3C4A499F1A18CB5F3C8CBF8D6FD518E17E1)
	I: Retrieving Packages 
	[ ... ]
	I: Base system installed successfully.
	#
```

### Hacer chroot (Conocido como jaula)

chroot Ejecuta un comando o un shell interactivo en un directorio especial que se utilizara como raíz. Para más ayuda man chroot.
Re-montamos los directorios (proc, dev y sys) de nuestro HOST en el directorio donde descargamos el Debian basico. Para saber más de estos directorios https://wiki.debian.org/es/FilesystemHierarchyStandard. Los montamos para tener acceso a la red y los disco.

```
	# mount -o bind /proc/ /media/jaula/proc/
	# mount -o bind /dev/ /media/jaula/dev/
	# mount -o bind /sys/ /media/jaula/sys/
```

Creamos la jaula con chroot.

```
	# chroot /media/jaula/
	#
```

### Algunas configuraciones basicas.

```
	# pwd
	/
	# vi /root/.bashrc
		# Note: PS1 and umask are already set in /etc/profile. You should not
		# need this unless you want different defaults for root.
		PS1='${debian_chroot:+($debian_chroot)}\h:\w\$ '
		umask 022

		# You may uncomment the following lines if you want `ls' to be colorized:
		# export LS_OPTIONS='--color=auto'
		# eval "`dircolors`"
		# alias ls='ls $LS_OPTIONS'
		# alias ll='ls $LS_OPTIONS -l'
		# alias l='ls $LS_OPTIONS -lA'
		#
		# Some more alias to avoid making mistakes:
		# alias rm='rm -i'
		# alias cp='cp -i'
		# alias mv='mv -i'
```

Nos salimos del chroot

```
	# exit
```

Ingresamos nuevamente 

```
	# chroot /media/jaula/
	debian:/#
```

Hacemos una prueba para verificar que nuestro chroot este operativo, que podamos ver los directorios montados, ver todos los discos y conectividad de red.

```
	debian:/# mount
	/dev/sdb1 on / type ext4 (rw,relatime,data=ordered)
	proc on /proc type proc (rw,nosuid,nodev,noexec,relatime)
	udev on /dev type devtmpfs (rw,relatime,size=10240k,nr_inodes=1013152,mode=755)
	sysfs on /sys type sysfs (rw,nosuid,nodev,noexec,relatime)
	debian:/# fdisk -l

	Disk /dev/sda: 465.8 GiB, 500107862016 bytes, 976773168 sectors
	Units: sectors of 1 * 512 = 512 bytes
	Sector size (logical/physical): 512 bytes / 512 bytes
	I/O size (minimum/optimal): 512 bytes / 512 bytes
	Disklabel type: dos
	Disk identifier: 0xae265c89

	Device     Boot     Start       End   Sectors   Size Id Type
	/dev/sda1  *         2048    206847    204800   100M  7 HPFS/NTFS/exFAT
	/dev/sda2          206848 122882047 122675200  58.5G  7 HPFS/NTFS/exFAT
	/dev/sda3       122884094 976771071 853886978 407.2G  5 Extended
	/dev/sda5       122884096 976771071 853886976 407.2G 8e Linux LVM

	Device     Boot Start     End Sectors  Size Id Type
	/dev/sdb1        2048 7839743 7837696  3.8G 83 Linux

	debian:/# ping -c 3 debian.org
	PING debian.org (130.89.148.14) 56(84) bytes of data.
	64 bytes from klecker4.snt.utwente.nl (130.89.148.14): icmp_seq=1 ttl=53 time=1501 ms
	64 bytes from klecker4.snt.utwente.nl (130.89.148.14): icmp_seq=2 ttl=53 time=524 ms
	64 bytes from klecker4.snt.utwente.nl (130.89.148.14): icmp_seq=3 ttl=53 time=178 ms

	--- debian.org ping statistics ---
	3 packets transmitted, 3 received, 0% packet loss, time 2111ms
	rtt min/avg/max/mdev = 178.055/734.870/1501.794/560.464 ms, pipe 2
```

Colocar una clave a root y creamos un usuario.

```
	debian:/# passwd root
	Introduzca la nueva contraseña de UNIX: 
	Vuelva a escribir la nueva contraseña de UNIX: 
	passwd: contraseña actualizada correctamente

	debian:/# adduser cgome
```

Instalar y configurar los repositorios

```
	debian:/# vi /etc/apt/sources.list
	deb http://ftp.debian.org/debian jessie main contrib non-free

	debian:/# apt-get update
```

Instalar y configurar locale

```
	debian:/# apt-get install locales

	debian:/# locale -a
	locale: Cannot set LC_CTYPE to default locale: No such file or directory
	locale: Cannot set LC_MESSAGES to default locale: No such file or directory
	locale: Cannot set LC_COLLATE to default locale: No such file or directory
	C
	C.UTF-8
	POSIX

	debian:/# vi /etc/locale.gen 
	  es_VE.UTF-8 UTF-8 # Buscamos esta opcion y la descomentamos

	debian:/# locale-gen 
	Generating locales (this might take a while)...
	  es_VE.UTF-8... done
	Generation complete.

	debian:/# update-locale LANG=es_VE.utf8 LANGUAGE=es_VE.utf8 LC_ALL=es_VE.utf8

	debian:/# cat /etc/default/locale 
	#  File generated by update-locale
	LANGUAGE=es_VE.utf8
	LANG=es_VE.utf8
	LC_ALL=es_VE.utf8
```

Salimos del chroot y volvemos a ingresar para verificar la configuración de locale.

```
	debian:/# exit
	#

	# chroot /media/jaula/
	debian:/#

	debian:/# locale
	LANG=es_VE.utf8
	LANGUAGE=
	LC_CTYPE="es_VE.utf8"
	LC_NUMERIC="es_VE.utf8"
	LC_TIME="es_VE.utf8"
	LC_COLLATE="es_VE.utf8"
	LC_MONETARY="es_VE.utf8"
	LC_MESSAGES="es_VE.utf8"
	LC_PAPER="es_VE.utf8"
	LC_NAME="es_VE.utf8"
	LC_ADDRESS="es_VE.utf8"
	LC_TELEPHONE="es_VE.utf8"
	LC_MEASUREMENT="es_VE.utf8"
	LC_IDENTIFICATION="es_VE.utf8"
	LC_ALL=

	debian:/# date
	jue abr 21 19:37:22 UTC 2016
	debian:/# man
	¿Qué página de manual desea?
```

### Instalar el Kernel.

Con debootstrap fue seleccionado la arquitectura amd64, deberíamos instalar un kernel acorde a esta arquitectura.

```
	debian:/# apt-get install linux-image-3.16.0-4-amd64 -y 
```

### Instalar Grub 2.

Instalamos grub2.

```
	debian:/# apt-get install grub2
```

Ahora la instalación del grub dependerá si es en un disco de bloque o en un pendrive

* Esta instalación la hacemos si es en un pendrive. https://www.debian.org/releases/jessie/amd64/ch04s03.html.en#usb-copy-flexible

```
	debian:/# grub-install --force /dev/sdb
	Installing for i386-pc platform.
	grub-install: aviso: Intentando instalar GRUB en un disco con múltiples etiquetas de partición.  Todavía no está soportado..
	grub-install: aviso: El embebido no es posible.  GRUB podrá ser instalado con esta configuración únicamente usando listas de bloques.  No obstante, las listas de bloques son INSEGURAS y su uso está desaconsejado..
	Instalación terminada. No se notificó ningún error.

	debian:/# update-grub
	Generating grub configuration file ...
	Encontrada imagen de linux: /boot/vmlinuz-3.16.0-4-amd64
	Encontrada imagen de memoria inicial: /boot/initrd.img-3.16.0-4-amd64
	Encontrado Windows 7 (loader) en /dev/sda1
	hecho
	debian:/#
```

* Esta instalación la hacemos si es en un disco de bloques

```
	debian:/# grub-install --force /dev/sdb
	Installing for i386-pc platform.
	Instalación terminada. No se notificó ningún error.
	
	debian:/# update-grub
	Generating grub configuration file ...
	Encontrada imagen de linux: /boot/vmlinuz-3.16.0-4-amd64
	Encontrada imagen de memoria inicial: /boot/initrd.img-3.16.0-4-amd64
	Encontrado Windows 7 (loader) en /dev/sda1
	hecho
	debian:/# 
```

### Preparar el fstab.

Capturar el UUID del disco

```
	root@debian:/home/cgome1# blkid | grep sdb
	/dev/sdb1: LABEL="debian-boot" UUID="26f5135b-d3ca-4b67-99df-d1f1d8833a5f" TYPE="ext4" PARTUUID="17f27ec1-01"
```

Editamos el vfstab

```
	debian:/# vi /etc/fstab
	UUID="26f5135b-d3ca-4b67-99df-d1f1d8833a5f"	/	ext4	errors=remount-ro	0	1
```

### Instalar firmware.

Instalamos algunas herramientas y firmwares para los adaptadores de red. 

```
	debian:/# apt-get install pciutils
```

Listar todos los pci del equipo e instalar los firmware que sean requeridos, en este caso los adaptadores de red 

```
	debian:/# lspci | egrep -i 'network|ethernet'
	03:00.0 Network controller: Ralink corp. RT3090 Wireless 802.11n 1T/1R PCIe
	04:00.0 Ethernet controller: Realtek Semiconductor Co., Ltd. RTL8111/8168/8411 PCI Express Gigabit Ethernet Controller (rev 03)

	debian:/# apt-get install firmware-ralink firmware-realtek -y
```

### Instalar entorno gráfico de escritorio.

Instalamos Gnome3 que son 357MB  aproximados, esto si que demora.

```
	debian:/# apt-get install gnome-session -y
```

Tambien puede escoger otro con 

```
	debian:/# tasksel
```

Reiniciamos el equipo con el disco usb conectado y le indicamos al BIOS que haga el arranque del BOOTLOADER desde este disco.



	



