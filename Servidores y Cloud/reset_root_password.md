# Restablecer la contraseña root (CentOS 7)

#### ¿Has perdido la contraseña de administrador (root) de tu servidor? aqui te mostramos cómo configurar una nueva a través de la consola KVM.

- Paso 1: Abre la consola KVM.
- Paso 2: Reinicia el servidor y pulsa la tecla `e` en el menú de `GRUB` para editar la entrada de inicio.
- Paso 3: Elimina los parámetros `rhgb y quiet` de la línea que comienza con `linux16`.
Añade los siguientes parámetros al final de la línea linux16:

```sh
    # rd.break enforcing=0
```

La línea debería tener un aspecto similar al siguiente:

```sh
    linux16 /vmlinuz-3.10.0-1062.4.1.el7.x86_64 root=/dev/mapper/centos-ro\ot ro crashkernel=auto rd.lvm.lv=centos/root rd.lvm.lv=centos/swap LANG=en_US.\UTF-8 rd.break enforcing=0
```

El parámetro `rd.break` hace que el proceso de arranque se interrumpa antes de que initramfs pase el control a systemd. Por lo tanto, el indicador de initramfs se puede utilizar para la entrada de comandos.

El parámetro `enforcing=0` pone a SELinux en modo permisivo. Así se ahorra el reetiquetado posterior, que puede llevar mucho tiempo, del sistema de archivos, que sería necesario cuando SELinux está desactivado.

- Paso 4: Presiona Ctrl+x para arrancar el sistema con los parámetros modificados.

Aparece el indicador switch_root de initramfs.

> Note: Si el sistema de archivos está encriptado, el mensaje para introducir la contraseña puede estar superpuesto por los mensajes del sistema y, por lo tanto, no ser visible. En este caso, pulsa la `tecla Return`. A continuación, se volverá a mostrar el mensaje.

- Paso 5: Dado que el sistema de archivos en /sysroot solo está montado con permisos de lectura, primero debes volver a montarlo con permisos de escritura:

```sh
    switch_root:/# mount -o remount,rw /sysroot
```

Ahora cambia a un entorno chroot:

```sh
    switch_root:/# chroot /sysrootEl indicador cambia a sh-4.2#.
```

Ahora puedes cambiar la contraseña mediante passwd:

```sh
    sh-4.2# passwd
    Enter new UNIX password:
    Retype new UNIX password:
    passwd: password updated successfull
```

> Note: Si con este último paso (passwd) se cancela el proceso y se muestra el mensaje Authentication token manipulation error, tienes que salir del entorno chroot de nuevo y volver a montar /sysroot con permisos de escritura (como se describe en el paso 5).

Para salir del entorno chroot, introduce el comando exit.
```sh
    exit
```

Reinicia el servidor:

```sh
    reboot
```

El servidor volverá a arrancar en el sistema normal. Luego podrás iniciar sesión con tu nueva contraseña.

