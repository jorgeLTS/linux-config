# Modifica el Volumen Logico de una Particion

#### Modificar el volumen logico después de ampliar o reducir le disco en **CentOS 7**.
En un Servidor el gestor de volúmenes lógicos (Logical Volume Manager, LVM) se utiliza para gestionar el espacio de almacenamiento. El LVM establece una capa lógica entre el sistema de archivos y las particiones del almacenamiento de datos utilizado. Esto te permite crear un sistema de archivos que abarque varias particiones y/o discos. De esta forma, se puede combinar el espacio de almacenamiento de varias particiones o discos. Además, el LVM te ofrece la posibilidad de ampliar un volumen lógico mientras se está ejecutando.

> Note: Te recomendamos que realices una `copia de seguridad` antes de ajustar manualmente el volumen lógico

## Preparación

- Reiniciaste el servidor después de ampliar el SSD o HDD.
- Iniciaste sesión en el servidor como administrador.

## Primeros Pasos

Para comprobar el espacio disponible del volumen lógico, introduce el siguiente comando:

```sh
[root@localhost ~]# df -h
```

Después de introducir el comando, se muestra la siguiente partición: 

```sh
Filesystem               Size  Used Avail Use% Mounted on
/dev/mapper/centos-root   78G  1.3G   77G   2% /
devtmpfs                 899M     0  899M   0% /dev
tmpfs                    910M     0  910M   0% /dev/shm
tmpfs                    910M   18M  893M   2% /run
tmpfs                    910M     0  910M   0% /sys/fs/cgroup
/dev/sda1                509M  213M  296M  42% /boot
tmpfs                    182M     0  182M   0% /run/user/0
```

Anota el volumen lógico que deseas ampliar. Está montado bajo /. En el ejemplo anterior, el volumen lógico /dev/mapper/centos-root se debe ampliar.

Para ver la partición del volumen lógico, escribe el comando que ve a continuación. A continuación, pulsa Intro:

```sh
[root@localhost ~]# fdisk -l
```

Después de introducir el comando, se muestra la estructura del sistema de archivos: 

```sh
[root@localhost ~]# fdisk -l

Disk /dev/sda: 107.4 GB, 107374182400 bytes, 209715200 sectors
Units = sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disk label type: dos
Disk identifier: 0x000b4f66

   Device Boot      Start         End      Blocks   Id  System
/dev/sda1   *        2048     1050623      524288   83  Linux
/dev/sda2         1050624   167772159    83360768   8e  Linux LVM

Disk /dev/mapper/centos-root: 83.2 GB, 83181436928 bytes, 162463744 sectors
Units = sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes

Disk /dev/mapper/centos-swap: 2147 MB, 2147483648 bytes, 4194304 sectors
Units = sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
```

En este ejemplo, la partición sda debe ampliarse manualmente. Para ello, anota el nombre de la partición y el sector de inicio, que ves en la columna Start. En este ejemplo se trata de 1050624

## Ampliar la partición con fdisk 

Para acceder a la partición `/dev/sda` en fdisk, escribe el siguiente comando:

```sh
[root@localhost ~]# fdisk /dev/sda
```

Después de introducir el comando, aparece el siguiente mensaje: 
 
```sh
[root@localhost ~]# fdisk /dev/sda
Welcome to fdisk (util-linux 2.23.2).

Changes will remain in memory only, until you decide to write them.
Be careful before using the write command.

Command (m for help):
```

> Nota: Para volver a mostrar la tabla de particiones, escribe `p` . A continuación, pulsa Intro.

Para eliminar la partición, escribe d. A continuación, pulsa Intro.

```sh
Command (m for help): d
```

Introduce el número de la partición. A continuación, pulsa Intro.

```sh
Partition number (1,2, default 2): 2
Partition 2 is deleted
```

Para añadir una partición, escribe n. A continuación, pulsa Intro.

```sh
Command (m for help): n
Partition type:
   p   primary (2 primary, 0 extended, 2 free)
   e   extended
```

Para seleccionar el tipo de partición principal, introduce p. A continuación, pulsa Intro.

```sh
Select (default p): p
```

Introduce el número de partición de la partición que eliminaste en el paso 4. Ejemplo: 

```sh
Partition number (2-4, default 2): 2
```

Introduce el sector de inicio: 

```sh
First sector (1050624-209715199, default 1050624): 1050624
```

Para utilizar toda la memoria disponible, pulsa Intro y después la siguiente información aparecerá: 

```sh
Last sector, +sectors or +size{K,M,G} (1050624-209715199, default 209715199):
Using default value 209715199
Partition 2 of type Linux and of size 99.5 GiB is set
```

Para cambiar el tipo de partición a Linux LVM, pulsa la tecla t. A continuación, pulsa Intro.

```sh
Command (m for help): t
```

Introduce el número de la partición: 

```sh
Partition number (1,2, default 2): 2
```

Opcional: Cuando se te pida que introduzcas un código hexadecimal, introduce el código hexadecimal 8e. Después de introducir el código hexadecimal, se cambia el tipo de partición:

```sh
Hex code (type L to list codes): 8e
Changed system type of partition 2 to 8e (Linux LVM)
```

Para comprobar la tabla de particiones modificada, escribe p. A continuación, pulsa Intro.

Para escribir la tabla de particiones en las SSD y salir del programa, escribe w. Después de introducirlo, aparece el siguiente mensaje: 

```sh
Command (m for help): w
The partition table has been altered!
```

> Nota:Si aparece el siguiente mensaje, reinicia el servidor:

```sh
WARNING: Re-reading the partition table failed with error 16: Device or resource busy.
The kernel still uses the old table. The new table will be used at
the next reboot or after you run partprobe(8) or kpartx(8)
Syncing disks.
```

Para comprobar si el tamaño del volumen lógico fue redimensionado, escribe el siguiente comando:

```sh
[root@localhost ~]# fdisk -l
```

##  Ampliar manualmente el volumen lógico 

Para obtener información detallada sobre los volúmenes físicos, escribe el siguiente comando:

```sh
[root@localhost ~]# pvdisplay
```

 Después de escribir el comando, se muestra, entre otro, la siguiente información: 
 
```sh
[root@localhost ~]# pvdisplay
  --- Physical volume ---
  PV Name               /dev/sda2
  VG Name               centos
  PV Size               <79.50 GiB / not usable 30.00 MiB
  Allocatable           yes (but full)
  PE Size               32.00 MiB
  Total PE              2543
  Free PE               0
  Allocated PE          2543
  PV UUID               6FKWEG-OnkG-QxZt-m7TB-wiDb-K9P6-I403lP
```

Para aumentar el volumen físico, introduce el siguiente comando:

```sh
[root@localhost ~]# pvresize /dev/sda2
```

Después de introducir el comando, se muestra la siguiente información:
 
```sh
[root@localhost ~]# pvresize /dev/sda2
  Physical volume "/dev/sda2" changed
  1 physical volume(s) resized or updated / 0 physical volume(s) not resized
```

Para comprobar el estado de los volúmenes lógicos, escribe el siguiente comando: 

```sh
[root@localhost ~]# lvdisplay
  --- Logical volume ---
  LV Path                /dev/centos/swap
  LV Name                swap
  VG Name                centos
  LV UUID                ZghzAz-F7hG-Kxsn-OEdM-idwf-HPmJ-esaD8s
  LV Write Access        read/write
  LV Creation host, time localhost, 2019-03-18 19:48:34 +0000
  LV Status              available
  # open                 2
  LV Size                2.00 GiB
  Current LE             64
  Segments               1
  Allocation             inherit
  Read ahead sectors     auto
  - currently set to     8192
  Block device           253:1

  --- Logical volume ---
  LV Path                /dev/centos/root
  LV Name                root
  VG Name                centos
  LV UUID                1ajYhy-gUdt-KUG4-9MaO-8ayT-g7Yi-Q3lGg1
  LV Write Access        read/write
  LV Creation host, time localhost, 2019-03-18 19:48:34 +0000
  LV Status              available
  # open                 1
  LV Size                <77.47 GiB
  Current LE             2479
  Segments               1
  Allocation             inherit
  Read ahead sectors     auto
  - currently set to     8192
  Block device           253:0
```

Anota la ruta del volumen lógico que deseas ampliar. En este ejemplo, la ruta es /dev/centos/root.

Para aumentar el volumen lógico con el programa lvresize, escribe el comando lvresize en el siguiente formato:

```sh
[root@localhost ~]# lvresize -l +100%FREE [RUTA DEL VOLUMEN LÓGICO]
```

### Ejemplo

```sh
[root@localhost ~]# lvresize -l+100%FREE /dev/centos/root
  Size of logical volume centos/root changed from <77.47 GiB (2479 extents) to <97.47 GiB (3119 extents).
  Logical volume centos/root successfully resized.
```

Cambia el tamaño del sistema de archivos para utilizar el nuevo espacio. Para redimensionar el sistema de archivos al nuevo tamaño con xfs_growfs, escribe el comando xfs_growfs en el siguiente formato:

```sh
[root@localhost ~]# xfs_growfs [RUTA DEL VOLUMEN LÓGICO]
```

### Ejemplo

```sh
[root@localhost ~]# xfs_growfs /dev/centos/root
meta-data=/dev/mapper/centos-root isize=512    agcount=42, agsize=489472 blks
         =                       sectsz=512   attr=2, projid32bit=1
         =                       crc=1        finobt=0 spinodes=0
data     =                       bsize=4096   blocks=20307968, imaxpct=25
         =                       sunit=0      swidth=0 blks
naming   =version 2              bsize=4096   ascii-ci=0 ftype=1
log      =internal               bsize=4096   blocks=2560, version=2
         =                       sectsz=512   sunit=0 blks, lazy-count=1
realtime =none                   extsz=4096   blocks=0, rtextents=0
data blocks changed from 20307968 to 25550848
```

Para comprobar si el sistema de archivos fue modificado, escribe el siguiente comando:

```sh
[root@localhost ~]# df -h
```
